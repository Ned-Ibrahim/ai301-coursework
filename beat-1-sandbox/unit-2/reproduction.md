# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy
kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**GitHub username**

Ned-Ibrahim

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/32#issuecomment-6385018915

I'm taking this one. `DELETE /profiles/{profile_id}` removes the profile's database rows, but `delete_profile` (`core/services/profile_service.py:74`) never touches the vector store, so its chunks should still sit in the `profile_{id}` Chroma collection. PRAYAG-PATEL-1998 and Lizyum already showed the chunks survive a direct `delete_profile()` call. Next I'll go through the HTTP route a user hits instead, on `f89c06f`, and check whether `HybridRetriever.retrieve()` still returns the deleted profile's chunk. I'll post my environment, steps, and output here.

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/32#issuecomment-6385836008

**Result:** reproduced on `f89c06f` (current `main`). After `DELETE /profiles/{id}` returns 204 and the follow-up `GET` returns 404, the profile's chunk is still in its `profile_{id}` Chroma collection, and `HybridRetriever.retrieve()` still returns it.

**Environment**

- macOS 27.0.1 (arm64), Python 3.11.15 in a uv venv
- Code: `f89c06f` on `main`
- Postgres 16.15 from `docker compose` (`db` service, port 5433)
- chromadb 1.5.9, the local `PersistentClient` that `VectorStore` opens (`.chromadb/` in the repo root)
- fastapi 0.143.1, sqlalchemy 2.1.4, uvicorn 0.54.0, httpx 0.28.1
- `.env` copied from `.env.example`: `LLM_PROVIDER=mock`, the repo's `MockEmbeddingProvider`, no API keys

**Steps** (from a fresh clone)

```bash
git clone https://github.com/codepath/pathreview-ai301-fa26-s1.git
cd pathreview-ai301-fa26-s1
git checkout f89c06f
cp .env.example .env
docker compose up -d
uv venv --python 3.11 .venv
uv pip install --python .venv/bin/python -e ".[dev]"
.venv/bin/alembic upgrade head
.venv/bin/python scripts/seed_db.py
.venv/bin/uvicorn api.main:app --port 8000   # terminal 1
.venv/bin/python repro_issue_32.py           # terminal 2, script below saved in the repo root
```

Login, profile create, delete, and get all go through the running API as the seeded `user1@example.com`. Ingestion isn't wired to any endpoint at `f89c06f`, so the script calls `IngestionPipeline.ingest_resume` directly and writes into the same `profile_{id}` collection the retriever reads.

<details>
<summary><code>repro_issue_32.py</code></summary>

