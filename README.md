# CloudPlayPlus — Claude Code plugin

Bridges Claude Code to a long-running local [CloudPlayPlus](https://www.cloudplayplus.com/) Flutter app over an MCP HTTP channel. Messages from remote CloudPlayPlus users are pushed to CC as channel notifications; CC replies go back through the `reply` tool.

## Prerequisites

1. A running CloudPlayPlus Flutter app exposing the MCP HTTP server on `http://127.0.0.1:7823/mcp`. The `cloudplayplus_agent` Dart/Flutter package starts it automatically.
2. Claude Code 2.1+.

## Install

```sh
claude plugin marketplace add cloudplayplus-local <owner>/<repo>
claude plugin install cloudplayplus@cloudplayplus-local --scope user
```

## Run

Start the Flutter app first, then:

```sh
claude --channels plugin:cloudplayplus@cloudplayplus-local
```

Messages from the CloudPlayPlus UI now push into Claude Code as channel events; Claude's replies appear back in the UI.

## How it works

- `.claude-plugin/marketplace.json` — declares this repo as a Claude Code marketplace containing the `cloudplayplus` plugin.
- `plugins/cloudplayplus/.claude-plugin/plugin.json` — plugin metadata.
- `plugins/cloudplayplus/.mcp.json` — tells CC to connect to `http://127.0.0.1:7823/mcp` over Streamable HTTP. No stdio spawning, because the Flutter app is long-running and hosts its own MCP server via `mcp_dart`.

The plugin itself carries no code — all logic lives in the Flutter app. Declaring it as a plugin (rather than a plain MCP server registered via `claude mcp add-json`) is what makes CC trust it enough to activate `experimental.claude/channel` push notifications without `--dangerously-load-development-channels`.
