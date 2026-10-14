---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: grading a PR against a rubric

## What you're here to do

Each run, you answer one question about one PR package: **is this pull
request ready to submit?**

A PR package is a candidate pull request (title, description, commit
list, unified diff, and test evidence) read next to two other things.
The first is the accepted plan it says it implements, including the
plan's scope, test plan, deviation notes, and the repro evidence it was
built on. The second is the issue that plan belongs to: the issue body,
its thread, and the repo's PR template and contribution policy.

Please stay inside that question. Don't critique style for its own sake,
don't second-guess whether the plan was a good one, don't propose a
better fix, and don't grade more than one package in a single run. The
plan has already been accepted. All you're checking is whether this PR
delivers it faithfully, proves that it does, is tidy enough for a
reviewer, and meets what the repo asks for. And don't go on instinct:
work through the components in this directory.

## Pick your mode

Look at what you were handed and decide the mode before anything else.

**Eval mode.** You're in eval mode if you were given a package bundle: a
markdown file headed `# Eval package: <id>`, or its `.json` twin. The
bundle is your entire world. It holds a "Repo facts" block, the "Issue"
section, "Thread highlights", the "Plan context" (the accepted plan plus
its repro evidence), and the "Candidate PR" (Title, Description,
Commits, Diff, Test evidence). Treat only the bundle text as evidence.
Don't fetch anything, don't open the `source:` issue on GitHub, and
don't read any file other than this tool's own components. Eval mode
always grades the full package: every check in `rubric.md`, with the
complete verdict rule. Use the bundle id (for example `pkg-07`) as the
JSON `item`.

**Live mode.** You're in live mode when a student asks you to grade
their own draft PR before they open it. Run from the top folder of their
fork's working copy, on their fix branch, and collect these inputs and
nothing else:

- **The plan:** `plan.md` in the working-copy root, including its
  `## Deviations` section if there is one. A house-chain student uses
  the house plan and the house repro pack their TF handed out. The same
  checks apply to the same things there.
- **The diff:** run `git diff main...HEAD` (three dots) from the working
  copy. That shows everything the branch changes relative to the default
  branch, and it's the only diff you grade. Also run
  `git log --oneline main..HEAD` for the commit list. Uncommitted
  changes aren't part of the PR, so ignore them, but if `git status`
  shows any, say so.
- **The draft PR title and description:** `pr_draft.md` in the
  working-copy root. The first line is the title, and everything below it
  is the description.
- **The test evidence:** `test_evidence.md` in the working-copy root
  (repro before and after, plus the repo's checks output).
- **The issue:** the issue URL the student gives you. Read the issue body
  and its thread (`gh issue view <n> --comments`, or the web) to pick up
  any direction from maintainers.
- **The repo's stated asks:** the PR template at
  `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md` in the
  working copy (or on GitHub, if they aren't there locally).

If something is missing in live mode, don't make it up. Grade any check
that depends on it as `unclear`, and name the missing file in that
check's evidence line. For the JSON `item`, use the issue URL, or the PR
URL if a PR already exists.

## The scope seam (live mode only)

In live mode, read `scope.md` in this directory before anything else. If
its `Repo:` line still holds the bracketed placeholder
(`<ORG>/<PATH-REVIEW-REPO>`, or anything in angle brackets), stop right
there. Don't grade. Tell the student to fill in the `Repo:` line of
`scope.md` with their section's Path Review repo. Never guess a scope.

If the line is filled in, check two things: that the working copy's
upstream (`git remote -v`) belongs to the scoped repo, and that the issue
URL does too. If either one doesn't, refuse to grade and say which one is
out of scope. Then treat the scope's house rules as reading rules. The PR
comes from a `fix/<issue-number>-<slug>` branch on the student's fork,
there's one PR per issue, the repo's template is always used, and a
classmate's PR on the same issue doesn't block this one.

In eval mode, skip `scope.md` completely.

## The voice seam (live mode only)

Once grading is done in live mode, read `voice-guide.md` in this
directory. Hold the outgoing text (the PR title and the description from
`pr_draft.md`) up against each of its rules and against its "Things I
never post" list. In the summary, above the JSON block, report every rule
the draft breaks, quoting the rule's name and the phrase that breaks it.
A broken voice rule never changes the verdict by itself. The rubric has
no check that reads the voice guide, because voice is the student's own
standard and not a readiness gate.

In eval mode, skip `voice-guide.md` completely.

## Reading the components

Read these in order, and use each one for its one job:

1. **`rubric.md`** defines the checks (name, evidence, pass condition,
   weight) and the verdict rule. It decides what gets graded and how the
   grades combine. In the JSON, use the check names exactly as written
   there.
2. **`references/evidence-guide.md`** is your map. It tells you where each
   family of evidence lives, both in a bundle and in live mode, and what
   good looks like once you find it.
3. **`procedure.md`** is how you actually run the grade: read order,
   evidence gathering, check execution, verdict assembly. Follow it as
   written, one step at a time.

If the procedure doesn't cover something you need, don't invent a step to
get around it. Grade with whatever the written steps gave you, and note
the gap in your summary as "procedure gap: <what was missing>".

If `rubric.md` has no filled-in check rows (just an empty table or
comments) or no verdict rule, or if `procedure.md` has no written steps
under its headings, refuse to grade. Say which file is still a template
and give no verdict. Instruction comments (`<!-- -->`) don't count as
content.

## The verdict and what you hand back

The verdict is binary. `accept` means the PR is ready to submit and
`reject` means hold it. There's no third option, no score, and no "accept
with changes". If you have reservations, put them in a check's evidence
line.

Write a short readable summary first. Give one line per check (its name,
its grade, and the fact that decided it), then the check that decided the
verdict, then any procedure gaps, and in live mode, any voice-guide rules
that were broken. After that, end your reply with the fenced JSON block
below, filled in, with one `checks` entry per rubric check in rubric
order. The JSON must be valid and must be the very last thing in your
reply. Nothing comes after it.

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## How to grade well

- **Evidence comes first.** Don't grade a check until you can name the
  fact or quote from the package that decided it. "Looks fine" and "seems
  good" aren't evidence.
- **Grade what the PR does, not how polished it looks.** Read the diff
  against the plan, the evidence against the test plan, the description
  against the diff, and the PR against the repo's stated asks. A terse PR
  that meets every pass condition is ready. A long, confident, beautifully
  formatted PR whose diff does something the plan never mentioned is not.
  What the description says is a claim, not proof. The diff and the test
  output are the proof.
- **The rubric decides, not your mood.** If a check meets its stated pass
  condition but still feels off, it passes. Mention the tension in the
  summary. And don't add checks the rubric doesn't have.
- **The procedure decides the how.** Follow `procedure.md` as it's
  written, and report its gaps instead of filling them with your own
  steps.
- **Unclear counts as a fail.** Reserve `unclear` for when the part of the
  package a check needs is truly absent. The rubric's verdict rule treats
  `unclear` as a failing grade, and wherever a rule is silent, a claim you
  can't verify counts as a failing one. If you can't verify a PR from the
  package, it isn't ready to submit.
- **Honest shortfalls aren't failures.** A PR that delivers less than the
  full plan but records the shortfall in the plan's deviation notes and
  repeats it in the description can still be ready. Only differences that
  nobody announced count against it.