```python
"""End-to-end reproduction for codepath/pathreview-ai301-fa26-s1#32.

DELETE /profiles/{profile_id} leaves the profile's embeddings in the vector store.

Uses only real components, no mocks:
  - the running API (uvicorn) for login, profile create, profile delete, profile get
  - the real IngestionPipeline write path (ingest_resume -> BatchEmbeddingProcessor)
  - the real HybridRetriever read path, which reads collection "profile_<id>"
  - the repo's own MockEmbeddingProvider (deterministic, no API key)

Nothing in the app wires ingestion to an endpoint yet, so step 3 calls the
pipeline directly with the collection the retriever reads.

Run from the repo root, with the API already running from the repo root:
    .venv/bin/uvicorn api.main:app --port 8000
    .venv/bin/python repro_issue_32.py   # this file, saved in the repo root
"""

import os
import sys
from pathlib import Path

import chromadb.api.client
import httpx

from ingestion.embeddings.provider import MockEmbeddingProvider
from ingestion.pipeline import IngestionPipeline
from rag.retriever.hybrid import HybridRetriever
from rag.retriever.keyword_search import KeywordSearcher
from rag.retriever.vector_store import VectorStore

API = os.environ.get("PATHREVIEW_API", "http://localhost:8000")
EMAIL, PASSWORD = "user1@example.com", "password1"
RESUME = Path("tests/fixtures/sample_resumes/mid_career.md")
QUERY = "Python projects and internship experience"


def fresh_store() -> VectorStore:
    # Drop this process's cached Chroma client so reads see what the API process wrote.
    chromadb.api.client.SharedSystemClient.clear_system_cache()
    return VectorStore()


def report(label: str, profile_id: str) -> int:
    store = fresh_store()
    collection = store.get_collection(f"profile_{profile_id}")
    by_profile = collection.get(where={"profile_id": profile_id})
    query_embedding = MockEmbeddingProvider().embed([QUERY])[0]
    hits = HybridRetriever(store, KeywordSearcher()).retrieve(
        QUERY, profile_id, query_embedding, min_score=0.0
    )
    print(f"[{label}] collection profile_{profile_id}: {collection.count()} chunk(s)")
    print(f"[{label}] chunks with metadata profile_id={profile_id}: {len(by_profile['ids'])}")
    print(f"[{label}] HybridRetriever.retrieve() returned {len(hits)} chunk(s)")
    if hits:
        print(f"[{label}]   first hit: {hits[0]['text'][:70]!r}")
    return len(by_profile["ids"])


def main() -> int:
    with httpx.Client(base_url=API, timeout=30) as http:
        # 1. Log in as a seeded user.
        r = http.post("/auth/login", data={"username": EMAIL, "password": PASSWORD})
        r.raise_for_status()
        http.headers["Authorization"] = f"Bearer {r.json()['access_token']}"
        print(f"[setup] POST /auth/login -> {r.status_code}")

        # 2. Create a profile with a resume, through the API.
        with RESUME.open("rb") as fh:
            r = http.post(
                "/profiles",
                data={"github_username": "repro-32"},
                files={"resume_file": (RESUME.name, fh, "text/markdown")},
            )
        r.raise_for_status()
        profile_id = r.json()["id"]
        print(f"[setup] POST /profiles -> {r.status_code}, profile_id={profile_id}")

        # 3. Ingest the resume with the real pipeline into the collection retrieval reads.
        collection = VectorStore().get_collection(f"profile_{profile_id}")
        pipeline = IngestionPipeline(
            vector_db=collection, db_session=None, embedding_provider=MockEmbeddingProvider()
        )
        result = pipeline.ingest_resume(profile_id, RESUME.read_text(), RESUME.name)
        print(f"[setup] IngestionPipeline.ingest_resume -> {result.chunk_count} chunk(s) stored")

        print()
        before = report("before delete", profile_id)

        # 4. Delete the profile through the API, then confirm the API says it is gone.
        print()
        r = http.delete(f"/profiles/{profile_id}")
        print(f"[delete] DELETE /profiles/{profile_id} -> {r.status_code}")
        r = http.get(f"/profiles/{profile_id}")
        print(f"[delete] GET /profiles/{profile_id} -> {r.status_code}")

        print()
        after = report("after delete", profile_id)

    print()
    if after:
        print(
            f"RESULT: BUG REPRODUCED. The profile is deleted (404) but {after} of its "
            f"{before} chunk(s) are still stored and retrievable."
        )
        return 1
    print(f"RESULT: NOT REPRODUCED. All {before} chunk(s) were removed with the profile.")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

</details>

**Output** (the script's own lines; the app's log lines are trimmed)

```text
[setup] POST /auth/login -> 200
[setup] POST /profiles -> 200, profile_id=ff32dcd3-ea89-4211-9448-6655e11b9002
[setup] IngestionPipeline.ingest_resume -> 1 chunk(s) stored
[before delete] collection profile_ff32dcd3-ea89-4211-9448-6655e11b9002: 1 chunk(s)
[before delete] chunks with metadata profile_id=ff32dcd3-ea89-4211-9448-6655e11b9002: 1
[before delete] HybridRetriever.retrieve() returned 1 chunk(s)
[before delete]   first hit: 'Priya Raman Backend engineer, four years building data services in Pyt'
[delete] DELETE /profiles/ff32dcd3-ea89-4211-9448-6655e11b9002 -> 204
[delete] GET /profiles/ff32dcd3-ea89-4211-9448-6655e11b9002 -> 404
[after delete] collection profile_ff32dcd3-ea89-4211-9448-6655e11b9002: 1 chunk(s)
[after delete] chunks with metadata profile_id=ff32dcd3-ea89-4211-9448-6655e11b9002: 1
[after delete] HybridRetriever.retrieve() returned 1 chunk(s)
[after delete]   first hit: 'Priya Raman Backend engineer, four years building data services in Pyt'
RESULT: BUG REPRODUCED. The profile is deleted (404) but 1 of its 1 chunk(s) are still stored and retrievable.
```

Expected: after the 204, the profile's chunks are gone from the vector store and `retrieve()` returns nothing for that profile.
Actual: the 1 stored chunk is still there and still returned, as shown above.

**What I checked in the code** (at `f89c06f`)

- `delete_profile` (`core/services/profile_service.py:74`) deletes the profile's Review, IngestedSource, and Profile rows and commits. Nothing in that file references the vector store.
- The `DELETE /profiles/{profile_id}` route (`api/routes/profiles.py:195`) only calls `delete_profile`.

I wrote the script with help from Claude Code, then ran it and checked each line of the output myself. Next I'll post a plan here before changing any code.

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. Calibration smoke run (`--include-calibration --only calib-01,calib-02,calib-03,calib-04`): 0/0 scored items; all 4 calibration packages agreed.
   Rubric v1: six required checks, `unclear` counts as fail, no preferred checks.
2. Run 1, full: 17/20, below the bar (clear-accept 5/8, disclosure 1/1, no-evidence 4/4, unfollowable-comms 3/3, wrong-target 4/4).
   All three misses were gold accepts my rubric rejected: pkg-03 (outcome-honest), pkg-05 (steps-rerunnable, behavior-matches), pkg-12 (steps-rerunnable).
3. Run 2, `--only pkg-03,pkg-05,pkg-12,pkg-20,pkg-18,pkg-02,pkg-08,pkg-15`: 8/8 (partial, so it does not decide the bar).
   Ran after loosening steps-rerunnable, behavior-matches, and outcome-honest; the last five ids are canaries.
4. Run 3, full, `--save-run eval-run.txt`: 19/20, PASS (clear-accept 7/8, disclosure 1/1, no-evidence 4/4, unfollowable-comms 3/3, wrong-target 4/4).
   No rubric change since run 2. This is the run in `eval-run.txt`.

**Package analysis**

pkg-09 (sharkdp/fd#2033, `--exec-batch` ordering).
Gold label: accept.
My rubric in the submitted run (run 3): reject, failed on behavior-matches; the other five checks passed.

The package is an honest cannot-reproduce.
The report tried to make the second `--exec-batch` command hit the argument-size limit first, logged every ONE batch before every TWO batch in each run, and then said its padding "may not achieve that".
My behavior-matches check passes a cannot-reproduce only if "an artifact shows the issue's trigger run and the output it actually produced".
The grader read "the issue's trigger" as the condition the issue names (the second command hits the limit first), and the report itself doubts it reached that condition; the grader also noted the padded variant has no command or output shown.
By my wording, the trigger was never shown to run, so the check failed.

The gold label reads it the other way: a real attempt at the trigger, with artifacts and a named list of what differed (uniform name lengths, a 2 MiB ARG_MAX), is what an honest cannot-reproduce looks like.
I think the gold label is right.
A cannot-reproduce often happens because the reporter could not reach the trigger, and saying so is the honest part.
The same clause passed pkg-09 in run 1, so this is an arguable read the grader resolves differently between runs, not a loud signal my rubric cannot see.

**Check rationale**

From `tools/repro-check/rubric.md`:

> | Check | Evidence | Pass condition | Weight |
> |---|---|---|---|
> | behavior-matches | The repro report's artifacts (pasted output, logs, visible screenshots), read against the failure the issue body describes; and the input the report used, read against the issue's trigger (evidence guide: Behavior shown) | Passes if an artifact shows the specific failure the issue describes (the same error, panic, wrong value, or missing effect), produced by the issue's trigger or by an equivalent input. A changed input is equivalent when the report names the property that triggers the bug and the artifact shows the issue's failure itself (same error text, same wrong value). For a cannot-reproduce outcome, passes if an artifact shows the issue's trigger run and the output it actually produced. Fails if the artifact shows a different failure (a different error from a changed input fails even when it is the same class of failure), or if no artifact is shown | required |

Run 1's version accepted a changed input only as "an input the report shows is equivalent", and failed any report whose input "differs from the issue's without being shown equivalent".
That rejected pkg-05, whose report used a minimal local `env.yml` with a `category:` section instead of the issue's conda-lock URL, even though its output was the issue's exact `EnvironmentSectionNotValid` text printed above the JSON.
I rewrote it so a changed input counts when the report names the property that triggers the bug and the artifact shows the issue's failure itself.
I rejected the simpler fix, dropping the input clause and judging the output alone, because a look-alike failure from a different trigger would then pass.
I kept "a different error from a changed input fails even when it is the same class of failure", which is the line that holds calib-03 (a colon instead of `=`, so a parse error instead of the panic) and the wrong-target packages.

**Trade-offs**

Loosening behavior-matches, along with steps-rerunnable and outcome-honest, risked turning rejects into accepts.
Before the confirming run I re-ran the three misses plus five canaries with `--only`: pkg-20 (the one disclosure package), pkg-18 (a repro that lives in a private monorepo, the steps risk), pkg-02 and pkg-08 (changed-input wrong-targets, the behavior risk), and pkg-15 (an unbacked root-cause claim, the honesty risk).
All eight agreed, and run 3 kept wrong-target at 4/4 and disclosure at 1/1.

The case I accept this check will miss: a cannot-reproduce whose report doubts it reached the trigger, like pkg-09, can be rejected even when the attempt is honest.
I did not tune for it.
Any rubric edit now needs a new full run so `eval-run.txt` matches the rubric I upload, and rewording a clause to flip one package is tuning to the eval set rather than to live repros.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
