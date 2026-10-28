# Plan: delete a profile's vectors when the profile is deleted (#32)

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/32
Built from my posted repro: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/32#issuecomment-6385836008
Code state: `f89c06f` (upstream `main`, unchanged since the repro).
Branch: `fix/32-drop-profile-collection-on-delete` on my fork `Ned-Ibrahim/pathreview-ai301-fa26-s1`, from `f89c06f`.

## Diagnosis

My repro went through the HTTP route on `f89c06f` and showed the profile gone from the API but its chunk still stored and still retrieved:

```text
[delete] DELETE /profiles/ff32dcd3-ea89-4211-9448-6655e11b9002 -> 204
[delete] GET /profiles/ff32dcd3-ea89-4211-9448-6655e11b9002 -> 404
[after delete] collection profile_ff32dcd3-ea89-4211-9448-6655e11b9002: 1 chunk(s)
[after delete] chunks with metadata profile_id=ff32dcd3-ea89-4211-9448-6655e11b9002: 1
[after delete] HybridRetriever.retrieve() returned 1 chunk(s)
```

Cause: nothing on the delete path calls the vector store.
`delete_profile` (`core/services/profile_service.py:74`) deletes the profile's Review, IngestedSource, and Profile rows and commits (`:104`), and the `DELETE /profiles/{profile_id}` route (`api/routes/profiles.py:196`) only calls `delete_profile` and holds no vector store.
Retrieval reads the collection `profile_{profile_id}` (`rag/retriever/hybrid.py:54`), so every chunk in that collection outlives the profile.
This is not a Chroma delete failing; no Chroma delete is ever issued, which is why the chunk count is the same before and after the 204.

## Scope

In: when a profile is deleted through `DELETE /profiles/{profile_id}`, its `profile_{profile_id}` Chroma collection is dropped as part of the delete.

Not in:

- Wiring ingestion to an endpoint. No app code calls `IngestionPipeline` yet; that is a feature, not this bug.
- The missing `profile_id` metadata in `VectorStore.add_chunks` (`rag/retriever/vector_store.py:57`) and the uncalled `delete_by_source_id` (`:109`). Dropping the whole collection makes this fix independent of chunk metadata.
- Cleaning up collections already orphaned by profiles deleted before this fix. That needs a one-off script or migration and its own issue.
- `VectorStore` using a local `PersistentClient` instead of `settings.vector_db_url` (`core/config.py:15`).
- `get_collection` creating a collection on a miss (`vector_store.py:20`) for read paths.

## Files

- `rag/retriever/vector_store.py`: new `VectorStore.delete_collection(name)` and a `get_vector_store()` dependency provider.
- `core/services/profile_service.py`: `delete_profile` takes the vector store and drops the profile's collection.
- `api/routes/profiles.py`: the DELETE route injects the vector store and passes it to `delete_profile`.
- `tests/unit/test_profile_service.py` (new): service tests for the delete path.
- `tests/unit/test_vector_store.py` (new): `delete_collection` tests against a temporary `PersistentClient`.

## Approach

1. Add `VectorStore.delete_collection(name)`. It calls `self.client.delete_collection(name)` and treats a missing collection as a no-op that is logged, not an error. It does not go through `get_collection`, which would create the collection first. Chroma 1.5.9 raises `chromadb.errors.NotFoundError` for a missing collection (checked directly) and older releases raise `ValueError`, and `pyproject.toml` allows `chromadb>=0.4.22`, so both count as "missing". Any other exception propagates.
2. Add `get_vector_store()` next to `VectorStore`, returning `VectorStore()` with its default persist dir, the same store the retriever reads.
3. Change `delete_profile(db, profile_id, user_id)` to `delete_profile(db, profile_id, user_id, vector_store)`. After the ownership lookup (`get_profile`), as the first statement inside the existing `try` and so before any DB delete, call `vector_store.delete_collection(f"profile_{profile_id}")`. A profile that is missing or not owned returns `False` before Chroma is touched.
4. Order and failure: vectors first, then the DB. If the Chroma drop raises, the existing `except` in `delete_profile` rolls back and re-raises, and the route's existing handler returns 500 with the DB rows untouched, so a retry is safe. If the DB commit fails after the drop, the profile survives without its chunks; the chunks are derived from the resume, and the DB row stays the source of truth.
5. In `delete_profile_endpoint`, add `vector_store: VectorStore = Depends(get_vector_store)` and pass it through. Tests can override the dependency or pass a fake store.

