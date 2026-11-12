# General-Purpose Agent

A guide to the **`general-purpose` sub-agent that is built into [Claude Code](https://code.claude.com/docs)**, written for hotel operations. There is nothing to install. This repo documents when to use the agent, how to brief it, and how to keep it safe around guest data.

> **Note:** Unlike most th1-ai agents, `general-purpose` is not a custom sub-agent. It ships with Claude Code. This repo has no `.claude/agents/` file to copy, and it must stay that way: see [Name collision](#name-collision-do-not-add-your-own-general-purpose-file). To use it from a scheduled, unattended hotel agent, see [Run it on its own](#run-it-on-its-own).

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

## Run it on its own

A hotel agent that runs on a schedule can hand a research task, or the steps of an approved plan, to `general-purpose`. This sub-agent has every tool, so an unattended run needs hard limits. [`AUTONOMY.md`](AUTONOMY.md) sets the limits. It ships at `Level: 1` (scheduled, report only). The file has one line `Level: <value>`, with the value `0`, `1`, `2`, `3` or `PAUSED`. A missing file, a missing line or an unknown value counts as `PAUSED`.

### The scheduled command

Run this from the folder of the hotel agent that delegates (for example your clone of `marketing-report-agent`):

```bash
claude -p "Read AUTONOMY.md first and obey it. Use the general-purpose subagent to compare this agent's files with the last entries in runs/LOG.md and to report the plan in runs/handoff/plan-agent.md with its approval status. Read only. Do not retry a denied call. Start the report with a section named 'Needs you'. Then append one entry to runs/LOG.md." \
  --permission-mode dontAsk \
  --permission-prompts none \
  --settings .claude/settings.example.json \
  --max-turns 30 \
  --output-format json
```

What each part does, from the Claude Code docs:

- `-p` runs without the interactive screen.
- `--permission-mode dontAsk` denies every tool call that would otherwise ask you. File reads in the folder and the read-only shell commands still run. So do calls that match an `allow` rule.
- `--permission-prompts none` tells Claude Code that nobody can answer a prompt, and tells Claude not to retry a denied call. Needs Claude Code v2.1.259 or later.
- `--settings` loads the permissions file for this run only (see below). The docs state that permission rules apply to the main conversation and to sub-agents.
- `--max-turns` stops a run that does not finish. `--max-budget-usd` is a second cap for runs that are billed to an API key. The docs state that sub-agent spend counts toward that cap.
- `--output-format json` returns the report in `result`, the denied calls in `permission_denials`, and an estimate in `total_cost_usd`.

Checked on Claude Code v2.1.323 in a 3-file test folder. With the example permissions file, the sub-agent wrote `runs/LOG.md`. A write to `runs/handoff/`, an edit of a file outside `runs/` and a read of `.env` were denied. The denied calls were listed in `permission_denials`.

Do not add `--bare` to this command. Bare mode skips `CLAUDE.md`, and it does not use your subscription login.

Do not use `--dangerously-skip-permissions` or `--permission-mode bypassPermissions` for a hotel agent. They remove the limits that this section depends on.

### Least-privilege permissions

[`.claude/settings.example.json`](.claude/settings.example.json) is an example for Level 1. It has the name `settings.example.json` so that a clone of this repo changes nothing. Copy it into the hotel agent's `.claude/` folder and pass it with `--settings`.

- `permissions.defaultMode: "dontAsk"` denies each call that no rule allows.
- `permissions.deny` removes `Edit`, `Write`, `NotebookEdit`, `WebFetch` and `WebSearch`, and denies `curl`, `wget`, `rm`, `git push` and reads of `.env` files.
- `permissions.allow` has one rule, `Edit(./runs/LOG.md)`, so the run can append its entry to the run log. The docs state that `Edit` rules apply to all built-in tools that edit files. No other file is writable.
- No rule allows your PMS CLI, your mail CLI or your MCP tools. They need approval, so an unattended run denies them.

For Level 2, make a second file. Change the allow rule to `Edit(./runs/**)`. In the same test folder, a write to `runs/handoff/` worked, and a write and an edit outside `runs/` were denied.

For Level 3, the calling agent's own files must be writable. Add one `Edit(...)` allow rule for each folder that an approved plan can change. Keep `Edit(./AUTONOMY.md)`, `Edit(./CLAUDE.md)`, `Edit(./.claude/**)` and `Edit(./.env*)` in `deny`. This Level 3 set-up was not tested here.

If the task needs live data, add an allow rule for one read-only command of your own CLI, for example `"allow": ["Bash(your-pms-cli arrivals *)"]`. The docs warn that Bash rules that constrain command arguments are fragile, and that a deny rule does not match the same program by path or inside `sh -c`. Give the scheduled run read-only credentials where your tools support that.

### A scheduler example

This is an example for cron on macOS or Linux. The Claude Code docs do not describe cron for `claude -p`, and this line was not run under cron here. The `grep` check is the kill switch. It was tested against `Level: 1`, `Level: 0`, `Level: PAUSED`, an unknown value and a missing file. Only `Level: 1`, `Level: 2` and `Level: 3` start a run.

```cron
# Every day 06:30: one report-only run. Use full paths; cron has a short PATH.
30 6 * * * cd /path/to/your-hotel-agent && grep -qE '^Level: [123]$' AUTONOMY.md && /path/to/claude -p "Read AUTONOMY.md first and obey it. Use the general-purpose subagent to report the plan in runs/handoff/plan-agent.md with its approval status. Read only. Start the report with 'Needs you'. Then append one entry to runs/LOG.md." --permission-mode dontAsk --permission-prompts none --settings .claude/settings.example.json --max-turns 30 --output-format json > runs/general-purpose-report.json 2>&1
```

`runs/general-purpose-report.json` is the raw output of the last run. The run summary that other agents read is the last entry of `runs/LOG.md`.

Claude Code also has its own schedulers (cloud scheduled tasks, Desktop scheduled tasks and `/loop`). See [Run prompts on a schedule](https://code.claude.com/docs/en/scheduled-tasks) for the comparison.

### Delegating to `general-purpose` from a scheduled agent

Mechanics and limits from the docs:

- **Every tool.** `general-purpose` gets the tools of the main conversation, MCP tools included. The permissions file is the limit, not the prompt.
- **`CLAUDE.md` loads.** Unlike `Explore` and `Plan`, `general-purpose` loads your `CLAUDE.md` files. Put standing house rules there. In a `-p` run you can also pass `--append-subagent-system-prompt "<your rules>"` (v2.1.205 or later), which adds the text to every sub-agent's system prompt.
- **No conversation history.** The sub-agent gets only the delegation prompt. See [Brief it well](#brief-it-well).
- **Not guaranteed.** Claude decides whether to delegate. Name the sub-agent in the prompt. The permissions hold in both cases.
- **From a custom sub-agent.** A sub-agent can start `general-purpose` only if its `tools` list includes `Agent` (or it has no `tools` list). Nesting stops 3 layers below the main conversation by default.
- **Hooks apply.** Hooks from settings files also run inside sub-agents. A `PreToolUse` hook that reads the `Level:` line can block a write as a second lock. This repo ships no hook.
- **Cost.** The sub-agent sends its own requests, and they count toward the same usage limits. See [Cost](#cost) for the model options. Keep the task narrow and name the files.

### What must never be delegated to `general-purpose`

- **A message to a guest, a partner or a supplier.** The dedicated agents carry the tone, language and safety rules. `general-purpose` does not.
- **A write to the PMS, the channel manager or a payment system, and any money movement.**
- **Approval of a plan.** Only the operator approves.
- **Decisions about a complaint, a legal matter, a medical matter or a safety matter.**
- **Changes to `AUTONOMY.md`, `CLAUDE.md`, settings, hooks, permissions or `.env` files.**
- **A job that a dedicated hotel agent does.**

The full list is in [`AUTONOMY.md`](AUTONOMY.md).

### Handoffs

- **From [`plan-agent`](https://github.com/th1-ai/plan-agent).** At its Level 2, the plan is saved to `runs/handoff/plan-agent.md` in the calling agent's folder. You approve it: add a line that starts with `Approved:` at the top of the file, with the date and your name. A `general-purpose` run then picks it up inside its own level. At Level 1 it reports the plan and its approval status. At Level 2 it writes the prepared steps under `runs/` and a summary to `runs/handoff/general-purpose-agent.md`. At Level 3 it applies the file changes to the agent's own files, inside the limit. A step that reaches the PMS, a guest or money goes to "Needs you" at every level. The 7 steps are in [`AUTONOMY.md`](AUTONOMY.md).
- **To [`morning-routine-agent`](https://github.com/th1-ai/morning-routine-agent).** The latest run summary is the last entry of `runs/LOG.md`. Its "Needs you" section is the part to show in the morning briefing. The handoff output is `runs/handoff/general-purpose-agent.md`.
- **From [`explore-agent`](https://github.com/th1-ai/explore-agent).** For a pure search question, use `Explore`. It cannot write.

Run reports can contain guest data. Add `runs/` to the `.gitignore` of the agent folder.

### Moving up a level

1. Run 7 days at the current level.
2. Read each run report. Check that "Needs you" was correct and that nothing unexpected was denied or changed.
3. Change the one line `Level:` in `AUTONOMY.md`, and pass the permissions file for the new level.
4. For the first 5 runs at the new level, the run does only what the level below allows and lists what it would have done. Read each report.

To stop everything, set `Level: 0` or `Level: PAUSED`.

### Subscription note

Scheduled `claude -p` runs on a personal Pro or Max plan are subject to Anthropic's usage policy, and the plan limits assume ordinary individual use; for heavy or business-critical schedules, use an API key or a team plan.

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
├── README.md                       # This file
├── AUTONOMY.md                     # Levels, action table and guard rails for scheduled delegation
├── .claude/
│   └── settings.example.json       # Example Level 1 permissions for an unattended run (not loaded by itself)
├── LICENSE
└── .gitignore
```

There is no `.claude/agents/` folder, no `.env.example` and no placeholders to fill in. The agent lives inside Claude Code.

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
- Run Claude Code programmatically (`claude -p`): https://code.claude.com/docs/en/headless
- Permission modes: https://code.claude.com/docs/en/permission-modes
- Permissions: https://code.claude.com/docs/en/permissions
- Run prompts on a schedule: https://code.claude.com/docs/en/scheduled-tasks

---

## License

MIT. See [LICENSE](LICENSE).
