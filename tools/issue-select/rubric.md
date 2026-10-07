# Rubric: is this a good first issue?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever checks
you define here. It ships empty on purpose: the judgment is your work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where. Name the source
     (repo-facts block, issue body, comment thread, or the locations in
     references/evidence-guide.md). "The repo" is not a source; "the last
     5 default-branch commit dates" is.
   - Pass condition: a condition someone else could apply and get your
     answer. Prefer thresholds with numbers ("a maintainer commented
     within 30 days") over adjectives ("maintainer is responsive").
   - Weight: `required` (a fail here rejects the issue) or `preferred`
     (never changes the verdict; a nice-to-have that helps rank the
     issues you accept).

2. A verdict rule below the table: how the check grades combine into
   accept or reject, including how `unclear` is treated. The verdict
   space is binary. If you write no rule for `unclear`, the skill treats
   it as fail.

Cover what actually kills first contributions. The lecture named four
families: the maintainer is alive, the repo is in use, the scope fits a
newcomer, and nobody else is already on it. A rubric that ignores a family
will fail eval issues designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| maintainer-alive | Repo facts: "last 5 default-branch commits" (date and author of each) and "maintainer first-response sample" (days to first owner/member/collaborator comment); in live mode also the repo's recent issue and pull request activity (closes and comments by OWNER/MEMBER/COLLABORATOR) | Both hold: (1) at least one of the last 5 default-branch commits is authored by a human, or is a bot merging a human's pull request, and is dated 90 days or less before the capture date (live mode: before today); (2) at least 1 of the 5 sampled issues shows a maintainer first response of 90 days or less ("no maintainer comment in thread" does not count), or an OWNER/MEMBER/COLLABORATOR closed or commented on any issue or pull request in the repo 90 days or less before the capture date (live mode: before today) | required |
| repo-in-use | Repo facts: "archived:" on the repo line, "latest release", and "last push to any branch" | "archived: no", AND either the latest release or the last push to any branch is dated 90 days or less before the capture date (live mode: before today). An archived repo always fails | required |
| newcomer-scope | Issue body; comment thread (each comment's author_association); Repo facts "this issue: linked PRs:" with state, plus PRs mentioned in comments | Passes unless any of these holds: (1) the thread shows the design still being debated and no OWNER/MEMBER/COLLABORATOR comment has settled it; (2) an OWNER/MEMBER/COLLABORATOR comment states the fix requires changes to core internals; (3) the issue is a usage or support question asking how to get something working, not a change to make; (4) 3 or more closed, unmerged PRs (linked or mentioned in comments) already attempted it; (5) the issue is explicitly an umbrella or tracking issue: its body or title calls itself one (umbrella, tracking, meta, or megaissue), or an OWNER/MEMBER/COLLABORATOR says it should be split into separate issues. A list of causes or candidate fixes for one problem is not an umbrella. A short body alone is not a failure | required |
| not-claimed | Repo facts "this issue: assignees:" and "linked PRs:" with state; comment thread for claim comments ("I'll take this", "can I work on this", "working on this") and PRs mentioned in comments | Fails if any of these holds: (1) assignees is not none; (2) an open PR is linked or mentioned in a comment; (3) a claim comment is dated 30 days or less before the capture date (live mode: before today). A closed, unmerged PR is not a claim. In live mode, apply the scope.md house rule on classmates' claims | required |
| ai-policy | Repo facts "contribution policy" line (CONTRIBUTING.md, AI policy files, PR templates it quotes or summarizes) | Fails only if the policy states that AI-assisted contributions are not accepted at all. Conditions (disclose AI use, understand and test every change, human review of AI output) pass. A ban on fully AI-generated work that still allows assistive AI use passes. No stated policy passes | required |
| newcomer-label | Issue header "labels:" line | Labels include "good first issue", "good-first-issue", or "beginner" (any capitalization) | preferred |
| maintainer-in-thread | Comment thread: each comment's author_association | At least one comment in this issue's thread is by an OWNER, MEMBER, or COLLABORATOR | preferred |

## Verdict rule

<!-- State how the grades above combine into accept or reject, and how
unclear is treated. Example shape (write your own): "accept if every
required check passes; preferred checks never change the verdict, they
rank accepted issues; unclear counts as fail." -->

Accept only if every required check passes; any required check that fails rejects the issue.
An `unclear` grade on maintainer-alive or not-claimed counts as fail; an `unclear` grade on any other check counts as pass.
Preferred checks never change the verdict; they only rank accepted issues.
