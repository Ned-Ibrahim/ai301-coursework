# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

Ned-Ibrahim

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/32#issuecomment-6480298243

Plan for this fix, built from my repro above: on `f89c06f`, after `DELETE /profiles/{id}` returned 204 and the `GET` returned 404, the `profile_{id}` collection still held 1 chunk and `HybridRetriever.retrieve()` still returned it.

**Cause:** nothing on the delete path calls the vector store. `delete_profile` (`core/services/profile_service.py:74`) deletes the Review, IngestedSource, and Profile rows and commits, and the route (`api/routes/profiles.py:196`) holds no store. Retrieval reads `profile_{id}` (`rag/retriever/hybrid.py:54`), so those chunks outlive the profile.

**Change:**

- `rag/retriever/vector_store.py`: add `VectorStore.delete_collection(name)`, which is a no-op when the collection doesn't exist (it won't go through `get_collection`, which creates on a miss), plus a `get_vector_store` dependency.
- `core/services/profile_service.py`: `delete_profile` takes the store and drops `profile_{id}` before the DB deletes. If Chroma fails, the request returns 500 and the DB rows stay, so a retry is safe.
- `api/routes/profiles.py`: the DELETE route injects the store and passes it through.

**Not in this change:** wiring ingestion to an endpoint, the missing `profile_id` metadata in `add_chunks` and the uncalled `delete_by_source_id`, cleaning up collections already orphaned by earlier deletes, and `VectorStore` ignoring `vector_db_url`.

**How I'll check it:** re-run my repro script on the branch. After the delete I expect `profile_<id>: 0 chunk(s)` and `retrieve()` returning 0 chunks. New unit tests cover the service (the drop happens before the commit, a Chroma failure leaves the DB alone, a missing profile never touches Chroma) and `delete_collection` on a missing name, and I'll run `make lint`, `make typecheck`, `make test-unit`, and `make test-integration`.

**Still open:** I've only run chromadb 1.5.9. Older releases raise `ValueError` for a missing collection; I'll handle it, but I can't test it here. If the DB commit fails after the drop, the profile survives without its chunks.

