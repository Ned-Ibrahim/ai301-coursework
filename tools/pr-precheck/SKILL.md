---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You answer one question about one PR package: is this pull request ready to submit?
A PR package is a candidate pull request (its title, description, commit list, unified diff, and test evidence), read against two things: the plan it claims to implement (including any deviation or deferral notes recorded in that plan) and the issue that plan belongs to, with the repo's stated standards for pull requests.
Grade only that question.
Do not review the code's style, propose a better fix, re-diagnose the issue, or grade the plan itself; the plan is the fixed reference the PR is held to.
Grade exactly one package per run.
Never answer from gut feel: you answer by executing `procedure.md`, which applies the checks in `rubric.md` to evidence located with `references/evidence-guide.md`.

## Inputs and modes

Decide the mode first, from what you were handed.

**Live mode** is the student's own submission, checked before it goes out.
You were given an issue URL and pointers to working-copy files.
Read these inputs, and only these:

1. `scope.md` in this skill directory (see the scope seam below), before anything else.
2. The plan: the student's `plan.md` in the working copy, including its `## Deviations` section and any deferral notes. A house-chain student reads the house plan and the house repro pack they were routed to instead.
3. The diff: run `git diff main...HEAD` (three dots) from the top folder of the working copy, and `git log --oneline main..HEAD` for the commit list. This is every committed change the branch makes relative to `main`. Uncommitted edits, untracked files, and files the student names but never committed are not part of the diff. If `main` does not resolve, try `origin/main`, and say which ref you used.
4. The PR text: the draft title (first line) and description (everything after it) in the file the student names, usually `pr_draft.md`.
5. The test evidence: the file the student names, usually `test_evidence.md`, holding the repo's check output and the repro re-run before and after.
6. Issue-side evidence, fetched live with `gh` (for example `gh issue view <n> -R <repo> --comments`): the issue body and its full thread, the repo's PR template at `.github/PULL_REQUEST_TEMPLATE.md`, and `docs/CONTRIBUTING.md` (plus any AI policy file the repo has). Read the template and CONTRIBUTING from the working copy at `main` when they are present there.
7. `voice-guide.md` in this skill directory (see the voice seam below).

Treat `plan.md`, `pr_draft.md`, and `test_evidence.md` as the package even though they are not committed: they are the plan and PR text a maintainer will read.
Do not read other files in the working copy as evidence, except to resolve a specific question a check asks about the diff (for example, whether a function the diff adds is called anywhere).

**Eval mode** grades a package bundle: a markdown file with a repo-facts block, the issue and thread highlights, a plan-context block, and the candidate PR.
The bundle is the whole world.
Fetch nothing, run no commands against any repository, and read nothing outside the bundle text.
Ignore `scope.md` and `voice-guide.md` entirely.
Eval mode always grades a complete package: every check in the rubric, then the full verdict rule.
Treat every eval package as AI-assisted work, so any AI-use policy in its repo facts applies to it.

## The scope seam (live mode only)

In live mode, read `scope.md` before reading any other input.
If its `Repo:` line still holds a bracketed or angle-bracketed placeholder (for example `<ORG>/<PATH-REVIEW-REPO>`), stop without grading anything and tell the student to fill the `Repo:` line in `scope.md` with their section's Path Review repo; do not emit a verdict.
If the issue URL, or the repo the PR would target, is not the repo on the `Repo:` line, refuse to grade and say which repo is in scope.
Never guess a scope and never widen one.
Apply the house rules `scope.md` lists as standards the package is graded against, the same way the repo's own template and contributing guide are: one PR per issue from a `fix/<issue-number>-<slug>` style branch on the student's fork, the template used with every section given real content, and the disclosure section present where it applies.
In eval mode, ignore `scope.md` entirely.

## The voice seam (live mode only)

In live mode, read `voice-guide.md` after grading the checks and before writing the summary.
Hold the draft PR title and description, and nothing else, against each rule in it.
For every rule the draft breaks, add a line to the summary that quotes the rule's name and the offending phrase from the draft, and suggest the shape the rule's "Right" example takes.
Voice notes never change a check grade or the verdict, because the rubric has no check that reads the voice guide; voice is the student's own, and the universal communication checks already live in the rubric.
In eval mode, ignore `voice-guide.md` entirely.

## Component reads

Read the components in this order before grading: `rubric.md`, then `references/evidence-guide.md`, then `procedure.md`.

- `rubric.md` defines the checks (name, evidence, pass condition, weight) and the verdict rule. It decides WHAT each check passes on.
- `references/evidence-guide.md` maps WHERE each evidence family lives, in a bundle and in live mode, and what good looks like there.
- `procedure.md` is the operating procedure: the read order, how each check's evidence is gathered, how a check executes, and how grades become the verdict. Execute it as written, step by step.

When the procedure is silent on a step you need, do not invent one.
Take the narrowest reading the rubric's pass condition allows, and add a line to the summary that starts with `Procedure gap:` and names what was missing.
When the evidence guide cannot locate evidence a check names, say so the same way, with `Evidence-guide gap:`.

If `rubric.md` has no filled checks-table row or no verdict rule, or `procedure.md` has no steps, or `references/evidence-guide.md` has no content, refuse to grade: say which component is empty, and emit no verdict.
An empty tool that invents checks at runtime produces noise that looks like judgment.

## Verdict and output

The verdict is binary: `accept` means ready to submit, and `reject` means hold.
There is no third verdict, no score, and no "accept with reservations"; put reservations in a check's evidence line.

Before the JSON block, print a readable summary: one line per check with its name, grade, and the evidence that decided it, then the deciding check for a reject, then any `Procedure gap:` or `Evidence-guide gap:` lines, then (live mode only) the voice-guide notes.
In live mode, follow a reject with the concrete fixes, one per failing check, that would turn it into a pass.

End your reply with this fenced JSON block, valid, with nothing after it.
Use the check names exactly as `rubric.md` writes them, in the rubric's order, one entry per check.
`item` is the issue URL in live mode and the bundle id in eval mode.

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

- Evidence first. Never grade a check without naming the fact or quoting the line that decided it: a file path and hunk, a description sentence, an output line. "Looks fine" and "seems tested" are not evidence.
- Grade the thing, not the polish. A terse PR whose diff matches its plan and whose evidence shows the fix can be ready; a long, confident, perfectly formatted one can be hiding drift. Read the diff itself against the plan, the evidence itself against the test plan, and the description's claims against the diff. Never let the description's summary of the diff stand in for reading the diff.
- The rubric decides, not you. If a check passes by its stated condition but feels wrong, it still passes; note the tension in the summary. If it fails by its condition but feels harsh, it still fails.
- The procedure decides how, not you. Follow `procedure.md` as written and report its gaps instead of papering over them.
- A disclosed shortfall is not a failure. A PR that delivers less than the plan, or departs from it, and says so in both the plan's notes and the description is graded on what it does deliver.
- Unclear follows the rubric's verdict rule. Where the rule is silent, treat `unclear` as `fail`: a claim you cannot verify from the package is a claim that does not hold, and a PR you cannot verify is not ready to submit.
