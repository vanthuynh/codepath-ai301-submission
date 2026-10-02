# Evidence guide: where proof lives in a reproduction package

<!--
HEADS UP: This is the map the grader uses. For every check in your rubric, this guide defines exactly Where it lives for the proof and WHAT a passing grade actually looks like in practice.

If your rubric asks for something that this guide can't locate, nobody else will be able to grade it either. Write the map you wish you had when you were grading.
-->

## Environment

**Where it lives**
In an eval bundle: Check the `Environment:` block at the top of the repro report, and cross-reference it with the `repo-facts` block (which tells you what OS/version the issue is actually targeting). In live mode: Look at the draft's environment line and check it against what the repo specifically asks for in their `README` or `CONTRIBUTING.md`, plus what the original reporter stated in the issue thread.

**What good look like**
The report explicitly lists the tool version, the OS, and the exact commit or tag. The version either perfectly matches what the issue targeted, or the author explicitly points out the difference (e.g., "The issue reported this on 1.18.0, but I am testing on 1.20.0"). Dropping a version number by itself without the OS or commit context is an automatic fail.

## Steps

**Where it lives**
In an eval bundle: Look at the `Steps` section of the repro report and compare it against the command the original issue reporter used. In live mode: Compare the draft's steps against the standard build/run process documented in the repo.

**What good look like**
The steps have to start from a rock-solid, reproducible baseline (like a fresh clone or a specific git checkout) and end exactly at the command that triggers the bug. They cannot rely on magic local state, global installs the author forgot to mention, or unmentioned files. A total stranger should be able to follow the steps blindly and hit the exact same state.

If a step requires a helper file (like an `env.yml` or `repro.mjs`), they don't have to dump the entire file as a code block to pass. It passes if: (a) the vital contents are shown inline, (b) it's described precisely enough that someone else could easily recreate it, or (c) it explicitly points to code blocks already provided in the original issue. Vague hand-waving like "set up a typical project" or "used my standard test file" is a hard fail because nobody else can replicate that.

## Behavior shown

**Where it lives**
In an eval bundle: Check the attached logs or output blocks in the repro report against the specific symptom the original issue reported (e.g., a specific error message, panic, or exit code). In live mode: Vet whatever artifact they attach—a terminal log, a screenshot, a linked CI run—against the failure description in the issue thread.

**What good look like**
The artifact must be the literal, unedited output of the command run in the steps. The failure signature needs to be a 1:1 match with the issue (same exact error type, same exit code family). If the artifact shows things crashing for a completely different reason (like a misconfigured setup), that's an adjacent bug, not proof of the reported issue. That fails.

## Honesty

**Where it lives**
In both eval and live modes: Compare the report's opening verdict or claim directly against the actual evidence provided in the Environment, Steps, and Behavior sections below it.

**What good look like**
The conclusion writes checks the evidence can actually cash. A solid, well-documented "I could not reproduce this" is a completely valid, highly useful report and gets a pass. Claiming you successfully reproduced the bug without the logs to back it up, or acting way more confident than a messy stack trace allows, is a hard fail regardless of how pretty the rest of the report is formatted.

## Comms

**Where it lives**
In an eval bundle: Read the draft comment meant for the issue thread and check it against the issue body and `repo-facts` block (checking for issue templates and AI disclosure rules). In live mode: Vet the actual comment about to be posted against the live issue thread and the repo's actual `CONTRIBUTING.md` or `CODE_OF_CONDUCT` for stated norms.

**What good look like**
The comment actually names the specific version and exact behavior instead of just saying "I'll fix this bug." It promises an investigation or a follow-up report, not a guaranteed merged PR by Friday. If the repo demands AI-use disclosure or first-time contributor tags, it includes them. Fluff like "Great project +1!" or generic boilerplate fails the vibe check—this check grades how well the communication fits the reality of the repo, not just the technical proof.
