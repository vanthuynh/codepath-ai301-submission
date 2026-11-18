# Rate limiting an API that already had a rate limiter

## Overview

PathReview is an AI portfolio-review service — a FastAPI backend with a React dashboard — and it shipped with a complete, 19-test rolling-window rate limiter in `safety/rate_limiter.py` that no code in `api/` ever called. The class was there, its tests passed, and `settings.rate_limit_per_minute` existed, so the gap was invisible in code review; in practice no request was ever refused, signed in or not, including unauthenticated `POST /auth/login`, which runs a bcrypt verification per call. For issue #28 I wrote a `RateLimitMiddleware` that calls the existing limiter once per request per bucket — by client IP always, and by user ID additionally when a bearer token decodes — without modifying the limiter itself, because the defect was a missing call site rather than a broken algorithm. That is [PR #285](https://github.com/codepath/pathreview-ai301-fa26-s3/pull/285): open, mergeable, all five CI jobs green, and as of 2026-11-17 it has no maintainer review and no comments. On the tooling side I wrote the parts the course skills leave empty — the `repro-check` rubric and voice guide, the `plan-check` rubric, and the `pr-precheck` rubric — and ran `pr-precheck` against this PR before opening it.

## What happened

**I started by proving the issue rather than believing it.** The issue says nothing calls the limiter. Four greps against `main` said the same thing, and this is the before-state the whole change rests on:

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
    --include="*.py" --include="*.ts" --include="*.tsx" --include="*.md" . | grep -v node_modules
(no output)
```

No production caller, no middleware slot to run in, a config knob nothing reads, and no code path anywhere that could emit a `429`.

**That left two explanations, and they need different fixes.** "The limiter is broken" and "the limiter is never called" both produce a flood with zero refusals. I separated them by calling `check_rate_limit` directly: it refuses the 4th request at a limit of 3, and `tests/unit/test_rate_limiter.py` passes untouched on my branch — including `test_ip_address_as_identifier`, which already passes `"192.168.1.1"` as the identifier:

```
$ .venv/bin/pytest tests/unit/test_rate_limiter.py -q
19 passed in 0.23s
```

Someone designed this class to be keyed by IP. So the fix was a call site, and `safety/rate_limiter.py` is not in my diff.

**The decision I spent the most thought on was middleware order.** Starlette's `add_middleware` inserts at index 0 and index 0 runs outermost, so registering rate limiting *first* puts it innermost: `RequestID → CORS → RateLimit → routes`. Sitting outside `CORSMiddleware` instead, a `429` would reach the browser stripped of `Access-Control-Allow-Origin`, and the frontend would report an opaque CORS failure for a rate limit — the user would see the wrong problem named. A backend ordering detail decides what error a frontend user reads.

**A change I made: my verification plan was impossible, and I rewrote it.** My plan's manual tests were `uvicorn api.main:app` plus `curl` loops. The server does not start:

```
asyncpg.exceptions.DuplicateTableError: relation "ix_profiles_user_id" already exists
[SQL: CREATE INDEX ix_profiles_user_id ON profiles (user_id)]
ERROR:    Application startup failed. Exiting.
```

My first guess was stale seeded data. I created a fresh database and it failed identically, which ruled that out. The real cause is in `core/models/profile.py`: `user_id` is declared `index=True` *and* `__table_args__` declares `Index("ix_profiles_user_id", "user_id")`, so `create_all` emits the same `CREATE INDEX` twice. It is pre-existing and unrelated to #28, and fixing it would have put a model change inside a rate-limiting PR. So I changed my own test plan instead of my scope: I drove the real `api.main:app` through Starlette's `TestClient` — which skips the broken startup hook — against the real Redis container, flooding `GET /reviews`, an endpoint that returns `401` from the auth dependency without touching the database, so a `429` is the only thing that can change the status code. Past 120 requests it began refusing:

```
[warning] rate_limit_exceeded  identifier=ip:198.51.100.7 limit=120 request_id=090e5094-...
[warning] rate_limit_rejected  identifier=ip:198.51.100.7 limit=120 method=GET path=/reviews
                               request_id=090e5094-... scope=ip

