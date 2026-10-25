# Rubric: is this reproduction package ready to post?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever
checks you define here. It ships empty on purpose: the judgment is your
work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four
   columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where in the package. Name
     the part (the claim comment, the repro report's environment
     record, the artifacts read against the issue's description, the
     repo-facts block) or a location from your
     references/evidence-guide.md. "The report" is not a source; "the
     output excerpt read against the error the issue describes" is.
   - Pass condition: a decision rule about the OUTCOME that someone
     else could apply and get your answer. Judge the thing itself (does
     the artifact show the issue's behavior?), never the write-up's
     shape (how many steps it has, how long it is, whether it uses a
     template's headings). Structure-shaped checks are what make
     graders disagree with themselves.
   - Weight: `required` (a fail here holds the package) or `preferred`
     (never changes the verdict).

2. A verdict rule below the table: how the check grades combine into
   accept (ready) or reject (hold), including how `unclear` is
   treated. The verdict space is binary. If you write no rule for
   `unclear`, the skill treats it as fail.

Cover what actually gets bad packages posted. The lecture named the
proof families: the environment is recorded, the steps are complete
and followable, the behavior shown matches the issue (not an adjacent
one), the outcome is stated honestly (an evidenced cannot-reproduce is
a pass, a confident wrong-target is not), and the words respect the
repo's conventions. A rubric that ignores a family will fail eval
packages designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| env-recorded | The repro report's environment record (project version, release, or commit; OS; runtime; build profile; install method), read against the environment in the issue body and the factors the issue ties to the bug (evidence guide: Environment) | Passes if the report names the version, release, or commit of the project it actually ran, and names every factor the issue ties to the bug (OS, runtime or language version, build profile, install method). If the version run differs from the issue's, the report says so. Fields that do not bear on the bug may be missing. Fails if no version, release, or commit of the project is named, or if a factor the issue ties to the bug is missing | required |
| steps-rerunnable | The repro report's steps from the recorded environment to the trigger, and every input they use (file contents, data, flags, UI actions) (evidence guide: Steps) | Passes if a stranger starting from the recorded environment could perform every step without guessing: each step is an explicit command, input, or UI action, and every input the trigger needs is either shown (in the comment or at a link the grader can open) or specified precisely enough to recreate the part that triggers the bug, such as the exact values the issue itself gives or the property of the input that triggers it. Running the issue's own command, quoted exactly, counts as a step. Fails if any step is a summary with no command or action behind it ("set up the project", "ran my script" with nothing about what it runs, "followed the issue's steps"), or depends on an input a stranger cannot obtain or recreate (a private repo, an unshared config, "a sample file") | required |
| behavior-matches | The repro report's artifacts (pasted output, logs, visible screenshots), read against the failure the issue body describes; and the input the report used, read against the issue's trigger (evidence guide: Behavior shown) | Passes if an artifact shows the specific failure the issue describes (the same error, panic, wrong value, or missing effect), produced by the issue's trigger or by an equivalent input. A changed input is equivalent when the report names the property that triggers the bug and the artifact shows the issue's failure itself (same error text, same wrong value). For a cannot-reproduce outcome, passes if an artifact shows the issue's trigger run and the output it actually produced. Fails if the artifact shows a different failure (a different error from a changed input fails even when it is the same class of failure), or if no artifact is shown | required |
| outcome-honest | Every outcome claim and every cause or scope claim in the claim comment and the repro report, each read against the artifacts the report shows (evidence guide: Honesty) | Passes if the stated outcome (reproduced, partially reproduced, or could not reproduce) is the outcome the artifacts show, and every cause or scope claim stated as fact is backed by shown output, cites the file and line it rests on, or is labeled as a guess. A scope claim widens the bug beyond what the artifacts show (other versions, other platforms, other users, how often it happens); a cause claim says which code or mechanism is at fault. A control or variant observation (the same steps with one flag removed behave correctly) is neither. An evidenced cannot-reproduce passes. Fails if the artifacts contradict the stated outcome, or if a cause or scope claim is stated as fact with nothing shown or cited | required |
| claim-specific | The claim comment, read against the issue title, body, and thread (evidence guide: Comms) | Passes if all three hold: (1) it names this issue's specific behavior, component, or code path, so it would not fit another issue unchanged; (2) it says what the author will do next; (3) it contains no +1, demand, complaint, or pressure on maintainers. In a claim-only draft, results that the draft does not show must be promised as next steps, not asserted as done. Fails if any of the three does not hold | required |
| repo-conventions | The repo-facts block's contribution policy line; in live mode, the repo's CONTRIBUTING file, AI policy files, issue templates, and the scope.md house rules; each read against the claim comment and the repro report (evidence guide: Comms) | Passes if the claim comment and repro report together meet every requirement the stated policy places on issue comments or on AI use. When the policy requires disclosing AI assistance, at least one of the two comments states plainly whether and how AI was used. A policy that states no requirement for comments or AI use passes. Bug-template fields are not judged here. Fails if any stated requirement is unmet | required |

## Verdict rule

<!-- State how the grades above combine into accept or reject, and how
unclear is treated. Example shape (write your own): "accept if every
required check passes; preferred checks never change the verdict;
unclear counts as fail." -->

Accept only if every required check passes; any required check that fails rejects the package.
An `unclear` grade counts as fail on every check.
There are no preferred checks.
Claim-only drafts (live mode): grade only claim-specific and repo-conventions, judging repo-conventions on the requirements that apply to the claim comment itself.
Report the other four checks as `unclear` with evidence "not yet applicable: claim-only draft" and leave them out of the verdict, so a claim-only draft is accepted only if both of those checks pass.
