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
3. They run `/plugin configure shipster@shipster` in Claude Code and paste the token at the prompt. The connector stays off until a token is set.

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

To run several agents under one supervisor, `npx @shipster/cli node up` runs them in the foreground until Ctrl-C. `npx @shipster/cli node service install` installs that as a service that survives the terminal closing. `npx @shipster/cli node --help` lists the rest. Setting one up changes the user's machine, so walk them through it and let them run each step.

## When a tool fails

A refusal comes back as a line starting "Error:" followed by a sentence. There is no code or status number to match, so read the sentence.

| What you see | What it means | What to tell the user |
|---|---|---|
| The Shipster tools are missing, or the connector will not connect | The token is wrong, expired or revoked | Create a new token and update the plugin's setting |
| "An assistant token can only read" | The token lacks the box for that tool | Create a new token with the box ticked |
| "reached its cap of ... goals" or "... messages in 24 hours" | The token's daily allowance is used up | Wait until tomorrow |
| "Too many requests in flight" | More than four calls at once | Call one tool at a time |
| Every pool reads "health unknown" | No machine is reporting | Start the node or agent on their machine |
| "Nothing needs you right now" | Not an error | Nothing is waiting |

Do not retry a refusal about the token or a cap. It will not change until the token or the day does.