6. CORS header on 429 response : http://localhost:5173
```

The second line is the one I cared about: it confirms the ordering decision above actually holds at runtime, not just in my reasoning.

**A change I made on the tool side.** My first `issue-scout` install was broken — the skill files sat one folder too deep, in `~/.claude/skills/issue-select/skill/`, so Claude Code never loaded the skill at all. I rebuilt it flat as `~/.claude/skills/issue-scout/`. Later, before opening the PR, I ran my `pr-precheck` rubric against the implementation, plan, and draft description; it held the package until I filled in real command output in place of claims, and accepted it after I updated the evidence. That rubric is the reason the PR's testing section quotes results rather than asserting passes.

**A result that landed: all five CI jobs are green on PR #285.**

```
$ gh pr checks 285
frontend          pass  15s
lint              pass   9s
test-integration  pass  1m16s
test-unit         pass  51s
typecheck         pass  1m0s
```

That mattered beyond a green tick. My local venv is Python 3.14.5 while the repo targets 3.11, and I had recorded in the PR that my local passes were "suggestive rather than decisive" because CI runs 3.11. CI running green is what actually closed that gap. Locally the same change gave `24 passed` on the new middleware tests and `399 passed, 53 xfailed` on the full unit suite — the 53 intact `xfail` markers being the evidence that I added a caller rather than quietly altering a seeded bug.

**What has not happened:** no maintainer has reviewed or commented on #285, and it is not merged. It has been open since 2026-11-16.

## Limitations and what's next

**The PR's diff does not match its own description.** The description claims six files and 639 insertions. The real PR is eight files, 839 insertions, one deletion:

```
$ gh pr view 285 --json files
plan.md: +90 -0
pr_draft.md: +70 -0
docs/API.md: +91 -1     # 40 of these belong to issue #37
```

I cut the branch from `fix/37-api-request-body-schema` instead of from `main`, so my previous issue's artifacts rode along. The fix is one command — `git rebase --onto main 69686d7 add/28-rate-limiting-per-user` — and it is the first thing I will do. The PR title has the same root cause: GitHub derived "Add/28 rate limiting per user" from a branch name that uses a type `docs/CONTRIBUTING.md` does not list and misnames an issue that is about per-IP limiting.

**This is also a limitation of my own tool, and the more interesting one.** My `pr-precheck` rubric has a `diff_reviewability` check whose pass condition is explicitly that the diff holds no "accidentally committed local files." It accepted this package anyway, because I pointed it at my local working tree rather than at what was actually pushed. The rubric was right and the input was wrong. Next: the precheck should read the PR's real file list (`gh pr view --json files`) and compare it against the plan's stated file table, so a mismatch between planned scope and pushed diff fails mechanically instead of depending on me to notice.

**The change has a design hole I chose not to close.** I read `request.client.host` and deliberately ignore `X-Forwarded-For`, because trusting that header without a trusted-proxy allowlist would let any caller mint a fresh identity per request and bypass the limit entirely. But if PathReview is deployed behind a proxy, every user collapses into one bucket and the limit becomes useless. I asked the maintainers which it is and have no answer yet. Next, pending that answer: a `rate_limit_trusted_proxies` setting, so the header is honoured only from known hops.

**Smaller limits I know about.** The limiter's check is read-then-write (`zcard` then `zadd`), so concurrent workers can admit slightly over the limit — fixing it means rewriting `check_rate_limit` as a Lua script, which is its own issue. A request refused by the user bucket has already spent one IP slot, because the IP check runs first. Several users behind one NAT share the IP ceiling, which is inherent to "per IP in addition to per user" and only mitigated by giving the IP budget 2× headroom. The defaults of 60/user/minute and 120/IP/minute are reasonable but not derived from any traffic data. And `frontend/src/services/api.ts` still funnels every non-OK response into a generic `Error`, so a throttled user sees a bare string rather than a "slow down" state — out of scope here, and the frontend issue I would open next.

**The startup bug I found and did not fix.** The duplicate index in `core/models/profile.py` stops `uvicorn api.main:app` from booting on a clean database. I reported it in the PR rather than folding it into this change. If it is not already tracked, filing it is the next thing I would do in this repo.
