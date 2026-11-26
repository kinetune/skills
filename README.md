<a href="https://kinetune.com/developers?utm_source=github&utm_medium=readme&utm_campaign=skills"><img src="https://kinetune.com/og/home.jpg" alt="Kinetune: your song, every video. AI music videos, lyric videos and Spotify Canvas" width="100%"></a>

<h1 align="center">Kinetune for AI agents</h1>

<p align="center">
<b>Give Claude, ChatGPT, Codex, Cursor or any MCP client a music studio.</b><br>
A skill, an MCP server and plugins that teach agents to make everything a release needs from one song.
</p>

<p align="center">
<a href="https://registry.modelcontextprotocol.io/v0/servers?search=com.kinetune/kinetune"><img src="https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%2Fcom.kinetune%252Fkinetune%2Fversions%2Flatest&query=%24.server.version&label=MCP%20Registry&prefix=v&logo=modelcontextprotocol&color=black" alt="MCP Registry"></a>
<a href="https://glama.ai/mcp/connectors/com.kinetune/kinetune"><img src="https://glama.ai/mcp/connectors/com.kinetune/kinetune/badges/score.svg" alt="Kinetune MCP connector"></a>
<a href="https://smithery.ai/servers/kinetune/kinetune"><img src="https://img.shields.io/badge/Smithery-kinetune%2Fkinetune-FF5601" alt="Smithery"></a>
<a href="https://mcp.so/servers/kinetune"><img src="https://img.shields.io/badge/mcp.so-Kinetune-black" alt="mcp.so"></a>
<a href="https://cursor.directory/plugins/kinetune"><img src="https://img.shields.io/badge/Cursor_Directory-kinetune-black?logo=cursor" alt="Cursor Directory"></a>
<a href="https://mcpservers.org/servers/kinetune-com-developers"><img src="https://img.shields.io/badge/mcpservers.org-Kinetune-black" alt="mcpservers.org"></a>
<br>
<a href="https://github.com/kinetune/skills/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-f2cf2e?labelColor=16161d" alt="License: MIT"></a>
<a href="https://kinetune.com/?utm_source=github&utm_medium=readme&utm_campaign=skills"><img src="https://img.shields.io/badge/Try_Kinetune-free-f2cf2e?labelColor=16161d" alt="Try Kinetune"></a>
</p>

<p align="center">
<a href="#quick-start">Quick start</a> ·
<a href="#install">Install</a> ·
<a href="#whats-inside">What's inside</a> ·
<a href="https://kinetune.com/developers?utm_source=github&utm_medium=readme&utm_campaign=skills">Developer docs</a>
</p>

> [!TIP]
> **New:** a song with no vocals makes a **Visualizer** (its cover and motion, no lyrics). Lyric videos take **Colours** apart from their lyric style (`look.colours`: `"style"`, `"cover"` or a set id from `get_options` `colours`). A video can leave out its clips or its song (`display.clips` / `display.song` false). And when the analysis missed a song's vocals, `transcribe_song` with `vocals: true` finds the words.

## What you can make

| | What you get |
|---|---|
| 🎬 | **AI music videos**: the artist performs the song, lip-synced, from 15 seconds to 10 minutes (MiniMax H3 Max, Seedance) |
| 🎤 | **Lyric videos**: every word on the beat, in 63 lyric styles, over clips cut to the music or a visualizer, in 9:16, 16:9 and 1:1 |
| 🔁 | **Spotify Canvas** loops |
| 🖼️ | **Cover art** made from what the song is about |
| 🎹 | **Songs made with AI**: lyrics written for you or your own, Mureka, Lyria, Eleven Music or MiniMax, even in the artist's own voice |
| 🌐 | **Artist pages** at `kinetune.com/@you`, and songs brought in from the catalog |

### Ask in plain words

