# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

Issue Link: **[codepath/pathreview-ai301-fa26-s3#37](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/37)**

[The individual Path Review issue page. A link to the repository or the issue list
does not satisfy this field.]

**Verdict output**

I ran in live mode against 4 good-first-issue candidates from Path
Review repo (`codepath/pathreview-ai301-fa26-s3`) and
all 4 were `accept`

- #37 API reference doc missing `POST /profile` request body schema
- #47 API docs don't include example `curl` command
- #26 Add a safety event count to the health check endpoint
- #56 Structural chunker silently drops documents that contain no headings

I chose issue #37 since I want to apply my backend knowledge in designing & reading database schema:

```json
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/37",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Last default-branch commit 2026-09-16 by Aburke225, 6 days before today (2026-09-22)."},
      {"name": "repo-in-use", "grade": "pass", "evidence": "archived: false and pushed_at 2026-09-16 (within 90 days); no releases, 3 stars."},
      {"name": "Scope-bounded", "grade": "pass", "evidence": "Documents request bodies for two named endpoints in a 32-line docs/API.md; a two-part deliverable, not an umbrella/tracking issue."},
      {"name": "issue-unclaimed", "grade": "pass", "evidence": "assignees: none; repo has 0 pull requests total; 0 comments on the thread."},
      {"name": "Contribution policy allows AI-assisted work", "grade": "pass", "evidence": "docs/CONTRIBUTING.md is the only policy doc and states no AI restriction; no AI_POLICY.md or PR template."},
      {"name": "Good-first-issue label", "grade": "pass", "evidence": "Labels: enhancement, good first issue, api, docs, tier-1."},
      {"name": "Clear acceptance criteria", "grade": "pass", "evidence": "Body specifies a description and example value per field for POST /profiles and POST /reviews, and flags that POST /profiles is multipart form data."}
    ],
    "verdict": "accept"
  },
```

---

## Eval iterations

Quote source text directly in each field below. Paraphrase does not satisfy them.

**Run history**

1. I first performed smoke-test run with `--limit 3` flag but encountered encoding bugs (`cp1252`) on my Windows laptop. This is due to several eval bundles contain emoji/arrow characters, and encoding them for the child process's stdin raised `UnicodeEncodeError`, so the harness refused to write `eval-run.txt` (partial/errored run).
2. Patched `run_eval.py` (via Claude's suggestion) by enforcing UTF-8 encoding in the `grade_one` subprocess (`encoding="utf-8", errors="replace"`). This overrides the problematic OS locale fallback of `text=True`. The fix is isolated to the CLI pipe and preserves the fingerprint integrity of both `rubric.md` and `SKILL.md`.
3. I then reran the full 20-issue test suite with no issue, harness wrote `eval-run.txt` and `results.json`. Scored 15/20 against gold (missed the 18/20 target) but successfully met the category floor requirement (`claimed 4/4  clear-accept 5/8  dead-repo 3/3  policy 1/1  scope 2/4`). The committed `eval-run.txt` reflects this specific execution.

**Issue analysis**

Scored issue analyzed: `issue-1`

My rubric verdict: `reject`

Gold-label: `accept` (Agree = NO)

My rubric reject `issue-1` even though issue 1 was opened within 90 days, have active status, unassigned, etc. My rubric couldn't decide on scope-bounded check. However, my rubric did correctly point out that issue-1 didn't pass the 2 preffered condition `good-first-issue` label as well as `clear acceptance criteria`, which doesn't affect the final condition but can be unclear. I didn't include `unclear` decision in Verdict section so that may be why my rubric rejected the issue.

**Check rationale**

Check: `Scope-bounded`

My scope condition:

> Fails if the issue is an umbrella/tracking issue listing multiple sub-items, if the thread shows an unresolved design debate with no maintainer decision, if a maintainer states the fix touches core internals, or if the issue is a pure usage/support question rather than a concrete bug or feature ask. A terse body or missing repro steps does not by itself fail this check.

I built this check to keep bounded first contributions separate from messy umbrella issues, design questions, support requests, and heavy core changes. After running some evaluations, I also updated it so that a maintainer listing a few causes for one bug doesn't accidentally get treated as multiple separate tasks.

**Trade-offs**

The rubric prioritizes issues demonstrating clear repository activity, accessible ownership, bounded scope, and contribution-policy compatibility, minimizing the selection of abandoned, claimed, or overly broad tasks. The trade-off is that nuanced issues may be misclassified if their true difficulty is buried in lengthy discussion histories or multiple abandoned attempts rather than a single explicit signal.

---

## Selection rationale

Graded on whether all three are answered, in your own words. Not on how good the
reasoning is, and not on length — a short honest answer to each earns the full marks.
This is also the basis for the claim comment you write in Unit 2.

**Selection rationale**

1. I'm currently more interested in backend engineering and this issue would allow me to apply my knowledge in database design, understanding new codebase, and fix the bug.
2. The verdict identified correctly all checks and graded pass for all, which satisfy all of my rubric for a good issue that I can work on.
3. This issue is tagged `tier 1` so it won't be very challenging and just enough for me to practice before stepping in Unit 2 where I need to understand the code, the issue, and reintroduce the bug

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
