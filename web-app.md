# Web App Guidelines

On top of [`AGENTS.md`](AGENTS.md), for repos with a UI or HTTP surface. If a section does not fit the stack, say so; do not skip it silently.

## 1. Every State, Not Just the Happy Path

* Handle loading, empty, partial, stale, error, offline, retry, permission-denied, and expired-session states.
* Handle double submits, slow responses, cancellation, and interruption.
* Narrow viewports, long content, touch targets. Refresh, deep links, back/forward, and shareable URLs all work.
* Keep user input after recoverable failures. Destructive actions spell out consequences, confirm when warranted, and offer recovery where practical.
* Never carry meaning by color, hover, animation, or client-side validation alone.
* Use the existing design system. Do not hand-roll a control it already has.

## 2. Accessibility

WCAG 2.2 AA unless the repo says otherwise.

* Semantic HTML before ARIA. Every control has an accessible name; labels, errors, and help text bind to their fields.
* Everything works by keyboard, with visible focus and logical order. Manage focus on route changes and dynamic content; trap and restore it in modals.
* Announce async status and errors. Meet contrast, support 200% zoom and reflow, honor `prefers-reduced-motion`.
* axe is not a keyboard pass. Say which you did.

## 3. Frontend State

* One source of truth per piece of state. Do not mirror server data into random client state.
* Draw server/client rendering boundaries on purpose: no hydration mismatches, layout shift, or stale updates. Drop superseded requests.
* Error boundaries so one broken component does not blank the page. Progressive enhancement where the stack supports it.
* Nothing server-only (config, secrets, internal IDs, privileged logic) ships in a browser bundle.

## 4. APIs

* Schemas for request, response, error, and auth. Generated clients and docs match the implementation.
* Consistent methods, status codes, and cache headers. Validate type, size, shape, ranges, and unknown fields at the edge.
* Retryable mutations are idempotent or take an idempotency key. Paginate with stable order and explicit limits.
* Define timeouts, cancellation, concurrency control, and rate limits.
* Do not leak resource existence across authz boundaries (404 over 403 where appropriate). Version and deprecate additively.

## 5. Auth

* Use the framework or an established library for auth, hashing, sessions, and crypto. Never roll your own.
* Authorize server-side on every protected action, against object, tenant, and role. Deny by default.
* **Every new query or endpoint touching tenant data gets a tenant-isolation test.**
* CSRF protection on cookie-auth mutations. Session cookies are `HttpOnly`, `Secure`, explicit `SameSite`. Rotate session IDs on login and privilege change; support logout, expiry, and revocation.
* No account enumeration. Allowlist redirect targets. Rate-limit sensitive endpoints.
* No long-lived credentials in URLs, no tokens in browser storage without a documented threat model.
* Audit-log privileged actions: actor, target, time.

## 6. Web Security Defaults

* Restrictive CSP, HSTS, `X-Content-Type-Options`, sane `Referrer-Policy`. CORS is an explicit allowlist, never a reflected origin with credentials.
* Escape output by context. User HTML, Markdown, and URLs are untrusted; sanitize with a maintained library.
* Uploads: check type and size by content (not extension), scan where required, store outside the web root under generated names, serve with safe headers.
* Guard server-side fetches of user URLs against SSRF. Pin third-party scripts; use SRI for CDN loads.

## 7. Databases

Every schema change is a migration with a compatibility plan.

* Expand, migrate, contract. Old and new app versions both work mid-rollout.
* Check locks, rewrites, index builds, and transaction length. Backfill in bounded, resumable batches.
* Validate existing data before adding constraints. Be explicit about defaults, nullability, ownership, and deletion behavior.
* Test on fresh and upgrade paths with realistic data.
* Never delete data in the same release that stops reading it. For a new store, prove restore, not just backup.
* UTC timestamps. Money as integer minor units or decimals, never floats. Opaque user-visible IDs.

## 8. Async Work and Webhooks

