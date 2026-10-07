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
1. **Start with the original problem report.** Write down exactly what the user says is broken, what they actually want to happen, what they want fixed, and any notes from the project managers.
2. **Look at the proof next.** Read the steps showing how the bug happens. Jot down the exact steps to recreate the problem, what the error looks like, what a normal working state looks like, and what the system environment is. Be very clear with yourself about what this proof actually guarantees versus what it leaves out.
3. **Now read the proposed plan.** With the problem fresh in your mind, read how they want to fix it. Take notes on what they think the root cause is, what boundaries they set for the work, the specific files they want to change, their step-by-step approach, how they will test it, and any risks or things they admit they aren't sure about.
4. **Read their draft message.** Read the message they plan to post to the team. Doing this after reading the plan helps you see if their message honestly matches the work they intend to do.
5. **Check the house rules.** Before you judge their communication style, read the project's general guidelines, rulebooks, templates, and any past conversations on the topic to understand the local culture.
6. **Stay grounded in the facts.** Never let a highly confident-sounding plan distract you from the hard evidence. The original problem report and the proof of the bug are your ultimate source of truth.

## Evidence gathering

<!-- For each evidence family your rubric's checks name, the concrete
gathering move: which part of the package (or, live, which page or
thread location per your evidence guide) to pull the fact from, and
what to record. A complete procedure leaves no check whose evidence an
executor would have to hunt for. -->
1. **To check the diagnosis (`diagnosis_grounded`):** Connect every reason the plan gives for the bug back to the actual proof. Write down whether the proof guarantees their reason is correct, or if their reason is just a really good guess.
2. **To check the boundaries (`scope_bounded`):** Write down what the user asked for, what the plan promises to do, what it promises *not* to do, and every file it touches. Highlight any extra fluff—like random code cleanup, redesigns, or new features—that don't belong in this specific bug fix.
3. **To check the root cause (`targets_cause`):** Note what they claim the problem is and exactly what code they plan to change to fix it. If they aren't totally sure what's causing the bug, write down how they plan to double-check their theory before changing the code.
4. **To check if it's doable (`executable_approach`):** List the specific files, areas, and step-by-step actions they wrote down. Note any missing steps where another worker would have to stop and guess what to do next.
5. **To check the test plan (`test_plan_decisive`):** Write down the exact steps that trigger the bug, then see if their test actually runs those steps. Write down what they expect to see once it's fixed, and note if they included checks to ensure they didn't accidentally break other parts of the system.
6. **To check for honesty (`uncertainty_honest`):** Collect all their assumptions, risks, unknowns, and bold claims. Compare these against what you actually know from the proof. Are they pretending to be certain about things they can't possibly know?
7. **To check communication (`comment_conventions`):** Write down any specific project rules about posting (like using specific templates or declaring the use of AI). Note whether the claims in their draft message match these rules and the actual plan.
8. **To check for brevity (`concise_comment` - bonus check):** Look at the draft message and compare it to their actual plan. Write down if the message clearly explains the "what, why, and how" without rambling about unrelated topics.

## Check execution

<!-- How one check runs against gathered evidence: in what order the
checks execute, what an executor does when evidence for a check is
genuinely absent, and when a check may be graded without re-reading
the whole package. A complete procedure makes two executors grade the
same package the same way. -->
1. **Grade `diagnosis_grounded` first.** Everything else depends on whether they actually understand why the bug is happening.
2. **Grade `scope_bounded` second.** This establishes the "fence" around the work so you know how big the project is.
3. **Grade `targets_cause`.** Compare their step-by-step fix to their diagnosis. Don't let them pass if they are just hiding an error message—they need to fix the underlying problem (unless the proof shows hiding it is the only option).
4. **Grade `executable_approach`.** Judge them only on the concrete steps they actually wrote down. Do not do the work for them by imagining the missing steps.
5. **Grade `test_plan_decisive`.** Check this against the bug trigger. Saying something generic like "I will run the tests" is a failure. They need to say exactly what they will look for.
6. **Grade `uncertainty_honest`.** Make sure their level of confidence matches the actual facts, and that they have a clear path to figure out the things they don't know.
7. **Grade `comment_conventions`.** Judge this strictly against the project's actual rules. If the project doesn't require a special AI disclosure, don't fail them for not having one.
8. **Grade `concise_comment` last.** This is just a bonus check to improve quality; it won't stop the plan from moving forward.
9. **Use strict grading terms.** Score a `pass` if the evidence clearly meets the goal. Score a `fail` if it falls short or contradicts the facts. Score an `unclear` if there just isn't enough information to tell. Never pretend information is there if it's missing.
10. **Reuse your notes.** You don't need to reread everything from scratch for every single check. Use the notes you took during the "Gathering your evidence" phase.

## Verdict assembly

<!-- How the per-check grades become the final accept or reject:
apply your rubric's verdict rule, state how unclear grades enter it,
and say what gets quoted in the output for the deciding check. A
complete procedure produces the same verdict from the same grades,
every time. -->
1. Gather up your grades for every check on the list.
2. Follow the absolute rule: for the plan to get an `accept` decision, **every single required check must pass**.
3. If a required check gets an `unclear` grade, treat it as a failure and `reject` the plan. If we don't have enough information, the plan isn't safe to use yet.
4. Never let the bonus (preferred) check change your final accept/reject decision.
5. If you have to reject the plan, clearly state which required checks failed. Pull a short, specific quote from the plan, the proof, or the rules to show exactly why it failed.
6. If you accept the plan, write a brief sentence for each required check explaining the solid evidence that made it pass.
7. Wrap up your review by providing the required system data block (in JSON format), listing the check names and your final, binary decision: `accept` or `reject`.
