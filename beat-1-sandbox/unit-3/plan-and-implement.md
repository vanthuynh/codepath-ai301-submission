# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**vanthuynh**

[Your GitHub username, exactly as it appears on your profile - no @, no
profile URL. Your comment upstream is identified by this name, and it is
the only thing that ties it to you. Several students may plan the same
house issue, so this is what keeps their comments off your score and
yours off theirs.]

**[Plan comment](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/37#issuecomment-6032661258)**

> Plan: Document request bodies for POST /profiles and POST /reviews (issue #37)
> Issue: #37
> Issue text: "docs/API.md lists endpoints with a short description but doesn't give any request bodies. Add them for POST /profiles and POST /reviews, with a description and an example value for each field. POST /profiles takes multipart form data, not JSON."

> Diagnosis
> This is a docs gap, not a code bug. docs/API.md describes both endpoints in one line each and gives no request body:

> POST /profiles — Create a profile with resume and GitHub username.
> POST /reviews — Request a new portfolio review for a profile.
> The request shapes live only in code:

> POST /profiles (api/routes/profiles.py, create_profile_endpoint) reads three fields from multipart form data, not JSON:
> github_username: str = Form(default=None), max 255 chars (ProfileCreate in api/schemas/profile.py)
> portfolio_url: str = Form(default=None), max 500 chars
> resume_file: UploadFile = File(default=None), accepted types application/pdf, text/markdown, text/plain; anything else returns 422 "Resume must be a PDF or Markdown file"
> All three are optional. The endpoint requires auth (get_current_user).
> POST /reviews (api/routes/reviews.py) takes a JSON body validated by ReviewCreate in api/schemas/review.py: one required field, profile_id: UUID. It returns a review with status="pending" immediately and processes it in the background.
> A mismatch worth noting: the existing doc line says "with resume and GitHub username", which implies they are required, but the code makes every field optional.

> Repro evidence (from my unit 2 comment on #37)
What the code requires

> POST /profiles (api/routes/profiles.py:23-27) — Form(...) and File(...) parameters, so the route takes multipart/form-data, not JSON: github_username (str), portfolio_url (str), resume_file (UploadFile). All three are default=None, i.e. optional.
> POST /reviews (api/routes/reviews.py:22-24) — binds data: ReviewCreate, so this one is a JSON body. ReviewCreate (api/schemas/review.py:14-15) has a single required field, profile_id, of type UUID.
> Validation
> The issue is accurate as filed. docs/API.md documents neither request body, and the two endpoints take different content types — POST /profiles is multipart while POST /reviews is JSON — which the current doc gives a reader no way to discover.

> Scope
> In scope

> Add a "Request body" subsection under POST /profiles and POST /reviews in docs/API.md.
> For each field: name, type, required/optional, constraints, description, example value.
> State the content type for each: multipart/form-data for profiles, application/json for reviews.
> One example request per endpoint (curl).
> Out of scope

> Response schemas and error tables for these endpoints.
> Request bodies for POST /auth/*, PUT /profiles/{id}, or any other endpoint. (Note PUT /profiles/{profile_id} exists in code but is not listed in API.md; I will not add it here.)
> Any change to application code, schemas or behavior, including making the optional-field behavior stricter.
> Files I'll touch
> docs/API.md — the only file changed.
> Read-only references: api/routes/profiles.py, api/schemas/profile.py, api/schemas/review.py, api/routes/reviews.py.

> Approach
> Under POST /profiles, add a field table (github_username, portfolio_url, resume_file) with type, required, constraints, description and example, plus a note that the body is multipart form data, not JSON.
> Add a curl example using -F flags, e.g.
> curl -X POST http://localhost:8000/profiles -H "Authorization: Bearer <token>" -F "github_username=octocat" -F "portfolio_url=https://octocat.dev" -F "resume_file=@resume.pdf;type=application/pdf"
> Under POST /reviews, add a table for profile_id (UUID, required) and a JSON example {"profile_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"}.
> Note the accepted resume types and that an invalid type returns 422.
> Cross-check every field name, type and limit against the code so the doc matches it. Keep the existing formatting style of API.md.
> Test plan
> Docs-only change, so verification is re-running my unit 2 repro steps and checking the doc against the live API.

> Re-run repro: open docs/API.md (and the repro steps from my unit 2 comment: TODO fill in exact steps). Before the fix: no request body for either endpoint. After the fix: both have a body section with every field, a description and an example.
> Check against the running API: start the app and open http://localhost:8000/docs. Expect the field names, types and required/optional status in Swagger to match the tables I wrote, and POST /profiles shown as multipart/form-data.
> Run the examples: run the POST /profiles curl with a valid PDF or .md resume. Expect a 200 with a ProfileResponse. Run it with a .png. Expect a 422 "Resume must be a PDF or Markdown file".
> Run the reviews example using the returned profile id. Expect a response with status: "pending".
C> onfirm git diff touches only docs/API.md.
> Risks and unknowns
> Doc/code drift: the code may not match the intended behavior. For example, the PDF branch calls PyPDF2.PdfReader(content) with raw bytes, which may fail at runtime for real PDFs. If step 3 returns 422 "Failed to parse PDF resume" for a valid PDF, I will document the working behavior (markdown) and report the bug in a separate issue rather than fix it here.
> Optional vs. required: I'm documenting what the code does (all optional). A maintainer may prefer to document the intent (e.g. at least one field). I'll ask in the PR if unsure.
> Auth: I'm assuming a Bearer JWT from POST /auth/login. I need to confirm the header format.
> Environment: I may not be able to run the full stack (DB, dependencies). If not, I'll verify against the code and Swagger only and say so.
> Unconfirmed repro details: the quoted evidence above must be filled in from my comment


---

## Your branch

**fix/37-api-request-body-schema**

[The name of the branch you built the change on, exactly as it appears in your fork. The
naming shape is a type prefix, then the issue number, then a short description. **The issue
number in the branch name must be the number of the issue you claimed** — a name carrying
any other number does not satisfy this field.]

**Evidence**

[Your Unit 2 reproduction steps re-run against the built change: the before, then the
after. Paste both, including the commands you ran and their output.]

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

[The agreement score of each run you did, in order. A single run is a complete answer if
only one run occurred. **The last score in your list must match the agreement line in the
`eval-run.txt` you committed** — that file is the record of your final run.]

**Package analysis**

[Pick one scored package (`pkg-01` through `pkg-20` — the four `calib-` packages are never
scored). Name it by id, say what your rubric decided and what the gold label said, and
explain why your rubric read it that way.]

**Check rationale**

[Quote one check from the `rubric.md` you uploaded to `tools/plan-check/`, exactly as it reads now.
Then say why it reads that way — what you revised to get there, or what you rejected in
favour of it.]

**Trade-offs**

[Every check gives something up. Any one of these is a complete answer: a package whose
result it changes, a canary you re-ran with `--only`, a case you accept it will miss, or a
stated reason nothing changed elsewhere. "Nothing changed, and here is how I know" earns
the point in full when the reason follows.]

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
