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

The project already have Rate limiting code definition (`RateLimiter.check_rate_limit`) but no code in `/api` calls it. `tests/unit/test_rate_limiter.py` exercises it directly — including `test_ip_address_as_identifier`, which passes an IP as the identifier.

**Expected Behavior**

[What should happen?]

**Current Behavior**

[What actually happens?]

**Affected Components**

[Which parts of the codebase are involved?]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

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

2. [Add or change Z]
3. Test Plan

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

[A running log, added to as you work. Date each entry. What you
built, decisions you made when you made them, wrong turns included.]

**[Date]**

[Entry]

**[Date]**

[Entry]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Pull Request

**PR Link:** [GitHub PR URL when opened]

**PR Description:** [what the PR says; much of the content above
adapts into it]

**Maintainer Feedback:**

- [Date]: [feedback received, or "none yet"; silence is a normal
  state and costs you nothing]
- [Date]: [how you addressed it]

**Status:** [Awaiting review / Iterating / Merged / Closed. The
status is a fact you record, never a grade: the PR is graded as
opened.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Learnings and Reflections

**Technical Skills Gained**

[What you learned technically, named concretely.]

**Challenges Overcome**

[What was hard and how you solved it; point at the real events, the
Toolkit blocks above are full of them.]

**What I'd Do Differently Next Time**

[Reflection on your process.]

---

## Resources Used

[What actually helped, not a reading list. List things this story
already shows you using: the doc you followed to set the environment
up, the thread that explained the bug, the person who answered you.
One real entry beats ten plausible ones.]

- [Link to documentation that helped]
- [Discussion, issue, or post that helped]
