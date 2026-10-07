# Kinetune for AI agents

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

Point any MCP client at `https://kinetune.com/mcp` (Streamable HTTP, OAuth sign-in). Setup for ChatGPT, Claude, Cursor and VS Code is at [kinetune.com/developers](https://kinetune.com/developers#mcp).

## What's inside

| Path | What it is |
|---|---|
| `skills/kinetune/SKILL.md` | How to make lyric videos and Canvases, and the rule to quote before spending credits |
| `skills/kinetune/references/cli.md` | Every CLI command |
| `.mcp.json` | The Kinetune MCP server, for the Claude Code plugin |
| `.claude-plugin/` | The Claude Code plugin and marketplace manifests |

## Links

- [Developer docs](https://kinetune.com/developers): REST API, MCP server, CLI and n8n
- [Full API reference for LLMs](https://kinetune.com/llms-full.txt)
- [Pricing](https://kinetune.com/pricing)
- Support: support@kinetune.com

MIT licensed.
