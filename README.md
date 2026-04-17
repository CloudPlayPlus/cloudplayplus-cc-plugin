# cloudplayplus

Bridges Claude Code to a running [CloudPlayPlus](https://www.cloudplayplus.com/)
Flutter app. Messages from remote CloudPlayPlus users are pushed into Claude
Code as channel events; Claude's replies go back through the `reply` tool.

Unlike `fakechat` / `discord` / `telegram`, this plugin carries **no server
code of its own**. The MCP server runs inside the long-running CloudPlayPlus
Flutter app (via the [`cloudplayplus_agent`](https://github.com/CloudPlayPlus/cloudplayplus_agent)
Dart package) and listens on `http://127.0.0.1:48989/mcp`. This plugin just
tells Claude Code where to connect.

## Prerequisites

1. A running CloudPlayPlus Flutter app exposing the MCP HTTP server on
   `http://127.0.0.1:48989/mcp`.
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
