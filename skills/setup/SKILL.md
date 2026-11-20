---
name: setup
description: Connect a repo to Shipster, get it a team of agents, and fix a Shipster server that is not working. Use when the user asks to connect, set up or join Shipster, how to get started, how to add agents, or why the Shipster tools return an error or nothing.
---

# Set up Shipster

Shipster coordinates coding agents across the user's repos. The agents run on the user's own machine or server.

This plugin adds two servers:

- `repo` runs on this machine, in the folder this session is open in. It connects that repo and then acts as the repo's own agent.
- `shipster` is the hosted connector. It shows every repo and agent on the account, from any Claude app. It needs a token and is optional.

Your tool list may show a tool with a longer prefixed name that ends in the name given here.

## Connect this repo

Do this when the user asks to connect, set up or join Shipster. It needs no terminal command and no restart.

1. Call `shipster_connect`. It returns the server address, a link and a code.
2. Tell the user the server address. Ask them to open the link and confirm the code. Only the user approves. Never open the link or approve it yourself.
3. Call `shipster_connect_status` every few seconds until it reports connected. It loads the repo agent's tools into this session.
4. Call `session_start` and tell the user which repo and server this session is connected to.

What the approval page shows:

- For a repo that is new to the account, it creates the repo and connects this agent as its orchestrator, the agent that plans work and routes it.
- For a repo the account already has, it connects this agent as a worker. Role and the other options are under Advanced. A repo takes one orchestrator.
- When a person approves into a new repo, or into one they own, and has not answered yet, it asks how changes merge: whether a reviewer agent's approval is enough, and whether merging deploys to production. These are the owner's decisions. Do not suggest answers.

If `shipster_connect` says the folder is not a git checkout, the session is open in the wrong folder. If `shipster_connect` is not in your tool list and the `repo` server's instructions say the checkout is already connected, use the tools of the checkout's own Shipster server.

## Get the repo a team

An orchestrator plans work. Nothing is built until the repo also has a writer and a reviewer. When the user wants work built and `shipster_request_team` is in your tool list:

1. Tell the user what a team is before you ask for one. It is three agents (architect, reviewer and writer) that run on this machine. Each gets its own git worktree under `~/shipster/<repo>`. A background service runs them, starts again at login, and runs commands in those worktrees without asking each time.
2. If they agree, call `shipster_request_team`. It returns a second link and code.
3. The user approves on the page titled "Give your planner a team". Only the user approves.
4. Call `shipster_request_team_status` every few seconds until it reports the team. Read the user the result: which agents are running, and the command that stops the service.

`shipster_request_team` is offered only to a repo's orchestrator, on macOS or Linux, in Claude Code or Cursor, in a repo that has at least one commit. If it is not in your tool list, say which of those is missing when you can tell, and offer the terminal route below.

Each agent runs a model CLI on that machine, so the CLI must be logged in there. If the team does not pick up work, the user logs in on that machine (for Claude Code: `claude`, then `/login`). With the connector set up, `fleet_status` says when agents cannot log in.

After the team is up, file a goal with the repo agent's own tools, or with the `goals` skill when the connector is set up.

## The terminal route

Use it for a client without this plugin, for a self-hosted Shipster server, or when the `repo` server is not running. Do not run these without asking: they register agents and write config files.

```bash
npx @shipster/cli join
```

`join` prints a code the user approves in the browser, then connects this agent and writes its MCP config. It detects Claude Code and Cursor. For any other agent, name it with `--agent`: `claude`, `cursor`, `gemini`, `copilot`, `windsurf`, `gpt` or `mcp`. For a self-hosted server, add `--api <url>`.

Restart the agent's session after `join`, because a running client keeps the tool list it started with.

To add another agent to a repo, give it its own git worktree and run `join` there. A second agent with the same name is numbered, so two Claude Code agents become `claude` and `claude-2`.

```bash
npx @shipster/cli doctor
```

`doctor` runs the setup checks and prints what is missing. Read its output to the user, and fix only what they ask you to fix.

To run several agents under one supervisor, `npx @shipster/cli node up` runs them in the foreground until Ctrl-C, and `npx @shipster/cli node service install` installs that as a service. `npx @shipster/cli node --help` lists the rest. Setting one up changes the user's machine, so walk them through it and let them run each step.

## The connector (optional)

The `shipster` server lets Claude see and steer every repo on the account, including from the Claude chat apps.

1. The user signs in at https://shipster.dev and opens Settings > Claude connector.
2. They create a token. It starts with `ast_` and is shown once. Two optional boxes widen it:
   - "Can create and start goals" allows the goal tools.
   - "Can message agents" allows answering a blocked task and asking an agent a question.
   With neither ticked, the token only reads.
3. They run `/plugin configure shipster@shipster` in Claude Code and paste the token at the prompt. The connector stays off until a token is set.

A token expires (the user picks 7, 30 or 90 days) and can be revoked from the same page. Never ask the user to paste a token into the chat.

## When something fails

A refusal comes back as a line starting "Error:" followed by a sentence. There is no code or status number to match, so read the sentence.

| What you see | What it means | What to tell the user |
|---|---|---|
| `shipster_connect` is not in the tool list, and the repo is not connected | The `repo` server did not start. It needs Node.js 20.12 or later and `npx` on the PATH | Check `/mcp` for the `repo` server's error, or use the terminal route |
| The `repo` server lists no tools and its instructions say the checkout is already connected | `join` set this checkout up earlier, through its own `.mcp.json` | Use the tools of the checkout's own Shipster server. Nothing is wrong |
| "This client entry names a Shipster server ... has not logged in to" | The session points at a self-hosted server this machine has not used | Use the terminal route with `--api <url>` |
| The connector tools are missing, or the connector will not connect | The token is unset, wrong, expired or revoked | Create a new token and run `/plugin configure shipster@shipster` |
| "An assistant token can only read" | The token lacks the box for that tool | Create a new token with the box ticked |
| "reached its cap of ... goals" or "... messages in 24 hours" | The token's daily allowance is used up | Wait until tomorrow |
| "Too many requests in flight" | More than four calls at once | Call one tool at a time |
| Every pool reads "health unknown" | No machine is reporting | Start the node or agent on their machine |
| "Nothing needs you right now" | Not an error | Nothing is waiting |

Do not retry a refusal about the token or a cap. It will not change until the token or the day does.
