# Procedure: how to grade a PR package

Work through these steps in order. Don't grade a single check until
you've finished Read order and Evidence gathering.
`references/evidence-guide.md` tells you where each fact lives. This
file tells you when to pull it and what to do with it. Most of the
gathering this week is side-by-side pairing: the diff against the plan,
the evidence against the test plan, the description against the diff,
and the PR against the repo's asks.

## Read order

1. Read `rubric.md`. List the check names in table order, along with
   their weights and the verdict rule. Then read
   `references/evidence-guide.md`.
2. Read the **plan first**. In eval mode that's the "Plan context"
   block. In live mode it's `plan.md` with its `## Deviations` section,
   or the house plan. Write down these lists:
   - PLAN-CHANGES: each change the plan commits to (code, test, docs,
     changelog), one line each.
   - PLAN-FILES: every file the plan names.
   - NOT-IN-SCOPE: everything the plan explicitly rules out.
   - NOTES: every deviation, deferral, or "known limit" note, word for
     word.
   - TEST-PLAN: each observable the test plan names (each repro or
     failure mode, each expected-after, each control).

   Read the plan before the PR so the PR's own story can't set the
   frame for you. The plan is the yardstick the diff gets measured
   against.
3. Read the **repro evidence** inside the plan context. Write down the
   failing step's exact inputs, its "Actual" result, and its "Expected"
   result, plus each control run and what it shows.
4. Read the **issue** and the **thread highlights**. Note any direction
   from an OWNER, MEMBER, or COLLABORATOR, since it may justify a note in
   the plan.
5. Read the **Repo facts** block. In live mode, that means
   `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md`. Write
   down a list of every stated PR ask: each template section, each
   checklist item, each required file entry (such as a changelog or
   whatsnew entry), the required form of the issue link, and
   tests-with-changes. Also copy down the AI-policy rule word for word.
6. Read the **Diff**. In live mode, that's `git diff main...HEAD`. Write
   down DIFF-FILES: every changed file, and for each file, each hunk in
   one line (for example, "ui.lua: escapes metacharacters in win32
   branch" or "fd.rs: rewords --hidden help text").
7. Read the **Commits** list.
8. Read the **Title** and **Description** last. Write down every factual
   claim they make about what the PR changes or contains.
9. Read the **Test evidence** section.

## Evidence gathering

For each check, pull exactly these facts:

- **diff-matches-plan:** Build a two-column pairing. For each line in
  DIFF-FILES, write the PLAN-CHANGES item or NOTES entry it belongs to,
  or write "UNPLANNED". A regression test for the planned change counts
  as belonging to it, and so does a changelog or docs entry that the plan
  or the repo asks for. Then go the other way. For each PLAN-CHANGES
  item, write the hunk that delivers it, or "MISSING" along with the
  NOTES entry that records its absence, or "MISSING, no note". Flag any
  hunk that does something listed in NOT-IN-SCOPE.
- **description-matches-diff:** For each claim from read step 8, write
  the hunk that makes it true, or "NOT IN DIFF". Fidelity claims
  ("exactly as planned", "no changes beyond the plan", "no functional
  changes outside X") are true only if the diff-matches-plan pairing has
  no UNPLANNED row.
- **decisive-evidence:** For each TEST-PLAN observable, quote the
  test-evidence line that shows it (the command plus the output shown),
  or write "NOT SHOWN". For each run you quote, compare its inputs to the
  repro's failing step. Is it the same inputs and code path, or is it a
  control, an unchanged path, or a different input? Note whether a
  before result is shown or quoted, and whether the after result matches
  the Expected line. Tag each TEST-PLAN observable as FAILURE-MODE (a
  repro of the reported bug) or SECONDARY (a control, an added warning or
  side behavior, a suite run). Only a FAILURE-MODE observable needs shown
  output. A SECONDARY one that's stated in prose counts as shown.
- **clean-diff:** Scan every `+` line and every hunk. List each debug
  print, each commented-out code line, each dead or unused function, and
  each stray TODO or FIXME. Also list each hunk that is only
  re-indentation, re-formatting, import reordering, or identical
  re-printed lines. Quote the commit messages.
- **template-asks:** For each ask from read step 5, write where it's met
  (the description heading or sentence, or the diff file for a required
  entry), or "NOT MET".
- **ai-disclosure:** Quote the AI rule from read step 5. Say whether it
  covers PRs or all AI use. Then quote the description's disclosure
  sentence, or write "no disclosure".
- **repo-checks-run / commit-hygiene:** Quote the suite or lint command
  and its outcome from the evidence. Quote the commit messages.

In eval mode, all of this comes only from the bundle. In live mode, the
diff and commits come from git, the candidate text comes from
`pr_draft.md`, the evidence comes from `test_evidence.md`, the plan comes
from `plan.md`, and the asks come from the repo's template and
CONTRIBUTING.md.

## Check execution

1. Run the checks in rubric table order: diff-matches-plan,
   description-matches-diff, decisive-evidence, clean-diff,
   template-asks, ai-disclosure, then the preferred checks.
2. Apply each pass condition word for word to the facts you gathered for
   it, and grade it `pass` or `fail`. Write one line of evidence: the
   quote, hunk, or missing item that decided it.
3. **diff-matches-plan** fails on any UNPLANNED row, any "MISSING, no
   note", or any NOT-IN-SCOPE hit. A MISSING item that has a NOTES entry
   passes, because that's an honest shortfall.
4. **description-matches-diff** fails on any "NOT IN DIFF" claim.
5. **decisive-evidence** fails in any of these cases: a FAILURE-MODE
   observable is "NOT SHOWN", the only runs shown use control or
   unchanged-path inputs, or the evidence is a bare assertion with no
   shown output.
6. **clean-diff** fails if the list from gathering has anything on it for
   the added lines. Comments that explain the fix don't count.
7. **template-asks** fails on any "NOT MET" ask. **ai-disclosure** fails
   when the rule covers PRs or all AI use and there's "no disclosure".
8. Grade the substance, not the writing. A terse PR that meets every
   condition passes. A confident, polished one that fails a condition
   fails. Never take the description's word for what the diff contains.
9. Grade `unclear` only when the part of the package a check needs
   doesn't exist (for example, there's no Test evidence section at all).
   If the part exists but is weak, grade it pass or fail. Before you
   grade anything `unclear`, look in every place the evidence guide lists
   for that family.
10. Grade every check, even after a required one has failed. A later
    check can be graded from the facts you've already gathered, without
    re-reading the package.
11. If the condition and your instinct disagree, the condition wins. Note
    the tension in the summary.

## Verdict assembly

1. Count `unclear` as `fail`.
2. If every `required` check is `pass`, the verdict is `accept`.
   Otherwise it's `reject`. `preferred` checks never change the verdict.
3. Name the deciding check in the summary. For a reject, that's the first
   failing `required` check in rubric table order, with its quoted
   evidence. For an accept, it's the required check that came closest to
   failing, along with why it still passed.
4. In live mode, after the verdict, list any voice-guide rule the title
   or description breaks. This never changes the verdict.
5. Emit the JSON block from SKILL.md last, with one `checks` entry per
   rubric check in table order, and nothing after it.