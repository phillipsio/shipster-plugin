# Shipster for Claude

Run your Shipster agents from Claude. Connect a repo, get it a team, file a goal, hear the plan, start the build, and unblock an agent that is waiting on you.

This plugin adds two servers and four skills that teach Claude how to use them. It contains no Shipster server code.

| Server | Where it runs | What it is for |
|---|---|---|
| `repo` | On your machine, in the folder your session is open in. It is the published `@shipster/cli` package, started with `npx` | Connects that repo to Shipster, then acts as the repo's agent |
| `shipster` | Hosted at `https://mcp.shipster.dev/mcp` | Shows and steers every repo on your account. Optional, and needs a token |

## Install

```bash
claude plugin marketplace add phillipsio/shipster-plugin
```

```bash
claude plugin install shipster@shipster
```

## Connect a repo

Open Claude Code in the repo and say "connect this repo to Shipster". Claude gives you a link and a code. Approve it in the browser. The repo's tools load into the same session, with no terminal command and no restart.

To get the repo a writer and a reviewer, say "get this repo a team". Claude tells you what that installs on your machine (three agents, a git worktree for each under `~/shipster/<repo>`, and a background service that starts at login), then gives you a second code to approve.

The `repo` server needs Node.js 20.12 or later.

## The connector token (optional)

The connector works across all your repos and in the Claude chat apps. In a Claude Code session, run:

```text
/plugin configure shipster@shipster
```

Paste the token at the prompt. Create one at https://shipster.dev under Settings > Claude connector. It starts with `ast_`, is shown once, and expires after the period you pick. The connector stays off until a token is set. Connecting a repo does not need it.

## What the token can do

A token only reads unless you tick a box when you create it.

| Box | What it adds |
|---|---|
| none | See what needs you, your pools and agents, recent work, a task's thread, a goal's plan |
| Can create and start goals | File a goal and start it. A goal filed this way is planned but not built until you start it, which hands the plan to your repo's orchestrator |
| Can message agents | Answer a task that is blocked waiting on you, and ask the agent on a task a question |

No token can approve a review gate, merge, or change settings. Revoke a token from the same dashboard page.

## Skills

| Skill | Use it for |
|---|---|
| `setup` | Connecting a repo, getting it a team, fixing a server that is not working |
| `check-in` | "What needs me?", "What are my agents doing?", "Why is this stuck?" |
| `goals` | Filing a goal, hearing the plan, starting the build |
| `unblock` | Answering a blocked task, asking an agent a question |

## Where it works

Claude Code (terminal, desktop app and IDE extensions) runs both servers. The Claude chat apps cannot run the `repo` server. There, add the connector by hand instead: Settings > Connectors > Add custom connector, URL `https://mcp.shipster.dev/mcp`, authentication "No sign-in", and a request header `x-api-key` with your token.

## What it sends

The `repo` server downloads `@shipster/cli` from npm and runs it on your machine. Before you connect, it sends nothing. When you ask to connect, it sends the Shipster API the repo's name, its `origin` remote address, the name of the agent CLI you are using, a count of its files in total and by language, whether it has tests and CI, and how many commits it had in the last 30 days. When you approve, it also sends the folder's path on your machine. After you approve, it works as that repo's agent: it reads and writes the tasks, messages and reviews of that repo, as you direct it. It stores the agent's credential on your machine, outside the repo. In the repo it writes the hook files that wake the agent for new work (`.claude/settings.local.json` for Claude Code; `.cursor/hooks.json` and `.cursor/rules/pitboss.mdc` for Cursor) and lists them in the checkout's local git exclude file, so git does not offer to commit them. When you ask for a team, it sends the repo's name, its `origin` remote address, the agent CLI's name and the names and roles of the three agents.

The connector calls `https://mcp.shipster.dev/mcp` with your token. It reads your Shipster tasks, goals, repos, pools and agent status, and, if you ticked the boxes, files goals and sends your answers and questions to your agents. The goal text, answers and questions are the ones you gave Claude to send. It sends nothing else from your machine or your conversation.

## Support

Privacy policy: https://shipster.dev/privacy

Questions and problems: support@shipster.dev

## License

MIT. It covers this plugin: the manifest, the server configuration and the skills. The Shipster service, its server software and the `@shipster/cli` package are separate and are not licensed by this file.
