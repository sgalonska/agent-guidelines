# Web App Guidelines

Applies on top of [`AGENTS.md`](AGENTS.md) to repositories with a UI or an HTTP surface. Repository-local instructions override these defaults. Where a section does not apply to the stack, say so rather than silently skipping it.

## 1. Complete User Experiences

For each user-visible flow, handle where relevant:

* Initial, loading, empty, success, partial, stale, error, offline, and retry states.
* Pending and duplicate submissions.
* Permission-denied and expired-session states.
* Slow responses, cancellation, and recovery after interruption.
* Responsive layouts, touch targets, long content, and narrow viewports.
* Refresh, deep links, back and forward navigation, shareable URLs.

Preserve user input after recoverable failures. Give destructive actions clear consequences, confirmation where warranted, and a recovery path where practical. Do not rely on color, animation, hover, or client-side validation alone to communicate meaning.

Reuse the existing design system, tokens, and components. Do not hand-roll a control the system already provides.

## 2. Accessibility

Target WCAG 2.2 AA unless the repository sets a different bar.

* Use semantic HTML before ARIA. Every control has an accessible name.
* All functionality works by keyboard with a visible focus indicator and a logical order.
* Manage focus on route changes, dialogs, and dynamic content. Trap and restore focus in modals.
* Announce async status and errors to assistive technology.
* Meet contrast requirements. Support zoom to 200% and reflow at narrow widths.
* Honor `prefers-reduced-motion` and avoid motion-only meaning.
* Associate labels, errors, and help text with form fields.

Automated checks (axe or equivalent) do not replace a manual keyboard pass. State which you did.

## 3. Frontend State and Boundaries

* Keep one source of truth for each piece of state. Do not duplicate server data into unrelated client state.
* Define server and client rendering boundaries deliberately. Prevent hydration mismatches, layout shift, and stale updates.
* Cancel or ignore obsolete requests when a newer action supersedes them.
* Use error boundaries so one failing component does not blank the page.
* Use progressive enhancement where the stack supports it.
* Never expose server-only configuration, secrets, internal identifiers, or privileged logic in browser bundles.

## 4. API Design

* Define request, response, error, and auth schemas, and keep generated clients and docs consistent with the implementation.
* Use HTTP methods, status codes, redirects, and cache headers consistently.
* Validate content type, size, shape, ranges, and unknown fields at the boundary.
* Make retryable mutations idempotent or support an idempotency key.
* Paginate with stable ordering and explicit limits.
* Define timeouts, cancellation, concurrency control, and rate limits.
* Do not leak resource existence across authorization boundaries (prefer 404 where the caller may not know).
* Document versioning and deprecation. Prefer additive change.

## 5. Authentication and Authorization

* Use the framework's or an established library's mechanisms for auth, password hashing, sessions, and crypto. Do not write your own.
* Enforce authorization server-side on every protected action, against the requested object, tenant, and role. Deny by default.
* Add a tenant-isolation test for every new query or endpoint that touches tenant data.
* Protect cookie-authenticated mutations against CSRF. Use `HttpOnly`, `Secure`, and an explicit `SameSite` on session cookies.
* Rotate session identifiers on login and privilege change. Support logout, expiry, revocation, and invalidation on credential change.
* Prevent account enumeration in login and recovery flows.
* Validate redirect targets against an allowlist.
* Rate-limit and monitor sensitive endpoints.
* No long-lived bearer credentials in URLs. No tokens in browser-accessible storage without a documented threat model.
* Audit-log privileged and security-relevant actions with actor, target, and time.

## 6. Web Security Defaults

* Set a restrictive Content-Security-Policy, HSTS, `X-Content-Type-Options`, and a sensible `Referrer-Policy`. Keep CORS to an explicit allowlist, never reflected origins with credentials.
* Escape output by context. Treat user-supplied HTML, Markdown, and URLs as untrusted; sanitize with a maintained library.
* File uploads: enforce type and size limits by content, not extension; scan where required; store outside the web root with generated names; serve with safe headers.
* Guard server-side fetches of user-supplied URLs against SSRF.
* Pin and review third-party scripts. Use subresource integrity where scripts load from a CDN.

## 7. Databases

Every schema change ships as a migration with a compatibility plan.

* Expand, migrate, then contract. Old and new app versions must both work during rollout.
* Assess locks, table rewrites, index builds, and transaction length.
* Backfill in bounded, observable, resumable batches.
* Add constraints only after validating existing data.
* State defaults, nullability, ownership, and deletion behavior explicitly.
* Test migrations against representative existing data, on both fresh and upgrade paths.
* Never combine irreversible data deletion with the release that first stops reading that data.
* Check that a restore works, not only that a backup exists, for any new store.

Store timestamps in UTC. Represent money as integer minor units or decimals, never floats. Use opaque identifiers for anything user-visible.

## 8. Async Work and Webhooks