## Test plan

Before, from my posted repro on `f89c06f`: the five lines quoted under Diagnosis, ending in

```text
RESULT: BUG REPRODUCED. The profile is deleted (404) but 1 of its 1 chunk(s) are still stored and retrievable.
```

After, on the branch: the same steps from my repro comment (fresh `docker compose up -d`, migrations, seed, `uvicorn api.main:app --port 8000`, then `repro_issue_32.py` from the repo root).
Expected, stated before the build:

```text
[delete] DELETE /profiles/<id> -> 204
[delete] GET /profiles/<id> -> 404
[after delete] collection profile_<id>: 0 chunk(s)
[after delete] chunks with metadata profile_id=<id>: 0
[after delete] HybridRetriever.retrieve() returned 0 chunk(s)
RESULT: NOT REPRODUCED. All 1 chunk(s) were removed with the profile.
```

The script's report opens the collection through `get_collection`, which recreates an empty one on a miss, so the after lines read `0 chunk(s)` rather than an error.

Unit tests (new, `@pytest.mark.unit`):

- `delete_profile` on an owned profile calls `delete_collection("profile_<id>")` before any `db.delete` or `db.commit`.
- When `delete_collection` raises, `delete_profile` raises, `db.commit` is never awaited, and `db.rollback` is.
- When the profile is not found, `delete_profile` returns `False` and `delete_collection` is never called.
- `VectorStore.delete_collection` on a temporary `PersistentClient`: an existing collection is gone afterwards, and a missing name returns without raising.

Repo checks, the four the PR template lists: `make lint`, `make typecheck`, `make test-unit`, `make test-integration`.
Baseline on `f89c06f` is ruff, black, and mypy clean with `375 passed, 53 xfailed`; after the change I expect the same plus the new tests passing.
Path Review has no integration tests yet, so `make test-integration` is expected to stop with "no tests ran", before and after.
Before each commit, `make check` (ruff, black, mypy), as CONTRIBUTING asks; new public functions get Google-style docstrings.
There is no `xfail` test for #32 (`grep -rn "issue #32:" tests` finds nothing), so no marker comes out.

## Risks and unknowns

- I have only run chromadb 1.5.9. The `ValueError` branch for older releases follows their documented behaviour, but I can't test it here.
- If the DB commit fails after the drop, the profile keeps its rows with no chunks, and nothing in the app can re-ingest them yet because ingestion has no endpoint.
- The API process and the repro script are separate processes sharing `.chromadb/` on disk. I expect a drop made in the API process to be visible to the script's fresh client, and the after run is what will show it.
- Classmates' PRs #134 and #171 already target this issue. Their descriptions say both drop the collection after the DB commit and log a cleanup failure; this plan orders it the other way (Approach step 4). It comes from my own repro and does not build on their diffs.

## Deviations

The plan held.
The code changes and their order are what Approach steps 1 to 5 describe, and the after run printed the expected output written under Test plan, line for line.
Three small differences, none of which makes the posted plan comment untrue:

- One unit test more than the plan listed: `test_other_collections_are_untouched` in `tests/unit/test_vector_store.py` checks that `delete_collection` removes only the named collection. It tests this fix, so it stays inside the scope.
- The docstrings of `delete_profile` and `delete_profile_endpoint` now say the profile's vector-store collection is deleted too, because that is the behaviour that changed.
- The repro ran against `uvicorn` on port 8032 instead of 8000, because an unrelated local app already held 8000. The script (the one from my repro comment, kept in `../issue-32/` next to the clone) reads the API address from `PATHREVIEW_API`, so the command was `PATHREVIEW_API=http://localhost:8032 .venv/bin/python ../issue-32/repro_issue_32.py`; no code or step changed.
