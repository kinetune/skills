<a href="https://kinetune.com/developers?utm_source=github&utm_medium=readme&utm_campaign=skills"><img src="https://kinetune.com/og/home.jpg" alt="Kinetune: your song, every video. AI music videos, lyric videos and Spotify Canvas" width="100%"></a>

# Kinetune for AI agents

[![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%2Fcom.kinetune%252Fkinetune%2Fversions%2Flatest&query=%24.server.version&label=MCP%20Registry&prefix=v&logo=modelcontextprotocol&color=black)](https://registry.modelcontextprotocol.io/v0/servers?search=com.kinetune/kinetune)
[![Kinetune MCP connector](https://glama.ai/mcp/connectors/com.kinetune/kinetune/badges/score.svg)](https://glama.ai/mcp/connectors/com.kinetune/kinetune)
[![Smithery](https://img.shields.io/badge/Smithery-kinetune%2Fkinetune-FF5601)](https://smithery.ai/servers/kinetune/kinetune)
[![mcp.so](https://img.shields.io/badge/mcp.so-Kinetune-black)](https://mcp.so/servers/kinetune)
[![Cursor Directory](https://img.shields.io/badge/Cursor_Directory-kinetune-black?logo=cursor)](https://cursor.directory/plugins/kinetune)
[![mcpservers.org](https://img.shields.io/badge/mcpservers.org-Kinetune-black)](https://mcpservers.org/servers/kinetune-com-developers)

Give Claude, ChatGPT, Codex, Cursor or any MCP client a music studio. [Kinetune](https://kinetune.com/?utm_source=github&utm_medium=readme&utm_campaign=skills) makes everything a release needs from one song, and this repo teaches agents to use it:

- **AI music videos**: the artist performs the song, lip-synced, from 15 seconds to 10 minutes (MiniMax H3 Max, Seedance)
- **Lyric videos**: every word on the beat, in 63 lyric styles, over clips cut to the music or a visualizer, in 9:16, 16:9 and 1:1
- **Spotify Canvas** loops, and **cover art** made from what the song is about
- **Songs made with AI**: lyrics written for you or your own, Mureka, Lyria, Eleven Music or MiniMax, even in the artist's own voice
- **Artist pages** at `kinetune.com/@you`, and songs brought in from the catalog

Ask in plain words:

- *"Make a vertical lyric video for Midnight Drive."*
- *"Give my new single a Spotify Canvas, something calm and rainy."*
- *"Write a song about a late drive home for Nova Lane and make two takes."*
- *"Plan a music video for the chorus: her on a rooftop at night."*

The agent finds the song, tells you exactly how many credits it costs, and makes it only after you say yes.

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

The skill works through the MCP server when it is connected, or through the [`kinetune` CLI](https://github.com/kinetune/cli) otherwise:

```
npm install -g @kinetune/cli
kinetune auth login
```

## MCP only

[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=kinetune&config=eyJ1cmwiOiJodHRwczovL2tpbmV0dW5lLmNvbS9tY3AifQ%3D%3D) [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Kinetune-0098FF?logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=kinetune&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fkinetune.com%2Fmcp%22%7D)

Point any MCP client at `https://kinetune.com/mcp` (Streamable HTTP, OAuth sign-in). It's listed in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=com.kinetune/kinetune) as `com.kinetune/kinetune`, and on [Glama](https://glama.ai/mcp/connectors/com.kinetune/kinetune), [Smithery](https://smithery.ai/servers/kinetune/kinetune), [mcp.so](https://mcp.so/servers/kinetune), [Cursor Directory](https://cursor.directory/plugins/kinetune) and [mcpservers.org](https://mcpservers.org/servers/kinetune-com-developers). In ChatGPT, add the [Kinetune app](https://chatgpt.com/plugins/plugin_asdk_app_6ac8644fd31c8191a871ef900fb7cb41) from the app directory. Setup for Claude, Cursor and VS Code is at [kinetune.com/developers](https://kinetune.com/developers#mcp).

## What's inside

| Path | What it is |
|---|---|
| `skills/kinetune/SKILL.md` | How to create songs and make lyric videos, music videos, Canvases and cover art, and the rule to quote before spending credits |
| `skills/kinetune/references/cli.md` | Every CLI command |
| `.mcp.json` | The Kinetune MCP server, for the Claude Code plugin |
| `.claude-plugin/` | The Claude Code plugin and marketplace manifests |
| `.codex-plugin/` | The plugin manifest for ChatGPT and Codex, with its listing details and review test cases |
| `assets/` | The icon and logo the plugin listings use |

## Links

- [Developer docs](https://kinetune.com/developers?utm_source=github&utm_medium=readme&utm_campaign=skills): REST API, MCP server, CLI and n8n
- [Full API reference for LLMs](https://kinetune.com/llms-full.txt)
- [Pricing](https://kinetune.com/pricing?utm_source=github&utm_medium=readme&utm_campaign=skills): the same prices on the web and in the apps
- [kinetune/cli](https://github.com/kinetune/cli): the `kinetune` command line, open source
- [kinetune/n8n-nodes-kinetune](https://github.com/kinetune/n8n-nodes-kinetune): the n8n node
- The Kinetune app for [iPhone](https://apps.apple.com/app/id6821154371) and [Android](https://play.google.com/store/apps/details?id=com.kinetune.app)
- [Kinetune in ChatGPT](https://chatgpt.com/plugins/plugin_asdk_app_6ac8644fd31c8191a871ef900fb7cb41)
- Support: support@kinetune.com

MIT licensed.