PRAYAG-PATEL-1998 (#134) and Lizyum (#171) already have PRs open on this. Both drop the collection after the DB commit and log a cleanup failure. This plan orders it the other way: the collection goes first, so a Chroma failure fails the request before any row is deleted, and the store reaches the route as a dependency. It comes from my own repro through the HTTP route. I'll build it with Claude Code and review every diff myself.

---

## Your branch

**Branch**

fix/32-drop-profile-collection-on-delete

**Evidence**

My Unit 2 repro steps, re-run on 2026-10-28 from the repo root of my fork's clone, once on `main` and once on the branch.
Setup, as in my repro comment: `docker compose up -d`, `.venv/bin/alembic upgrade head`, `.venv/bin/python scripts/seed_db.py`, then `.venv/bin/uvicorn api.main:app --port 8032` in a second terminal.
The API ran on port 8032 instead of 8000 because an unrelated local app held 8000; the script, the one pasted in my repro comment and kept in `../issue-32/` next to the clone, takes the address from `PATHREVIEW_API` (recorded under Deviations in `plan.md`).

Before, on `main`:

```text
$ git switch main
$ git rev-parse --short HEAD
f89c06f
$ PATHREVIEW_API=http://localhost:8032 .venv/bin/python ../issue-32/repro_issue_32.py
[setup] POST /auth/login -> 200
[setup] POST /profiles -> 200, profile_id=a010654c-f34c-4f81-9dfc-cde20f8961a9
[setup] IngestionPipeline.ingest_resume -> 1 chunk(s) stored
[before delete] collection profile_a010654c-f34c-4f81-9dfc-cde20f8961a9: 1 chunk(s)
[before delete] chunks with metadata profile_id=a010654c-f34c-4f81-9dfc-cde20f8961a9: 1
[before delete] HybridRetriever.retrieve() returned 1 chunk(s)
[before delete]   first hit: 'Priya Raman Backend engineer, four years building data services in Pyt'
[delete] DELETE /profiles/a010654c-f34c-4f81-9dfc-cde20f8961a9 -> 204
[delete] GET /profiles/a010654c-f34c-4f81-9dfc-cde20f8961a9 -> 404
[after delete] collection profile_a010654c-f34c-4f81-9dfc-cde20f8961a9: 1 chunk(s)
[after delete] chunks with metadata profile_id=a010654c-f34c-4f81-9dfc-cde20f8961a9: 1
[after delete] HybridRetriever.retrieve() returned 1 chunk(s)
[after delete]   first hit: 'Priya Raman Backend engineer, four years building data services in Pyt'
RESULT: BUG REPRODUCED. The profile is deleted (404) but 1 of its 1 chunk(s) are still stored and retrievable.
```

After, on the branch:

```text
$ git switch fix/32-drop-profile-collection-on-delete
$ git rev-parse --short HEAD
ec8d577
$ PATHREVIEW_API=http://localhost:8032 .venv/bin/python ../issue-32/repro_issue_32.py
[setup] POST /auth/login -> 200
[setup] POST /profiles -> 200, profile_id=c06bed20-9837-4fc5-874a-8bc14f46c267
[setup] IngestionPipeline.ingest_resume -> 1 chunk(s) stored
[before delete] collection profile_c06bed20-9837-4fc5-874a-8bc14f46c267: 1 chunk(s)
[before delete] chunks with metadata profile_id=c06bed20-9837-4fc5-874a-8bc14f46c267: 1
[before delete] HybridRetriever.retrieve() returned 1 chunk(s)
[before delete]   first hit: 'Priya Raman Backend engineer, four years building data services in Pyt'
[delete] DELETE /profiles/c06bed20-9837-4fc5-874a-8bc14f46c267 -> 204
[delete] GET /profiles/c06bed20-9837-4fc5-874a-8bc14f46c267 -> 404
[after delete] collection profile_c06bed20-9837-4fc5-874a-8bc14f46c267: 0 chunk(s)
[after delete] chunks with metadata profile_id=c06bed20-9837-4fc5-874a-8bc14f46c267: 0
[after delete] HybridRetriever.retrieve() returned 0 chunk(s)
RESULT: NOT REPRODUCED. All 1 chunk(s) were removed with the profile.
```

The API log for the after run shows the drop happening before the row deletes: `deleted_collection collection_name=profile_c06bed20-9837-4fc5-874a-8bc14f46c267`, then `profile_deleted_cascade`, then `profile_deleted`.

Repo checks on the branch:

```text
$ make lint
.venv/bin/ruff check .
All checks passed!
$ make typecheck
.venv/bin/mypy api/ core/ ingestion/ rag/ agent/ safety/
Success: no issues found in 76 source files
$ make test-unit
381 passed, 53 xfailed, 4 warnings in 5.34s
$ make test-integration
no tests ran in 0.05s
make: *** [test-integration] Error 5
```

`make test-unit` was `375 passed, 53 xfailed` on `main`; the 6 new tests in `tests/unit/test_profile_service.py` and `tests/unit/test_vector_store.py` account for the difference, and all 6 fail against `main`'s code.
`make test-integration` collects nothing because Path Review has no integration tests yet.

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. Calibration smoke run (`--include-calibration --only calib-01,calib-02,calib-03,calib-04`): 0/0 scored items; all 4 calibration packages agreed.
   Rubric v1: seven required checks, `unclear` counts as fail, no preferred checks.
2. Run 1, full: 16/20, below the bar (clear-accept 3/7, scope-creep 4/4, thread-convention 2/2, unbuildable 3/3, wrong-cause 4/4).
   All four misses were gold accepts my rubric rejected: pkg-02 (scope-bounded, honest-claims, comment-matches-plan), pkg-03 (comment-matches-plan), pkg-09 (honest-claims, thread-and-conventions), pkg-14 (executable, honest-claims).
3. Run 2, `--only pkg-02,pkg-03,pkg-09,pkg-14,pkg-20,pkg-04,pkg-15,pkg-11,pkg-17`: 8/9 (partial, so it does not decide the bar).
   Ran after loosening five checks; the last five ids are canaries. pkg-09 still failed honest-claims.
4. Run 3, full, `--save-run eval-run.txt`: 19/20, PASS (clear-accept 6/7, scope-creep 4/4, thread-convention 2/2, unbuildable 3/3, wrong-cause 4/4).
   No rubric change since run 2. This is the run in `eval-run.txt`.

**Package analysis**

pkg-09 (sharkdp/fd, `--glob --full-path` patterns matching nothing on native Windows).
Gold label: accept.
My rubric in the submitted run (run 3): reject, failed on honest-claims; the other six checks passed.

The plan's cause, scope, and test plan all hold: the regex-mode control shows the candidates carry backslashes at match time, the change is one normalization at the `walk.rs` match site, and the test plan names the exact path each pattern spelling must print.
What failed is one attribution.
The plan says it takes "option 2 from the thread, which the collaborator noted is the simpler route and globset already does internally".
In the thread, tmccombs (COLLABORATOR) lists option 2 as "normalize the path to use `/` instead of `\` for the comparison, which could simplify other code paths", and it is petrroll (NONE) who "offers option 2 as the simpler and less regex-fragile route" and "notes globset already does the same normalization internally".
My honest-claims check fails a plan if "a claim contradicts the package", and part (1) covers every claim stated as fact, so the grader read the misattribution as a claim the thread contradicts.

I think the gold label is right.
honest-claims is there to catch false confidence about what causes the bug, how far it reaches, and what the fix will do; a misattributed quote changes none of those, and the collaborator did list option 2 and say it could simplify code.
In run 1 the same package also failed thread-and-conventions for not naming every option the collaborator listed, and that one I did fix, because naming the option you take is how a real comment engages a list of alternatives.

**Check rationale**

From `tools/plan-check/rubric.md`:

> | Check | Evidence | Pass condition | Weight |
> |---|---|---|---|
> | honest-claims | Every claim the plan and the plan comment state as fact, and every unknown the package raises (a maintainer's warning, a repro observation the plan does not explain), each read against the repro evidence, the issue body, the thread highlights, and the code the plan cites (evidence guide: Honesty) | Passes if all three hold: (1) no claim stated as fact is contradicted by the repro evidence, the issue, or the thread; (2) every claim that the bug reaches further than the repro shows (other platforms, versions, or code paths), or that the change achieves more than the test plan checks, rests on the issue, the thread, cited code, or a guess label; (3) every unknown the package raises is stated as open, not settled. Inferences the plan draws from the repro evidence count as backed. The cause itself is graded by diagnosis-grounded, not here. Fails if a claim contradicts the package, a reach or outcome claim has nothing behind it, or a raised unknown is presented as settled | required |

Run 1's version passed a plan only if "every claim stated as fact is shown in the repro evidence, rests on code the plan cites (a file, function, or line), or is labeled as a guess or open question", and said "A claim taken from another commenter counts as backed only as far as the repro evidence supports it."
That rejected pkg-14 for "0.44.1, which predates the reattach-path change in 0.44.2", which is an inference from the repro's own regression window, and pkg-02 for "so the cursor never overshoots", which rests on the issue author's report of the sibling site.
I rewrote it in three parts: nothing stated as fact may contradict the package, a claim that reaches past the repro or promises more than the test plan checks needs backing, and raised unknowns stay open.
I added "Inferences the plan draws from the repro evidence count as backed" and moved the cause itself to diagnosis-grounded, so one wrong cause no longer fails two checks.
I rejected dropping the check and folding it into diagnosis-grounded, because the plans it still catches are about reach and outcome, not cause: pkg-15's "covers the whole failure surface" and "undici handles Happy Eyeballs natively", which nothing in the package backs and the test plan never checks.

**Trade-offs**

The rewrite loosened honest-claims, and four other checks loosened with it in the same revision, so before the confirming run I re-ran the four misses plus five canaries with `--only`: pkg-20 and pkg-04 (the two thread-convention packages, where the category floor lives), pkg-15 (scope-creep, which leans on honest-claims), pkg-11 (wrong-cause), and pkg-17 (unbuildable).
All five canaries still rejected, and run 3 kept every reject category at full: scope-creep 4/4, thread-convention 2/2, unbuildable 3/3, wrong-cause 4/4.

What the check gives up: a claim taken from the thread now passes unless the package contradicts it, so a plan that repeats a commenter's unverified claim about reach can get through honest-claims; diagnosis-grounded is the guard on cause, not this check.
The case I accept it will miss is pkg-09's: a sound plan with one misattributed quote.
I did not tune for it, because narrowing part (1) to cause, reach, and outcome claims would exist only to flip that package, and any rubric edit now needs a new full run so `eval-run.txt` matches the rubric I upload.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
