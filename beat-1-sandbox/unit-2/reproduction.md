# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy
kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**vanthuynh**

---

## Posted upstream

**[codepath/pathreview-ai301-fa26-s3/#37](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/37#issuecomment-5988728689)**

Hi, I'd like to tackle this issue as my first contribution.

I read docs/API.md at 2f4e82f5 and both endpoints get a single description
line with no request body section:

`POST /profiles` — Create a profile with resume and GitHub username.
`POST /reviews` — Request a new portfolio review for a profile.
The issue also notes that POST /profiles takes multipart form data rather
than JSON, which the doc doesn't call out anywhere — that's the part I most
want to confirm against the handler before writing anything down.

Next: I'll cross-reference api/routes/profiles.py and api/schemas/review.py
to pull the exact form fields and schema, and I'll report back with what I find
before I draft an update.

**[codepath/pathreview-ai301-fa26-s3/#37](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/37#issuecomment-5988730731)**

### Reproduction Report

**Environment**
- **Project:** `vanthuynh/pathreview-ai301-fa26-s3`
- **Target Commit:** `2f4e82f52efbcfcc57d65b3fa5348672163ca088` (`2f4e82f5`, `main`)
- **System:** macOS 27 Golden Gate
- **Python:** 3.14.5
- **Method:** read-only source inspection — no running server needed to confirm a
  documentation gap, just the checked-out files.

**Steps**

1. Clone and sync to the target commit:

```bash
git clone git@github.com:vanthuynh/pathreview-ai301-fa26-s3.git
cd pathreview-ai301-fa26-s3
git checkout 2f4e82f52efbcfcc57d65b3fa5348672163ca088
```

2. Read the two endpoints in the API reference:

```console
$ sed -n '16,28p' docs/API.md
### Profiles

`POST /profiles` — Create a profile with resume and GitHub username.
`GET /profiles/{profile_id}` — Retrieve a profile.
`DELETE /profiles/{profile_id}` — Delete a profile and associated data.

### Reviews

`POST /reviews` — Request a new portfolio review for a profile.
`GET /reviews/{review_id}` — Retrieve a completed review.
`GET /reviews` — List reviews for the authenticated user (paginated).

## Interactive Docs
```

Both endpoints get a single description line, then the doc moves on to
`## Interactive Docs`. No content type, no field list, no example body.

3. Trace the route handlers and the request schema:

```console
$ rg -n -A6 '@router.post' api/routes/profiles.py
23:@router.post("", response_model=ProfileResponse)
24-async def create_profile_endpoint(
25-    github_username: str = Form(default=None),
26-    portfolio_url: str = Form(default=None),
27-    resume_file: UploadFile = File(default=None),
28-    current_user: User = Depends(get_current_user),
29-    db=Depends(get_db),

$ rg -n -A4 '@router.post' api/routes/reviews.py
22:@router.post("", response_model=ReviewResponse)
23-async def create_review_endpoint(
24-    data: ReviewCreate,
25-    background_tasks: BackgroundTasks,
26-    current_user: User = Depends(get_current_user),

$ rg -n -A2 'class ReviewCreate' api/schemas/review.py
14:class ReviewCreate(BaseModel):
15-    profile_id: UUID
16-
```

**What the code requires**

- **`POST /profiles`** (`api/routes/profiles.py:23-27`) — `Form(...)` and `File(...)`
  parameters, so the route takes `multipart/form-data`, not JSON:
  `github_username` (str), `portfolio_url` (str), `resume_file` (UploadFile).
  All three are `default=None`, i.e. optional.
- **`POST /reviews`** (`api/routes/reviews.py:22-24`) — binds `data: ReviewCreate`,
  so this one *is* a JSON body. `ReviewCreate` (`api/schemas/review.py:14-15`)
  has a single required field, `profile_id`, of type `UUID`.

**Validation**

The issue is accurate as filed. `docs/API.md` documents neither request body,
and the two endpoints take different content types — `POST /profiles` is
multipart while `POST /reviews` is JSON — which the current doc gives a reader
no way to discover.


## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

- Run 1: categories: clear-accept 5/8  disclosure 1/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4
agreement: 17/20 scored items  (bar: 18/20: below the bar). `pkg-01` failed due to "repo_conventions" check, `pkg-07` failed due to "repo_conventions" check, `pkg-09` failed due to "behavior_matches_issue"

- Run 2: clear-accept 6/8  disclosure 1/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4
agreement: 18/20 scored items  (bar: 18/20: PASS). `pkg-05` failed due to "step_followable" check, `pkg-09` failed due to "behavior_matches_issue"

**Package analysis**

`pkg-01` (httpie/cli#1640)

Gold label:	`accept`. The note says: "faithful offline repro of the missing Content-Type with a control run; env recorded; claim specific and modest."

My rubric: reject `failed: repo_conventions`

Explain: 

The package is a good repro. It has the environment recorded, an offline run, and a control run with no custom header. The grader passed every other check, and repo_conventions was the only one that failed.
repo_conventions asks whether the comments "follow the repo's specific rules". The claim comment "needs to link to the issue and offer to look into it."
pkg-01's claim comment never links to the issue. It says "Hi, I'm working through this as a first contribution," but it has no #1640 or URL, so the grader read it as failing the rule.
httpie states no AI policy, so disclosure wasn't the cause. The missing link was.
This failed the whole package because the copy of the rubric that was graded marks repo_conventions as required. Your project copy marks it preferred. Under the verdict rule, one failed required check rejects the package.

**Check rationale**

> Passes if the comments follow the repo's specific rules. A "claim" comment needs to link to the issue and offer to look into it, without prematurely promising a fix or confirming the bug. The repro comment needs to accurately reflect the work done. If the repo demands AI-generated content disclosure, it must be there (if the repo is silent on this, silence is fine)

This check ensures the claim comment to be transparent with focus on showing they have looking into the issue and offer to help; also make sure the contributor have succesfully reproduced the issue under the same environment and exact steps that they did (with/without AI assistant).


**Trade-offs**

This validation is intentionally strict to verify the exact reported behavior. While this successfully filters out irrelevant packages, it might occasionally reject evidence of closely related failures. This is a deliberate trade-off: a valid reproduction must prove the specific bug being claimed, not just highlight that something nearby is broken.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
