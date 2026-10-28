# Procedure: how this tool grades a PR package

## Read order

1. Decide the mode. A markdown bundle with a `## Candidate PR` section is eval mode: the bundle is the whole world, fetch and run nothing, and ignore `scope.md` and `voice-guide.md`. An issue URL plus pointers to `plan.md`, the branch, a PR draft, and test evidence is live mode: read `scope.md` first, and stop as SKILL.md directs if the `Repo:` line is a placeholder or the issue is outside the scoped repo.
2. Read the reference side before the PR side, in the order below, and take notes from each part before opening the next. The plan and the repo's asks are what the PR is held to; reading the description first would frame the diff through the author's own summary, which is exactly what silent drift hides behind.
3. Repo facts (live: `.github/PULL_REQUEST_TEMPLATE.md`, `docs/CONTRIBUTING.md`, the `scope.md` house rules, and the course's AI-disclosure ask in the evidence guide). Write a numbered list of the asks: each template section, each checklist item, each required line (such as "closes #N"), each required file entry (changelog, release notes), each title, branch, or commit convention, each required check or suite, and the AI policy quoted (or "no stated AI policy").
4. Issue and thread highlights (live: `gh issue view <n> -R <repo> --comments`). Note the issue number and the behavior the issue says is broken.
5. Plan context (live: `plan.md`, then the student's repro comment on the issue). From the repro evidence, note the trigger and the failing output. From the plan, write four lists: (a) deliverables, one line per change, file, test, doc, or changelog promise in its scope; (b) the not-in-scope items; (c) every deferral, known-limit, or deviation note, quoted; (d) the test plan, one line per repro re-run, failure mode, and control, each with its stated expected-after, and one line per suite or check command it names.
6. Diff and commits, before the PR text (live: `git diff main...HEAD`, `git diff main...HEAD --stat`, `git log --oneline main..HEAD`). Number every hunk as `H1`, `H2`, ... with its file and a few words on what it changes. Copy the commit messages.
7. Title and description last (live: `pr_draft.md`, first line the title). Note every statement about what changed, what is included, how faithful it is to the plan, and what was tested, each quoted; note each section heading and checklist answer; note any AI-use disclosure, quoted.
8. Test evidence (live: `test_evidence.md`). Note each command run, with the output pasted for it, labeled before or after.

## Evidence gathering

Build one record per check from the notes above, using the locations in `references/evidence-guide.md`. Quote; do not paraphrase.

1. diff-within-plan: for each hunk `H1`, `H2`, ..., write the plan deliverable, repo ask, or deviation note it serves, quoted, or "not in plan". Add each commit message that announces work tagged "not in plan". Mark any hunk that touches a not-in-scope item.
2. plan-delivered: for each deliverable in list (a), write the hunk number that delivers it, or "absent". For each "absent", quote the deferral from list (c) and the sentence in the description that states it, or write "no plan note" or "not in description".
3. description-matches-diff: for each statement from Read order step 7 about what changed or what is included, write the hunk that shows it, or "not in diff". For each fidelity claim ("exactly", "in full", "nothing else"), write the hunks the records of steps 1 and 2 marked "not in plan" or "absent", or "none".
4. repro-evidence-decisive: sort each item in list (d) into "reproduced failure" (a trigger or failure mode the repro evidence shows failing) or "other" (a control, a smoke check, a scenario for new behavior, a parity comparison). For each reproduced failure, copy the evidence lines that show it run on the changed code, and write the observed result next to the stated expected-after; write "not shown" when the evidence never mentions it, "characterized" when it states a result with no command or steps behind it, and "different trigger" when the run uses another input or path than the plan's trigger. For each other item, copy the sentence or named test that reports its outcome, or write "not mentioned".
5. repo-checks-reported: for each suite or check command in list (d) and each required check in the asks from Read order step 3, copy the outcome line from the evidence or description, or write "not reported". For each failing outcome, copy its stated reason, or write "no reason".
6. diff-free-of-debris: scan every added line and hunk for the six debris kinds in the rubric and the evidence guide's Diff quality tells. List each hit with its hunk number and the line quoted. List each commit message that names no change. Write "none found" if the scan finds nothing.
7. template-and-asks-met: for each ask in the numbered list from Read order step 3 (except the AI policy), write where the PR meets it (the section, the line, the hunk), "not applicable" with the reason, or "not met".
8. ai-disclosure-met: quote the AI policy's disclosure requirement, or write "no disclosure requirement". If one exists, quote the description's disclosure, or write "none", and list anything the policy requires the disclosure to state that it leaves out.

## Check execution

1. Grade the checks in the rubric table's order, each against its own pass condition in `rubric.md`, read literally.
2. Grade from the check's record. Re-read a package part only when the record lacks a fact the pass condition asks about, and add that fact to the record before grading.
3. Grade each check on its own condition. A failure on one check never decides another, except where a pass condition says so (description-matches-diff reads the hunk lists from the diff-within-plan and plan-delivered records to test fidelity claims). Excellent evidence does not rescue a drifting diff, and a clean diff does not rescue missing evidence.
4. When the package lacks something a pass requires (no test evidence, no named suite outcome, no disclosure a policy demands), grade `fail` and write the evidence as "absent: <what is missing>".
5. When a condition has nothing to apply to (no template and no stated asks, no AI policy, a test plan that names no suite), that part passes; write the evidence as, for example, "repo states no PR asks" or "no stated AI policy".
6. Grade `unclear` only when the package supports two readings you cannot settle from the text (the plan says two different things about the same file). Write both readings in the evidence.
7. Every grade carries one line of evidence that decided it: a hunk with its file, a quoted description sentence, or a quoted output line.

## Verdict assembly

1. Collect the eight grades. Count every `unclear` as `fail`, as the verdict rule says.
2. Verdict `accept` if all eight pass; otherwise `reject`.
3. In the readable summary, print one line per check in rubric order: name, grade, evidence.
4. For a reject, name the deciding check: the first failing check in rubric order. Quote its evidence line, then list the other failing checks after it.
5. Print any `Procedure gap:` or `Evidence-guide gap:` lines next.
6. In live mode, add the voice-guide notes after that, each quoting the broken rule and the phrase that breaks it; they never change the verdict. Then, for a reject, list one concrete fix per failing check.
7. End with the fenced JSON block from SKILL.md: `item` (issue URL in live mode, bundle id in eval mode), the eight checks in rubric order with names exactly as the rubric writes them, the verdict, and nothing after it.
