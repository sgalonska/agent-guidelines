# agent-guidelines

Tool-neutral guidelines for coding agents (Claude Code, Codex, Cursor, and others), written to drop into a repository as `AGENTS.md`.

| File | Use |
| --- | --- |
| [`AGENTS.md`](AGENTS.md) | Core rules for any task: priority, scope, contracts, security, testing, boundaries, Definition of Done. |
| [`web-app.md`](web-app.md) | UI or HTTP surface: UX states, accessibility, auth, APIs, databases, observability, browser verification, privacy. |
| [`infrastructure.md`](infrastructure.md) | Infrastructure as code, deploy scripts, migrations: PR-only changes, idempotency, access scoping, environments, state. |
| [`multi-agent.md`](multi-agent.md) | Sub-agents: when to spawn, model routing, briefing, reviewing results, second opinions. |

## Using it

Copy the files into one directory (the project root, or a folder you point to) and keep them together: `AGENTS.md` refers to its companions by relative path, and agents only open them when told to. If you merge `AGENTS.md` into an existing project file, keep those read-before instructions and fix the paths. Project-local rules win.

Keep the core short. Project-specific facts (accounts, tools, routing tables) belong in the project's own instructions, not here.

## History

Started as a [gist](https://gist.github.com/sgalonska/9f77ee4d4531aa9d37ca5aa9f1958c81). The content now lives here.
