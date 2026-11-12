# General-Purpose Agent

A guide to the **`general-purpose` sub-agent that is built into [Claude Code](https://code.claude.com/docs)**, written for hotel operations. There is nothing to install. This repo documents when to use the agent, how to brief it, and how to keep it safe around guest data.

> **Note:** Unlike most th1-ai agents, `general-purpose` is not a custom sub-agent. It ships with Claude Code. This repo has no `.claude/agents/` file to copy, and it must stay that way: see [Name collision](#name-collision-do-not-add-your-own-general-purpose-file).

Checked against the [Claude Code sub-agents docs](https://code.claude.com/docs/en/sub-agents) in November 2026. Claude Code changes often, so the docs win if they disagree with this page.

---

## What it is

`general-purpose` is the built-in sub-agent for work that needs searching, reasoning and changing things in one task. The docs describe it as follows:

- **Purpose:** complex research, multi-step operations, code modifications.
- **Tools:** every tool that sub-agents can have. This includes file edits, shell commands, web access and your MCP connections.
- **Model:** the model of your main conversation, unless you set `CLAUDE_CODE_SUBAGENT_MODEL` (see [Cost](#cost)).

It runs in its own context window. It does not see your conversation, and only its final report comes back. That keeps long searches out of your main session.

---

## When to use it

Use it for:

- Questions that cross several areas, for example finance, bookings and operations together.
- Decisions that need several sources compared, for example memory files, a finance sheet and recent guest feedback.
- One-off investigations that do not justify a new custom agent or skill.

Do not use it for:

- **A job that a dedicated hotel agent already does.** Use that agent (for example `email-triage-agent` or `concierge-agent`). Its prompt and safety rules are tuned for the job.
- **Read-only lookups.** Use the built-in `Explore` agent ([`explore-agent`](https://github.com/th1-ai/explore-agent)). It cannot write or edit, so it is the safer choice.
- **Planning before a change.** Use the built-in `Plan` agent ([`plan-agent`](https://github.com/th1-ai/plan-agent)).

`general-purpose` is the fallback, not the default.

---

## When Claude Code picks it automatically

Per the docs, Claude delegates to `general-purpose` when the task needs both exploration and modification, complex reasoning to interpret results, or several steps that depend on each other.

Claude chooses between sub-agents by their `description`. A custom agent with a sharp description wins the tasks it describes. `general-purpose` gets the multi-step work that no custom agent claims.

---

## How to invoke it

Name it in plain language inside a Claude Code session:

```
Use the general-purpose subagent to find which booking channel had the most disputed bookings last quarter. Read only. Report the count per channel and where each number came from.
```

Or mention it directly:

```
@agent-general-purpose compare our cancellation terms across every channel listing and list the differences
```

Or describe an open-ended task and let Claude decide:

```
Look across our memory files, the finance sheet and recent emails. What is the cheapest single change that would cut admin time? Propose it, change nothing.
```

---

## Brief it well

The agent starts with a fresh context. It gets the task message that Claude writes, plus your `CLAUDE.md` files. It does not get your chat history, the files Claude already read, or your auto memory. Sub-agents also cannot put a question to you part-way through a run.

So give the full brief in one message:

1. **Goal:** the question to answer or the result you want.
2. **Sources:** which files, sheets, mailboxes or systems to use, and which to leave alone.
3. **Limits:** what it must not do (see the next section).
4. **Report format:** for example "a table, one row per channel, with the source of each number".
5. **Unknowns:** tell it to list open questions in the report instead of guessing.

Put standing rules in `CLAUDE.md`, because every `general-purpose` run loads that file.

---

## Safe defaults for hotel work

`general-purpose` has every tool, and it has none of the guard rails that the dedicated hotel agents carry in their prompts. State the limits yourself, in the brief or in `CLAUDE.md`:

- **Draft, do not send.** No email, WhatsApp or other message to a guest, agent or supplier goes out without your approval.
- **No writes to the PMS, channel manager or payment system** without your approval. Ask for a proposed change list first.
- **No invented facts.** Prices, availability, policies and provider names come from a named source or are marked "not found".
- **Read-only unless told otherwise.** Add "change nothing" to research tasks.

Wording in a prompt is a request, not a lock. For a hard limit, use Claude Code permissions in `.claude/settings.json`. This example turns the agent off completely:

```json
{
  "permissions": {
    "deny": ["Agent(general-purpose)"]
  }
}
```

The same `deny` list can block single tools or commands for the whole session. See the [permissions docs](https://code.claude.com/docs/en/permissions).

---

## Cost

`general-purpose` runs on the model of your main conversation by default, so a long run on a large model uses a lot of your plan. Two options from the docs:

- Set the environment variable `CLAUDE_CODE_SUBAGENT_MODEL` to a model alias such as `sonnet` or `haiku`. It is a default for sub-agents that do not set their own model. It does not change the built-in `Explore` and `Plan` agents by itself.
- Ask for a model in the request, for example "use the general-purpose subagent on sonnet". Claude can pass a model for a single run.

---

## Name collision: do not add your own `general-purpose` file

Do not create `.claude/agents/general-purpose.md` (or the same file in `~/.claude/agents/`) unless you mean to replace the built-in agent.

What the docs say:

- When several sub-agents share a name, Claude Code uses the one from the higher-priority location. The order is managed settings, the `--agents` flag, the project `.claude/agents/`, the user `~/.claude/agents/`, then plugins.
- A user or project sub-agent named `Explore` overrides the built-in `Explore`.

The docs state the override for `Explore` only. They do not name `general-purpose`. Expect the same result: your file replaces the built-in agent, with your prompt, your `tools` list and your `model`. This repo has not tested it. If you want a narrower or cheaper helper, give it a different `name` and its own `description`, and leave the built-in agent as it is.

This is the reason the repo ships no agent file. A clone of this repo changes nothing in your Claude Code setup.

---

## Verify it works

1. Open Claude Code in any project folder.
2. Ask: `Which sub-agents can you use in this session?` The list must include `general-purpose`.
3. Run a harmless task: `Use the general-purpose subagent to list the markdown files in this folder and summarise each in one line. Change nothing.`
4. Check that Claude reports the result of a sub-agent run, not its own direct answer.

## Troubleshooting

- **Claude does the task itself and does not delegate.** Name the agent in the request, or use `@agent-general-purpose`. Small tasks often stay in the main session, which is fine.
- **`general-purpose` is missing from the list.** Look for `Agent(general-purpose)` in a `deny` list in `.claude/settings.json`, `~/.claude/settings.json` or your organisation's managed settings.
- **It behaves unlike this page describes.** Look for a file with `name: general-purpose` in `.claude/agents/` or `~/.claude/agents/`. See [Name collision](#name-collision-do-not-add-your-own-general-purpose-file).
- **A dedicated hotel agent was skipped in favour of `general-purpose`.** Sharpen that agent's `description`, or name the agent in your request.
- **The report has gaps or guesses.** The brief was too thin. Add sources and a report format, and run it again.

---

## File structure

```
general-purpose-agent/
├── README.md       # This file: the whole repo
├── LICENSE
└── .gitignore
```

There is no `.claude/` folder, no `.env.example` and no placeholders to fill in. The agent lives inside Claude Code.

---

## Related agents

Built-in Claude Code agents documented in this org:

- [`explore-agent`](https://github.com/th1-ai/explore-agent): read-only search
- [`plan-agent`](https://github.com/th1-ai/plan-agent): research for a plan before a change

Custom hotel agents that often sit next to `general-purpose` work:

- [`morning-routine-agent`](https://github.com/th1-ai/morning-routine-agent)
- [`web-research-agent`](https://github.com/th1-ai/web-research-agent)
- [`marketing-report-agent`](https://github.com/th1-ai/marketing-report-agent)
- [`villa-analytics-agent`](https://github.com/th1-ai/villa-analytics-agent)

The full set of 21 agents: https://th1.ai/agent-stack

---

## Further reading

- Claude Code sub-agents docs: https://code.claude.com/docs/en/sub-agents

---

## License

MIT. See [LICENSE](LICENSE).
