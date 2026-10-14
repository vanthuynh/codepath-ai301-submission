# Rubric: is this pull request ready to submit?

<!--
THIS PART IS YOURS TO WRITE (fourth week running; this is the rubric's
final form in the sandbox). The frame in SKILL.md runs whatever checks
you put here, using your procedure.md. It ships empty on purpose,
because the judgment calls are the part that's yours.

A finished rubric needs two things:

1. At least one row in the checks table, with all four columns filled in:
   - Check: a short name. It shows up in the output JSON.
   - Evidence: exactly what to look at, and where in the package. Name
     the part (the diff read against the plan's scope, the test
     evidence read against the plan's test plan, the description read
     against the diff, the template asks in the repo-facts block), or
     point to a location in your references/evidence-guide.md. "The PR"
     isn't a source. "The diff's changed files read against the plan's
     stated boundary" is.
   - Pass condition: a decision rule about the outcome, written so
     someone else could apply it and land on your answer. Judge the
     thing itself (does the diff fall inside the plan plus its
     deviation notes? can the claimed evidence actually be observed?)
     and never the shape of the write-up (how long the description is,
     how many commits there are). Checks built around structure are
     what make graders disagree with themselves.
   - Weight: `required` (a fail here holds the package) or `preferred`
     (never changes the verdict).

2. A verdict rule under the table that says how the check grades
   combine into accept (submit) or reject (hold). The verdict is
   binary, and the rule has to say how `unclear` is treated. The frame
   here is your SKILL.md, so a rubric that says nothing on this is only
   covered if your frame's grading discipline spells out what happens
   (the contract's own default is that a claim nobody can verify
   fails).

Cover what actually gets bad PRs submitted. The failure families from
the lecture are the harness's scoring categories, with the same names
as the eval README: silent drift (the diff quietly does more or less
than the posted plan, or the description claims fidelity the diff
contradicts), not tested (the evidence proves nothing observable, or
the repo's own checks were never run), unreviewable (debris or
unrelated hunks bury the change), and standards wall (the repo's stated
template and disclosure asks get ignored). The four headings in your
evidence guide line up with these one to one: plan fidelity is silent
drift, test evidence is not tested, diff quality is unreviewable, and
standards and comms is standards wall. The category floor is scored on
exactly these names, plus clear accept. If your rubric skips a
category, it will fail the eval packages built around that category.

And remember the honest-outcome rule, fourth week running: a PR that
openly discloses a shortfall can be ready. A rubric that treats "less
than everything" as "hold" fails the set.
-->


## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| plan_fidelity | The candidate plan's scope, approach, and Deviations, read against the PR diff's changed files and behavior. Also the PR description's claims about how it was implemented. | Passes when every material change in the diff is either covered by the plan or explained in Deviations, and the implementation tackles the issue the plan was written for. Fails when the diff quietly adds unrelated functionality, drops a promised core change without saying so, or contradicts the plan while claiming to follow it. A relevant deviation that's honestly documented doesn't fail on its own. | required |
| description_matches_diff | The candidate PR's title, summary, change list, and issue reference, read against the actual diff and the plan. | Passes when the description correctly names the issue, describes the behavior that was really implemented, and neither claims changes that aren't in the diff nor hides material ones. If the implementation is narrower than originally planned but the limitation or unfinished item is honestly disclosed, that alone is not a failure. | required |
| test_evidence | The candidate's test-evidence block and the testing claims in its description, read against the plan's test plan and the original reproduction evidence. | Passes when the evidence shows commands that were actually run and results you can observe, and those results speak directly to the original failure and show the implemented behavior working. Before/after observations (or equivalent regression evidence that settles the question) must establish whether the targeted behavior changed. Fails when the PR only promises testing, makes pass claims nothing supports, or shows only unrelated tests. A failing check that's honestly reported is fine, as long as its relevance and limits are explained. | required |
| repository_checks | The candidate's test evidence, read against the repo-facts block's stated checks, contribution policy, and PR template. | Passes when the checks the repo requires were run and their real outcomes are recorded, including failures, skipped suites, or tests that weren't available. Fails when required checks are left out, falsely reported as passing, or swapped for unrelated commands with no explanation. A disclosed failure, or an empty test suite, doesn't automatically fail as long as the evidence accurately says what happened. | required |
| diff_reviewability | The PR diff, its changed-file list, and the commit contents, read against the planned scope and the issue. | Passes when the diff holds only issue-relevant implementation, tests, and supporting changes that are truly needed, with no debug statements, accidentally committed local files, commented-out experiments, generated debris, or unrelated refactors getting in a reviewer's way. Fails when unrelated or throwaway changes are still in the submitted diff. | required |
| standards_and_disclosure | The candidate PR's title and description, read against the repo-facts contribution policy, the PR template, the issue thread, and any AI-use disclosure requirements. | Passes when everything the PR template explicitly requires is there and the repo's contribution conditions are met, including AI-use disclosure where it's required. Fails when a mandatory section, issue reference, required disclosure, or stated policy condition is missing. A brief answer, an explained N/A, or an honestly disclosed shortfall is acceptable where the repo allows it. | required |
| reviewer_clarity | The candidate PR description and the diff, read together. | Passes when a reviewer can work out the problem, the changed behavior, the affected files, and the test outcomes without having to piece the submission together from unrelated material. Otherwise it fails. | preferred |

## Verdict rule

Return `accept` only when every required check passes. A `fail` on any
required check means `reject`. An `unclear` on a required check also
means `reject`, since the package in hand doesn't show that the PR is
ready.

Preferred checks are there for feedback and can help explain why one PR
is better than another, but they never change the final binary verdict.

Grade every check, even if an earlier required one has already failed.
Use only the evidence you have and the pass conditions above. Don't
reject a PR just because a relevant deviation, limitation, test failure,
or unavailable test is honestly documented. Instead, decide whether the
evidence is sufficient and whether the repo's requirements were met.