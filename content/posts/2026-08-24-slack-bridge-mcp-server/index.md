---
title: An MCP Server That Bridges a Resident Claude Code Session to Slack
slug: "slack-bridge-mcp-server"
description: An MCP server that bridges a resident Claude Code session to a Slack channel. The whole back-and-forth of instructing the LLM stays in the channel, so teammates can give feedback mid-flight even on solo work.
date: "2026-08-24T02:00:00+09:00"
public: true
tags: ["go","oss","mcp","slack","claude-code"]
archives: ["2026-08"]
alternate: true
---

I released [slack-bridge-mcp-server](https://github.com/ngs/slack-bridge-mcp-server), an MCP server that bridges a resident Claude Code session on my Mac to a Slack channel.

I send a message from Slack, the local session picks it up, does the work, and replies where I sent it.

<!--more-->

## Motivation

When you code alone with an LLM, the entire process of instructing it stays inside your terminal where nobody else can see it.

Teammates can see the resulting pull request, but not why the design went one way or where the plan changed mid-course, so the earliest they can give feedback is after everything is done.

If the instructions happen as a conversation in a Slack channel instead, the process itself becomes visible, and people can push back while things are still in flight.

The other motivation was the billing boundary: rather than running a bot on the metered API, I wanted to reach the session that is already running inside my subscription.

Being able to say "keep going on that thing" from the sofa turned out to be a nice side effect.

## Usage

### The resident loop

You give the session a loop like this (the full prompt is in the README).

```
You are bridged to my Slack via the slack-bridge MCP server. Run this loop and
do not stop:

1. Call slack_wait.
2. If it returns timed_out, go back to step 1.
3. For each message: do what it asks, then reply with slack_post. ...
```

### Signals during a turn

From there it behaves like a well-mannered coworker:

- The server reacts with 👀 on its own the moment the session takes a message in — immediately when it is waiting, at the next wait when it is mid-task
- If a reply takes more than ten seconds, "⏳ Working… (1m 05s)" appears where the turn is happening — on the channel surface, or in the thread when the exchange is threaded — keeps counting, and disappears with the reply
- When the session starts something long, `slack_progress` puts the reason next to the timer: "⏳ Working… (2m 10s) — waiting for CI"
- When it needs a decision, `slack_ask` posts buttons and the tapped choice goes straight back to the session
- Ask it to "summarize the discussion above" and it reads everyone's messages through `slack_history`

### Conversations outside the home channel

Outside the home channel it works by mention: in any channel the app has been added to, mentioning it opens a conversation — the thread under that message when you mention it on the channel surface, the thread itself when you mention it inside one — and from then on you talk there without mentioning it again.

Each message arrives with the channel it was sent in, and the session passes that back when it posts, so the reply lands where the conversation is instead of in the home channel.

Only your own messages are ever relayed, in every channel — a colleague mentioning the app does not reach the session.

Catch-up out there is best effort: at most twenty open threads are re-read per reconnect, and new mentions are looked for only in the newest hundred messages of up to twenty channels the app belongs to.

### The trust boundary

The owner filter is exactly as strong as the Slack account and the signed-in device behind it, and what it guards is an agent with local tool access, so treat access to the channel as terminal access on the machine and set the session's tool permissions accordingly.

### Recovery after sleep

Sleep is not a failure mode: messages stay in Slack, and the session catches up from its cursor on wake, thread replies included — it scans the newest 200 messages for replied-to threads and reads at most 20 of them per reconnect, past which a reply is genuinely missed.

### Release day

Most of what I just described — the progress label, the buttons, the history reader, thread-reply recovery — was implemented on release day, by me sending instructions from my phone.

I shipped v0.1.0 in the early afternoon, kept filing bug reports and feature requests from Slack, and the project reached v0.2.1 the same day, with the entire exchange still sitting in the channel.

## Installation

```bash
brew tap ngs/tap
brew install slack-bridge-mcp-server
```

The from-scratch setup guide, including a ready-made Slack app manifest with the right scopes, lives in [docs/setup.md](https://github.com/ngs/slack-bridge-mcp-server/blob/main/docs/setup.md).

The full version of the loop, packaged as a ready-to-use Claude Code skill, ships with the repository as [examples/attend/SKILL.md](https://github.com/ngs/slack-bridge-mcp-server/blob/main/examples/attend/SKILL.md).

Set four environment variables, register the server in Claude Code's `.mcp.json`, and you are done.

```json
{
  "mcpServers": {
    "slack-bridge": {
      "command": "slack-bridge-mcp-server",
      "args": []
    }
  }
}
```

## Under the hood

It is written in Go with the official [MCP Go SDK](https://github.com/modelcontextprotocol/go-sdk).

The server runs as a stdio child of Claude Code, so there is no daemon, no port, and no launchd — it lives and dies with the session.

The Socket Mode WebSocket is opened lazily on the first tool call that needs Slack, so having the server in every project's `.mcp.json` costs nothing until a session actually attends.

Reliability comes from catch-up rather than redelivery: the timestamp of the last message handed to the session is persisted as a cursor, and on reconnect the server walks `conversations.history` and `conversations.replies` forward from it.

`slack_wait` blocks for 300 seconds by default and 1,500 at most, a number worked back from the 30-minute idle limit Claude Code applies to MCP tool calls.

The closest project I know is [claude-slack-bridge](https://github.com/tomeraitz/claude-slack-bridge), a Docker daemon that both lets Claude ask a question mid-task through `ask_on_slack` and spawns a fresh `claude -p` run for every mention in Slack.

This one talks to a session that is already running instead of starting one per message, with no daemon and with sleep tolerance built out of the cursor.

## Feedback

Bug reports, feature requests, and pull requests are all welcome.

https://github.com/ngs/slack-bridge-mcp-server

Being able to ask your own session "how is that going?" from Slack is more pleasant than I expected — if you keep a resident session around, give it a try 📱
