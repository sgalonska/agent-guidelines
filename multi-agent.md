# Multi-Agent Guidelines

On top of [`AGENTS.md`](AGENTS.md), whenever an agent spawns sub-agents, delegates to another agent CLI, or asks another agent for a second opinion. Harness-agnostic.

Terms: the **main session** talks to the user. A **sub-agent** is anything it launches. A **tier** is a model class: *lower* (small, cheap, fast) or *strongest* (largest available).

## 1. When to Spawn

Spawn when it buys time, cost, or quality:

* Independent work that can run in parallel.
* A side task that should not block the main thread.
* Broad searches where you need the conclusion, not the file dumps.
* A review that must not be anchored on your own reasoning.
* Mechanical work a cheaper tier can do.

Do not spawn for a handful of tool calls or a lookup whose target you know. Do not delegate a check you can run inline. Never redo a task you already delegated.

Sub-agents do not spawn their own unless the brief allows it. Keep fan-out under ten unless the user asks for scale.

## 2. Route by Difficulty

Pick the tier per task. Do not default everything to the session's own model.

**Routing applies only when more than one model is actually available to you.** Single model, or no way to choose? Run everything on it and skip this section. Never assume a model exists, never invent a model name, never hack around the harness to switch models. A requested tier is unavailable? Use what you have and, for high-judgment work, say so in the report.

**Lower tier:** mechanical work where a wrong draft is cheap to catch.

* Templated code following an existing pattern, boilerplate, renames
* Docs edits, test scaffolding
* Log and plan-output triage
* Inventories, data compilation, formatting, simple lookups
* First drafts from a well-defined structure

**Strongest tier:** work where a wrong call is expensive.

* Access, identity, and permission design
* Production cutover and data migration
* Security, compliance, financial, contractual, or legal tradeoffs
* Synthesis across many sources; prioritization and sequencing
* Foundational decisions other work depends on
* Client-facing framing
* The final review of the whole change

**Unsure? It is high-judgment.** Never downgrade to save tokens. When two tiers both fit, start lower and escalate if the result falls short.

A project's own routing table beats this section.

## 3. Brief Like a Peer

A sub-agent knows only what the brief says. Every brief states:

* The goal and why it matters.
* Exact scope: which files, directories, and systems it may touch, and which it may not.
* What is already known or ruled out.
* The hard rules (security, read-only systems, style, forbidden actions). Sub-agents inherit none of your context; restate them.
* The output shape (list, diff, verdict with evidence) and length.
* Blockers: stop and report, never improvise around a restriction.

Point to files instead of retyping them. Never hand over your hypothesis as fact: an agent given a conclusion tends to confirm it. For a review, give the code, not your verdict.

Cannot write a clear brief? You do not understand the task well enough to delegate it.

## 4. Who Owns What

* The **main session owns every external action**: pushing, opening and merging PRs, editing trackers, messaging people, deploying. Sub-agents return work and touch nothing external unless the brief explicitly grants it.
* The **main session owns the plan and the final answer.**
* Parallel sub-agents get **disjoint files** or separate worktrees or branches. Two agents in one file is a conflict you created.
* Share state through files (plans, handover notes, results), not remembered summaries.
* Sub-agents carry the same operating boundaries. Delegation never widens authority.

## 5. Review Every Result

A sub-agent's report says what it meant to do, not necessarily what it did.

* Read the actual diff or output before it enters the change.
* Re-run the relevant checks yourself. "Tests pass" without evidence is not evidence.
* Verify claims against the code. Confident wrong answers look exactly like right ones.
* Lower-tier output gets a light review. Strongest-tier work is still yours to review.
* Before opening a PR, have the strongest tier review the whole diff against the acceptance criteria. Real findings become fixes, not notes.
* Misunderstood task? Reject and re-brief; do not patch around it.

## 6. Second Opinions

Use a different agent or model as independent reviewer for designs, plans, and risky diffs, **only if one is actually available and authorized.** If not, use a fresh sub-agent of the same model briefed with the code but not your conclusion, or state plainly that the review was not independent.

* Advisory runs default to a **read-only** sandbox. Writes only when the task is to create files.
* Run CLIs non-interactively and **close stdin** (`< /dev/null`) or they may hang.
* Give it the material and the question, never your answer to rubber-stamp.
* Its output is input, not authority. Reconcile explicitly: where you agree, where you do not, and why.
* Call agent CLIs directly. No custom server or connector wrapper unless the project asks for one.
* No secrets, customer data, or private material to an external service without authorization.

Consilium (two or more agents on one question): collect each opinion independently first, then compare. State the combined call and what stays disputed.

## 7. Automated Review Loops

When bots or other agents review a PR:

1. Tie every check to the **head commit SHA**. "Clear" means a completed review of the current head, not an absence of comments. Never merge on silence.
2. Request a fresh review after every fix push, even if you verified the fix yourself.
3. Wait with a bounded polling loop (`until <condition>; do sleep N; done`), never a bare sleep, never forever.
4. Collect everything: inline comments, summaries, reviews, CI status.
5. Verify each finding against the code. Fix valid ones with the smallest change, one commit per finding where practical, and reply on the thread.
6. Wrong or debatable finding? Do not change code to appease the bot. Reply with reasoning and ask the human before merging.
7. Stop after a fixed number of rounds (default 5) and report instead of looping.
8. Merge only where you are authorized to. Otherwise report the PR as clear and leave it.

## 8. Fail Safely

* Hit a permission denial, restriction, or connection failure? Report it. Never retry the blocked action or work around the guard.
* Flaky integrations (OAuth sessions, remote MCP servers) drop routinely. Ask the human to reconnect; do not root-cause it.
* Cap retries and rounds. At the cap, summarize what was tried and stop.
* A result that contradicts a hard rule gets discarded, and the user gets told.

## 9. Voice in External Text

Every sub-agent writing tickets, PR bodies, or commit messages gets the repo's voice rules in its brief, including the core rule on no tool branding and role language ([`AGENTS.md`](AGENTS.md) section 12).

## 10. Report

End a delegated task with:

* What was delegated, to which tier, and why.
* What each sub-agent returned and what you verified yourself.
* Disagreements between agents and how they were resolved.
* What remains unverified or left to a human.
