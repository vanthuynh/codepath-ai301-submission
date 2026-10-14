# Evidence guide: where to find evidence in a PR package

For each family of evidence, this guide tells you three things: where it
lives in an eval bundle (the sections of the `pkg-*.md` / `calib-*.md`
files, or the same fields in the `.json` twin), where it lives in live
mode (your working copy and GitHub), and what good looks like once you
find it. The four families map one to one onto the eval categories: plan
fidelity is silent-drift, test evidence is not-tested, diff quality is
unreviewable, and standards and comms is standards-wall.

## Plan fidelity (harness category: silent-drift)

- **Where to find it (eval):** The plan's scope is in the "Plan context"
  block. Look at the "Plan:" paragraph for its "Scope" or "one bounded
  change" sentence, the listed changes, the "Not in scope:" line, and the
  "Files:" line. Also look for any "Known limit", deferral, or deviation
  sentence. What actually changed is in the "### Diff" section under
  "Candidate PR": the `--- a/` and `+++ b/` file headers and each `@@`
  hunk. Claims of fidelity are in "### Title" and "### Description".
- **Where to find it (live):** Use your `plan.md` (its scope, files,
  changes, and `## Deviations` section). For the diff, run
  `git diff main...HEAD` in the top folder of your fork's clone. Your
  draft title and description are in `pr_draft.md`, with the title on the
  first line.
- **What good looks like:** Every file header in the diff appears in the
  plan's Files line (or is the regression test or changelog for the
  planned change). Every hunk does a change the plan lists, and every
  change the plan lists has a hunk. Anything else is covered by a note in
  the plan that the description restates.

  Silent drift in the "more" direction looks like a renaming pass, a new
  config option or CLI flag, a rewrite of a neighboring function,
  reworded unrelated help text, or a hunk in a file the plan never names,
  all under a description that says "exactly as planned". Silent drift in
  the "less" direction is when the plan promised a docs update or a second
  fix and the diff has none, with no note, or the description claims it's
  there. An honest shortfall (the plan's deferral note, restated in the
  description) counts as fidelity, not drift.

## Test evidence (harness category: not-tested)

- **Where to find it (eval):** Look at the "### Test evidence" section of
  the Candidate PR, plus any before/after blocks inside the Description.
  Read these against the plan's "Test plan:" sentence and the "Repro
  evidence" paragraph (its failing command, Actual, Expected, and
  controls). The repo's own test command is in the Repo facts or the
  contribution line.
- **Where to find it (live):** Use `test_evidence.md` next to `plan.md`.
  It should hold your unit-2 repro steps re-run on `main` for the before
  and on your branch for the after, plus the output of `make test-unit`,
  `make test-integration`, `make lint`, and `make typecheck`. The test
  plan is in `plan.md`.
- **What good looks like:** The repro's own failing command, with the
  same inputs, is re-run on the branch with the output shown, next to the
  before result, and the after matches the plan's expected-after (a full
  URL and exit code 0, both table rows present, a byte-identical
  round-trip). Every failure mode the test plan names is shown, not just
  the first. The repo's checks are named with their outcomes, and a
  failing check that's honestly reported with its reason still counts as
  evidence.

  Things that are not decisive: "tested locally, works now", "verified on
  my machine for a day", or "cargo test passes" on its own. Neither is a
  run of the single-file control or the GET path when the bug was the
  two-file case or the POST path, or a new test that asserts a different
  behavior than the one the issue reports.

## Diff quality (harness category: unreviewable)

- **Where to find it (eval):** Check the "### Diff" section (every `+`
  line and every hunk) and the "### Commits" list.
- **Where to find it (live):** Use `git diff main...HEAD` and
  `git log --oneline main..HEAD`. The output of
  `git diff main...HEAD --stat` should list only the files your change
  needs.
- **What good looks like:** Every added line is the fix, its test, or a
  comment explaining the fix, so a reviewer can see the change in one
  read. Here are the tell-tale signs of debris:
  - `println!`, `eprintln!`, `print(`, `console.log`, or `dbg!` calls, or
    "DEBUG" output, added to the code.
  - `+ // old_code(...)` commented-out lines, or a commented-out first
    attempt.
  - A function nothing calls, `#[allow(dead_code)]`, or names like
    `_unused_*`.
  - A TODO added on an untouched line.
  - Hunks that only re-indent, re-wrap, re-order imports, or remove and
    re-add identical lines.
  - Commits named "wip", "fix", "fmt + cleanup", or "misc cleanups while
    debugging".

## Standards and comms (harness category: standards-wall)

- **Where to find it (eval):** Look in the "## Repo facts" block. The
  "pull requests:" line holds the template sections, required checklist
  items, the issue-link form (like "closes #xxxx" or "fixes #<n>"),
  required changelog or whatsnew entries, and tests-with-changes. The
  "contribution policy" line holds the AI rules. Compliance shows up in
  the Title, the Description (section headings or equivalent content,
  checklist answers, issue link, AI disclosure), and the Diff (the
  changelog or whatsnew file).
- **Where to find it (live):** Use the Path Review repo's
  `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md` (the
  AI-use policy and checks list) in your working copy. Compliance shows
  up in `pr_draft.md` (every template section filled, "Closes #<n>", and
  the Notes for Reviewers AI disclosure) and in your diff.
- **What good looks like:** Every section the template names has real
  content in the description, either under its heading or in unmistakable
  equivalent words ("Type of changes: Bug fix"; "Requirements:
  cheatsheets regenerated (no diff); integration test added"). Every
  required entry (CHANGELOG.md, whatsnew) is in the diff or marked n/a
  with a reason, and the required issue link appears.

  When the policy requires disclosing all AI use (or AI use in PRs and
  comments), the description should say which tool helped, how much, and
  that the author reviewed and understands the work, in the author's own
  words. A repo with no template and no AI rule asks nothing beyond what
  its contribution line states.

  A visibly ignored ask looks like a required checklist that's absent, no
  "closes #", no whatsnew entry when the template requires one for bug
  fixes, or no disclosure under a strict AI policy. (Whether the
  description's claims match the diff is a plan-fidelity question, covered
  above.)