# Procedure: how this skill grades a plan package

<!--
THIS IS THE PART YOU WRITE, and it is a new kind of part. Weeks 1 and
2, SKILL.md carried a numbered workflow and you only wrote judgment
files. This week the workflow is gone from the frame: SKILL.md says
"execute procedure.md", and these are the operating steps you author.
The machinery is in your hands now.

Your operator swap is the design brief. When your executor stalled
because your rubric said WHAT to decide but not HOW to find the
evidence, that was a procedure gap. This file is where those gaps get
closed: a complete procedure lets someone who has never seen a plan
package before (a groupmate, or the skill itself) grade one exactly the
way you would.

Under each stage heading below, write the concrete steps for that
stage. The one-line note under each heading says what a complete
procedure must decide there. Write steps, not intentions: "read the
repro evidence before the plan, and note what behavior it pins down"
is a step; "understand the context" is a wish.
-->

## Read order

<!-- What gets read, in what order, before any check is graded, and
what to note down from each part while reading. A complete procedure
decides the order (issue first? repro evidence first?) and says why
the order matters for the checks that come later. -->

1. Decide the mode. A markdown bundle with `## Candidate plan` is eval mode: the bundle is the whole world, fetch nothing, ignore `scope.md` and `voice-guide.md`. Draft files plus an issue URL is live mode: read `scope.md` first and stop as it directs if the issue is outside the scoped repo or the `Repo:` line is a placeholder.
2. Read the context, in this order, before reading any candidate text, so the plan is judged against the evidence rather than the evidence read through the plan's framing.
3. Repo facts (live: the policy files named in the evidence guide's Comms section). Note each requirement the contribution policy places on issue comments, proposals, or AI use, quoted. If it states none, note "no requirement".
4. Issue. Note the trigger, the expected behavior, and the failure, in the issue's own words.
5. Thread highlights (live: the full thread). For each comment by a maintainer role, note its direction verbatim. For each cause or fix proposed by anyone else, note it as unverified.
6. Repro evidence (live: the student's posted repro comment). Note the Actual line and every control, variant, and timing observation as a numbered list. This list is what diagnosis-grounded and test-decisive are graded against.
7. Candidate plan (live: `plan.md`). Note the stated cause, each listed change with its file or location, any in, not-in, or deferred statement, the test plan with its expected result, any risks or unknowns, and every sentence stated as fact.
8. Candidate plan comment, last (live: the comment draft). Note every promise, every claim, any date, and every mention of the thread or the repo's policy.

## Evidence gathering

<!-- For each evidence family your rubric's checks name, the concrete
gathering move: which part of the package (or, live, which page or
thread location per your evidence guide) to pull the fact from, and
what to record. A complete procedure leaves no check whose evidence an
executor would have to hunt for. -->

Build one record per check from the notes above, using the locations in `references/evidence-guide.md`. Quote; do not paraphrase.

1. diagnosis-grounded: quote the stated cause. Against each numbered repro observation from Read order step 6, write "explained", "contradicted", or "not addressed". Write whether the planned change acts on the cause or only where the failure shows up.
2. scope-bounded: list every change the plan names. Tag each "fixes the reproduced behavior", "tests the fix", "documents the changed behavior", or "not needed by the issue". Quote any deferral statement and any open-ended phrase.
3. executable: for each change, record the file, function, or component named and the action taken there. Quote any hedged location or goal-only step.
4. test-decisive: quote the test plan. Record which repro step or code path it runs and the expected after-result it states, or "none stated".
5. honest-claims: list every claim of cause, reach, or outcome from the plan and the comment. For each, record its backing: the repro line, the cited code, a guess label, or "none". List every unknown the package raises (maintainer warnings, unexplained repro observations) and whether the plan leaves it open.
6. comment-matches-plan: list every promise and claim in the comment and, next to each, the plan sentence that contains it, or "not in plan". Quote any date, +1, or pressure. Record whether the comment states the approach and where it lands.
7. thread-and-conventions: next to each maintainer direction from Read order step 5, quote where the comment or plan addresses it, or "not addressed". Next to each policy requirement from Read order step 3, quote where it is met, or "not met".

## Check execution

<!-- How one check runs against gathered evidence: in what order the
checks execute, what an executor does when evidence for a check is
genuinely absent, and when a check may be graded without re-reading
the whole package. A complete procedure makes two executors grade the
same package the same way. -->

1. Grade the checks in the rubric table's order, each one against its own pass condition in `rubric.md`, read literally.
2. Grade from the check's record. Re-read a package part only when the record lacks a fact the pass condition asks about, and add that fact to the record before grading.
3. Grade each check on its own condition. A failure on one check never decides another: a wrong diagnosis does not by itself fail test-decisive, and a good test plan does not rescue a wrong diagnosis.
4. When the package lacks something a pass requires (no test plan, no location, no stated cause), grade fail and write the evidence as "absent: <what is missing>".
5. When a condition has nothing to apply to (a thread with no maintainer comment, a policy with no requirement), that part passes; write the evidence as "no maintainer direction in thread" or "policy states no requirement".
6. Grade unclear only when the package gives two readings you cannot settle (the plan says two different things about the same change). Write both readings in the evidence.
7. Every grade carries one line of evidence quoted from the package that decided it.

## Verdict assembly

<!-- How the per-check grades become the final accept or reject:
apply your rubric's verdict rule, state how unclear grades enter it,
and say what gets quoted in the output for the deciding check. A
complete procedure produces the same verdict from the same grades,
every time. -->

1. Collect the seven grades. Count every `unclear` as fail, as the verdict rule says.
2. Verdict `accept` if all seven pass; otherwise `reject`.
3. In the readable summary, print one line per check with its grade and evidence. For a reject, name the first failing check in table order as the deciding check and quote its evidence; list the other failing checks after it.
4. In live mode, add the voice-guide notes after the check lines, quoting each broken rule. They never change the verdict.
5. End with the fenced JSON block from SKILL.md, check names exactly as the rubric writes them, and nothing after it.
