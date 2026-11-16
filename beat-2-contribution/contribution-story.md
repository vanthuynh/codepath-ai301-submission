# Contribution: [Issue Title]

Path: `beat-2-contribution/contribution-story.md`

This file is Assignment 5, and it is graded at the path above; a copy kept anywhere else in
the repository is not read. **Keep every section header below under its exact name** — the
format criteria grade exactly that. Your course repo link is the whole submission: no tool
files and no evidence uploads are part of Assignment 5.

**Contribution Number:** [1]
**Student:** Van Huynh
**GitHub Username:** vanthuynh
**Issue:** [codepath/pathreview-ai301-fa26-s3#28](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/28)
**Pull Request:** [GitHub PR link, once opened]

One repository, built across units 5 to 8, telling the whole story of your real
contribution. Fill it as you work, not at the end — the sections below ask for evidence that
is far easier to write down as it happens than to reconstruct in the last weekend.

Contribution Number stays `[1]` unless you make more than one contribution this term; it
just numbers them.

---

## Toolkit

Every story section below, from Why I Chose This Issue through Pull
Request, ends with a Toolkit block: which tool did that phase's work
and how you used it, what it produced, what difficulties or
shortcomings it presented, and what you changed or would change
because of them. Together the blocks are the thread the rubric's
Toolkit points read, and they are the raw material for your unit-9
portfolio, so keep them current as you work rather than
reconstructing them later.

A Toolkit block looks like this (copy it into each story section and
fill it there):

```text
**Toolkit**

- Tool: [which tool did this phase's work, and how you used it]
- What it produced: [described in a sentence or a short excerpt]
- Difficulties: [what fought you or fell short, or "none met"]
- What changed: [the patch, revision, or workaround, what you would
  update, or "nothing"]
```

This section also keeps one short dated entry per week of tool work,
starting below: what you built or changed that week, what broke, and
what you changed because of it. The blocks tell the story where it
happened; these entries are the toolkit's own timeline.

All four entries are graded. Keep every heading below, and fill each
one in the week it belongs to.

**Unit 5: [one line on the tool work]**

My first install was broken: the skill files sat one folder too deep, in `~/.claude/skills/issue-select/skill/`, so Claude Code never loaded the skill. I rebuilt it as a flat `~/.claude/skills/issue-scout/`. Even though I did install the `issue-scout.md` and implemented `search-strategy.md`, I didn't choose any issue from the search result. I selected Issue #28 during the PathReview classroom contribution process prior to Unit 5. Because I did not save a verified issue-scout scan or a Unit 5 search-strategy file, I cannot confirm whether the scout evaluated or recommended this specific issue. This experience highlighted the importance of documenting my workflow. In the future, I will retain my search queries, ranking criteria, and issue verdicts to clearly demonstrate how tool-based insights influence my decisions.

**Unit 6: [one line on the tool work]**

To ensure my issue comments were specific, evidence-based, and repository-appropriate, I utilized structured draft reviews early in the process. I also followed strict voice guidelines to stick to verified facts and avoid making promises I couldn't deliver. I installed both `skill/repro-check-scope-wild.md` to `~/.claude/skills/repro-check/scope.md` and `plan/repro-check-scope-wild.md` to `~/.claude/skills/plan-check/scope.md`. 

**Unit 7: [one line on the tool work]**

Before opening the PR, I used my `pr-precheck` skill to compare the implementation, plan, test evidence, and PR description. After running the checks and updating the evidence, the tool accepted the PR

**Unit 8: [one line on the tool work]**

TODO before submitting: install the PR monitor (Week 8 steps 1-3), write the playbook, run monitor.py on PR #656, and replace this paragraph with what it read, what it drafted, and what you did with the draft. As of 2026-11-09 the PR has only the bot's welcome comment and no review.

---

## Why I Chose This Issue

**1. The hard part is already built and tested, so the work is wiring, not invention.**
`safety/rate_limiter.py` already implements a rolling-window limiter on Redis
sorted sets, and `tests/unit/test_rate_limiter.py` covers it with 19 tests —
including `test_ip_address_as_identifier`, which passes `"192.168.1.1"` as the
identifier. Somebody designed this class to be called with an IP. Nothing calls
it. That makes the issue a well-bounded integration task with a clear finish
line, rather than an open-ended design problem I could sink 20 hours into.

**2. It is a real security gap.**
`POST /auth/login` and `POST /auth/register` are unauthenticated, and login
runs a bcrypt verification on every call. With no limiter in front of them, a
single host can brute-force credentials or burn CPU for free. `POST /reviews`
is worse in a different way: it kicks off an LLM-backed review in the
background, so unbounded requests mean unbounded spend. "Rate limiting exists
as a class but is never called" is exactly the kind of gap that looks fine in
code review and bites in production.

**3. It sits exactly on my Python backend experience.**
Starlette/FastAPI middleware, Redis, and JWT decoding are the stack I have
actually built in. I know where the sharp edges are here — middleware ordering,
blocking a sync Redis client inside an async handler, deciding whether
`X-Forwarded-For` can be trusted — rather than discovering them from scratch.

**Toolkit**

- Tool: I ran `issue-scout`, but didn't find any issues that match my interest. So I followed issue-select workflow and manual review of the PathReview issue.
- What it produced: I selected Issue #28, investigated its scope, and posted a claim comment describing my intention to reproduce the empty-corpus failure.
- Difficulties: searching from `issue-scout` results
- What changed: I would preserve search queries, ranking decisions, and issue verdicts as part of my future selection workflow.

---

## Understanding the Issue

**Problem Description**

PathReview ships a complete, unit-tested rate limiter that nothing ever calls.
`RateLimiter.check_rate_limit` in `safety/rate_limiter.py` implements a rolling
window on Redis sorted sets and accepts any string identifier, but no module
under `api/` imports it. The result is an API with no rate limiting at all,
while `README.md` and `docs/ARCHITECTURE.md` both describe the API layer as
having it. The gap is invisible in code review — the class is there, the tests
pass, the setting exists — and only shows up under traffic.

**Expected Behavior**

Every request is counted against a rolling window and refused once it exceeds a
limit:

- by client IP, for all traffic, signed in or not;
- additionally by user ID, when the request carries a bearer token.

Requests over either limit get `429 Too Many Requests` with a `Retry-After`
header, instead of being served.

**Current Behavior**

No request is ever refused for rate reasons. Four checks against the tree, each
reproducible from a clone:

```
$ grep -rn "rate_limiter\|RateLimiter" api/ core/
(no output)

$ grep -rn "add_middleware" api/
api/main.py:43:app.add_middleware(
api/main.py:44:    CORSMiddleware,
api/main.py:52:app.add_middleware(RequestIDMiddleware)

$ grep -rn "rate_limit_per_minute" --include="*.py" .
core/config.py:42:    rate_limit_per_minute: int = Field(default=60)

$ grep -rn "429\|TOO_MANY_REQUESTS\|Retry-After\|X-RateLimit" \
    --include="*.py" --include="*.ts" --include="*.tsx" --include="*.md" . \
    | grep -v node_modules
(no output)
```

Read in order: the limiter has no production caller; the middleware stack is
`CORSMiddleware` plus `RequestIDMiddleware`, so there is no seam where a limit
could be enforced; `rate_limit_per_minute` is declared and read by nothing, so
the knob is inert at any value; and no code path anywhere can produce a `429`.

Two consequences the issue text does not spell out:

- `POST /auth/login` is unauthenticated and runs a bcrypt verification per
  call, so this is a brute-force surface, not only a throughput concern.
- `POST /reviews` starts an LLM-backed background job, so request volume maps
  directly to spend.

**Affected Components**

| Component | Role in this issue |
| --- | --- |
| `safety/rate_limiter.py` | The working limiter. Has no caller. Not changed by the fix. |
| `api/main.py` | Builds the middleware stack. The missing registration point. |
| `api/middleware/` | Where request-scoped logic lives (`auth.py`, `request_id.py`). Where the new middleware belongs. |
| `core/config.py` | Declares `rate_limit_per_minute`, which nothing reads. |
| `core/security.py` | `decode_access_token`, needed to identify a user without a DB lookup. |
| `frontend/src/services/api.ts` | Has no `429` handling. Out of scope, but affected downstream. |

**Toolkit**

- Tool: Claude Code, used to read the issue against the code and to run the
  four `grep` checks above before writing anything.
- What it produced: the four-check static proof that the limiter has no caller,
  plus the two consequences above (the bcrypt cost on `/auth/login`, the LLM
  cost on `/reviews`) that the issue text does not mention.
- Difficulties: the model's first instinct was to accept the issue's framing
  and start designing middleware. Reading is not proof, and a static grep is
  not a reproduction — both needed pushing before the design work started.
- What changed: I made the static checks a precondition for the plan rather
  than a substitute for the runtime repro, and kept "what I verified" and "what
  I only read" as separate lists from this point onward.

---

## Reproduction Process

**Environment Setup**
| Item | Value |
| --- | --- |
| OS | macOS 27.2 (Darwin 27.2.0), arm64 |
| Python | 3.14.5 — `python --version` inside `.venv` (repo targets 3.11; `pyproject.toml` sets `requires-python = ">=3.11"`) |
| Repo | forked from `codepath/pathreview-ai301-fa26-s3` |
| Branch | `add/28-rate-limiting-per-user` — see note below |
| Services | `docker compose up -d` → postgres:16-alpine (host port 5433), redis:7-alpine (6379), chromadb 0.4.22 (8001) |
| Config | stock `.env.example` copied to `.env`; `RATE_LIMIT_*` unset, so `rate_limit_per_minute` is its default `60` |

- `GET /health` reported TODO before the run


**Steps to Reproduce**

Unauthenticated, 200 requests to `GET /reviews` (which returns `401` without a
token, so a `429` would be the only thing that could change the code):

```bash
for i in $(seq 1 200); do
  curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8000/reviews
done | sort | uniq -c
```

**Step 2 — no rate limit headers on any response**

```bash
curl -sS -D - -o /dev/null http://localhost:8000/reviews | grep -i -E "ratelimit|retry-after|^HTTP"
```

**Expected if the issue is real:** the `HTTP/1.1 401` status line and no
`X-RateLimit-*` or `Retry-After` header at all.

**Step 3 — authenticated flood**

This proves the gap applies to signed-in traffic too, which is the "per user"
half of the issue title.

```bash
# Register a throwaway user and capture the token
TOKEN=$(curl -s -X POST http://localhost:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"repro28@example.com","password":"repro-password-123"}' \
  | python -c "import sys,json; print(json.load(sys.stdin)['access_token'])")

echo "${TOKEN:0:12}..."   # sanity check that a token came back

for i in $(seq 1 200); do
  curl -s -o /dev/null -w "%{http_code}\n" \
    -H "Authorization: Bearer $TOKEN" http://localhost:8000/reviews
done | sort | uniq -c
```

**Expected if the issue is real:** `200` responses of `200`, and zero `429`.

> If the email already exists, `POST /auth/register` returns `400 Email already
> registered`. Either change the email or log in instead with
> `curl -s -X POST http://localhost:8000/auth/login -d "username=...&password=..."`.


**Step 4 — unauthenticated flood against the login endpoint**

The security case, and the one most worth showing a maintainer. `POST
/auth/login` is unauthenticated and runs a bcrypt verification per call.

```bash
for i in $(seq 1 100); do
  curl -s -o /dev/null -w "%{http_code}\n" -X POST http://localhost:8000/auth/login \
    -d "username=repro28@example.com&password=wrong-password"
done | sort | uniq -c
```

**Expected if the issue is real:** `100` responses of `401`, and zero `429` —
i.e. nothing stops a password-guessing loop.


```
$ grep -rn "rate_limiter\|RateLimiter" api/ core/
(no output)

$ grep -rn "add_middleware" api/
api/main.py:43:app.add_middleware(
api/main.py:44:    CORSMiddleware,
api/main.py:52:app.add_middleware(RequestIDMiddleware)

$ grep -rn "rate_limit_per_minute" --include="*.py" .
core/config.py:42:    rate_limit_per_minute: int = Field(default=60)
```

**Reproduction Evidence**

The issue is accurate as filed. `RateLimiter` works correctly when it is
called — the control run refuses the 4th request at a limit of 3 — but nothing
in the request path calls it. The middleware stack is `CORSMiddleware` and
`RequestIDMiddleware` only, and `settings.rate_limit_per_minute` is read by no
code, so the configuration knob has no effect at any value.

Two things beyond the issue text that seem worth recording:

- `POST /auth/login` is unauthenticated and runs a bcrypt verification per
  call, so the gap is a brute-force surface and not only a throughput concern.
- The frontend has no handling for `429` either — `frontend/src/services/api.ts`
  turns any non-OK response into a generic `Error`. Out of scope for #28, but
  whoever adds the limit should expect the UI to show a bare error string.

### What I did not verify

- Behavior behind a reverse proxy. Everything above was measured against
  uvicorn directly, so `request.client.host` is the real peer. If the app is
  deployed behind a proxy, client-IP detection becomes its own question.
- Whether `/health` should be exempt from limiting. I have an opinion but no
  evidence, so I'll raise it in the plan rather than assert it here.

---

**Toolkit**

- Tool: `curl`, `grep`, Claude Code, 
- What it produced: Errors from different API calls
- Difficulties: a lot of reading
- What changed: nothing was changed as I was just running API calls from terminal

---

## Solution Approach

**Analysis**

This is a missing-integration gap, not a defect in the limiter. The evidence
separates the two cleanly:

- **The limiter works.** `RateLimiter.check_rate_limit`
  ([safety/rate_limiter.py](safety/rate_limiter.py)) keeps a rolling window in a
  Redis sorted set: it evicts entries older than `now - window_seconds`, counts
  what remains, and admits or refuses. `tests/unit/test_rate_limiter.py` covers
  it with 19 tests, including `test_ip_address_as_identifier`, which already
  passes an IP as the identifier. The control step in my repro calls it directly
  and watches it refuse the 4th request at a limit of 3.
- **Nothing calls it.** `grep -rn "rate_limiter\|RateLimiter" api/ core/` returns
  nothing. The only callers are tests.
- **There is no seam for it to run in.** `grep -rn "add_middleware" api/` shows
  the full middleware stack is `CORSMiddleware` + `RequestIDMiddleware`
  ([api/main.py:43-52](api/main.py#L43-L52)).
- **The config knob is inert.** `rate_limit_per_minute` is declared at
  [core/config.py:42](core/config.py#L42) and read by no code, so it has no
  effect at any value.
- **No refusal path exists at all.** `429`, `TOO_MANY_REQUESTS`, `Retry-After`
  and `X-RateLimit` appear nowhere in the Python, TypeScript or Markdown
  sources.

So the cause is precise and proven: **a working limiter with no call site in the
request path.** The fix is therefore a call site — middleware — and not any
change to `safety/rate_limiter.py`, which stays untouched.

**Proposed Solution**

- A new `RateLimitMiddleware` that calls the existing `RateLimiter` once per
  request per applicable bucket.
- Per-IP bucket on every request; per-user bucket additionally when the request
  carries a decodable bearer token. Both must pass.
- Registration in `api/main.py`, positioned deliberately in the stack.
- Settings for the two limits, the window, and an on/off switch, plus the
  matching `.env.example` entries.
- A `429` response with `Retry-After` and `X-RateLimit-*` headers, and those
  headers on allowed responses too.
- Unit tests for the middleware.
- A rate-limiting section in `docs/API.md`.

**Implementation Plan**

1. Files to be modified

| File | Change |
| --- | --- |
| `api/middleware/rate_limit.py` | **New.** The middleware. |
| `api/main.py` | Import and register it, positioned inside CORS. |
| `core/config.py` | Add `rate_limit_enabled`, `rate_limit_per_ip_per_minute`, `rate_limit_window_seconds`; keep `rate_limit_per_minute` as the per-user budget. |
| `.env.example` | Document the four knobs. |
| `docs/API.md` | Add a "Rate limiting" section: limits, headers, `429` shape, exemptions. |
| `tests/unit/test_rate_limit_middleware.py` | **New.** Unit tests. |

2. Test Plan

Re-running the three repro loops: unauthenticated `GET /reviews` should start returning `429` after ~120 requests instead of `401` forever; the authenticated loop should cut off at ~60 with `"scope": "user"`; the login loop should stop accepting attempts. Plus regression checks that `/health` still answers during a flood, that a caller recovers after the window rolls off, and that a normal register → profile → review flow is untouched. Unit tests cover the middleware with a fake limiter so `tests/unit/` stays Docker-free.

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Testing Strategy


### Automated

**1. Run the new unit tests.**

```bash
.venv/bin/pytest tests/unit/test_rate_limit_middleware.py -v
```

They drive the middleware with an in-memory fake limiter and cover: under-limit
requests allowed; the over-limit request refused with `429`; `Retry-After` and
`X-RateLimit-*` present; separate IPs having separate budgets; a token adding a
`user:` bucket; a user limit biting inside an unexhausted IP budget; two users
sharing one IP's budget; two users having independent user budgets; a garbage
token falling back to IP-only *without* being rejected; `/health` and `OPTIONS`
exempt; a `429` keeping its CORS header; a raising limiter failing open; and the
middleware being installed on the real app, inside CORS.

**2. Full unit suite — nothing else regressed.**

```bash
make test-unit
```

`tests/unit/test_rate_limiter.py` must still pass untouched, which is the
evidence that this change added a caller rather than altering the limiter.

**3. Lint, format, types.**

```bash
make lint && make typecheck && .venv/bin/black --check .
```

### Manual, against the running stack

Same setup as the repro: `docker compose up -d`, `uvicorn api.main:app`, second
terminal. Confirm Redis is genuinely up first
(`docker compose exec redis redis-cli ping` → `PONG`), because fail-open means a
dead Redis would silently reproduce the *old* behavior and look like a failed fix.

**4. Re-run repro step 1 — unauthenticated flood.**

```bash
for i in $(seq 1 200); do
  curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8000/reviews
done | sort | uniq -c
```

Before: 200 × `401`, zero `429`. **Expected after: ~120 × `401`, then `429` for
the rest.** The exact split depends on the rolling window and how long the loop
takes; what must be true is that `429` appears and the `401` count is near the
IP limit.

**5. Re-run repro step 3 — authenticated flood.**

Same loop with a bearer token. Before: 200 × `200`. **Expected after: ~60 × `200`
then `429`** — the user limit (60) bites before the IP limit (120), and the body
reports `"scope": "user"`.

**6. Re-run repro step 4 — login brute force.**

```bash
for i in $(seq 1 100); do
  curl -s -o /dev/null -w "%{http_code}\n" -X POST http://localhost:8000/auth/login \
    -d "username=repro28@example.com&password=wrong-password"
done | sort | uniq -c
```

Before: 100 × `401`. **Expected after: `401` up to the IP limit, then `429`.**

**Toolkit**

- Tool: 
- What it produced:
- Difficulties: figuring out different test for different API call behaviors
- What changed:

---

## Implementation Notes

**2026-11-15 — static verification before any code**

Ran the four greps in Understanding the Issue. Confirmed the limiter has no
caller, the middleware stack has two entries, the config knob is inert, and no
`429` path exists anywhere. Decided on the basis of this that
`safety/rate_limiter.py` stays untouched: the defect is a missing call site, not
a broken algorithm, and editing a correct, 19-test-covered class would widen the
diff for nothing.

**2026-11-15 22:24 — config and env first (`f602972`)**

Added `rate_limit_enabled`, `rate_limit_per_ip_per_minute` and
`rate_limit_window_seconds` to `core/config.py`, keeping the existing
`rate_limit_per_minute` as the per-user budget, and documented all four in
`.env.example`. Set the IP budget (120) to double the user budget (60), because
one IP can legitimately carry several users behind NAT while a user ID is one
person.

One thing I deliberately did not copy: `api/routes/health.py` builds its Redis
client from `settings.redis_host` and `settings.redis_port`, neither of which
exists on `Settings`. That is seeded bug #62, already suppressed in
`pyproject.toml`. The new middleware uses `settings.redis_url`, which is the
attribute that actually exists.

**2026-11-15 22:26 — documented the contract before implementing it (`9677570`)**

Wrote the "Rate limiting" section of `docs/API.md` — the two buckets, the
headers, the `429` body shape, the exemption list — before writing the
middleware. Writing the contract first is what surfaced the `Retry-After`
problem: `check_rate_limit` returns `(allowed, remaining)` and never says when
the oldest entry leaves the window, so there is no way to compute an exact
retry time without changing the limiter's return type. Settled on advertising
the full window length as an honest upper bound, and said so in the doc.

**2026-11-15 22:27 — the middleware and its tests (`b79a13f`)**

Four decisions, made while writing it:

1. **Position inside `CORSMiddleware`.** Starlette's `add_middleware` inserts at
   index 0 and index 0 runs outermost, so registering rate limiting *first*
   puts it innermost: `RequestID → CORS → RateLimit → routes`. If it sat
   outside CORS, a `429` would reach the browser without
   `Access-Control-Allow-Origin` and the frontend would report an opaque CORS
   failure instead of a rate limit — the user would see the wrong problem named.
2. **`decode_access_token`, not `get_current_user`.** The auth dependency needs
   a DB session and cannot run in middleware. Decoding the JWT in-process gives
   the `sub` claim with no database round trip. The middleware never returns a
   `401`: an invalid token just means IP-only limiting, and rejecting it stays
   the route's job. Authentication keeps one home.
3. **`asyncio.to_thread` around the limiter call.** `RateLimiter` is
   synchronous and `dispatch` is async, so calling it directly would block the
   event loop for every concurrent request — worst exactly during the flood the
   feature exists to stop. The thread hop keeps `safety/rate_limiter.py`
   unchanged.
4. **Namespaced keys.** Identifiers are `ip:<addr>` and `user:<id>` so the two
   buckets cannot collide inside the limiter's shared `rate_limit:{identifier}`
   keyspace.

Also exempted `/`, `/health`, `/docs`, `/redoc`, `/openapi.json` and every
`OPTIONS` request. `/health` because throttling a monitoring endpoint makes the
service look down during exactly the incident monitoring is watching for;
`OPTIONS` because a throttled preflight shows the browser a CORS error for a
limit the user never hit. Nothing in the repo asked for either exemption — both
are my judgment, and both are one line to reverse, so I flagged them for a
maintainer rather than burying them.

**2026-11-15 — wrong turn: I could not start the server**

The plan's manual steps all assumed `uvicorn api.main:app` and `curl`. It does
not start:

```
asyncpg.exceptions.DuplicateTableError: relation "ix_profiles_user_id" already exists
[SQL: CREATE INDEX ix_profiles_user_id ON profiles (user_id)]
ERROR:    Application startup failed. Exiting.
```

I first assumed stale state from the seeded database and created a fresh one.
It failed identically, which ruled that out. The actual cause is in
`core/models/profile.py`: `user_id` is declared `index=True` *and*
`__table_args__` declares `Index("ix_profiles_user_id", "user_id")`, so
`create_all` issues the same `CREATE INDEX` twice. Pre-existing, unrelated to
#28, and not mine to fix inside this PR.

Worked around it instead of widening scope: drove the real `api.main:app`
through Starlette's `TestClient` (which skips the startup hook) against the
real Redis container, flooding `GET /reviews` — which returns `401` from the
auth dependency without touching the database, so a `429` is the only thing
that can change the status code. That produced real refusals
(`rate_limit_rejected ... limit=120 scope=ip`) and confirmed the `429` carries
`access-control-allow-origin: http://localhost:5173`, which was the specific
thing decision 1 existed to guarantee.

**2026-11-15 — repo checks**

```
24 passed                                    # tests/unit/test_rate_limit_middleware.py
399 passed, 53 xfailed                       # full unit suite
All checks passed!                           # ruff
Success: no issues found in 77 source files  # mypy
112 files would be left unchanged            # black --check
```

The 53 `xfailed` matter as much as the 399 passed: every seeded bug's marker is
still intact, which is the evidence that this change added a caller rather than
quietly altering something else.

Caveat worth recording: the venv is Python 3.14.5 while the repo targets 3.11
(`requires-python = ">=3.11"`). All of the above passed on 3.14, but CI runs
3.11 and that is the number that counts.

**2026-11-15 — deviation I need to fix before opening the PR**

The branch is `add/28-rate-limiting-per-user`, and the commit subjects are
`add: ...` and `docs/add ...`. `docs/CONTRIBUTING.md` lists the allowed types as
`fix, feat, test, docs, refactor, perf, chore, ci` — `add` is not one, the
format is `<type>(<scope>): <description>`, and "per-user" misnames an issue
whose point is per-IP. It should have been `feat/28-ip-rate-limiting` with
`feat(api): ...` subjects.

Worse: I cut the branch from `fix/37-api-request-body-schema` instead of from
`main`, so `git log main..HEAD` still carries the #37 docs commit plus `plan.md`
and `pr_draft.md`. A PR opened from here would show #37's work as part of #28.
That has to be rebased onto `main` before the PR goes out.

**Toolkit**

- Tool: Claude Code, for the implementation, the test suite, and running the
  repo's five checks.
- What it produced: `api/middleware/rate_limit.py`, its registration in
  `api/main.py`, the config and `.env.example` entries, the `docs/API.md`
  section, and 24 unit tests driving the middleware with an in-memory fake
  limiter so `tests/unit/` stays Docker-free.
- Difficulties: two real ones. It wrote the manual test plan around `uvicorn`
  and `curl` without checking the server actually starts, which cost the detour
  above. And it initially reported the implementation as finished while the
  tests had never been run — the claim and the evidence were separate things,
  and only the second one counted.
- What changed: I stopped accepting "implemented" as a status and started
  requiring the command output. Every pass claim in this document is now a
  pasted result, and the things that were not run are listed as not run.

---

## Pull Request

**PR Link:** [github.com/codepath/pathreview-ai301-fa26-s3](https://github.com/codepath/pathreview-ai301-fa26-s3/pull/285)

**PR Description:**

feat(api): rate limit requests by client IP and by user

## Summary

`safety/rate_limiter.py` has shipped a working rolling-window rate limiter since the repo was created, but nothing in the request path ever called it: no module under `api/` imported it, the middleware stack was `CORSMiddleware` plus `RequestIDMiddleware` only, and `settings.rate_limit_per_minute` was read by no code. The result was an API with no rate limiting at all — signed in or not — while `README.md` and `docs/ARCHITECTURE.md` both described the API layer as having it. This PR adds a `RateLimitMiddleware` that calls the existing limiter once per request per bucket: every request is counted against its client IP, and requests carrying a decodable bearer token are counted against the user ID as well. Both buckets must have capacity, so signing in does not raise the per-IP ceiling. `safety/rate_limiter.py` is not modified — the defect was a missing call site, not a broken algorithm.

## Issue

Closes #28

## Changes

- Added `api/middleware/rate_limit.py`: a `BaseHTTPMiddleware` subclass that checks each request against an `ip:<client host>` bucket and, when a bearer token decodes, additionally against a `user:<sub>` bucket. Either bucket can reject.
- Registered it in `api/main.py` **before** `CORSMiddleware` in source order, which places it *inside* CORS at runtime — Starlette runs the last-registered middleware outermost. Execution order is now `RequestID → CORS → RateLimit → routes`. This is deliberate: a `429` returned from outside `CORSMiddleware` reaches the browser without `Access-Control-Allow-Origin`, and the frontend reports an opaque CORS failure instead of a rate limit.
- Identifies the user with `decode_access_token` from `core/security.py` rather than the `get_current_user` dependency, which needs a DB session and cannot run in middleware. There is no database round trip in the request path. The middleware never returns `401`: an invalid, expired or malformed token simply means IP-only limiting, and rejecting it stays the route's job.
- Wrapped the synchronous limiter call in `asyncio.to_thread`. `RateLimiter` is sync and `dispatch` is async, so a direct call would block the event loop for every concurrent request — worst precisely during the flood this feature exists to stop.
- Namespaced the identifiers as `ip:<addr>` and `user:<id>` so the two buckets cannot collide inside the limiter's shared `rate_limit:{identifier}` keyspace.
- Added settings in `core/config.py`: `rate_limit_enabled` (default `true`), `rate_limit_per_ip_per_minute` (default `120`), `rate_limit_window_seconds` (default `60`). The existing `rate_limit_per_minute` (default `60`) becomes the per-user budget and is now actually read. The IP budget is double the user budget because one address can legitimately carry several users behind NAT, while a user ID is one person.
- Documented all four knobs in `.env.example`.
- Exempted `/`, `/health`, `/docs`, `/redoc`, `/openapi.json` and every `OPTIONS` request. `/health` because throttling a monitoring endpoint makes the service look down during exactly the incident monitoring exists to catch; `OPTIONS` because a throttled CORS preflight shows the browser a CORS error for a limit the user never hit.
- `429` responses carry `Retry-After`, `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Window`, with a body of `{detail, scope, request_id}` where `scope` is `"ip"` or `"user"`. Allowed responses carry the `X-RateLimit-*` headers for whichever bucket is closest to exhaustion.
- Fails open. The limiter already swallows Redis errors and returns "allowed"; the middleware adds a second guard around the call and logs `rate_limit_check_failed`. A Redis outage degrades the limit, not the API.
- Added `tests/unit/test_rate_limit_middleware.py`: 24 tests driving the middleware with an in-memory fake limiter, so `tests/unit/` stays Docker-free.
- Added a "Rate limiting" section to `docs/API.md`: the two buckets, the configuration, the exemption list, the response headers and the `429` body shape.

## Testing

- [ ] **CI is green on this PR (all five jobs)** — required for review
- [X] Unit tests pass (`make test-unit`)
- [ ] Integration tests pass (`make test-integration`) — see note below; the suite is empty
- [X] Linter passes (`make lint`)
- [X] Type checker passes (`make typecheck`)
- [X] New/updated tests cover the changes
- [ ] If this fixes a seeded bug: removed its `@pytest.mark.xfail` marker (and any
      matching suppression in `pyproject.toml`) — N/A, this is an enhancement with no xfail marker or suppression attached to it

**Before** — four checks against `main`, each reproducible from a clone:

```
$ grep -rn "rate_limiter\|RateLimiter" api/ core/
(no output)

$ grep -rn "add_middleware" api/
api/main.py:43:app.add_middleware(
api/main.py:44:    CORSMiddleware,
api/main.py:52:app.add_middleware(RequestIDMiddleware)

$ grep -rn "rate_limit_per_minute" --include="*.py" .
core/config.py:42:    rate_limit_per_minute: int = Field(default=60)

$ grep -rn "429\|TOO_MANY_REQUESTS\|Retry-After\|X-RateLimit" \
    --include="*.py" --include="*.ts" --include="*.tsx" --include="*.md" . \
    | grep -v node_modules
(no output)
```

No production caller, no middleware slot, an inert config knob, and no code path anywhere that can emit a `429`.

**Control — the limiter itself was never the problem.** `tests/unit/test_rate_limiter.py` passes untouched on this branch, including `test_ip_address_as_identifier`, which already passed an IP as the identifier. That is the evidence that this PR added a caller rather than altering behavior.

**After — new unit tests:**

```
$ .venv/bin/pytest tests/unit/test_rate_limit_middleware.py -v
24 passed, 3 warnings in 1.26s
```

They cover: under-limit requests allowed; the over-limit request refused with `429`; `Retry-After` and `X-RateLimit-*` present; separate IPs having separate budgets; a token adding a `user:` bucket; a user limit biting inside an unexhausted IP budget; two users sharing one IP's budget; two users having independent user budgets; four shapes of bad `Authorization` header falling back to IP-only *without* being rejected; `/health` and `OPTIONS` exempt; a `429` keeping its CORS header; a raising limiter failing open; and the middleware being installed on the real app, inside CORS.

**After — full unit suite, no regressions:**

```
$ .venv/bin/pytest tests/unit -m unit -q
399 passed, 53 xfailed, 3 warnings in 7.39s
```

The 53 `xfailed` matter as much as the 399 passed: every seeded bug's marker is still intact, so nothing was fixed or broken by accident.

**After — repo checks:**

```
$ .venv/bin/ruff check .
All checks passed!

$ .venv/bin/mypy api/ core/ ingestion/ rag/ agent/ safety/
Success: no issues found in 77 source files

$ .venv/bin/black --check .
112 files would be left unchanged.
```

**After — the middleware actually refusing requests against real Redis.** I drove `api.main:app` through Starlette's `TestClient` against the running `redis:7-alpine` container, flooding `GET /reviews` from a fixed client IP. `GET /reviews` returns `401` from the auth dependency without touching the database, so a `429` is the only thing that can change the status code. Past 120 requests the middleware began refusing, logging:

```
[warning] rate_limit_exceeded  identifier=ip:198.51.100.7 limit=120 request_id=090e5094-...
[warning] rate_limit_rejected  identifier=ip:198.51.100.7 limit=120 method=GET path=/reviews
                               request_id=090e5094-... scope=ip
```

and the `429` carried the CORS header, which is the specific thing the middleware ordering exists to guarantee:

```
6. CORS header on 429 response : http://localhost:5173
```

**Check: diff scope.** `git diff --stat` against the branch point touches six files, 639 insertions, zero deletions: `api/middleware/rate_limit.py` (new), `tests/unit/test_rate_limit_middleware.py` (new), `api/main.py`, `core/config.py`, `.env.example`, `docs/API.md`. No change to `safety/rate_limiter.py`.

**Not verified, and why:**

- **The end-to-end `curl` run against a live `uvicorn`.** The server does not start on this branch or on `main` — see "A separate bug I found" below. The `TestClient` run above exercises the same app object, the same middleware stack and the same Redis, but over an in-process transport rather than a socket.
- **`make test-integration`** collects zero tests — `tests/integration/` contains only `__init__.py`. It reports "no tests ran" rather than passing, so I have not ticked it.
- **Python version.** My venv is 3.14.5; the repo targets 3.11 (`requires-python = ">=3.11"`). Everything above passed on 3.14, but CI runs 3.11 and that is the number that counts.
- **Behavior behind a reverse proxy.** See the maintainer question below.

## Screenshots / Demo

N/A — this change has no UI surface. The observable behavior is the status code and the response headers, both shown in the Testing section above.

## Notes for Reviewers

**AI use.** I read the issue and the relevant files myself and formed the diagnosis — that the limiter is correct and simply has no caller — before writing any code. I used implemented the middleware with Claude Code assistant and the test suite from that diagnosis, and to run the repo's five checks. I reviewed every design decision in the diff against the code it touches: the middleware ordering against Starlette's `add_middleware` semantics, the JWT decode path against `api/middleware/auth.py` and `core/security.py`, and the Redis client construction against `core/config.py`. The scope boundaries — not touching `safety/rate_limiter.py`, leaving `X-Forwarded-For` out, not fixing the startup bug I ran into — were my calls.

**Changed from my plan.** My plan's test section was built around `uvicorn` plus `curl` loops. After building out the middleware and API call, I implemented unit tests for the new logic without typing in the terminal


**Questions for a maintainer.**

- **Is the app expected to run behind a proxy?** If it is, `request.client.host` becomes the proxy's address and every user collapses into one bucket, which would make the limit useless. The fix would be a `rate_limit_trusted_proxies` setting, which I would rather add deliberately than guess at. This is the one design choice that could invalidate the approach.
- **Should `/health` be exempt?** I exempted it on my own judgment — nothing in the repo asks for it. It is one entry in a frozenset if you disagree.


**Maintainer Feedback:**

- [Date]: None

**Status:** Awaiting Review

**Toolkit**

- Tool: Markdown
- What it produced: Markdown Pull Request draft
- Difficulties: NA
- What changed: NA

---

## Learnings and Reflections

**Technical Skills Gained**

- **Starlette middleware ordering, and why it is not cosmetic.**
  `add_middleware` inserts at index 0, and index 0 runs *outermost*, so
  registration order is the reverse of execution order. Getting this backwards
  would have put rate limiting outside CORS, and every `429` would have reached
  the browser stripped of `Access-Control-Allow-Origin` — the frontend would
  have reported a CORS failure for a rate limit. A backend ordering detail
  decides what error a frontend user sees.
- **Async middleware cannot call sync Redis directly.** A synchronous client in
  an async `dispatch` blocks the event loop for every concurrent request. That
  is tolerable at idle and worst precisely during a flood, which is the one
  situation this feature exists for. `asyncio.to_thread` solved it without
  touching the limiter.
- **Authentication in middleware is a different problem from authentication in
  a route.** `get_current_user` is a FastAPI dependency and needs a DB session,
  so it cannot run here. Decoding the JWT in-process gets the user ID with no
  database round trip — and the middleware must *not* reject a bad token, or
  there are suddenly two places that decide what a `401` means.
- **Proving an absence needs a control.** The issue claims requests are never
  refused. A flood showing no `429` is equally consistent with "the limiter is
  never called" and "the limiter is broken." Calling `check_rate_limit`
  directly and watching it refuse the 4th request at a limit of 3 is what
  separates the two, and it is what made the diagnosis specific enough to act
  on.

**Challenges Overcome**
- **Choosing an endpoint that could only answer one question.** Most endpoints
  need a database, so a failure could mean anything. `GET /reviews` without a
  token returns `401` from the auth dependency before touching the DB, which
  makes a `429` the only thing that can change the status code. Picking the
  target carefully is what made the result mean something.
- **A version mismatch I could not remove.** The venv is Python 3.14.5; the repo
  targets 3.11. Everything passes locally, but CI runs 3.11, so the local green
  is suggestive rather than decisive. I recorded the gap instead of implying there's a match
- **Separating what I built from what I had verified.** For a stretch the
  implementation was described as done while no test had been run. Those are two
  different claims, and only the second is evidence.

**What I'd Do Differently Next Time**

- **Read `CONTRIBUTING.md`'s naming rules before the first commit, not after the
  third.** `add` is not an allowed type, the subjects do not follow
  `<type>(<scope>): <description>`, and the branch name says "per-user" for an
  issue about per-IP. Cheap to get right up front, annoying to rewrite later.
- **Keep the working-file artifacts out of the repo, or out of the PR branch.**
  `plan.md`, `pr_draft.md` and the `issue-28-*.md` drafts are useful to me and
  noise to a reviewer.

---

## Resources Used

In-repo, and the ones that did the most work:

- **`safety/rate_limiter.py` and `tests/unit/test_rate_limiter.py`** — the
  limiter's own tests were the best documentation of its intended use.
  `test_ip_address_as_identifier` passes `"192.168.1.1"` as the identifier,
  which is how I knew keying by IP was the design's intent and not my
  invention.
- **`api/middleware/request_id.py`** — the existing `BaseHTTPMiddleware`
  subclass in this repo. I matched its structure, logging style and docstring
  format so the new middleware reads like it belongs.
- **`core/security.py`** — `decode_access_token` is what made per-user limiting
  possible without a DB round trip in middleware.
- **`docs/CONTRIBUTING.md`** — branch naming, Conventional Commits, the five CI
  jobs, and the warnings about seeded bugs and `ruff --unsafe-fixes`. Also the
  document that told me my own branch and commit names were wrong.

External:

- **Starlette middleware docs** (https://www.starlette.io/middleware/) — confirmed
  that `add_middleware` inserts at the front and that the first entry runs
  outermost, which is the fact the CORS ordering decision rests on.
- **FastAPI CORS docs** (https://fastapi.tiangolo.com/tutorial/cors/) — for how
  `CORSMiddleware` attaches headers to responses generated by inner middleware.
- **MDN on `429` and `Retry-After`**
  (https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429) — the response
  contract I wrote into `docs/API.md`.
- **Claude Code** — used within every steps for writing test from terminal help research unfamiliar topics