* Assume delivery can be duplicated, delayed, reordered, or lost. Make consumers idempotent.
* Bound retries with backoff and jitter. Separate retryable from permanent failures.
* Define timeout, dead-letter, replay, and operator-recovery behavior.
* Propagate correlation IDs.
* Authenticate webhooks and defend against replay.
* Commit required durable state before acknowledging work.
* Document ordering and consistency guarantees; do not imply stronger ones.

## 9. Configuration and Secrets

* Keep configuration explicit, typed, and validated at startup. Fail fast on missing or malformed required values.
* Distinguish build-time, deploy-time, and runtime values.
* Document new variables in the example file with safe placeholders.
* Keep secrets out of source control, logs, errors, fixtures, URLs, and browser bundles. Use the repository's secret manager.
* Scope credentials to the minimum permission and lifetime.
* No silent fallback to insecure defaults in production.
* Tests must not depend on production credentials or services.

## 10. Observability

* Structured logs with stable event names and severity. Redact secrets and personal data.
* Propagate request, trace, and job IDs across services and queues.
* Record latency, throughput, error, and saturation metrics for new critical paths.
* Provide health and readiness signals.
* Alerts are actionable and tied to user impact. Expected user errors do not page anyone.

## 11. CI and Deployment

* Locked dependencies and reproducible build commands.
* CI runs format, lint, type, test, build, migration, and security checks. Never make a gate pass by skipping work or lowering coverage.
* Build one immutable artifact and promote it across environments.
* Sequence app, migration, worker, and cache changes safely. Define rollback or forward recovery before high-risk releases.
* Environment differences live in configuration, not branches.
* Feature flags have an owner, a safe default, an observable rollout, and a removal condition.
* Pin third-party CI actions. Verify health and one critical path after deploy.

## 12. Performance Budgets

Use the repository's budgets. If none exist, take a baseline before claiming an improvement or regression.

* Server: latency, query count, connection-pool use.
* Browser: JavaScript, CSS, image, and font weight; request count; layout stability; interaction latency.
* Background: queue depth and job duration.
* Caching: hit rate and invalidation correctness.

Measure with representative data, devices, and networks on a production-like build. Treat material regressions as failures unless explicitly accepted. Check the bundle-size impact of every new dependency.

## 13. Verify in a Real Browser

For user-facing changes:

* Run the app (production-like build where feasible) and drive the changed flow end to end in a real browser, not only a DOM test.
* Check narrow and wide viewports, keyboard operation, zoom, and reduced motion.
* Verify refresh, deep links, back and forward, and expired sessions.
* Inspect console errors, failed requests, redirects, cookies, and caching.
* Exercise loading, empty, error, offline, and slow-network states.
* Capture a screenshot when layout or interaction correctness matters.

State which browsers, viewports, and flows were verified. If you could not run the UI, say "not verified visually". Do not claim cross-browser support from one browser.

Cover each critical user journey with an end-to-end test; keep E2E tests few, stable, and independent.

## 14. SEO (public pages only)

* Unique title, description, headings, and canonical URL per page.
* Correct status codes for success, redirects, missing content, and errors. Stable URLs, or permanent redirects when they change.
* `robots` directives and sitemaps. Important content and links available to crawlers without JavaScript where the stack allows.
* Structured data only when it matches visible content.
* Keep private, placeholder, and duplicate pages out of the index. Verify social preview metadata.

Do not apply this to authenticated or intentionally non-indexable surfaces.

## 15. Internationalization

When localization exists or is reasonably expected:

* Keep user-facing text out of business logic and never concatenate translated fragments.
* Support plurals, grammatical variation, and text expansion.
* Format dates, numbers, currencies, names, and addresses per locale. Display times in the relevant timezone.
* Define locale detection, fallback, URLs, and persistence. Support right-to-left when in scope.
* No text in images.

Test one expanded translation and one non-default locale.

## 16. Privacy by Design

Before collecting or exposing personal data, identify purpose, owner, users, retention, and deletion path.

* Collect and retain only what the purpose requires.
* Restrict access by role, tenant, environment, and need.
* Define export, correction, deletion, and account-closure behavior, including logs, analytics, caches, search indexes, replicas, and backups.
* Record consent where required and load no optional tracking before it.
* Review third-party processors and cross-border transfers.
* Use synthetic or protected data outside production.

Do not claim legal compliance from technical checks. Escalate jurisdiction-specific questions to qualified privacy or legal review.

## 17. Web Completion Checklist

In addition to the core Definition of Done, where relevant:

* All UX states implemented; accessibility checked by keyboard.
* Authorization verified server-side, with a tenant-isolation test.
* API and data compatibility assessed; migration and recovery path tested.
* Observability covers the new failure modes.
* Configuration and operational docs current.
* Performance budgets preserved.
* Browsers, viewports, and flows exercised and named in the report.
* Privacy, SEO, and localization impacts considered.
* Deployment and rollback understood.

The final report separates passed, failed, and not-run checks.
