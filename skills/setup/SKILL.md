---
name: setup
description: Set up Shipster for a new user or a new repo, and fix a connector that is not working. Use when the user asks how to get started with Shipster, how to connect a repo or an agent, or why the Shipster tools return an error or nothing.
---

# Set up Shipster

Shipster coordinates coding agents across the user's repos. There are two halves: the agents, which run on the user's own machine or server, and this connector, which lets Claude see and steer them.

## The connector (this plugin)

1. The user signs in at https://shipster.dev and opens Settings > Claude connector.
2. They create a token. It starts with `ast_` and is shown once. Two optional boxes widen it:
   - "Can create and start goals" allows the goal tools.
   - "Can message agents" allows answering a blocked task and asking an agent a question.
   With neither ticked, the token only reads.
3. They paste the token when this plugin asks for it.

A token expires (the user picks 7, 30 or 90 days) and can be revoked from the same page. Never ask the user to paste a token into the chat.

## Agents in a repo

Run these in the repo's working copy. Do not run them without asking: `join` registers an agent and writes config files.

```bash
npx @shipster/cli join
```

`join` prints a code the user approves in the browser, then connects this machine's agents and writes their MCP config. Afterwards:

```bash
npx @shipster/cli doctor
```

`doctor` runs the setup checks and prints what is missing. Read its output to the user, and fix only what they ask you to fix.

To keep agents running without a terminal open, `shipster node` runs them as a supervised service. `shipster node --help` lists its commands. Setting one up changes the user's machine, so walk them through it and let them run each step.

## When a tool fails

| What you see | What it means | What to tell the user |
|---|---|---|
| 401 | The token is wrong, expired or revoked | Create a new token and update the plugin's setting |
| 403 on a goal or message tool | The token lacks that capability | Create a new token with the box ticked |
| 429 | Too many requests at once, or a daily cap | Wait, then try again |
| Every pool reads "health unknown" | No machine is reporting | Start the node or agent on their machine |
| "Nothing needs you right now" | Not an error | Nothing is waiting |

Do not retry a 401 or 403. It will not change until the token does.
