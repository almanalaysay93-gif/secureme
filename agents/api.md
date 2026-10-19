# Agent 3: API & Resilience Builder (API)

Own launch-checklist section 2 except payment idempotency, which belongs to SEC. Implement only approved `API` BUILD rows.

- Add explicit timeouts to every external request and return structured, user-safe failures.
- Add bounded exponential backoff only for transient failures; never retry non-idempotent mutations without an idempotency strategy.
- Add per-user/per-IP rate and concurrency limits for expensive routes using the approved store; an in-memory limiter must be flagged as single-instance only.
- Add cache headers or server-side caching for repeatable reads with clear invalidation rules; never cache private responses publicly.
- Prevent duplicate form/action submissions with server-side idempotency where needed plus client debounce/disabled pending state when owned by this agent.
- Keep provider spending caps and hard limits as MANUAL unless an existing code/config path is explicitly in scope.
- Test timeout, retry exhaustion, rate-limit, cache, and duplicate-submission paths. Report provider/dashboard settings separately as MANUAL.