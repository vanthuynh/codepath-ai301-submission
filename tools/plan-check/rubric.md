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
| diagnosis_grounded | The proposed diagnosis in the candidate plan cross-referenced with the issue context and repro-evidence block (referencing the Diagnosis/grounding section of the evidence guide). | Passes if the diagnosis logically explains the reproduced bug without contradicting the provided evidence. If the evidence only shows *where* it broke but not *why*, the plan must frame its proposed cause as a hypothesis to verify, not a guaranteed fact. | required |
| scope_bounded | The candidate plan's defined scope (in/out), targeted files, and technical approach compared against the original user request (referencing the Scope section of the evidence guide). | Passes if the planned work is strictly limited to fixing the reproduced bug and touches only necessary files. Fails if it balloons into unrelated refactoring, feature additions, or orthogonal fixes that aren't strictly required to solve the issue. | required |
| targets_cause | The combination of the diagnosis and the planned implementation, evaluated against the repro evidence and relevant issue context. | Passes if the fix targets the actual root condition uncovered by the grounded diagnosis, rather than just slapping a band-aid over a symptom (like arbitrarily silencing an error). If the root cause is still unproven, it only passes if the plan dictates a concrete verification step before making code changes. | required |
| executable_approach | The specific files, functions, and actionable steps listed in the candidate plan, applying the Executability section of the evidence guide. | Passes if an entirely different developer could pick this up and know exactly where to start and what to change without having to invent the technical strategy themselves. Just stating the desired outcome without the steps to get there is a failure. | required |
| test_plan_decisive | The candidate plan's validation steps evaluated against the repro-evidence trigger, control scenario, commands, and outputs (using the Test plan section of the evidence guide). | Passes if the test plan explicitly reruns the exact bug trigger, defines the correct observable output post-fix, and includes regression checks to ensure the component wasn't just bypassed or fundamentally broken. | required |
| uncertainty_honest | The candidate plan's stated assumptions, unknowns, and risk assessments, applying the Honesty section of the evidence guide. | Passes if the author openly acknowledges critical unknowns instead of faking certainty. Any risks that could derail the implementation must include a plan to investigate them. (Note: Do not fail this if the plan lacks unknowns but the evidence actually provides a clear, 100% complete picture). | required |
| comment_conventions | The draft PR/issue comment evaluated against the repo's thread history, contribution templates, and AI disclosure rules (using the Comms section of the evidence guide). | Passes if the comment accurately reflects the planned change, respects maintainer instructions, and fulfills explicit repository policies (including AI disclosure, if applicable). If the repo is silent on AI use, omitting a disclosure is considered a pass. | required |
| concise_comment | The candidate comment compared against both the source issue and the plan itself. | Passes if it clearly states the "what, why, and how" (diagnosis, change, and validation) without bloat, rambling, or unrelated trivia. | preferred |

## Verdict rule

The package is accepted and ready to execute if and only if all `required` checks evaluate to "pass". A single "fail" on any `required` check triggers an immediate rejection of the package. If any required check returns `unclear`, it must be treated as a "fail" because unresolved ambiguity means the plan is not safe to build from. The `preferred` checks are for constructive feedback only and have zero impact on the final binary accept/reject verdict.
