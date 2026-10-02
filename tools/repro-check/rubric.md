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

| Check                  | Evidence                                                                                                                                        | Pass condition                                                                                                                                                                                                                                                                                                                                                                   | Weight    |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------- |
| environment_recorded   | The environment block in the repro report, cross-checked against the original issue context and repository details.                             | Passes if the report clearly lists the setup details needed to understand the outcome (OS, language/runtime versions, git branch/commit). If the tester's environment differs from what the issue reported, they must explicitly call that out. Mark it `unclear` if there's not enough info to know how they ran it.                                                            | required  |
| steps_followable       | The setup state and step-by-step actions in the report, checked against the original issue and standard repo setup docs.                        | Passes if any other dev could take the stated environment, follow the steps exactly, and hit the same result without having to guess missing commands or prerequisites. Fails if crucial setup steps, inputs, or triggers are left out.                                                                                                                                          | required  |
| behavior_matches_issue | The actual artifacts (logs, screenshots, stack traces, test outputs) compared directly to the symptoms described in the original issue.         | Passes if the attached evidence shows the exact bug the user reported, or clearly proves the bug didn't happen during this specific test. Fails if the evidence just shows a different error, a crash from bad setup, or tangential behavior that doesn't validate the actual issue.                                                                                             | required  |
| outcome_honest         | The final verdict stated in the report versus what the steps and artifacts actually prove.                                                      | Passes if the author doesn't over-promise. If they claim they reproduced it, the evidence must back that up. If they claim they _couldn't_ reproduce it, they need to show they tried the right things and the bug just didn't happen (while noting any environment differences). Fails if they confidently state a result that their logs/screenshots don't support.            | required  |
| repo_conventions       | The actual text of the claim and repro comments, reviewed against the repo's contribution guidelines, issue templates, and AI disclosure rules. | Passes if the comments follow the repo's specific rules. A "claim" comment needs to link to the issue and offer to look into it, without prematurely promising a fix or confirming the bug. The repro comment needs to accurately reflect the work done. If the repo demands AI-generated content disclosure, it must be there (if the repo is silent on this, silence is fine). | required  |
| issue_specificity      | The text of the claim and repro comments compared to the original issue details.                                                                | Passes if the comments use specific context from the issue (naming the exact component, error message, or input). Fails if it reads like generic, copy-paste boilerplate that could be dropped onto any random issue.                                                                                                                                                            | preferred |

## Verdict rule

A package is only approved to post if it passes 100% of the `required` checks. If a single `required` check fails, reject the whole package. Treat any `unclear` rating on a required check as a failure—if we don't have enough evidence to be sure, it's not ready to ship. Checks marked `preferred` are just for constructive feedback and will never cause a package to be rejected or accepted on their own.
