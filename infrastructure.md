# Infrastructure Guidelines

Applies on top of [`AGENTS.md`](AGENTS.md) to infrastructure as code (Terraform, OpenTofu, Pulumi, CloudFormation, Kubernetes manifests, Ansible), deployment scripts, user-data, and database migrations. Examples use Terraform vocabulary; the rules apply to any tool. Repository-local instructions override these defaults.

## 1. Change Path

* Infrastructure changes ship as a reviewed pull request. A pipeline applies them after merge. Do not apply, import, destroy, or mutate state from an interactive session.
* Reads are fine: plan without refresh where the tool allows it, validate, describe, list. Prefer the read mechanism the project designates (a scoped read-only credential or server) over a general-access profile.
* One-off operational mutations that are not code-managed (creating a secret, running an admin command on a host) are written out as an exact command for a human to run. Do not run them.
* Do not run any command that refreshes remote state or acquires a state lock unless the project allows it.
* If a task requires a write to a live environment, stop and explain which guardrail must be deliberately removed by the owner first.

## 2. Idempotency

A change is not done until every item holds:

* Applying twice produces no diff.
* No `timestamp()`, `uuid()`, unkeyed random values, or other per-run values feed resource arguments. Where a per-run value is deliberate (an image tag), isolate it and do not copy the pattern.
* Resources are keyed by stable identifiers: `for_each` over stable keys, not list indexes that shift when an element is removed.
* No destroy-and-recreate from a harmless edit. Check replacement behavior, set `prevent_destroy` on stateful resources, use `create_before_destroy` where replacement is needed.
* Renames use `moved` blocks (or the tool's equivalent), not delete plus create.
* Existing resources are adopted deliberately through a declared import that the pipeline applies, never recreated.
* Conditional resources (`count` or `for_each` gated by a flag) behave correctly with the flag on and off, and toggling does not touch unrelated resources.
* State the expected plan in the PR body: what is created, changed, destroyed. Anything unexpected is a bug.
* A second apply with no resource changes is the real proof. Record it when you can observe it.

## 3. Scripts, User-Data, and Jobs

Safe to re-run:

* Guard with existence checks and `IF NOT EXISTS` equivalents.
* Shell scripts use `set -euo pipefail`.
* A second run loses no data and changes nothing.
* Fail loudly on partial failure; do not continue past an error.

## 4. Migrations

Follow the migration rules in [`web-app.md`](web-app.md) section 7, and additionally:

* Tables are created only in schema or migration files, never in application code. Application code assumes tables exist and fails loudly if one is missing. Tests that need a database create their own tables in fixture setup.
* Do not write destructive migrations: no `DROP TABLE`, `DROP COLUMN`, `DELETE FROM`, or `TRUNCATE`, and no overwriting of existing row data.
* A table rebuild (for example to add a constraint) is allowed only if the migration copies every existing row first. Verify the copy covers every column.
* Rebuilds of derived data (indexes, full-text indexes) that regenerate from existing rows are fine.
* If a migration could lose data under any circumstance, stop and ask before writing it.

## 5. Access and Identity

* Least privilege for every role, policy, and credential, at the narrowest scope and shortest lifetime that works.
* Prefer short-lived federated credentials (OIDC) over long-lived keys. Restrict trust policies by repository, branch, and environment.
* Scope policies to named resources, not wildcards, unless the wildcard is the documented requirement.
* **Deploy-role coverage:** if the pipeline runs under a restricted role that refreshes the whole stack, any new resource it must read (bucket, secret, parameter, log group) must be added to that role's read permissions in the same change. Missing this makes the next unrelated deploy fail with permission errors.
* Treat access and identity design as high-judgment work (see [`multi-agent.md`](multi-agent.md) section 2).

## 6. Environments

* Name the environments that exist. Do not invent a "staging" that does not exist, and note when two names mean the same environment.
* Frame a change around the environment where the real problem lives (usually production). Use a lower environment as secondary validation, never as a substitute for the production fix and never as the only subject of a change unless the problem is genuinely there.
* Test new infrastructure in a production-like environment before it goes to review for production, where one exists.
* A change must remain correct after a lower environment is reset from the main branch. Do not rely on a fix that exists only in the lower environment.
* Keep environment differences in variable files or configuration, not in branches or copied modules.

## 7. State and Drift

* Never hand-edit state. Use the tool's supported commands, and only through the change path in section 1.
* Treat unexpected drift as a finding: report it with evidence, do not silently overwrite it.
* Keep state locking and remote state protections enabled. Do not bypass a lock.
* Stateful resources (databases, buckets, volumes, keys) carry deletion protection and backups. Verify restore, not only backup.

## 8. Secrets

* Never commit secrets or put them in state-visible plain text where the tool offers a sensitive or external reference.
* Reference secrets from the platform's secret store. Creating the secret value is a human step (section 1).
* Do not echo secrets in plan output, logs, or PR text.

## 9. Validation

Locally, without touching remote state:

* Format check, validate, and any policy or security scanners the repository defines.
* The repository's own test files for modules where they exist.
* Fix failures. Do not paper over a failing check, skip it, or lower its severity.

## 10. Reusable Infrastructure Code

If the code is meant to be reused across customers or projects:

* Keep generic logic in modules and put environment-, customer-, and account-specific values in variable files or a separate configuration layer that another consumer could swap.
* Write policy-as-code rules generically ("storage buckets must have encryption enabled") and keep named exceptions in a separate override file.
* Keep confidential material (contracts, personal data, pricing, internal policy text) out of reusable code from the start. Do not rely on redacting it later.
* Make the generic layer liftable as a whole, with specific glue referencing it and never the reverse.

## 11. Completion Checklist

In addition to the core Definition of Done:

* Plan expectation stated; second-apply no-op confirmed or noted as unverified.
* Stable keys, replacement behavior, and gated-resource toggling checked.
* Scripts and migrations safe to re-run; no destructive statements.
* Deploy-role permissions cover new resources.
* Least-privilege access reviewed.
* Validation run locally; results stated, including checks not run.
* Anything requiring a human-run command is listed with the exact command.
