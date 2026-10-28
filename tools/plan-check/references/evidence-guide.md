# Evidence guide: where evidence lives in a plan package

<!--
THIS IS THE PART YOU WRITE (second week running: the judgment files
stay in your hands). The skill uses this guide as its map: for every
kind of evidence a rubric check names, this file says WHERE to find it
in a plan package and WHAT GOOD LOOKS LIKE when you do.

Under each family heading below, write:

- Where it lives: the exact places to look. In an eval bundle (which
  section of the package: the issue context, the repro-evidence block,
  the candidate plan's scope statement or test plan, the plan comment,
  the repo-facts block). In live mode (where on GitHub or in the
  draft: the issue thread, the student's posted repro comment, the
  repo's docs, the draft plan and comment).
- What good looks like: one or two sentences someone else could apply.
  Prefer observable conditions ("the stated cause cites behavior the
  repro evidence actually shows") over adjectives ("diagnosis is
  solid").

A rubric check whose evidence this guide cannot locate is a check
nobody else can execute, and this week that cuts twice: your
procedure.md tells the skill WHEN to gather each family, and this
guide tells it WHERE. Write the map you wish your executor had.
-->

An eval bundle has seven parts: the header (`source:`, `captured:`), `## Repo facts`, `## Issue`, `## Thread highlights`, `## Repro evidence`, `## Candidate plan`, and `## Candidate plan comment`.
Only the last two are the candidate's work; everything else is context to read them against.
In live mode the candidate side is only the draft files the student names (for example `plan.md` and `comment.md`), and the repro evidence is the student's own posted repro comment on the issue.
Other files in the student's working directory are not part of the package.
Comments by anyone else in the thread (classmates' plans, proposed causes, linked pull requests) are context: they can carry maintainer direction, but they never stand in for the candidate's own evidence.

## Diagnosis and grounding

<!-- Where the plan states its cause, and where the repro evidence
pins down the behavior that cause must explain. What it means for a
diagnosis to follow from the evidence rather than contradict or
ignore it. -->

Where it lives:

- Cause, eval: the `Cause:` or `### Diagnosis` part of `## Candidate plan`, or its opening sentences when the plan has no headings, plus any "traced to" or "the problem is" line in `## Candidate plan comment`.
- Behavior the cause must explain, eval: `## Repro evidence`, every line of it. Read the steps, the controls ("without the toggle, Cmd+Z works"), the variants and timings ("`--color=never`: under 0.3 s"), and the `Actual:` line, then the failure described in the `## Issue` body.
- Live: the diagnosis in `plan.md` and the comment draft, read against the student's posted repro comment on the issue (`gh api repos/<owner>/<repo>/issues/<n>/comments`, the comment by the student that carries the environment, steps, and output).

What good looks like:

- The cause names a code location or a mechanism (a function, a file, a refresh scope, a rounding rule), not a restatement of the symptom.
- Every repro observation fits it, controls and variants included. One observation that the cause cannot explain, such as the slowdown persisting with the pager removed, is a contradiction, and it decides the check.
- The planned change acts on the named cause. Patching the last stage where the failure becomes visible, while the repro points at an earlier stage, is a symptom fix.
- A cause adopted from another commenter is judged like any other: it holds only if the repro evidence fits it.

## Scope

<!-- Where the plan bounds itself: the in-scope statement, the
not-in-scope line, the files or areas named. What one bounded change
looks like next to a drive-by rewrite. -->

Where it lives:

- Eval: the `Change:`, `### Scope`, `### Changes`, or `### Approach` parts of `## Candidate plan`: every listed change, the `In:` and `Out:` or "Not in scope" lines, the files named, and any sentence that defers part of the issue.
- Live: the scope, files, and approach sections of `plan.md`.

What good looks like:

- Each listed change is needed to fix the reproduced behavior, to test that fix, or to document the behavior the fix changes.
- The same defect at a sibling site that the issue or a maintainer names is part of the issue, and an audit that only reports what it finds adds no scope.
- A not-in line, when present, names the neighbouring work the plan leaves alone. It helps a reviewer, but a bounded plan without one is still bounded.
- A deliberate scope-down names what it defers ("this covers the absolute-prefix case only"); that is ready, not incomplete.
- Creep: "while I'm here" renames, refactors, config or dependency migrations, new flags or options, a docs pass beyond the changed behavior, fixes for other issues, or open-ended phrasing ("and clean up anything else I find").

## Executability

<!-- Where the plan says what will actually be done: files or areas,
approach, order of work. What it means for a stranger to be able to
start executing without asking the author anything. -->

Where it lives:

- Eval: the files, functions, and numbered steps in the change, approach, or scope parts of `## Candidate plan`.
- Live: the files and approach sections of `plan.md`.

What good looks like:

- Every change names where it happens (a file, function, callback, or component) and what is done there ("add the commits context to the post-push refresh scope in `sync_controller.go`").
- A stranger who knows the codebase could make the first edit from the plan alone.
- Naming the component and the action, then a concrete locating step inside it ("pin the exact function by tracing the query issuance with `--debug` logs"), is executable; the first edit starts after a step anyone can run.
- Not executable: hedged locations ("somewhere in the editor code", "poke around", "figure out where the history lives"), goals in place of actions ("make undo work across toggles"), or a first step that waits on a decision the plan leaves open.

## Test plan

<!-- Where the plan says how success will be observed, and how that
maps onto the repro evidence's steps and artifacts. What a decisive
test plan names that a vague one does not. -->

Where it lives:

- Eval: the `Test:` or `### Test plan` part of `## Candidate plan`, read against the steps and the `Actual:` line of `## Repro evidence`.
- Live: the test plan section of `plan.md`, read against the steps and output of the student's posted repro comment.

What good looks like:

- It re-runs the repro's trigger, or a test that drives the same code path, and states the result expected after the fix before any code is written: "at step 3 the color flips without leaving the view", "the collection holds 0 chunks and `retrieve()` returns nothing".
- The expected result is the observable the repro shows failing today (an output line, a value, a state, a timing).
- Extra regression checks are welcome beside it, never in place of it.
- Not decisive: "run the full test suite", "make sure nothing regresses", "undo works after toggling", "verify the fix".

## Honesty

<!-- Where claims meet uncertainty: risks, unknowns, and deviations.
How to tell stated unknowns from false confidence, and where an
honest mid-build deviation gets recorded. -->

Where it lives:

- Eval and live: every sentence in the plan and the plan comment that states a cause, a reach ("the same bug swallows the search keys"), or an outcome ("brings back instant shift+G"), plus any risks, unknowns, or open-questions part of the plan.
- Live, after the build: the `## Deviations` section of `plan.md`, which is where a change from the posted plan is recorded.

What good looks like:

- Nothing the plan states as fact is contradicted by the repro, the issue, or the thread. Inferences drawn from the repro (a regression window, a control that isolates the trigger) are backed.
- A claim that the bug reaches further than the repro shows, or that the change achieves more than the test plan checks, rests on the issue, the thread, cited code, or a label ("I think", "unconfirmed", "open question").
- The cause itself is judged under Diagnosis and grounding, not here.
- Unknowns the package raises stay open in the plan: a maintainer's "the fix is hard", a repro observation the plan does not explain, an untested platform.
- False confidence: a cause repeated from the thread as settled when the repro does not support it, a promise that the change fixes more than the plan tests, "no risk" with nothing checked.
- A deviation recorded in `## Deviations` with what changed and why is honest work; a deviation that only exists in the diff is not.

## Comms

<!-- Where the words meet the thread and the repo: the plan comment
read against the issue's maintainer signals (thread highlights, or
the live thread) and against the repo-facts block's stated templates,
contributing asks, and contribution policy (including AI-use
disclosure requirements). What thread-aware looks like next to
boilerplate. -->

Where it lives:

- Comment against plan: `## Candidate plan comment` read against `## Candidate plan` (live: `comment.md` against `plan.md`).
- Maintainer direction, eval: `## Thread highlights`, the entries whose author role is OWNER, MEMBER, or COLLABORATOR, or whom the thread names as a maintainer. Comments by NONE or CONTRIBUTOR authors are context, not direction.
- Maintainer direction, live: the issue thread (`gh issue view <n> -R <repo> --comments`) and any pull request or issue a maintainer links there.
- Policy, eval: the `contribution policy` line of `## Repo facts`. The `bug reports` line does not apply to plans.
- Policy, live: `docs/CONTRIBUTING.md` or `CONTRIBUTING.md`, any AI policy file (`AI_POLICY.md`, `AGENTS.md`, an AI section of the README or CONTRIBUTING), `.github/PULL_REQUEST_TEMPLATE.md` for asks that bind a proposal, and the house rules in this skill's `scope.md`. Read them with `gh api repos/<owner>/<repo>/contents/<path>` or on the web.

What good looks like:

- The comment states the plan's approach and where it lands, so a maintainer can say yes or no, and it promises nothing the plan does not contain: no extra scope, no result the test plan does not check, no date.
- Coordination notes are not plan content: a cross-reference, a report-back promise, "if #3314 lands first I'll add my tests to it instead", a question for the maintainers.
- Every maintainer direction is named back in the comment or the plan ("given the note that the deep fix in the `ignore` crate is hard, this stays at flag time"), and the plan follows it or says why it departs and leaves the call to the maintainer.
- When a maintainer lists alternatives, naming the option the plan takes engages that comment; every alternative and link need not be repeated.
- A pointer to another issue or pull request from a maintainer is acknowledged, not skipped.
- Every policy requirement on comments, proposals, or AI use is met; where the policy asks for AI disclosure or human-written comments, the comment says plainly how AI was used.
- Boilerplate: "I can fix this, please assign me", enthusiasm with no plan, "should have a PR up soon, wish me luck".
