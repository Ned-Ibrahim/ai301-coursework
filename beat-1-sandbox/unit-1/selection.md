# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/32

**Verdict output**

Live-mode run on #32, #67 and #33, 2026-10-06. #32 records `accept`.

**The verdict must record `accept` for this issue.** Choose an issue your own skill
accepts. If your skill rejects every candidate you try, that is a signal about your
rubric rather than about the issues: revise it and re-run — retries are unlimited and a
partial re-run costs about $0.20 — or run the skill on different candidates. Output
recording `reject` for the issue you chose earns no credit for this field.

````
## Ranked read-out (live mode, measured against 2026-10-06)

The three issues share the same repo-level results:
- **maintainer-alive: pass.** The newest commit, `f89c06f`, is by Aburke225 (a human) on 2026-09-16, 20 days ago. Aburke225 (COLLABORATOR) also commented on and closed #43 and #52 that same day.
- **repo-in-use: pass.** The repo isn't archived and has no releases. The last push was 2026-09-16, 20 days ago.
- **ai-policy: pass.** `docs/CONTRIBUTING.md`, `README.md` and `.github/PULL_REQUEST_TEMPLATE.md` say nothing about AI. Silence passes.

**All three accepted.** None has a newcomer label and none has a maintainer comment in its thread, so the preferred checks don't separate them. Fit decides the order.

1. **#33: per-profile lock for concurrent reviews.** Top fit. This is the closest thing in the set to the distributed-systems work you asked for. Serializing concurrent requests per key is a locking problem. An in-process lock only works with a single worker; across workers you need a shared lock in Postgres or Redis (classmates' setup comments show Redis already runs in the stack). It's backend work in Python, estimated at 6–9 hours (tier-3), well inside your 2-week budget. It also has zero comments and zero PRs, so the design is yours.
2. **#32: deleted profile leaves its embeddings in the vector store.** Good storage fit. Postgres and Chroma drift apart on delete. Two classmates have already posted reproductions and one has posted a plan, so less is left to discover.
3. **#67: review creation skips the profile ownership check.** This is the auth item in your profile, but it's the smallest fix (one ownership lookup plus a 404). Three classmates have already reproduced it, and ChinoUkaegbu said on 2026-10-05 that their build followed the posted plan.

**Rejected:** none.

**Where the result is shaky:**
- On #33, the thread doesn't debate the lock design; it just leaves it open (in-process lock vs. distributed lock). With no maintainer in the thread, you'd be making that call alone. That's exactly what makes it fit, but plan to explain your choice in the PR.
- On #67, if ChinoUkaegbu opens a PR, #67 will fail the not-claimed check: the house rule excuses classmates' claim comments, not open PRs. No such PR existed today.
- On #32 and #67, the claim comments from classmates are all from the last 30 days. Without the house rule, not-claimed would fail on both.

```json
[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/33",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Last commit f89c06f by human Aburke225 on 2026-09-16 (20 days); Aburke225 (COLLABORATOR) commented on and closed #43 and #52 on 2026-09-16"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "isArchived false; no releases; last push 2026-09-16 (20 days)"},
      {"name": "newcomer-scope", "grade": "pass", "evidence": "Body asks for one bounded change: 'Add a per-profile lock to serialize concurrent reviews', 2 files, 6-9 hours; no comments, no PRs, not an umbrella"},
      {"name": "not-claimed", "grade": "pass", "evidence": "assignees none; no linked or mentioned PRs; zero comments"},
      {"name": "ai-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md, README.md and the PR template state no AI policy"},
      {"name": "newcomer-label", "grade": "fail", "evidence": "labels: enhancement, api, tier-3"},
      {"name": "maintainer-in-thread", "grade": "fail", "evidence": "Thread has zero comments"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/32",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Last commit f89c06f by human Aburke225 on 2026-09-16 (20 days); Aburke225 (COLLABORATOR) commented on and closed #43 and #52 on 2026-09-16"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "isArchived false; no releases; last push 2026-09-16 (20 days)"},
      {"name": "newcomer-scope", "grade": "pass", "evidence": "Bounded bug: delete_profile leaves vector-store chunks, 3 files, 4-6 hours; no design debate, no closed unmerged PRs, not an umbrella"},
      {"name": "not-claimed", "grade": "pass", "evidence": "assignees none; no linked or mentioned PRs; claim comments by PRAYAG-PATEL-1998 (2026-09-27) and Lizyum (2026-09-28) are classmates (NONE), ignored per scope.md house rule"},
      {"name": "ai-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md, README.md and the PR template state no AI policy"},
      {"name": "newcomer-label", "grade": "fail", "evidence": "labels: bug, api, devops, tier-2"},
      {"name": "maintainer-in-thread", "grade": "fail", "evidence": "All 5 comments have author_association NONE"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/67",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Last commit f89c06f by human Aburke225 on 2026-09-16 (20 days); Aburke225 (COLLABORATOR) commented on and closed #43 and #52 on 2026-09-16"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "isArchived false; no releases; last push 2026-09-16 (20 days)"},
      {"name": "newcomer-scope", "grade": "pass", "evidence": "Bounded bug: create_review() ignores user_id and skips the profile ownership check; no design debate, no PRs, not an umbrella"},
      {"name": "not-claimed", "grade": "pass", "evidence": "assignees none; no linked or mentioned PRs, and none by commenters; claims by ChinoUkaegbu, tahiya-nm and b6tang (NONE, 2026-09-24 to 2026-10-06) ignored per scope.md house rule"},
      {"name": "ai-policy", "grade": "pass", "evidence": "docs/CONTRIBUTING.md, README.md and the PR template state no AI policy"},
      {"name": "newcomer-label", "grade": "fail", "evidence": "labels: bug, tier-2"},
      {"name": "maintainer-in-thread", "grade": "fail", "evidence": "All 9 comments have author_association NONE"}
    ],
    "verdict": "accept"
  }
]
```
````

