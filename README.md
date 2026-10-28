# agent-guidelines

Tool-neutral guidelines for coding agents (Claude Code, Codex, Cursor, and others). Written to be dropped into a repository as `AGENTS.md`.

| File | Use |
| --- | --- |
| [`AGENTS.md`](AGENTS.md) | Core rules for any coding task: priority, scope, contracts, security, testing, boundaries, Definition of Done. |
| [`web-app.md`](web-app.md) | Extra rules for repositories with a UI or HTTP surface: UX states, accessibility, auth, APIs, migrations, observability, browser verification, privacy. |
| [`multi-agent.md`](multi-agent.md) | Rules for spawning and briefing sub-agents, routing work by model tier, reviewing results, and second opinions from another agent. |

| [`infrastructure.md`](infrastructure.md) | Extra rules for infrastructure as code, deploy scripts, and migrations: PR-only changes, idempotency, access scoping, environments, state. |

## Using it

Copy or link `AGENTS.md` into a project root, and add `web-app.md` or `multi-agent.md` next to it when relevant. Reference the companions from the project's own `AGENTS.md`. Project-local rules override these defaults.

Keep the core file short. Put project-specific facts (accounts, tools, routing tables) in the project's own instructions, not here.

## History

Started as a [gist](https://gist.github.com/sgalonska/9f77ee4d4531aa9d37ca5aa9f1958c81). The content now lives here.
