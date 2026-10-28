# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/245

**Branch**

`fix/32-drop-profile-collection-on-delete`

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

Four runs, in order:

1. Partial, `--only pkg-02,pkg-13,pkg-20`: 3/3 scored. This was a format check on three packages I expected to be hard: an accept whose diff re-prints unchanged lines, an accept with a disclosed deferral, and the AI-disclosure wall.
2. Full: `agreement: 18/20 scored items  (bar: 18/20: PASS)`, with `clear-accept 5/7`. pkg-05 and pkg-11 came back reject against a gold accept, and so did calib-01.
3. Partial, `--include-calibration --only pkg-05,pkg-11,pkg-20,pkg-04,pkg-07,pkg-10,pkg-14,pkg-17,calib-01,calib-04`: 8/8 scored, and both calibration packages matched.
4. Full, saved with `--save-run`: `agreement: 20/20 scored items  (bar: 18/20: PASS)`, with `categories: clear-accept 7/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`. This is the run in `eval-run.txt`.

Run 2 already cleared the bar.
I kept going because all three misses were accepts failing the same check, and a pass condition that rejects honest PRs here would reject mine in live mode too.

**Package analysis**

pkg-11 (BurntSushi/ripgrep#3477, clear-accept).
Gold label: accept.
My rubric's verdict: reject in run 2, accept in run 4.

In run 2 it failed two checks.
repro-evidence-decisive failed with "After result is annotation only ('after: searches normally, no error', ''foo ' not listed'), no pasted output or exit code".
description-matches-diff failed because the description says "One function touched" while the diff edits `parse_gitignore_line` and adds `trim_unescaped_trailing_spaces`.

Both fails followed my wording at the time.
The old pass condition asked for "pasted output, a transcript of the steps with the exact output, or the exact observed value", and pkg-11 writes its after result as a comment beside the command: `$ rg hi # after: searches normally, no error`.
That comment is still a specific observable on the exact trigger from the repro: the `dangling '\'` error is gone, and `rg --files` no longer lists `foo `.
It is nothing like pkg-04's "colors work now", which names no command at all.
The "one function" line is loose about structure, but the work it describes is in the diff.

After the revision, run 4 graded repro-evidence-decisive pass with "Same trigger re-run: 'rg hi # after: searches normally, no error' and rg --files omits 'foo ', matching expected-after", and description-matches-diff pass with "'One function touched' is loose arrangement wording only".
ai-disclosure-met passed in both runs, on "this change was AI-assisted with me reviewing and understanding every line".

**Check rationale**

> | repro-evidence-decisive | The test evidence section and any before/after in the description, read against the plan's test plan (each repro re-run it names, each failure mode, and the expected-after it states) and the repro evidence in the plan context (evidence guide: Test evidence) | Passes if both hold. (1) Every reproduced failure, meaning each trigger or failure mode the repro evidence shows failing and the test plan says to re-run, is run on the changed code with its observed result shown: pasted output, or the exact command or steps with the specific observable each produced (the error message gone, a named file listed or not listed, an exit code, a value, a toast's text). The observed result is the expected-after the plan states, and the before is shown in the evidence or the description or is the plan's own repro evidence. (2) Every other item the test plan names (a control, a smoke check, a scenario for new behavior the fix adds, a parity comparison) has its outcome reported, in a sentence or by a named passing test. Fails if a reproduced failure is characterized instead of shown ("works now", "fixed on my machine", "ran it for a while with no errors", "too small to need testing", a result with no command or steps behind it); if it is run with a different input or path than the plan's trigger (a control, a simpler case that never triggered the bug, a different input shape, a path the fix does not change); if a reproduced failure or a test-plan item is never mentioned; or if the observed after differs from the expected-after. A new unit test passing does not stand in for re-running a reproduced failure | required |

In runs 1 and 2 the check let only items the plan called controls off with a sentence.
Everything else had to meet one bar: "for every repro trigger or failure mode the plan's test plan says to re-run, the evidence shows that same trigger run on the changed code with its observed result".
So a smoke check or a new-behavior scenario reported in one sentence failed the same way a missing repro did.
pkg-05 failed on "Same-key redefine prints the one-time warning.", which tests new behavior, not the reported bug.
calib-01 failed on "renders gruvmax-fang unchanged", a smoke check.
calib-04 shows the case the check exists for: its plan names two reproduced failures, and the evidence never mentions the second one (the `--exec` abort).
So I split the condition in two.
A reproduced failure needs its result shown, everything else needs its outcome stated, and an item the evidence never mentions fails either way.

I rejected one shortcut: letting a passing unit test count as the re-run.
pkg-07 would pass under it, since its evidence says "Unit test added above passes" while it only runs the one-file case and never the plan's two-file repro.
The check's last sentence is there to block that.

**Trade-offs**

The split loosened a check, so run 3 re-ran all four not-tested packages (pkg-04, pkg-07, pkg-10, pkg-14) and calib-04 as canaries, plus pkg-17 for the description-matches-diff change.
All five not-tested canaries stayed reject on repro-evidence-decisive.
pkg-17 still failed description-matches-diff, since "the reactivity-core documentation now documents the limitation" has no docs hunk behind it.

What the check gives up: it now takes a written observable at face value.
A PR that types `# after: no error` beside a command it never ran passes part (1), and the tool can't tell a typed line from a pasted one.
I accept that miss.
Asking for raw output on every line rejected three honest accepts in run 2, and a reviewer who re-runs the command will catch a fake one.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
