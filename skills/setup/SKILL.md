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

`join` prints a code the user approves in the browser, then connects this agent and writes its MCP config. It detects Claude Code and Cursor. For any other agent, name it:

```bash
npx @shipster/cli join --agent gemini
```

The names are `claude`, `cursor`, `gemini`, `copilot`, `windsurf`, `gpt` and `mcp`. Use `mcp` for a client not on that list: `join` then prints a server command to add to that client by hand, with no secrets in it.

The approval page needs no choices in the common case:

- On a new account it asks for a repo name and connects this agent as the user's top-level agent, which coordinates every repo they add.
- On an account that already has repos it connects this agent as a worker in the repo the user picks. Role and the other options are under Advanced.

To add another agent to a repo, give it its own git worktree and run `join` there. A second agent with the same name is numbered, so two Claude Code agents become `claude` and `claude-2`. A repo takes one orchestrator; the page says so if the user picks a second.

A new account's top-level agent gets a short setup list from its first `session_start` call (connect a repo, add a planner, a writer and a reviewer, file a first goal). Walk the user through it in order.

A goal is not worked on until three more agents are running: a planner, a writer and a reviewer. The top-level agent cannot be any of them and cannot add a planner or a reviewer itself. A planner is an agent that joins with Role set to Orchestrator under Advanced, or an architect at tier 2. The shortest way to get all three is one command, which asks for one approval and then prints a `node up` command that keeps them running:

```bash
npx @shipster/cli onboard <github-url>
```

Each of those agents runs a model CLI on that machine, so the CLI must be logged in there. If `fleet_status` says agents cannot log in, the user logs in on that machine (for Claude Code: `claude`, then `/login`).

Restart the agent's session after `join`, because a running client keeps the tool list it started with. Afterwards:

```bash
npx @shipster/cli doctor
```

`doctor` runs the setup checks and prints what is missing. Read its output to the user, and fix only what they ask you to fix. A repo with one agent fails the "reviewer and a writer" check until the user adds those agents. That is expected on a first setup.

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
