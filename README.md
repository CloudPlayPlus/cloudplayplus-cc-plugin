# cloudplayplus

Channel plugin that bridges Claude Code to a long-running local Flutter app.
Messages typed in the app are pushed into Claude Code as channel events; the
assistant's replies go back through the `reply` tool and render in the app.

This repo ([CloudPlayPlus/cloudplayplus-cc-plugin](https://github.com/CloudPlayPlus/cloudplayplus-cc-plugin))
is just the plugin manifest — no runtime code. The MCP server lives in a
separate Dart package, [`cloudplayplus_agent`](https://github.com/CloudPlayPlus/cloudplayplus_agent),
which ships a ready-to-run Flutter example you can build on macOS / Windows /
Linux. The example binds `http://127.0.0.1:48989/mcp` and is exactly what
the plugin points at. Eventually the full [CloudPlayPlus](https://www.cloudplayplus.com/)
product will embed this package so remote CloudPlayPlus users can chat with
a Claude Code session on the host machine, but the Flutter example already
shows the whole flow end-to-end.

## Prerequisites

1. A Flutter app running the [`cloudplayplus_agent`](https://github.com/CloudPlayPlus/cloudplayplus_agent)
   MCP HTTP server on `http://127.0.0.1:48989/mcp`. The
   [`example/`](https://github.com/CloudPlayPlus/cloudplayplus_agent/tree/main/example)
   directory in that repo has a minimal chat UI you can `flutter run` right
   now; any host app that embeds `CloudplayAgent` works.
2. Claude Code 2.1 or later.

## Setup

Run `claude` to start a session, then:

```
/plugin marketplace add CloudPlayPlus/cloudplayplus-cc-plugin
/plugin install cloudplayplus@cloudplayplus
```

**Relaunch with the channel flag** — the plugin won't activate without it.
Exit your session and start a new one:

```sh
claude --channels plugin:cloudplayplus@cloudplayplus
```

Start the Flutter app. Type messages from any paired CloudPlayPlus client —
they arrive in Claude Code as channel events. Claude's replies show up back
in the CloudPlayPlus UI.

## Tools

| Tool | Purpose |
| --- | --- |
| `reply` | Send a reply back to the CloudPlayPlus UI. Takes `chat_id` + `text`; optionally `reply_to` (message ID) and `files`. |
| `react` | Attach an emoji reaction to an existing message. |
| `edit_message` | Edit a previously-sent message in place. |
| `fetch_messages` | Look back at recent messages in a chat. |
| `download_attachment` | Materialize attachments of a specific message as local files. |

## Skip permission prompts for channel tools

Because this plugin connects over a local HTTP MCP endpoint (not a Claude
Code-spawned stdio subprocess like `fakechat` / `discord`), Claude Code
treats the tools as ordinary MCP calls and asks for permission on first
use. Pre-approve all five channel tools once by adding this to
`~/.claude/settings.json` (user scope) or `.claude/settings.local.json` in
your project:

```json
{
  "permissions": {
    "allow": [
      "mcp__plugin_cloudplayplus_cloudplayplus__*"
    ]
  }
}
```

Rule name format is `mcp__plugin_<plugin>_<server>__<tool>`; the wildcard
covers `reply` / `react` / `edit_message` / `fetch_messages` /
`download_attachment`. You can also click **Yes, don't ask again** the
first time each prompt appears — Claude Code writes the same rules itself.

## Security

CloudPlayPlus authenticates remote users before forwarding their messages to
the MCP layer. The MCP server itself binds to `127.0.0.1` and declares
`experimental.claude/channel/permission`, meaning it can also relay tool-use
permission prompts back to the user. Your CloudPlayPlus transport is
responsible for gating those prompts to authorized users only.

## How it works

- `.claude-plugin/marketplace.json` — declares this repo as a Claude Code
  marketplace containing the `cloudplayplus` plugin.
- `plugins/cloudplayplus/.claude-plugin/plugin.json` — plugin metadata.
- `plugins/cloudplayplus/.mcp.json` — points Claude Code at the Streamable
  HTTP endpoint the Flutter app exposes. No stdio process is spawned.

## License

Apache-2.0
