# The-claude-best-harness

An opt-in model-routing harness for Claude Code. The main session plans, decides and verifies. Light, read-heavy work goes to cheaper subagents so the main session spends fewer tokens, without lowering the quality of results.

## What is in here

| Path | What it is |
|---|---|
| `skills/harness/SKILL.md` | The `/harness` skill: roles, routing table, size threshold, how to write a brief, report format, quality rules, the on-request Fable section, and the hard switch |
| `agents/scout.md` | Read-only subagent on Haiku for searching, listing and reading, with a report that must list what it searched |
| `agents/worker.md` | Subagent on Sonnet for small, well-specified edits on named files |

## How it works

- **Off by default.** Nothing is delegated until you type `/harness` or say "harness on". It applies to that chat only. "harness off" stops it.
- **Routing:** search and reading go to `scout`; small single-file edits go to `worker`; planning, design, multi-file coding and review stay in the main session. Fable is used only when you name it, for a hard sub-problem.
- **Size threshold:** delegate only when you expect more than about 5k tokens of raw output, or more than about 5 files, for the whole task. Below that, doing it directly is cheaper (measured in one small trial; indicative, not a benchmark).
- **Per-chat memory:** agents keep a short notes file in the session scratchpad, so it starts empty in every new chat.
- **Hard switch (enforced):** add `Agent(scout)` and `Agent(worker)` to `permissions.deny` in your settings to block the agents in every session. A change applies within about one tool call, no restart.

## Install

Copy the files into your Claude Code config folder:

```
~/.claude/agents/scout.md
~/.claude/agents/worker.md
~/.claude/skills/harness/SKILL.md
```

Then type `/harness` in a chat to turn it on. A newly added agent file can take a short while to appear in a session that is already open.

## Status

Tested in small trials only: a read-heavy inventory, a one-file search, a wide search with a 30-hit table, a worker draft, and a Fable subagent review. A large multi-file task has not been tested.
