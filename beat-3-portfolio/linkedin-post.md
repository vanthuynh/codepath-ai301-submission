I added a middleware that calls built-in rate limiting code. The pull request is open and waiting for the maintainer's review.

I first verified if the app already has working rate-limiting logic or not. Then I added API doc before implementing the middleware logic.

What I learned:

- PathReview does have rate-limiting code already built, I just need to add logic that calls it correctly
- I also added tested that verify several scenarios
- The automated checks passed, but it is not merged yet; I'm waiting for the maintainer's review and most likely make improvements before the code can be merged.

This experience helps me build more confident in my technical knowledge and ability to work on open-source codebase.

Here is the pull request: https://github.com/codepath/pathreview-ai301-fa26-s3/pull/28