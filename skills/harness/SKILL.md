---
name: harness
description: Opt-in model-routing harness. Use only when the user types /harness or says "harness on". Delegates light work to the scout (Haiku) and worker (Sonnet) subagents to cut main-session tokens, while the main session keeps planning, coding and verification. "harness off" stops it for the chat.
---

# Harness: model-routing delegation (opt-in)

Active only from the moment the user invokes `/harness` or says "harness on", and only for this chat. When the user says "harness off", stop delegating and do everything in the main session for the rest of the chat. Never turn it on by yourself.

## Roles

- **Main session (orchestrator):** plans, decides, verifies, and does all multi-file or design-heavy coding. Never delegated down.
- **scout** (Haiku, read-only): searching, grepping, listing, reading many files, logs and manual lookups.
- **worker** (Sonnet): single-file small edits, drafts, extraction, summaries, on files named in the brief.

## Routing table

| Task | Who |
|---|---|
| Search, list, read many files, log or manual lookups | scout |
| Small single-file edit, draft, extraction, summary | worker |
| Planning, architecture, design choices, multi-file or non-trivial coding, unclear-cause debugging, code review | main session |
| Anything that changes a live plant, SCADA system, settings or permissions | main session only, with the usual preview then confirm |

Tie-breaker: if a task is ambiguous or touches more than one file, go up a tier. Do not delegate a quick one-file read or one-line edit; the cold start costs more than it saves.

**Size threshold (measured):** delegate only when you expect, for the task as a whole (not per scout), more than about 5k tokens of raw output, or reading more than about 5 files, such as long logs, wide searches or many manual pages. Below that, do it directly in the main session. In a trial, a small inventory (about 1.2k tokens of output) cost roughly 1.3k main-session tokens directly versus roughly 2.2k delegated plus 38k inside the scout. One run per method, so treat it as indicative.

## How to delegate

1. Write a **self-contained brief**; subagents do not see the conversation. Include: goal, exact files or areas, constraints, the report format below, and the session scratchpad path with the instruction to use `<scratchpad>/agent-memory/<agent-name>.md` as notes. The scratchpad is per chat, so memory starts empty in every new chat.
2. Run independent scouts **in parallel**, one per area.
3. Work from the **reports only**. Open a file in the main session only when a report's evidence is not enough.
4. Decide, then hand the edit to the worker with exact instructions, or make the edit yourself if it is tricky.

## Report format (put in every brief)

Conclusion first; evidence as `file:line` references (and the check that was run, for the worker); open questions. About 300 words by default; scale it to the task (for a wide search, a table of up to 30 `file:line` hits is fine). No file dumps. Scout reports also list the patterns and paths searched with hit counts.

## Quality rules

- Every claim in a report needs a `file:line` or a source. A claim without one gets re-checked.
- Spot-check the one or two facts the decision depends on before relying on a report.
- **Negative results need proof.** A "not found" or "none" cannot carry a `file:line`, so scouts must list the exact patterns and paths searched and the hit counts. For any negative the decision depends on, re-run one search yourself in the main session before relying on it.
- An empty, vague or contradictory report is not accepted. A failed scout report goes straight to the main session with a narrowed question (not to the worker, which would get the same weak brief and cost a second cold start). A failed worker report also goes to the main session.
- Worker edits count as done only after the named test, lint or diff review passes. The worker runs that check itself and returns only pass or fail plus the failing lines, so long test or lint output stays out of the main context.
- A cheap model never makes the final decision; it only gathers or drafts.

## Fable (on request only)

Fable is the strongest model and has no fixed role. Opus stays the main session and the foundation; Fable is never chosen automatically (it has its own weekly limit).

- **When:** only if the user names Fable for a task, and the task is a hard sub-problem: a design choice, a hard-to-diagnose bug, a second opinion on a plan, or a tricky piece of reasoning. Never for work scout or worker can do.
- **How:** delegate with `model: fable` on the Agent call (a general-purpose agent is fine), with a self-contained brief and the usual report format. Opus then reads the report, spot-checks the facts the decision depends on, and integrates the result. Opus makes the final decision.
- **Heavy implementation** stays with the main session (Opus). If a separate Opus helper is ever needed, use `model: opus` on the Agent call; no dedicated agent file.
- **If the user wants the whole main session on Fable** (a different mode, not delegation): say it changes the session model for everything after, needs their approval, and that Opus would then become the delegated heavy-coding role via `model: opus`. Do not do this unless they ask.
- **Hard switch note:** the deny rules cover scout and worker by name; a Fable call through another agent type is not blocked by them.

## Hard switch (enforced off)

If the user wants the harness fully blocked, offer to edit `~/.claude/settings.json` and add the rules `Agent(scout)` and `Agent(worker)` to `permissions.deny`; removing them turns it back on. The change applies within about one tool call, no restart. Only do this when the user asks for that exact action, and say that the setting is global to every session, not just this chat.
