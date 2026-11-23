I contributed to PathReview app, an open-source AI-powered portfolio review assistant that helps early-career developers strengthen their professional portfolios. The project already have working code for limiting requests one visitor can make, but nothing in the app ever ran it; as a result, the app doesn't prevent someone from hammering the login page with thousands of password guesses, or from running up the bill on a feature that costs money every time it is called.

Issue #28 asked for that gap to be closed. My fix connects the existing limiting code to the app's request path, so every request is counted and turned away once it goes over the limit — by IP address for everyone, and by account as well for anyone signed in.

What I learned:

- Prove the problem before fixing it. "Requests are never turned away" can mean the limiting code is broken, or that it is never run — two problems with completely different fixes. I ran the code on its own and watched it correctly refuse the 4th request when the limit was 3. That told me the code was fine and only the connection was missing, which decided the entire approach.
- Small ordering decisions reach the user. Where I placed the new code in the chain of steps a request passes through decided whether a blocked user sees "you're going too fast" or an unrelated, confusing browser error. I verified the right one by watching a real blocked response come back.
- Done is not merged. All five automated checks pass and I added 24 tests, but the pull request is still open and waiting for a maintainer's review. I expect to make changes before it is merged — starting with a question I raised myself about how the app is deployed, which could change part of the design.

Working in a codebase I did not write, on a problem I had to prove before I could solve it, has made me more confident about contributing to open source.

Pull request #285 (for issue #28): https://github.com/codepath/pathreview-ai301-fa26-s3/pull/285
