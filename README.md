# Shipster for Claude

Run your Shipster agents from Claude. Ask what needs you, file a goal, hear the plan, start the build, and unblock an agent that is waiting on you.

This plugin adds the Shipster connector and four skills that teach Claude how to use it. It contains no Shipster server code.

## Install

```bash
claude plugin marketplace add phillipsio/shipster-plugin
```

```bash
claude plugin install shipster@shipster
```

Claude asks for a connector token when you enable the plugin. Create one at https://shipster.dev under Settings > Claude connector. It starts with `ast_`, is shown once, and expires after the period you pick.

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
| `setup` | Getting started, connecting a repo, fixing a connector error |
| `check-in` | "What needs me?", "What are my agents doing?", "Why is this stuck?" |
| `goals` | Filing a goal, hearing the plan, starting the build |
| `unblock` | Answering a blocked task, asking an agent a question |

## Where it works

Claude Code (terminal, desktop app and IDE extensions) asks for the token and connects. In the Claude chat apps, add the connector by hand instead: Settings > Connectors > Add custom connector, URL `https://mcp.shipster.dev/mcp`, authentication "No sign-in", and a request header `x-api-key` with your token.

## What it sends

The connector calls `https://mcp.shipster.dev/mcp` with your token. It reads your Shipster tasks, goals, repos, pools and agent status, and, if you ticked the boxes, files goals and sends your answers and questions to your agents. The goal text, answers and questions are the ones you gave Claude to send. It sends nothing else from your machine or your conversation.

## Support

Privacy policy: https://shipster.dev/privacy

Questions and problems: support@shipster.dev

## License

MIT. It covers this plugin: the manifest, the connector configuration and the skills. The Shipster service and its server software are separate and are not licensed by this file.
