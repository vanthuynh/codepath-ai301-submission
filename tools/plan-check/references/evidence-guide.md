# Evidence guide: where evidence lives in a plan package

<!--
THIS IS THE PART YOU WRITE (second week running: the judgment files
stay in your hands). The skill uses this guide as its map: for every
kind of evidence a rubric check names, this file says WHERE to find it
in a plan package and WHAT GOOD LOOKS LIKE when you do.

Under each family heading below, write:

- Where it lives: the exact places to look. In an eval bundle (which
  section of the package: the issue context, the repro-evidence block,
  the candidate plan's scope statement or test plan, the plan comment,
  the repo-facts block). In live mode (where on GitHub or in the
  draft: the issue thread, the student's posted repro comment, the
  repo's docs, the draft plan and comment).
- What good looks like: one or two sentences someone else could apply.
  Prefer observable conditions ("the stated cause cites behavior the
  repro evidence actually shows") over adjectives ("diagnosis is
  solid").

A rubric check whose evidence this guide cannot locate is a check
nobody else can execute, and this week that cuts twice: your
procedure.md tells the skill WHEN to gather each family, and this
guide tells it WHERE. Write the map you wish your executor had.
-->

## Diagnosis and grounding

**Where to look:** Find the proposed cause in the first paragraph of the `## Candidate plan` (often called "Cause" or "Diagnosis"). To see if this cause makes sense, look at the `## Repro evidence` section, which includes the step-by-step proof, the "Actual" result, and any "Control runs" (tests done to compare normal vs. broken behavior). You can also read the `## Issue` section to see how the user originally described the problem. In a real project, the cause is in `plan.md`, and the proof is the comment posted on the issue.

**What good looks like:** A good explanation points to a specific broken part (like a specific file, function, or rule) and matches all the facts from the test proof. If a "control run" worked perfectly just by changing one small thing, the explanation must make sense of why that happened. A bad explanation blames something the test already proved was working, just repeats the problem ("the color doesn't change") instead of explaining *why*, or ignores the test facts completely.

## Scope

**Where to look:** Look in the `## Candidate plan` under headings like "Change", "Scope", or "In" to see what they plan to touch, and under "Out" or "Not in scope" for what they promise to leave alone. The specific files or areas mentioned make up the boundary of the work. Compare this list to the root cause they identified to make sure they match.

**What good looks like:** The plan focuses on just one specific task. Every file it mentions is strictly necessary to fix the root cause, and it clearly lists what it will intentionally ignore. It fails if it tries to sneak in extra, unrelated work—like cleaning up old code, upgrading systems, or changing background setups—that isn't needed to fix the actual bug.

## Executability

**Where to look:** Read the "Change" or "Approach" part of the `## Candidate plan`. Compare it with the `## Repo facts` to make sure the files and folders they mention actually exist in this project.

**What good looks like:** A complete stranger could read the plan, open the code, and get straight to work. A good plan gives clear, numbered steps specifying exactly which files, settings, or functions to change, and what to do with them. Vague instructions like "look into it and fix," "make it better," or leaving major decisions up in the air are signs of a bad plan.

## Test plan

**Where to look:** Look at the "Test" or "Test plan" section inside the `## Candidate plan`. Compare their testing steps against the original step-by-step proof in the `## Repro evidence`, especially the "Expected" and "Actual" results.

**What good looks like:** The test repeats the exact steps that caused the bug in the first place and clearly describes what the successful, fixed result will look like (such as a specific message, screen change, or successful completion). Doing the test manually is perfectly fine. Vague statements like "make sure it works" or "add tests" aren't good enough. The test must actually prove the fix worked; a test that would pass even if the bug was still there is a failure.

## Honesty

**Where to look:** Search the `## Candidate plan` for sections labeled "Risks", "Unknowns", or "Not yet checked". Also, look closely at any bold claims made in the plan or the `## Candidate plan comment` that weren't actually proven by the test evidence. In a real project, check the `## Deviations` section at the end of `plan.md` to see if things changed during the work.

**What good looks like:** If the test didn't prove something 100%, the plan honestly calls it an unknown or a detail that needs checking. It doesn't make overly confident statements (like "this fixes it for everyone everywhere") without proof. If the final work ended up different from the original plan, those changes—and the reasons for them—are clearly written out, not just hidden in the final code changes.

## Comms

**Where to look:** Read the `## Candidate plan comment` and compare it against the `## Thread highlights` (to see what the project leaders previously asked for) and the `## Repo facts` (to check the project's rulebooks, templates, or policies about using AI). In a real project, compare this to the GitHub discussion, the project's rulebook, and issue templates.

**What good looks like:** The draft message directly answers any specific instructions given by the project leaders, like testing a specific part or trying a suggested fix. It strictly follows all the project's rules, including mentioning AI use if the rules require it. If the message suggests a completely different fix than what a project leader asked for without explaining *why*, it fails. If there were no specific instructions and no special rules, a simple, polite message is totally fine.
