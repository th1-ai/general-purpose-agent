# General-Purpose Agent

A pointer to the **`general-purpose` sub-agent built into [Claude Code](https://claude.ai/code)**. There is nothing to install — this repo exists to document when and how to use it from a hotel-operations workflow.

> **Note:** Unlike the other th1-ai agents, `general-purpose` is not a custom sub-agent definition. It is the default fallback that Claude Code routes to when no specialised sub-agent fits. There is no `agent.md` to copy.

---

## What it does

`general-purpose` is the catch-all sub-agent. It has access to the **full toolset** the main agent has, and is the right choice for:

- Open-ended research questions ("should we even do this?")
- Multi-step tasks that don't fit a single specialised sub-agent
- Broad searches that combine reading code, reading docs, and reasoning
- Anything where the right next move is genuinely unclear and you want a sub-agent to figure it out

Because it has every tool available, it is more expensive than `Explore` (read-only) or `Plan` (design-only). Reach for it when the task actually warrants the breadth.

---

## What it is **not**

`general-purpose` is not for:

- **Fast file lookups** — use [`explore-agent`](https://github.com/th1-ai/explore-agent), it's cheaper
- **Pure architectural design** — use [`plan-agent`](https://github.com/th1-ai/plan-agent), it's more focused
- **Hotel-domain workflows that already have a custom agent** — use the dedicated agent (e.g. `email-triage-agent`, `concierge-agent`), the prompts and memory are already tuned

If a custom hotel agent fits the job, prefer it. `general-purpose` is the fallback, not the default.

---

## When Claude Code picks it automatically

Claude Code routes to `general-purpose` when:

- The task is multi-step and doesn't match any custom sub-agent's `description`
- The user asks something open-ended where the path involves searching, reading, and reasoning together
- A specialised sub-agent has been tried and bounced back as the wrong fit

---

## How to invoke it explicitly

From inside a Claude Code session:

```
> Use the general-purpose subagent to figure out which of our channel managers is generating the most disputed bookings, and propose which one to drop.
```

Or, more naturally, just describe the open-ended task:

```
> Look across our memory files, the finance sheet, and recent emails — what's the cheapest single change that would meaningfully cut the operator's admin time?
```

Claude Code will route to `general-purpose` if no narrower sub-agent matches.

---

## Suggested usage with hotel agents

Reach for `general-purpose` for:

- Cross-cutting questions that touch multiple agents' domains (finance + bookings + ops)
- Decisions that need synthesis across sources (memory files + sheet data + recent guest feedback)
- One-off investigations that don't justify building a new skill

Don't use it for:

- Anything a single dedicated agent already handles end-to-end (use that agent)
- Pure read-only lookups (use `Explore`)
- Pure design work (use `Plan`)

---

## File structure

```
general-purpose-agent/
├── README.md       # This file — the whole repo, essentially
├── LICENSE
└── .gitignore
```

There is no `.claude/` folder, no `.env.example`, and no `agent.md`. The agent lives inside Claude Code itself.

---

## Related agents

This sub-agent is one of three built-ins documented in this org:

- [`explore-agent`](https://github.com/th1-ai/explore-agent) — fast read-only search
- [`plan-agent`](https://github.com/th1-ai/plan-agent) — design implementation plans before changing code

Hotel-domain custom agents that often live alongside `general-purpose` work:

- [`morning-routine-agent`](https://github.com/th1-ai/morning-routine-agent)
- [`web-research-agent`](https://github.com/th1-ai/web-research-agent)
- [`marketing-report-agent`](https://github.com/th1-ai/marketing-report-agent)
- [`villa-analytics-agent`](https://github.com/th1-ai/villa-analytics-agent)

---

## Further reading

- Official Claude Code sub-agents docs: https://docs.claude.com/en/docs/claude-code/sub-agents

---

## License

MIT — see [LICENSE](LICENSE).
