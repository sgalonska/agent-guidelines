# Infrastructure Guidelines

On top of [`AGENTS.md`](AGENTS.md), for infrastructure as code (Terraform, Pulumi, CloudFormation, Kubernetes manifests, Ansible, and the like), deploy scripts, user-data, and database migrations. Examples use Terraform vocabulary; the rules apply to any tool.

## 1. Change Path

* Infrastructure ships as a reviewed pull request, applied by a pipeline after merge. Never apply, import, destroy, or mutate state from an interactive session.
* Reads are fine: validate, describe, list, plan without refresh where the tool allows it. Use the project's designated read mechanism (a scoped read-only credential or server) over a general-access profile.
* One-off mutations that are not code-managed (creating a secret, an admin command on a host) get written out as an exact command for a human to run. You do not run them.
* Nothing that refreshes remote state or takes a state lock unless the project allows it.
* A task needs a write to a live environment? Stop and name the guardrail the owner must deliberately remove first.

## 2. Idempotency

Not done until every item holds:

* Applying twice produces no diff.
* No `timestamp()`, `uuid()`, unkeyed random values, or other per-run values in resource arguments. A deliberate per-run value (an image tag) stays isolated; do not copy the pattern.
* Resources keyed by stable identifiers: `for_each` over stable keys, never list indexes that shift when an element goes.
* No destroy-and-recreate from a harmless edit. Check replacement behavior, `prevent_destroy` on stateful resources, `create_before_destroy` where replacement is unavoidable.
* Renames use `moved` blocks (or the equivalent), not delete plus create.
* Existing resources are adopted through a declared import the pipeline applies, never recreated.
* Flag-gated resources (`count` or `for_each`) work with the flag on and off, and toggling leaves unrelated resources alone.
* The PR body states the expected plan: created, changed, destroyed. Anything unexpected is a bug.
* A second no-op apply is the real proof. Record it when you can see it.

## 3. Scripts, User-Data, and Jobs

Safe to re-run, always:

* Existence checks and `IF NOT EXISTS` equivalents.
* Shell scripts start with `set -euo pipefail`.
* A second run changes nothing and loses nothing.
* Partial failure stops loudly; never continue past an error.

## 4. Migrations

[`web-app.md`](web-app.md) section 7 applies, plus:

* Tables are created only in schema or migration files, never in application code. App code assumes tables exist and fails loudly when one is missing. Tests create their own tables in fixture setup.
* No destructive migrations: no `DROP TABLE`, `DROP COLUMN`, `DELETE FROM`, `TRUNCATE`, and no overwriting existing rows.
* A table rebuild (say, to add a constraint) copies every existing row first. Verify the copy covers every column.
* Regenerating derived data (indexes, full-text indexes) from existing rows is fine.
* Could it lose data under any circumstance? Stop and ask before writing it.

## 5. Access and Identity

* Least privilege for every role, policy, and credential: narrowest scope, shortest lifetime that works.
* Short-lived federated credentials (OIDC) over long-lived keys. Restrict trust policies by repository, branch, and environment.
* Scope policies to named resources, not wildcards, unless the wildcard is the documented requirement.
* **Deploy-role coverage:** if the pipeline runs under a restricted role that refreshes the whole stack, every new resource it must read (bucket, secret, parameter, log group) gets added to that role's read permissions in the same change. Miss it and the next unrelated deploy dies on permission errors.
* Access and identity design is high-judgment work (see [`multi-agent.md`](multi-agent.md) section 2).

## 6. Environments

* Name the environments that actually exist. Do not invent a "staging"; note when two names mean the same environment.
* Frame a change around where the real problem lives (usually production). A lower environment is secondary validation, never a substitute for the production fix, and never the sole subject unless the problem is genuinely only there.
* Where a production-like environment exists, test new infrastructure there before it goes to review for production.
* A change must stay correct after a lower environment is reset from the main branch. Never depend on a fix that lives only there.
* Environment differences live in variable files or config, not branches or copied modules.

## 7. State and Drift

* Never hand-edit state. Supported commands only, and only through the change path in section 1.
* Unexpected drift is a finding: report it with evidence, never silently overwrite it.
* Keep locking and remote-state protections on. Never bypass a lock.
* Stateful resources (databases, buckets, volumes, keys) get deletion protection and backups. Verify restore, not just backup.

## 8. Secrets

* No plaintext secrets in code or state where the tool offers a sensitive or external reference. Reference the platform's secret store.
* Creating the secret value is a human step (section 1).
* Never echo secrets in plan output, logs, or PR text.

## 9. Validation

Locally, without touching remote state:

* Format, validate, and every policy or security scanner the repo defines.
* The repo's own module tests where they exist.
* Fix failures. Never skip a check, paper over it, or lower its severity.

## 10. Reusable Infrastructure Code

When the code is meant for more than one customer or project:

* Generic logic in modules; environment-, customer-, and account-specific values in variable files or a separate config layer another consumer could swap in.
* Policy-as-code rules stay generic ("storage buckets must be encrypted"); named exceptions go in a separate override file.
* Confidential material (contracts, personal data, pricing, internal policy text) stays out of reusable code from day one. Redacting later is not a plan.
* The generic layer must lift out whole: specific glue references it, never the reverse.

## 11. Completion Checklist

On top of the core Definition of Done:

* Expected plan stated; second-apply no-op confirmed or flagged unverified.
* Stable keys, replacement behavior, and flag toggling checked.
* Scripts and migrations re-runnable; no destructive statements.
* Deploy role covers new resources. Least privilege reviewed.
* Local validation run; results stated, including checks not run.
* Every human-run command listed verbatim.
