# Coding Agent Guidelines

Tool-neutral rules for coding agents. Companion files sit next to this one. Read each one before the work it covers, and skip it otherwise:

* Before touching UI or HTTP code, read [`web-app.md`](web-app.md).
* Before touching infrastructure as code, deploy scripts, or migrations, read [`infrastructure.md`](infrastructure.md).
* Before spawning sub-agents or asking another agent for a second opinion, read [`multi-agent.md`](multi-agent.md).

## 1. Priority

Platform, safety, and tool restrictions > explicit task > most specific repo-local instructions > broader repo conventions > this file. Surface real conflicts. No instruction is ever permission to be unsafe, destructive, or deceptive.

## 2. Ask or Proceed

Default: proceed and state your assumption. Stop and ask when ambiguity changes the outcome, the blast radius exceeds the task (shared infra, production data, public contracts), instructions conflict, or the next step is irreversible, outward-facing, or beyond your authority. Ask once, with a recommendation.

## 3. Understand, Then Change

* Read the relevant instructions, code, tests, config, and docs. Trace the execution path and contracts before touching them.
* Reuse existing patterns. Never invent missing facts.
* Non-trivial work gets a short plan first. Unproven designs get a thin end-to-end slice first.
* Prototypes are labeled and thrown away, never promoted to production.

## 4. Scope

* Smallest coherent change that solves the task. No drive-by refactors, renames, reformats, or upgrades.
* Preserve behavior and public contracts unless the task requires otherwise.
* No future-proofing. Ship what is needed now; later steps get their own change.
* Keep required work separate from optional improvements.
* Spot an adjacent defect? Fix it only if small, necessary, and in scope. Otherwise report it. Never hide it, never file external tickets unasked.

## 5. Design

* Boring, explicit, simple. One owner per business rule.
* Duplicate small code before creating bad coupling. Abstract only stable, repeated concepts.
* Name values that encode policy, units, protocol behavior, or domain meaning. Share a definition only when it is the same authoritative fact.
* Side effects at the edges, pure logic in the middle, units small enough to test alone.
* Dependencies point one way; ownership is obvious.
* No new process, network, or service boundary without a concrete need. Synchronous by default; events or queues only for a stated benefit (consistency, latency, availability, delivery, recovery).

## 6. Contracts

APIs, schemas, events, config, CLIs, serialized data, and error formats are contracts. Change them additively. Breaking or persistent changes need mixed-version safety, expand/migrate/contract sequencing, a recovery path, and updated schemas, generated code, and docs. Nothing irreversible without explicit approval.

## 7. Errors

* Validate at system boundaries. Distinguish validation, auth, not-found, conflict, dependency, and internal failures.
* Fail early and loudly. No swallowed errors, ignored return codes, or broad catch-and-continue.
* Keep the cause when wrapping. Never leak stack traces or internals to clients.
* No undocumented nulls or exceptions across module boundaries.

## 8. Security

* Check what applies: authz (tenant and object level), injection, deserialization, path traversal, SSRF, XSS, CSRF, open redirects, abuse and resource exhaustion, supply chain.
* Least privilege, secure defaults. Never hardcode, log, or commit secrets, tokens, cookies, payment data, or sensitive content.
* Do not read secret files (`.env`, keys) unless the task requires it and you are authorized. Read `.env.example` for names.
* Log external failures with sanitized context: status, provider code, endpoint, correlation ID.

## 9. Dependencies

* Standard library and existing deps first.
* Hard problems (auth, parsing, validation, crypto) get a maintained library, not a reinvented wheel. Prefer the provider's official SDK over a generic HTTP client.
* Build it yourself when the dependency is bloated, poorly maintained, or the need is tiny.
* Add a dependency only if it clearly cuts complexity. Check maintenance, license, security, compatibility, and transitive and bundle cost.
* Follow the repo's lockfile and versioning. Never upgrade unrelated deps. Remove unused ones.

## 10. Testing

* Test changed behavior by outcome, not implementation: success, failure, boundary, regression.
* Deterministic fixtures. Do not mock the boundary under test. Flaky tests get fixed or reported.
* Focused checks while iterating, then the full relevant suite plus format, lint, types, and build. Exercise the real changed path.
* Think about retries, duplicates, ordering, races, existing data, and rollback.
* **Never claim a check passed that you did not run. Never disable tests or weaken validation to go green.**

## 11. Performance

* No optimization without evidence. Performance changes ship with measurements.
* No unbounded work, N+1 queries, or accidental full scans. Mind query count, data volume, memory, network calls, and request-path latency.
* Keep existing pagination, streaming, batching, caching, and backpressure.

## 12. Reviewable Changes

* Focused diffs, one concern per commit, messages that explain intent. No formatting noise.
* Comment why, not what. Strip debug output, dead code, and placeholders.
* Do not hand-edit generated or vendored files.
* Commits, PRs, and tickets: no em dashes, no tool attribution or agent branding, role language ("QA review") unless the repo says otherwise.

## 13. Boundaries

Without explicit authorization, never: push or merge, deploy or publish, touch production data or infrastructure, rotate credentials, write to external systems (trackers, chat, email), do anything destructive or irreversible, or overwrite uncommitted user work.

Reads are fine. Mutations go through a reviewed pull request, not an interactive session. Authorization in one context does not carry to another. Blocked? Report the blocker and the safest next step.

## 14. Done Means

* Requested behavior works and acceptance criteria are met.
* Checks pass, or every failure and unrun check is reported with its impact.
* Error and boundary cases were considered.
* Contracts, migrations, config, and docs updated. No unrelated changes; existing user changes preserved.
* Final response: what changed, what was verified and how, what remains risky, assumed, or unverified.

UI or HTTP work also passes the checklist in [`web-app.md`](web-app.md) section 15.