---

## Eval iterations

Quote source text directly in each field below. Paraphrase does not satisfy them.

**Run history**

1. Smoke run, `--limit 3`: `agreement: 2/3 scored items`
2. Full run 1: `agreement: 14/20 scored items  (bar: 18/20: below the bar)`
3. Re-run, `--only issue-01,issue-06,issue-09,issue-14,issue-16,issue-10`, after changing maintainer-alive from 30 to 90 days and from 2 of 5 to 1 of 5 sampled replies: `agreement: 5/6 scored items`
4. Full run 2: `agreement: 18/20 scored items  (bar: 18/20: PASS)`
5. Full run 3, after live mode rejected every Path Review candidate and I widened maintainer-alive's reply clause to count a maintainer close or comment on any issue or pull request: `agreement: 17/20 scored items  (bar: 18/20: below the bar)`
6. Full run 4, after adding an umbrella-issue condition to newcomer-scope: `agreement: 16/20 scored items  (bar: 18/20: below the bar)`. The condition's wording ("a list of sub-items") was too broad and failed issue-19, a bug report listing causes and fixes; issue-15 flipped from `unclear` to `pass` on not-claimed, because only 40 of its 97 comments are shown.
7. Re-run, `--only issue-10,issue-15,issue-19`, after narrowing the umbrella condition to issues that call themselves one or that a maintainer says to split: `agreement: 3/3 scored items`
8. Full run 5, confirming, saved as `eval-run.txt`: `agreement: 17/20 scored items  (bar: 18/20: below the bar)`. issue-15 graded accept again: with only 40 of 97 comments visible, its years of design debate and two abandoned PRs are not reliably caught by my scope and claim checks.

**Issue analysis**

issue-20. The harness line: `issue-20  reject  accept   NO     graded accept`.
On issue-20 my rubric said accept and gold says reject.
It passed because every required check found passing evidence: recent human commits, two fast maintainer replies, no assignee, PR or claim, no AI policy, and with zero comments there was nothing for my scope conditions to trip on.
What my rubric couldn't see is that my scope check only fails on design debate, core internals, support questions, or 3+ abandoned PRs.
None of those covers a feature wish whose spec leaves a product decision open ("Logo asset TBD."), opened by `cursor[bot]` with no maintainer saying they want it, so an unscoped wish looked like bounded work.

**Check rationale**

> | maintainer-alive | Repo facts: "last 5 default-branch commits" (date and author of each) and "maintainer first-response sample" (days to first owner/member/collaborator comment); in live mode also the repo's recent issue and pull request activity (closes and comments by OWNER/MEMBER/COLLABORATOR) | Both hold: (1) at least one of the last 5 default-branch commits is authored by a human, or is a bot merging a human's pull request, and is dated 90 days or less before the capture date (live mode: before today); (2) at least 1 of the 5 sampled issues shows a maintainer first response of 90 days or less ("no maintainer comment in thread" does not count), or an OWNER/MEMBER/COLLABORATOR closed or commented on any issue or pull request in the repo 90 days or less before the capture date (live mode: before today) | required |

Both halves are required because commits alone can come from a maintainer who ignores issues, and replies alone can come from a repo that stopped shipping.
I started at 30 days and 2 of 5 replies.
Run 1 rejected 5 gold-accept issues that all had fresh commits but only 0 or 1 sampled replies.
One reply within 90 days still shows that a human answers people, which is what a newcomer's PR needs.
Live mode on Path Review then rejected all three candidates on this check, because staff never reply in issue threads; their only maintainer activity was two closing notes on 2026-09-16 (#43, #52).
Closing issues is a human acting, so the reply clause now also accepts a maintainer close or comment on any issue or pull request within 90 days.

**Trade-offs**

Loosening to 1 of 5 still misses issue-14.
Its commits are fresh but its sample has 0 maintainer replies, so it stays rejected even though gold accepts it.
I accept that miss: a repo where nobody answered any of 5 issues is a real risk for a first PR.
I kept issue-10 (gold reject, also 0 sampled replies) as a canary.
It stayed rejected in the `--only` re-run after the 1 of 5 change.
Once the reply clause also counted a maintainer comment anywhere, issue-10 passed this check: a MEMBER had commented 41 days before capture.
That was right about liveness, and it showed my rubric had been rejecting issue-10 for the wrong reason: it is a self-described megaissue, and newcomer-scope had no umbrella condition.
I added one to newcomer-scope instead of tightening this check back.
---

## Selection rationale

Graded on whether all three are answered, in your own words. Not on how good the
reasoning is, and not on length — a short honest answer to each earns the full marks.
This is also the basis for the claim comment you write in Unit 2.

**Selection rationale**

1. #32 is a storage consistency bug: deleting a profile clears the database rows but leaves its chunks in the vector store, so the two stores drift apart.
   That's the distributed data problem I want to work on.
   The issue estimates 4 to 6 hours, well inside my 2 weeks, which leaves room for tests.
2. The verdict correctly found it unassigned, with no open PR, in a live repo, with a bounded scope of three named files.
   What it couldn't weigh: two classmates have already posted repros and one posted a plan.
   The house rule says that doesn't block me, but it means I need to bring something of my own, like a test proving the chunks are gone.
3. Two classmates claimed it in late September and one has posted a plan, so my claim comment has to show my own repro rather than repeat theirs.
   I'll also need to set up ChromaDB locally to reproduce the leftover chunks.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
