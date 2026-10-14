# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

[codepath/pathreview-ai301-fa26-s3](https://github.com/codepath/pathreview-ai301-fa26-s3/pull/231)

**Branch**

`fix/37-api-request-body-schema`

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

- I first run with `--limit 3`. The output correctly pass 3/3 scored items

- I then ran the full `run_eval.py --skill /Users/van.huynh/.claude/skills/pr-precheck --save-run eval-run.txt `

> grading 20 package(s) with the tool at /Users/van.huynh/.claude/skills/pr-precheck, model sonnet, 5 worker(s)...
>  ...

> categories: clear-accept 7/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3 agreement: 20/20 scored items  (bar: 18/20: PASS)

**Package analysis**

[Pick one scored package (`pkg-01` through `pkg-20` — the four `calib-` packages are never
scored). Name it by id, say what your rubric decided and what the gold label said, and
explain why your rubric read it that way.]

**Check rationale**

> standards_and_disclosure: 
> - The candidate PR's title and description, read against the repo-facts contribution policy, the PR template, the issue thread, and any AI-use disclosure requirements. | Passes when everything the PR template explicitly requires is there and the repo's contribution conditions are met, including AI-use disclosure where it's required. Fails when a mandatory section, issue reference, required disclosure, or stated policy condition is missing. A brief answer, an explained N/A, or an honestly disclosed shortfall is acceptable where the repo allows it.

**Trade-offs**

[Every check gives something up. Any one of these is a complete answer: a package whose
result it changes, a canary you re-ran with `--only`, a case you accept it will miss, or a
stated reason nothing changed elsewhere. "Nothing changed, and here is how I know" earns
the point in full when the reason follows.]

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
