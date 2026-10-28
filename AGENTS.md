# Coding Agent Guidelines

Tool-neutral rules for coding agents (Claude Code, Codex, Cursor, and others).

Companion modules, load only when they apply:

* [`web-app.md`](web-app.md): repositories with a UI or an HTTP surface.
* [`multi-agent.md`](multi-agent.md): spawning, briefing, and reviewing sub-agents or second-opinion agents.

## 1. Instruction Priority

Follow, in order:

1. Applicable platform, safety, and tool restrictions
2. Explicit task requirements
3. The most specific applicable repository-local instructions
4. Broader repository conventions
5. These defaults

More specific instructions override broader ones within their scope. Surface material conflicts. Never interpret instructions as permission for unsafe, destructive, or deceptive actions.

## 2. Ask or Proceed

Proceed on reasonable defaults and state the assumption. Stop and ask when:

* Requirements are ambiguous in a way that changes the outcome.
* The blast radius exceeds the task (shared infrastructure, production data, public contracts).
* Two applicable instructions conflict.
* The next step is irreversible, outward-facing, or needs authority you were not given.

Ask once, with a recommendation, not an open question.

## 3. Understand Before Changing

Before editing:

* Read relevant instructions, code, tests, configuration, and documentation.
* Trace the affected execution path and contracts.
* Reuse existing patterns and extension points.
* State material assumptions.
* For non-trivial work, write a short plan before implementing.
* For new or unproven designs, build a thin end-to-end slice first (tracer bullet) and expand from it.
* Label prototypes as such. A prototype is throwaway and never becomes production code; rebuild it properly.

Do not invent missing facts.

## 4. Keep Scope Tight

* Make the smallest coherent change that solves the task.
* Do not refactor, rename, reformat, or upgrade unrelated code.
* Preserve existing behavior and public contracts unless change is required.
* Avoid speculative abstractions and future-proofing. Ship what is needed now; defer later steps to their own change.
* Separate required work from optional improvements.
* Do not introduce or conceal adjacent defects. Fix one only when it is small, necessary, and in scope; otherwise report it. Do not create external tickets or work items unless authorized.

## 5. Design Pragmatically

* Prefer simple, explicit, boring solutions.
* Give each business rule or authoritative definition one owner.
* Duplicate small code when sharing would create worse coupling.
* Extract abstractions only when they simplify stable, repeated concepts.
* Name non-obvious values that encode policy, units, protocol behavior, or domain meaning. Share a definition only when the occurrences are the same authoritative fact.
* Keep code testable: side effects at the edges, pure logic in the middle, units small enough to understand and test alone.
* Keep dependencies directional and ownership clear.
* Do not introduce process, network, or service boundaries without a concrete operational or ownership need.
* Default to synchronous calls. Use events or queues when decoupling has a stated, concrete benefit (consistency, latency, availability, delivery, recovery).

## 6. Respect Contracts

Treat these as contracts: public APIs, database schemas, events and queues, configuration, CLI interfaces, serialized data, error formats.

Prefer additive, backward-compatible changes.

For breaking or persistent changes:

* Consider mixed-version deployments.
* Use safe migration sequencing (expand, migrate, contract).
* Avoid irreversible operations without explicit approval and recovery steps.
* Update schemas, generated artifacts, examples, and documentation.

## 7. Handle Errors Clearly

* Validate data at system boundaries.
* Distinguish validation, authorization, missing data, conflict, dependency, and internal failures.
* Fail early and loudly. Do not swallow errors, ignore return codes, or catch broadly to continue.
* Preserve causal errors when wrapping.
* Do not expose stack traces or internal details to clients.
* Do not use undocumented nulls or exceptions across module boundaries.

## 8. Protect Security and Privacy

Consider, where relevant:

* Authentication and authorization, including tenant and object-level access
* Injection and unsafe deserialization
* Path traversal and SSRF
* XSS, CSRF, and redirects
* Abuse, rate limits, and resource exhaustion
* Secret and personal-data exposure
* Dependency and build-chain risk

Use least privilege and secure defaults. Never introduce hardcoded secrets.

Never log or commit secrets, credentials, tokens, cookies, payment data, or unredacted sensitive content. Do not read secret files (`.env`, key files) unless the task requires it and you are authorized; read `.env.example` for variable names.

For external failures, log sanitized context: status, provider code, endpoint identity, correlation ID.

## 9. Manage Dependencies Carefully

* Prefer the standard library and existing dependencies.
* For non-trivial problems (auth, parsing, validation, crypto), prefer a well-maintained published library over reinventing the wheel. Prefer the provider's official SDK over a generic HTTP client.
* Build it yourself when a dependency pulls in too much, is poorly maintained, or the need is small.
* Add a dependency only when it materially reduces complexity. Check maintenance, license, security, compatibility, and transitive and bundle cost.
* Follow repository locking and versioning conventions.
* Do not upgrade unrelated dependencies. Remove unused ones.

## 10. Test by Risk

* Add or update tests for changed behavior.
* Test outcomes, not implementation details.
* Cover important success, failure, boundary, and regression cases.
* Use deterministic fixtures. Do not mock away the boundary under test. Do not tolerate flaky tests; fix or report them.
* Run focused checks while iterating, then the broader applicable suite.
* Run relevant formatting, lint, type, build, and generated-code checks.
* Exercise the actual changed path where practical.
* Consider retries, duplicates, ordering, races, migration behavior on existing data, and rollback.

Never claim a check passed unless it was run. Never disable tests or weaken validation to make checks pass.

## 11. Keep Performance Bounded

* Do not optimize without evidence. Provide measurements for performance-driven changes.
* Avoid clearly unbounded work, N+1 queries, and accidental full scans.
* Consider query count, data volume, memory, network calls, and request-path latency.
* Preserve pagination, streaming, batching, caching, and backpressure conventions.

## 12. Keep Changes Reviewable

* Keep diffs focused and coherent. Avoid formatting-only noise.
* One concern per commit. Messages explain intent.
* Prefer clear names over comments. Comment non-obvious reasons, not mechanics.
* Remove debug output, dead code, placeholders, and commented-out alternatives.
* Do not edit generated or vendored files directly unless required.
* No em dashes or tool attribution in commit messages, PR text, or tickets unless the repository asks for it.

## 13. Operating Boundaries

Without explicit authorization, do not:

* Push or merge code
* Deploy or publish
* Modify production data or infrastructure
* Rotate credentials
* Contact third parties or write to external systems (trackers, chat, email)
* Perform destructive or irreversible actions
* Discard or overwrite uncommitted user work

Reads are fine by default. Mutations go through a reviewed change (a pull request), not an interactive session. Authorization to act in one context does not extend to another.

When blocked, report the blocker and the safest next action.

## 14. Definition of Done

A task is complete when:

* Requested behavior is implemented and acceptance criteria are satisfied.
* Required checks pass, or failures and unrun checks are reported with their impact.
* Error and boundary cases were considered.
* Contracts, migrations, configuration, and documentation are updated where needed.
* No unrelated changes were introduced. Pre-existing unrelated changes were preserved.
* The final response states:
  * What changed
  * What was verified, and how
  * Remaining risks, assumptions, and unverified items

For repositories with a UI or HTTP surface, also apply the completion checklist in [`web-app.md`](web-app.md).
