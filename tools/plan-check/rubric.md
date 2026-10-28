# Rubric: is this plan ready to post and build from?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever
checks you define here (via your procedure.md). It ships empty on
purpose: the judgment is your work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four
   columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where in the package. Name
     the part (the plan's scope statement, the test plan read against
     the repro evidence's steps, the plan comment read against the
     thread highlights, the repo-facts block) or a location from your
     references/evidence-guide.md. "The plan" is not a source; "the
     plan's stated cause read against what the repro evidence shows"
     is.
   - Pass condition: a decision rule about the OUTCOME that someone
     else could apply and get your answer. Judge the thing itself (is
     this one bounded change? could a stranger start executing it?),
     never the write-up's shape (how many sections it has, how long it
     is, whether it uses headings). Structure-shaped checks are what
     make graders disagree with themselves.
   - Weight: `required` (a fail here holds the package) or `preferred`
     (never changes the verdict).

2. A verdict rule below the table: how the check grades combine into
   accept (ready) or reject (hold), including how `unclear` is
   treated. The verdict space is binary. If you write no rule for
   `unclear`, the skill treats it as fail.

Cover what actually gets bad plans posted. The lecture named the
failure families: the diagnosis ignores or contradicts the reproduced
evidence, the change is unbounded (scope creep), the plan targets the
symptom while the evidence points at the cause, a stranger could not
start executing it, the test plan proves nothing observable, the
unknowns are dressed up as certainty, and the comment ignores what the
thread or the repo's stated conventions ask. A rubric that ignores a
family will fail eval packages designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diagnosis-grounded | The plan's stated cause (its diagnosis or cause line, and any "traced to" line in the plan comment), read against every observation in the repro evidence (steps, controls, variants, timings, the Actual line) and the issue body (evidence guide: Diagnosis and grounding) | Passes if the stated cause names a code location or mechanism, explains the behavior the repro evidence pins down, and no repro observation contradicts it (for example, the failure persisting in a run where the named cause was removed or bypassed), and the planned change acts on that cause rather than only on the place where the failure shows up. A cause labeled as a hypothesis passes on the same terms. Fails if a repro observation contradicts the cause, if the cause only restates the symptom ("the undo stack gets lost when the editors switch") or points nowhere ("somewhere in the editor code"), or if the change patches where the failure surfaces while the repro evidence points upstream | required |
| scope-bounded | The plan's change list, its in-scope and not-in-scope statements, and the files it names, each read against the behavior the issue and the repro evidence describe (evidence guide: Scope) | Passes if every change the plan lists is needed to fix the reproduced behavior, to test that fix, or to document the behavior the fix changes, and no item is open-ended. A change at another site that the issue or a maintainer names as the same defect counts as needed, and so does a search or audit whose findings the plan only reports without fixing them. A plan that covers less than the issue passes when it names the part it defers. A missing not-in line does not fail on its own. Fails if any listed change is work the issue does not need (a rename, refactor, reformat, dependency or config migration, a new feature, flag, or option, a docs pass beyond the changed behavior, a fix for a different issue), or if the plan leaves its scope open ("and anything else I find along the way") | required |
| executable | The plan's files or areas, its approach or change steps, and its order of work (evidence guide: Executability) | Passes if the plan names each file, function, or component it will change and the action it takes there, specifically enough that a stranger who knows the codebase could make the first edit without asking the author anything. A plan that names the component and the action may leave the exact function to a stated locating step inside that component (a named trace, log, or search). Fails if a location is unnamed or hedged with no locating step ("somewhere in", "poke around", "figure out where it lives"), if a step states a goal instead of an action ("make undo survive the toggle"), or if the first step depends on a decision the plan leaves open | required |
| test-decisive | The plan's test plan, read against the repro evidence's steps and its Actual line (evidence guide: Test plan) | Passes if the test plan re-runs the repro's trigger, or a test that drives the same code path, and states in advance the observable result expected after the fix (the output, value, state, or timing that the repro shows failing today). Extra checks may sit beside it. Fails if the test plan names no observable result ("run the full test suite", "make sure nothing regresses", "it works after toggling"), or if what it runs does not exercise the reproduced trigger | required |
| honest-claims | Every claim the plan and the plan comment state as fact, and every unknown the package raises (a maintainer's warning, a repro observation the plan does not explain), each read against the repro evidence, the issue body, the thread highlights, and the code the plan cites (evidence guide: Honesty) | Passes if all three hold: (1) no claim stated as fact is contradicted by the repro evidence, the issue, or the thread; (2) every claim that the bug reaches further than the repro shows (other platforms, versions, or code paths), or that the change achieves more than the test plan checks, rests on the issue, the thread, cited code, or a guess label; (3) every unknown the package raises is stated as open, not settled. Inferences the plan draws from the repro evidence count as backed. The cause itself is graded by diagnosis-grounded, not here. Fails if a claim contradicts the package, a reach or outcome claim has nothing behind it, or a raised unknown is presented as settled | required |
| comment-matches-plan | The candidate plan comment, read against the candidate plan (evidence guide: Comms) | Passes if the comment states the plan's approach (what will change, and where) so a maintainer could say yes or no to it, and every approach, scope, test, or result it promises is in the plan. Coordination notes (a cross-reference to another issue or pull request, a report-back promise, what the author will do if another pull request lands first, a question for the maintainers) are not plan content and need not appear in the plan. Fails if the comment promises or claims something the plan does not contain or contradicts, commits to a specific date or deadline ("this weekend", "by Friday"), carries no plan content (enthusiasm, "I'll fix this", "wish me luck"), or contains a +1, a complaint, or pressure on maintainers | required |
| thread-and-conventions | The plan comment and the plan, read against every comment by a maintainer role (OWNER, MEMBER, COLLABORATOR, or a person the thread names as a maintainer) in the thread highlights, and against the contribution policy line of the repo facts; in live mode, the issue thread, CONTRIBUTING, any AI policy file, and the scope.md house rules (evidence guide: Comms) | Passes if both hold: (1) every direction a maintainer gave in the thread (a constraint, a preferred or rejected approach, "working as intended", "discuss first", "the fix is hard", a pointer to another issue or pull request) is engaged in the comment or the plan: named, and either followed or departed from with a reason that leaves the call to the maintainer. When a maintainer lists alternative approaches, naming the one the plan takes engages that comment; the plan need not repeat every alternative or link; (2) the comment and the plan meet every requirement the stated policy places on issue comments, proposals, or AI use (for example AI-use disclosure, comments written by a human, discussion before a pull request). A thread with no maintainer direction passes part (1), and a policy that states no such requirement passes part (2). Fails if either part does not hold | required |

## Verdict rule

<!-- State how the grades above combine into accept or reject, and how
unclear is treated. Example shape (write your own): "accept if every
required check passes; preferred checks never change the verdict;
unclear counts as fail." -->

Accept only if every required check passes; any required check that fails rejects the package.
An `unclear` grade counts as fail on every check.
There are no preferred checks.
A plan that is smaller than the issue is not a failure in itself: scope-bounded passes it when the plan names what it defers, and every other check grades only the part the plan takes on.