* Delivery will be duplicated, late, reordered, or lost. Consumers are idempotent.
* Bounded retries with backoff and jitter; separate retryable from permanent failures. Define dead-letter, replay, and operator recovery.
* Commit durable state before acking. Propagate correlation IDs. Authenticate webhooks and block replays.
* Do not promise ordering or consistency you do not have.

## 9. Config and Observability

* Config is typed and validated at startup; missing or malformed required values crash immediately. No silent insecure fallback in production.
* Keep build-time, deploy-time, and runtime values distinct. Secrets come from the repo's secret manager and never reach fixtures, URLs, errors, or bundles.
* New variables go in the example file with safe placeholders. Tests never need production credentials.
* Structured logs with stable event names, secrets and personal data redacted. Propagate request, trace, and job IDs.
* New critical paths get latency, error, and saturation metrics plus health and readiness signals. Alerts are actionable; user errors page nobody.

## 10. CI and Deployment

* Locked deps, reproducible builds, one immutable artifact promoted across environments.
* CI runs format, lint, types, tests, build, migrations, and security checks. Never make a gate pass by skipping work.
* Sequence app, migration, worker, and cache changes safely. Know the rollback or forward fix before a risky release.
* Environment differences live in config, not branches. Feature flags have an owner, a safe default, and a removal condition.
* Pin third-party CI actions. After deploy, check health and one critical path.

## 11. Performance

Use the repo's budgets; without them, baseline before claiming anything. Watch server latency, query count, and pool use; JS, CSS, image, and font weight, request count, layout stability, and interaction latency; queue depth and job duration; cache hit rate and invalidation. Measure on a production-like build with realistic data, devices, and networks. Material regressions fail unless explicitly accepted. Every new dependency gets a bundle-size check.

## 12. Verify in a Real Browser

User-facing change? Run the app and drive the changed flow end to end in a real browser, not just a DOM test.

* Narrow and wide viewports, keyboard, zoom, reduced motion.
* Refresh, deep links, back/forward, expired session.
* Console errors, failed requests, redirects, cookies, caching.
* Loading, empty, error, offline, and slow-network states. Screenshot when layout matters.

Name the browsers, viewports, and flows you checked. Could not run it? Say "not verified visually". One browser is not cross-browser. Critical journeys get a few stable, independent E2E tests.

## 13. SEO and Locales

**SEO (public pages only, never authenticated surfaces):** unique title, description, headings, and canonical URL; correct status codes; stable URLs or permanent redirects; robots and sitemaps; key content and links crawlable without JS where possible; structured data only if it matches visible content; private, placeholder, and duplicate pages kept out of the index; social preview metadata verified.

**i18n (when localization exists or is expected):** no concatenated translations, no text in images, no copy in business logic. Handle plurals, grammatical variation, and text expansion. Format dates, numbers, currencies, names, and addresses per locale; show times in the relevant timezone. Define locale detection, fallback, URLs, and persistence; RTL if in scope. Test one long translation and one non-default locale.

## 14. Privacy by Design

Before collecting or exposing personal data, know its purpose, owner, users, retention, and deletion path.

* Collect and keep only what the purpose needs. Restrict access by role, tenant, environment, and need.
* Define export, correction, deletion, and account closure, reaching logs, analytics, caches, search indexes, replicas, and backups.
* Record consent where required. No optional tracking before it.
* Review third-party processors and cross-border transfers. Synthetic or protected data outside production.
* Never claim legal compliance from a technical check. Escalate jurisdiction questions to qualified privacy or legal review.

## 15. Web Checklist

On top of the core Definition of Done, where relevant:

* All UX states built; keyboard pass done.
* Server-side authz with a tenant-isolation test.
* API and data compatibility assessed; migration and recovery path tested.
* Observability covers new failure modes; config docs current.
* Performance budgets held.
* Browsers, viewports, and flows named in the report.
* Privacy, SEO, i18n, and rollback considered.

Report passed, failed, and not-run checks separately.