> *"Make a vertical lyric video for Midnight Drive."*<br>
> *"Give my new single a Spotify Canvas, something calm and rainy."*<br>
> *"Write a song about a late drive home for Nova Lane and make two takes."*<br>
> *"Plan a music video for the chorus: her on a rooftop at night."*

The agent finds the song, tells you exactly how many credits it costs, and makes it only after you say yes.

## Quick start

**1. Install** the plugin in Claude Code (or [pick your agent](#install)):

```
/plugin marketplace add kinetune/skills
/plugin install kinetune@kinetune
```

**2. Sign in.** Run `/mcp`, choose **kinetune**, then **Authenticate** with your Kinetune account.

**3. Ask.** *"Make a vertical lyric video for Midnight Drive."*

## Install

| Where | How |
|---|---|
| **Claude Code** | `/plugin marketplace add kinetune/skills`, then `/plugin install kinetune@kinetune`. The plugin adds the Kinetune skill and connects the Kinetune MCP server. |
| **Codex, Cursor, opencode and other agents** | `npx skills add kinetune/skills` with the [skills](https://github.com/vercel-labs/skills) installer, or copy `skills/kinetune` into your agent's skills folder. |
| **ChatGPT** | Add the [Kinetune app](https://chatgpt.com/plugins/plugin_asdk_app_6ac8644fd31c8191a871ef900fb7cb41) from the app directory. |
| **Any MCP client** | Point it at `https://kinetune.com/mcp` (Streamable HTTP, OAuth sign-in). |

<details>
<summary><b>Codex, Cursor, opencode and other agents</b></summary>

<br>

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

</details>

<details>
<summary><b>MCP only</b></summary>

<br>

[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=kinetune&config=eyJ1cmwiOiJodHRwczovL2tpbmV0dW5lLmNvbS9tY3AifQ%3D%3D) [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Kinetune-0098FF?logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=kinetune&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fkinetune.com%2Fmcp%22%7D)

Point any MCP client at `https://kinetune.com/mcp` (Streamable HTTP, OAuth sign-in). Setup for Claude, Cursor and VS Code is at [kinetune.com/developers](https://kinetune.com/developers?utm_source=github&utm_medium=readme&utm_campaign=skills#mcp).

It's listed in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=com.kinetune/kinetune) as `com.kinetune/kinetune`, and on [Glama](https://glama.ai/mcp/connectors/com.kinetune/kinetune), [Smithery](https://smithery.ai/servers/kinetune/kinetune), [mcp.so](https://mcp.so/servers/kinetune), [Cursor Directory](https://cursor.directory/plugins/kinetune) and [mcpservers.org](https://mcpservers.org/servers/kinetune-com-developers).

</details>

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

| | |
|---|---|
| 📚 **Docs** | [Developer docs](https://kinetune.com/developers?utm_source=github&utm_medium=readme&utm_campaign=skills): REST API, MCP server, CLI and n8n · [Full API reference for LLMs](https://kinetune.com/llms-full.txt?utm_source=github&utm_medium=readme&utm_campaign=skills) |
| 💳 **Pricing** | [Pricing](https://kinetune.com/pricing?utm_source=github&utm_medium=readme&utm_campaign=skills): the same prices on the web and in the apps |
| ⌨️ **Terminal** | [kinetune/cli](https://github.com/kinetune/cli): the `kinetune` command line, open source |
| ⚙️ **Automations** | [kinetune/n8n-nodes-kinetune](https://github.com/kinetune/n8n-nodes-kinetune): the n8n node |
| 📱 **Apps** | The Kinetune app for [iPhone](https://apps.apple.com/app/id6821154371) and [Android](https://play.google.com/store/apps/details?id=com.kinetune.app) · [Kinetune in ChatGPT](https://chatgpt.com/plugins/plugin_asdk_app_6ac8644fd31c8191a871ef900fb7cb41) |
| 💬 **Support** | support@kinetune.com |

<p align="center"><sub>MIT licensed.</sub></p>
