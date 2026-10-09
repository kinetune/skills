# Kinetune for AI agents

[![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%2Fcom.kinetune%252Fkinetune%2Fversions%2Flatest&query=%24.server.version&label=MCP%20Registry&prefix=v&logo=modelcontextprotocol&color=black)](https://registry.modelcontextprotocol.io/v0/servers?search=com.kinetune/kinetune)
[![Kinetune MCP connector](https://glama.ai/mcp/connectors/com.kinetune/kinetune/badges/score.svg)](https://glama.ai/mcp/connectors/com.kinetune/kinetune)
[![Smithery](https://img.shields.io/badge/Smithery-kinetune%2Fkinetune-FF5601)](https://smithery.ai/servers/kinetune/kinetune)
[![mcp.so](https://img.shields.io/badge/mcp.so-Kinetune-black)](https://mcp.so/servers/kinetune)
[![Cursor Directory](https://img.shields.io/badge/Cursor_Directory-kinetune-black?logo=cursor)](https://cursor.directory/plugins/kinetune)
[![mcpservers.org](https://img.shields.io/badge/mcpservers.org-Kinetune-black)](https://mcpservers.org/servers/kinetune-com-developers)

[Kinetune](https://kinetune.com) turns a song into release-ready video: lyric videos with word-synced lyrics and audio-reactive visuals in 9:16, 16:9 and 1:1, and Spotify Canvas loops designed from the song's cover.

This repository teaches AI agents to use it. Ask in plain words, for example *"make a vertical lyric video for Midnight Drive"* or *"give my new single a Canvas"*. The agent finds the song, tells you exactly how many credits it costs, and makes the video only after you say yes.

## Claude Code

```
/plugin marketplace add kinetune/skills
/plugin install kinetune@kinetune
```

The plugin adds the Kinetune skill and connects the Kinetune MCP server. Run `/mcp`, choose **kinetune**, then **Authenticate** to sign in with your Kinetune account.

## Codex, Cursor, opencode and other agents

Install the skill with the [skills](https://github.com/vercel-labs/skills) installer:

```
npx skills add kinetune/skills
```

You can also copy `skills/kinetune` into your agent's skills folder.

The skill works through the MCP server when it is connected, or through the [`kinetune` CLI](https://www.npmjs.com/package/@kinetune/cli) otherwise:

```
npm install -g @kinetune/cli
kinetune auth login
```

## MCP only

[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=kinetune&config=eyJ1cmwiOiJodHRwczovL2tpbmV0dW5lLmNvbS9tY3AifQ%3D%3D) [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Kinetune-0098FF?logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=kinetune&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fkinetune.com%2Fmcp%22%7D)

Point any MCP client at `https://kinetune.com/mcp` (Streamable HTTP, OAuth sign-in). It's listed in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=com.kinetune/kinetune) as `com.kinetune/kinetune`, and on [Glama](https://glama.ai/mcp/connectors/com.kinetune/kinetune), [Smithery](https://smithery.ai/servers/kinetune/kinetune), [mcp.so](https://mcp.so/servers/kinetune), [Cursor Directory](https://cursor.directory/plugins/kinetune) and [mcpservers.org](https://mcpservers.org/servers/kinetune-com-developers). Setup for ChatGPT, Claude, Cursor and VS Code is at [kinetune.com/developers](https://kinetune.com/developers#mcp).

## What's inside

| Path | What it is |
|---|---|
| `skills/kinetune/SKILL.md` | How to make lyric videos and Canvases, and the rule to quote before spending credits |
| `skills/kinetune/references/cli.md` | Every CLI command |
| `.mcp.json` | The Kinetune MCP server, for the Claude Code plugin |
| `.claude-plugin/` | The Claude Code plugin and marketplace manifests |
| `.codex-plugin/` | The plugin manifest for ChatGPT and Codex, with its listing details and review test cases |
| `assets/` | The icon and logo the plugin listings use |

## Links

- [Developer docs](https://kinetune.com/developers): REST API, MCP server, CLI and n8n
- [Full API reference for LLMs](https://kinetune.com/llms-full.txt)
- [Pricing](https://kinetune.com/pricing)
- Support: support@kinetune.com

MIT licensed.
