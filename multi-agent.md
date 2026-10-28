# Multi-Agent Guidelines

Applies on top of [`AGENTS.md`](AGENTS.md) whenever an agent spawns sub-agents, delegates to another agent CLI, or asks another agent for a second opinion. Works the same for Claude, Codex, and any other harness.

Terms: the **main session** is the agent talking to the user. A **sub-agent** is any agent it launches. A **tier** is a class of model: *lower tier* (small, cheap, fast) or *strongest* (largest available).

## 1. When to Spawn

Spawn when it saves time or cost or improves quality:

* Independent pieces of work that can run in parallel.
* A side task that should not block the main thread.
* Broad searches or reads whose conclusion matters but whose file dumps do not.
* A review that should not be anchored on your own reasoning.
* Mechanical work that a cheaper tier can do.

Do not spawn for a handful of tool calls or a lookup whose target you know. Do not delegate a check you can run inline. Do not run the same task yourself after delegating it.

Sub-agents do not spawn further sub-agents unless the brief says they may. Keep the fan-out small by default (under ten), and raise it only when the user asks for scale.

## 2. Route by Difficulty

Pick the tier per task. Do not default everything to the session's own model.

Routing only applies when more than one model is actually available to you. If the harness offers a single model, or the project's configuration does not let you choose, run everything on that model and skip this section. Never assume a model exists, never invent a model name, and never change the model by working around the harness. If a requested tier is unavailable, use the model you have and, for high-judgment work, say so in the report.

**Lower tier:** mechanical, low-judgment work where a wrong draft is cheaply caught in review.

* Templated code following an existing pattern, boilerplate, renames
* Docs and README edits, test scaffolding
* Log and plan-output triage
* Inventory and data compilation, formatting, simple lookups
* First drafts from a well-defined structure

**Strongest tier:** work where a wrong call is expensive.

* Access, identity, and permission design
* Production cutover and data migration
* Security, compliance, financial, contractual, or legal tradeoffs
* Synthesis across many sources, prioritization and sequencing
* Foundational decisions that other work depends on
* Client-facing framing
* The final review of the whole change

**When unsure, treat the task as high-judgment.** Never downgrade a task to save tokens. When two tiers both fit, start with the lower and escalate if the result is not good enough.

Projects may add a routing table in their own instructions. That table wins over this section.

## 3. Brief Like a Peer

A sub-agent knows only what the brief says. Every brief states:

* The goal and why it matters.
* The exact scope: files, directories, systems it may touch and may not.
* What is already known or ruled out, so it does not redo or contradict it.
* The hard rules that apply (security, read-only systems, style, forbidden actions). Sub-agents do not inherit your context; restate them.
* The expected output shape (a list, a diff, a verdict with evidence) and how long it should be.
* How to report blockers: stop and report, do not improvise around a restriction.

Point to files and docs instead of retyping them. Do not hand over your hypothesis as fact: an agent given a conclusion tends to confirm it. For a review, give the code, not your verdict.

If you cannot write a clear brief, you do not understand the task well enough to delegate it.

## 4. Who Owns What

* The **main session owns every external action**: pushing, opening and merging PRs, editing trackers, messaging people, deploying. Sub-agents return work; they do not push, open PRs, or touch external systems unless the brief explicitly grants it.
* The **main session owns the plan and the final answer.** Sub-agents contribute to them.
* Give parallel sub-agents **disjoint files** or separate worktrees or branches. Two agents editing the same file is a conflict you created.
* Share state through files (plans, handover notes, result files), not through remembered summaries.
* Sub-agents inherit the same operating boundaries as the main session. Delegation never widens authority.

## 5. Review Every Result

A sub-agent's report describes what it intended, not necessarily what it did.

* Read the actual diff or output before it enters the change.
* Re-run the relevant checks yourself. Do not accept "tests pass" without evidence.
* Verify claims against the code. Confident wrong answers look the same as right ones.
* For lower-tier output, a light review is enough. For strongest-tier-only work, the review is still yours.
* Before opening a PR, have the strongest tier review the whole diff against the acceptance criteria. Record real findings as fixes, not notes.
* Reject and re-brief rather than patching around a misunderstood task.

## 6. Second Opinions From Another Agent

Use a different agent or model as an independent reviewer for designs, plans, and risky diffs, but only if one is actually available and authorized. If it is not, review with a fresh sub-agent of the same model briefed with the code but not your conclusion, or say the review was not independent.

* Default to a **read-only** sandbox for advisory runs. Allow writes only when the task is to create files.
* Run CLIs non-interactively and **close stdin** (`< /dev/null`), or they may hang waiting for input.
* Give it the material and the question. Do not give it your answer to confirm.
* Treat its output as input, not authority. Reconcile disagreements explicitly: say where you agree, where you do not, and why.
* Call agent CLIs directly. Do not wrap them in a custom server or connector unless the project asks for one.
* Do not send secrets, customer data, or private material to an external service without authorization.

For a consilium (two or more agents on one question), collect each opinion independently first, then compare. State the combined recommendation and what remains disputed.

## 7. Automated Review Loops

When automated reviewers (code-review bots, other agents) review a PR:

1. Tie every check to the **head commit SHA**. "Clear" means a completed review of the current head, not an absence of comments. Never merge on silence.
2. Request a fresh review after every fix push, even if you verified the fix yourself.
3. Wait with a bounded polling loop (`until <condition>; do sleep N; done`), never a bare sleep and never forever.
4. Collect all remarks: inline, summaries, reviews, and CI status.
5. Verify each finding against the code before acting. Fix valid ones with the smallest change, one commit per finding where practical, and reply on the thread.
6. If a finding is wrong or debatable, do not change code to appease the bot. Reply with reasoning and ask the human before merging.
7. Stop after a fixed number of rounds (default 5) and report instead of looping.
8. Merge only where you are authorized to merge. Otherwise report the PR as clear and leave it.

## 8. Fail Safely

* A sub-agent that hits a permission denial, a restriction, or a connection failure reports it. It does not retry the same blocked action or work around the guard.
* Flaky integrations (OAuth sessions, remote MCP servers) fail routinely. Ask the human to reconnect rather than root-causing it.
* Cap retries and rounds. After the cap, summarize what was tried and stop.
* If a sub-agent's result contradicts a hard rule, discard the result and tell the user.

## 9. Keep Text Free of Tooling Branding

When external-facing text is audience-specific (tickets, PR bodies, commit messages), follow the repository's convention on voice. If it says to use role language ("implementation", "QA review") and not name agents or tools, do so, and tell every sub-agent that writes such text.

## 10. Report

End a delegated task with:

* What was delegated, to which tier, and why.
* What each sub-agent returned and what you verified yourself.
* Disagreements between agents and how they were resolved.
* What remains unverified or left to a human.
