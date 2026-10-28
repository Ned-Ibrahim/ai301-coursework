# Evidence guide: where evidence lives in a PR package

Every rubric check names one of the four families below.
Each family says where its evidence lives in an eval bundle, where it lives in live mode, and what good looks like.
In an eval bundle the sections are fixed: `## Repo facts`, `## Issue`, `## Thread highlights`, `## Plan context` (the repro evidence paragraph, then the paragraph starting `Plan:`), and `## Candidate PR` with `### Title`, `### Description`, `### Commits`, `### Diff`, and `### Test evidence`.
In live mode the same parts come from the working copy and GitHub: `plan.md`, `git diff main...HEAD`, `git log --oneline main..HEAD`, `pr_draft.md`, `test_evidence.md`, and the issue thread, PR template, and `docs/CONTRIBUTING.md`.

## Plan fidelity (harness category: silent-drift)

Checks: diff-within-plan, plan-delivered, description-matches-diff.

Where it lives in an eval bundle:

- The plan's boundary is the `Plan:` paragraph under `## Plan context`. Pull out its scope statement ("Scope, one bounded change: ..." or "Scope, two named deliverables: ..."), its `Files:` list, its `Not in scope:` line, and any known-limit, deferral, or deviation sentence ("Deliberately deferred, stated in the plan", "Known limit, stated in the plan").
- What actually changed is `### Diff`: each `--- a/` and `+++ b/` pair names a file, and each `@@` header starts a hunk. `### Commits` names the work each commit claims; a commit like "add a --dry-run option" announces scope the diff then has to account for.
- The fidelity claims are in `### Title` and `### Description`: phrases like "implements the plan exactly", "in full", "nothing else changed", and every claimed inclusion ("tests added", "CHANGELOG entry added", "docs updated").

Where it lives in live mode:

- The plan's boundary is `plan.md`: its scope (in and not-in), its files, its approach, and its `## Deviations` section. A deviation counts only if it is written in `plan.md`.
- What actually changed is `git diff main...HEAD` (three dots), run from the top of the working copy; `git diff main...HEAD --stat` lists the files, and `git log --oneline main..HEAD` lists the commits. Only committed changes count.
- The fidelity claims are in `pr_draft.md`: the title on its first line, the Summary, and the Changes bullets.
- Path Review has two kinds of hunk the plan may not name but the repo requires for a seeded bug: deleting the `@pytest.mark.xfail` marker whose reason names the issue number, and deleting a `pyproject.toml` suppression whose comment names it. Both count as planned (CONTRIBUTING.md asks for them).

What good looks like:

- Every hunk maps to one of: the planned change, a test of the planned behavior, an edit to a named file for that change, an entry the repo requires (changelog, whatsnew, i18n string, regenerated file, removed xfail marker or suppression), an import the change needs, or a deviation note. Write the mapping out hunk by hunk.
- Every deliverable the plan names (each change, each test, each doc or changelog promise) is in the diff, or its absence is stated in both the plan's notes and the description. A PR that defers part of the plan and says so in both places is honest scope, not drift.
- Silent drift in the "more" direction: a hunk that adds a flag, option, setting, rename pass, refactor, or wording change no plan sentence names, often in its own commit. Silent drift in the "less" direction: a planned deliverable (a docs update, a second fix) absent from the diff while the description claims it, or with no word about it at all.
- A description claiming more or less than the diff delivers is drift, not a comms problem: "docs updated" over a diff with no docs file fails description-matches-diff.

## Test evidence (harness category: not-tested)

Checks: repro-evidence-decisive, repo-checks-reported.

Where it lives in an eval bundle:

- What must be shown is the plan's `Test plan:` sentence inside the `Plan:` paragraph: each repro re-run it names ("re-run the repro", "re-run both repro commands", "repro 1 ... repro 2 ..."), the expected-after it states ("expect exit code 0 and the full output", "expect the argument error and a normal exit code"), each control, and each suite command ("`cargo test`", "`npm test` on the touched package").
- The before is the repro evidence paragraph at the top of `## Plan context`, and sometimes a before line in the evidence itself.
- What was shown is `### Test evidence`, plus any Before/After block in `### Description`.
- The repo's required checks are in `## Repo facts`: the pull-request line ("all code checks passed", "tests added and passed") and the contribution-policy line ("the test suite must pass", "run the formatter, linter, and tests").

Where it lives in live mode:

- What must be shown is the test plan in `plan.md`, and the repro steps from the student's posted repro comment on the issue (or the house repro pack).
- What was shown is `test_evidence.md`, plus the Testing section of `pr_draft.md`.
- The repo's required checks are the four commands in the Testing section of `.github/PULL_REQUEST_TEMPLATE.md`: `make test-unit`, `make test-integration`, `make lint`, and `make typecheck`. Path Review has no integration tests yet, so `make test-integration` stopping with "no tests ran" is an expected outcome with a stated reason. The "CI is green" box runs only after the PR is opened, so it is left unticked and is not required before submission.

What good looks like:

