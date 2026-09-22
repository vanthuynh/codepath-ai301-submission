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

| Check                                       | Evidence                                                                                                                                                                                  | Pass condition                                                                                                                                                                                                                                                                                                                                                                       | Weight    |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- |
| maintainer-alive                            | Repo facts: "last 5 default-branch commits" dates, and "maintainer first-response sample" (or, live mode, the commit list and recent issue replies from Owner/Member/Collaborator badges) | At least one default-branch commit within 90 days of the capture/today date, OR the maintainer first-response sample shows a reply from an Owner/Member/Collaborator within 14 days of an issue being opened                                                                                                                                                                         | required  |
| repo-in-use                                 | Repo facts: "archived:" flag, "latest release" date, "last push to any branch" date, star count (or the equivalent live-mode sidebar signals)                                             | Not archived, AND (latest release within the last 12 months OR last push within the last 90 days OR stars >= 100)                                                                                                                                                                                                                                                                    | required  |
| Scope-bounded                               | Issue body and comment thread text                                                                                                                                                        | Fails if the issue is an umbrella/tracking issue listing multiple sub-items, if the thread shows an unresolved design debate with no maintainer decision, if a maintainer states the fix touches core internals, or if the issue is a pure usage/support question rather than a concrete bug or feature ask. A terse body or missing repro steps does not by itself fail this check. | required  |
| issue-unclaimed                             | Repo facts: "this issue: assignees:", "linked PRs:" with state, and the Comments section for claim comments (or the live-mode Assignees box, Development box, and thread)                 | All of: assignees is "none", an open linked PR addressing the issue, or a claim comment ("I'll take this" / "working on this") within the last 14 days that went unanswered/unchallenged by a maintainer. A closed, unmerged linked PR (abandoned attempt) does not fail this check.                                                                                                 | required  |
| Contribution policy allows AI-assisted work | Repo facts: "contribution policy" line (or CONTRIBUTING.md / AI_POLICY.md / PR template on github.com)                                                                                    | Fails only on an outright ban on AI-generated/AI-assisted contributions. Disclosure, human-review, or testing conditions pass. Silence passes.                                                                                                                                                                                                                                       | required  |
| Good-first-issue label                      | Issue labels in the repo-facts block or issue sidebar                                                                                                                                     | Issue carries a "good first issue" (or equivalent newcomer-friendly) label                                                                                                                                                                                                                                                                                                           | preferred |
| Clear acceptance criteria                   | Issue body                                                                                                                                                                                | Issue states concrete acceptance criteria or reproduction steps a newcomer could work from without further clarification                                                                                                                                                                                                                                                             | preferred |

## Verdict rule

<!-- State how the grades above combine into accept or reject, and how
unclear is treated. Example shape (write your own): "accept if every
required check passes; preferred checks never change the verdict, they
rank accepted issues; unclear counts as fail." -->

Accept if all required checks pass. Reject the issue if any required check `fail` with `clear` evidence. Preferred checks never change the verdict; they only rank issues that are already accepted, most preferred-passes first.
