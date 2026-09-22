# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**[codepath/pathreview-ai301-fa26-s3#37](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/37)**

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

[One scored issue, identified by id (`issue-01` through `issue-20`; the `calib-`
issues are not scored). State your rubric's decision, the gold label, and the
reasoning that produced your rubric's result.]

**Check rationale**

[One check from the `rubric.md` uploaded to `tools/issue-select/`, quoted as it is
currently written, with the reasoning behind its current form.]

**Trade-offs**

[What the quoted check gives up. Any one of these is a complete answer: an issue whose
result it changes, a canary you re-ran with `--only`, a case you accept it will miss, or a
stated reason nothing changed elsewhere. "Nothing changed, and here is how I know" earns
the point in full when the reason follows.]

---

## Selection rationale

Graded on whether all three are answered, in your own words. Not on how good the
reasoning is, and not on length — a short honest answer to each earns the full marks.
This is also the basis for the claim comment you write in Unit 2.

**Selection rationale**

[Answer all three:

1. The issue's fit to your interests and to the time available.
2. What the verdict identified correctly, and what you weighed that the rubric could
   not.
3. The anticipated difficulty in claiming it.]

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