- Decisive: each reproduced failure (a trigger or failure mode the repro evidence shows failing) is re-run on the changed code and its observed result is shown: pasted output, or the exact command or steps with the specific observable each produced (a command and its output, `# after: no error` next to the command, a transcript of UI steps with the exact message text, an exit code), and it equals the expected-after the plan stated. Before and after sit side by side, or the before is the plan's repro evidence.
- The rest of the test plan (controls, smoke checks, scenarios for new behavior the fix adds) needs its outcome reported, and a sentence ("the control still prints both rows") or a named passing test is enough. Sort the test plan's items into these two kinds before grading: a reproduced failure needs shown output, everything else needs a stated outcome.
- Not decisive: a characterization ("works now", "fixed on my machine", "too small to need more testing"); a run of a control or a neighboring path instead of the trigger (a simpler input that never triggered the bug, a different request or code path than the one the fix changes); or a test plan that names two reproduced failure modes with evidence for only one, the other never mentioned.
- Repo checks: each suite or check the plan or repo names appears with its outcome ("`make test-unit`: 412 passed, 53 xfailed", "lint and typecheck clean"). A failing check reported with its reason still counts as reported; a check never mentioned, or "all tests pass" with no suite named while one was required, does not.

## Diff quality (harness category: unreviewable)

Check: diff-free-of-debris.

Where it lives in an eval bundle:

- The added lines (`+`) and the hunks of `### Diff`, and the messages in `### Commits`.

Where it lives in live mode:

- `git diff main...HEAD` and `git log --oneline main..HEAD`. To decide whether a function the diff adds is ever called, search the working copy for its name.

What good looks like:

- Reviewable: every added line serves the fix or its test, and every hunk changes behavior or tests it. Prose comments explaining why the change is right are fine.
- Debris tells, any one of which fails the check:
  - debug output left in: `print("DEBUG ...")`, `eprintln!("DBG ...")`, `console.log` of internal state, a log line the plan does not name as user-facing;
  - commented-out code: `# print(...)`, `// let x = old_call(...)`, a commented-out first attempt "keeping for reference";
  - dead code: an added function nothing calls, a name prefixed `_unused`, a function behind `allow(dead_code)` or `# noqa: F841`, a variable assigned and never read;
  - a new `TODO`, `FIXME`, or `XXX` comment;
  - churn: a hunk that only re-indents, reformats, or regroups imports, or removes and re-adds identical lines (a loop or a block of print lines printed back unchanged) in code the fix does not otherwise touch;
  - commit messages that name no change: "wip", "fix", "stuff", "cleanup while debugging".
- Not debris: unchanged lines re-printed inside the same statement as a real edit (a call whose one argument changed shows its other arguments again), and an import the change needs.

## Standards and comms (harness category: standards-wall)

Checks: template-and-asks-met, ai-disclosure-met.

Where it lives in an eval bundle:

- The asks are in `## Repo facts`: the `pull requests:` line lists the template's sections or checklist and any required lines, files, or conventions ("closes #xxxx", "an entry in the release notes", "a CHANGELOG.md entry", "a conventional-commit PR title", "a Problem section and a Solution section"); the `contribution policy:` line states the AI policy, or says "no stated AI policy".
- Where they are honored: `### Title`, `### Description` (sections and checklist answers), `### Commits` (message conventions), and `### Diff` (required file entries such as `CHANGELOG.md` or `whatsnew`, required tests).

Where it lives in live mode:

- The asks are `.github/PULL_REQUEST_TEMPLATE.md` (sections Summary, Issue with `Closes #`, Changes, Testing, Screenshots / Demo, Notes for Reviewers), `docs/CONTRIBUTING.md` (branch naming `<type>/<issue-number>-<short-description>`, Conventional Commits `<type>(<scope>): <description>`, remove the issue's xfail marker and suppressions, every code change includes or updates tests, Google-style docstrings on public functions), and the house rules in `scope.md` (PR from the student's own fork branch, one PR per issue, every template section with real content).
- The AI ask in live mode: Path Review's files state no AI policy, but the course always asks for an AI-use disclosure, and the house rules say the disclosure section is filled where it applies. Treat that as the stated policy: the description's Notes for Reviewers must say, in the student's own words, how AI was used on the change.
- Where they are honored: `pr_draft.md`, the branch name (`git branch --show-current`), `git log main..HEAD` for the commit messages, and the diff.

What good looks like:

- Compliant: every section the template names is present with content specific to this PR; checklist items are answered truthfully, ticked or in prose ("Checklist: tests added; code formatted; docs n/a, no user-facing change"); the required issue line carries the real number ("Closes #32"); required file entries are hunks in the diff; titles, branches, and commits follow the stated conventions. An ask that cannot apply is marked not applicable ("Screenshots / Demo: N/A, no UI change").
- A visibly ignored ask: a required checklist with no closes line and no required release-notes entry; a template section missing or left as `<!-- ... -->` placeholder; a CI box ticked before CI ran.
- AI disclosure: when the policy requires it, a sentence like "This change was AI-assisted: I used Claude Code to ..., and I reviewed and understand every line" that names what the policy asks for (the tool, the extent). Silence never meets a disclosure requirement, because every package is AI-assisted work. A repo with "no stated AI policy" passes without a disclosure.
- Whether the description's claims match the diff is Plan fidelity, above, not this family.
