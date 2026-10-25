# Evidence guide: where proof lives in a reproduction package

<!--
THIS IS THE PART YOU WRITE (new this week: Unit 1 handed you this file
finished; the scaffolding fades). The skill uses this guide as its map:
for every kind of proof a rubric check names, this file says WHERE to
find it in a package and WHAT GOOD LOOKS LIKE when you do.

Under each family heading below, write:

- Where it lives: the exact places to look. In an eval bundle (which
  section of the package: the issue context, the repo-facts block, the
  claim comment, the repro report and its parts). In live mode (where
  on GitHub or in the draft: the issue thread, the repo's docs, the
  student's draft comment).
- What good looks like: one or two sentences someone else could apply.
  Prefer observable conditions ("the versions named match what the
  issue targets, or the difference is called out") over adjectives
  ("environment is thorough").

A rubric check whose evidence this guide cannot locate is a check
nobody else can execute; the rubric swap showed you what that feels
like. Write the map you wish your grader had.
-->

An eval bundle has six parts: the header (`source:`, `captured:`), `## Repo facts`, `## Issue`, `## Thread highlights`, `## Candidate claim comment`, and `## Candidate repro report`.
Only the last two are the candidate's work; everything else is context to read them against.
In live mode the candidate side is only the draft files the student names (for example `claim.md` and `repro.md`), read as a stranger on the thread would read them once posted.
Other files in the student's working directory are not part of the package.
Comments by anyone else in the thread (classmates' repros, "pushed a fix", maintainer notes) are context and never proof: they cannot fill a gap in the candidate's package.

## Environment

<!-- Where the environment record lives, and what a sufficient one
looks like against the issue's stated target. -->

Where it lives:

- Eval: the environment line or section of `## Candidate repro report`, usually its first lines (project version or commit, OS, runtime, install method, build profile). Compare it with the environment the reporter gives in the `## Issue` body, and with the factors the issue ties to the bug (for example "release builds wrap, debug builds panic", or "on Windows").
- Live: the environment section of the repro draft, compared with the issue body (`gh issue view <n> -R <repo>`) and the repo's bug template (`.github/ISSUE_TEMPLATE/bug_report.md`).

What good looks like:

- The project version, release, or commit that was actually run is named, and every factor the issue ties to the bug is named.
- When the version run differs from the issue's, the report says so ("filed against 0.63.1, tested 0.64.1"), rather than swapping it silently.
- "Same environment as the issue" counts only when the report names that version; "same as the comment above" borrows nothing.
- A field that does not bear on the bug (OS for a bug the issue never ties to an OS) may be missing.

## Steps

<!-- Where the reproduction steps live, and what makes them followable
by a stranger, starting state to trigger. -->

Where it lives:

- Eval: the commands, numbered steps, and UI actions in `## Candidate repro report` (code blocks with `$` prompts, "focus X, press Y"), plus the inputs those steps use (file contents shown with `cat`, request bodies, data).
- Live: the same parts of the repro draft. A script or input file counts only if its content is pasted in the draft or sits at a link the grader can open (a gist, a file on the student's fork).

What good looks like:

- Starting from the recorded environment, a stranger could type or click every step in order without guessing.
- Every input the trigger needs is visible, or specified precisely enough to recreate the part that triggers the bug: "a script with the issue's two `format` calls, rangeStart 19 and 31" points at exact values the issue gives; "an env file with a valid `dependencies:` list plus a `category:` section" names the triggering property.
- Quoting the issue's own command exactly counts as a step.
- Not followable: "set up the project", "ran my script" with nothing about what it runs, "followed the issue's steps", or a trigger that depends on something a stranger cannot get (a private repo, an unshared config, "a sample file").

## Behavior shown

<!-- Where the artifacts live (output excerpts, logs, screenshots),
and what it means for an artifact to show the issue's behavior rather
than an adjacent one. -->

Where it lives:

- Eval: the output blocks and the "Actual:" lines in `## Candidate repro report`, read against the failure in the `## Issue` body (its error text, panic site, wrong value, or missing effect). Read the report's input next to the issue's trigger: the command flags, the input file, the starting state.
- Live: the pasted output in the repro draft, read against the issue body.
- A link or image counts only if the grader can open it and it shows the behavior. In eval mode nothing outside the bundle can be opened, so "see screenshot" with nothing visible is absent evidence.

What good looks like:

- The artifact shows the same failure the issue names (same error message or panic site, same wrong value, same missing effect), produced by the issue's trigger or by an equivalent input.
- A changed input (a local file instead of the issue's URL, a minimal case) is equivalent when the report names the property that triggers the bug and the artifact shows the issue's failure itself.
- Adjacent, and a fail: a different error from a changed input (input syntax that differs from the issue's, a different command), or a different symptom in the same feature.
- An honest cannot-reproduce shows the issue's trigger run and the output it really produced.
- Narrative with no output ("same here, it disappears every time") shows nothing.

## Honesty

<!-- Where claims and their backing meet: how to tell a report that
says exactly what happened (including an honest cannot-reproduce) from
one that claims more than its evidence shows. -->

Where it lives:

- Eval and live: every outcome statement ("reproduced", "could not reproduce", "Actual:", analysis or conclusion paragraphs) and every cause or scope claim, in both the claim comment and the repro report. Read each one against the artifacts the report shows.

What good looks like:

- The headline outcome is the one the output shows, including a partial result or a cannot-reproduce stated as such.
- A scope claim (the bug reaches other versions, platforms, or users, or happens every time) or a cause claim (which code or mechanism is at fault) is shown in output, cites the file and line it rests on, or is labeled a guess ("likely", "I think").
- A control or variant observation (the same command without one flag behaves correctly) is not a scope or cause claim; it adds detail without widening the bug.
- Claiming more than the evidence: "exactly the class of failure the issue describes" over a different error, "everyone I know has this", "it's obviously the null handling" with no code cited, "confirmed on every version" with one run shown.

## Comms

<!-- Where the words meet the repo: the claim comment against the
issue, the comments against the repo's stated templates and
contribution policy (including AI-use disclosure requirements), and
what specific-and-honest looks like next to boilerplate. -->

Where it lives:

- Claim: `## Candidate claim comment` (live: the claim draft), read against the issue title, body, and thread.
- Policy, eval: the `contribution policy` line of `## Repo facts`. The `bug reports` line is context for Environment and Steps only.
- Policy, live: `CONTRIBUTING.md` or `docs/CONTRIBUTING.md`, any AI policy file (`AI_POLICY.md`, `AGENTS.md`, a contributing or AI section of the README), `.github/ISSUE_TEMPLATE/*`, and the house rules in `scope.md`. Read them with `gh api repos/<owner>/<repo>/contents/<path>` or on the web.

What good looks like:

- The claim names this issue's behavior, component, or code path, so it would not fit another issue unchanged, and says what the author does next.
- The claim has no +1, emoji pleading, complaint about the project, or pressure on maintainers ("should be top priority", "please fix soon").
- A claim-only draft promises the reproduction and report as next steps; it does not assert results it does not show.
- Every requirement the policy places on issue comments or AI use is met. Where the policy requires disclosing AI assistance, one of the comments says plainly whether and how AI was used; silence fails.
- No stated requirement passes.
- Boilerplate: "I'd like to work on this" fits any issue and names nothing.
