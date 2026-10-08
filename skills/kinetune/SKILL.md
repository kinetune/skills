---
name: kinetune
description: Make lyric videos (9:16, 16:9, 1:1) and Spotify Canvas loops from a user's songs with Kinetune, through its MCP tools or the `kinetune` CLI. Use when the user wants a lyric video, a Spotify Canvas, a visualizer or promo video for a song, wants to upload a song or artist photos to Kinetune, browse Looks, check their credits, or mentions Kinetune.
---

# Kinetune

Kinetune turns a song into release-ready video:

- **Lyric videos**: word-synced lyrics over a designed background with audio-reactive visuals, in 9:16 (vertical), 16:9 (wide) and 1:1 (square).
- **Spotify Canvas**: a 5–8 second vertical loop designed from the song's cover, uploaded by the artist in Spotify for Artists.

Every video uses a **Look**, a complete reusable design. Reuse a saved Look, or have a new one designed from the song's cover art.

## The one rule: quote, tell, wait for the go

Creating a video spends the user's credits. Always:

1. Quote the exact request (`quote_lyric_video` / `quote_canvas`, or `kinetune quote …`).
2. Tell the user the credits, what they get, and their balance.
3. Create only after they say yes, with the **same** input plus the `quote_id`. A quote is valid for 30 minutes.

Never create without a quote and the user's agreement, and never pass the CLI's `--yes` on your own. If the user has already agreed to a budget, pass `--max-credits N` so the CLI refuses anything pricier.

## Pick the interface

1. **MCP tools** if they are connected (`get_account`, `list_songs`, `quote_canvas`…). The MCP server is `https://kinetune.com/mcp`; the user signs in with their Kinetune account.
2. **The CLI** for local agents and scripts: `kinetune` (`npm install -g @kinetune/cli`, or `npx @kinetune/cli …` without installing). Check sign-in with `kinetune auth status`; if not signed in, ask the user to run `kinetune auth login` (it opens their browser) or set `KINETUNE_API_KEY`. Every command prints JSON when piped. See [references/cli.md](references/cli.md).
3. **REST** at `https://kinetune.com/api/v1` with `Authorization: Bearer <key>`, only when writing code for the user. The spec is at `https://kinetune.com/api/v1/openapi.json`.

The three are the same operations with the same inputs, so the steps below name the MCP tool and the CLI command together.

## Make a video

1. **Check the account**: `get_account` / `kinetune account`. Note the credit balance.
2. **Find the song**: `list_songs` with `q` (part of the title or artist) / `kinetune songs list --q "…"`. Only songs with `analysis.status` `"ready"` can be used. Lists return 50 per page; page with `limit`/`offset` rather than listing everything.
3. **No song yet?** Find or create the artist (`list_artists` / `create_artist`), then upload: `create_song` / `kinetune songs create --artist-id … --title … --audio ./song.wav --cover-art ./cover.jpg`. Audio is MP3, WAV or M4A up to 250 MB; the cover is a square JPEG or PNG, 1000–6000 px (3000×3000 recommended). Analysis (lyrics, beats, sections) takes about a minute: poll `get_song` until `analysis.status` is `"ready"`. If it fails, `reanalyze_song`.
4. **Choose what to make** (below), then **quote** it and tell the user the credits.
5. **Create** with the same input plus `quote_id`: `create_lyric_video` / `create_canvas`, or `kinetune create … --quote-id …`. It returns the video ids at once.
6. **Wait**: `wait_for_video` (call again while `status` is `queued` or `processing`) / `kinetune videos wait VIDEO_ID`.
7. **Deliver**: a completed video has one output per format with `download_url` (full quality), `web_url` (720p) and `poster_url`, signed for 7 days. Share the links, or `kinetune videos download VIDEO_ID --out ./videos`.

## Choosing a lyric video

Read `get_options` / `kinetune options` for the current categories, background sources, quality tiers, headlines and credit prices. Don't guess ids.

- **Look**:
  - `{"mode":"existing","id":"look_…"}` reuses a saved Look exactly. It is the cheapest choice. Browse with `list_looks` (`scope`: `official`, `community`, `mine`; filter by `category`; pass the song's `artist_id` to leave out Looks that show another artist).
  - `{"mode":"new","category":"…","background":{"source":"ai-image","quality":"high"},"direction":"…","visibility":"private"}` has a new Look designed from the cover.
    - `direction` is the user's mood or references in plain words.
    - The singer stays out of the background unless asked: `"feature_artist":"always"` shows them. It needs an AI background, artist photos (`add_artist_photos`), `"display":{"cover":false}` (the cover would hide them) and a private Look, which then only serves that artist's songs.
- **Background sources** for a new Look: `ai-image`, `ai-video` (an AI image brought to life as a loop; costs more), `stock-photo`, `stock-video`. AI sources take a `quality` tier.
- **`variations`** (1–4): several different New Looks, one video each.
- **`aspect_ratios`**: any of `"9:16"`, `"16:9"`, `"1:1"`, each its own file. Each extra format adds credits.
- **`display`**: which elements show (`title`, `artist`, `cover`, `lyrics`, `badges`, `headline`) and their sizes.
- **`trim`**: render only part of the song (at least 8 seconds).

Ask the user only what matters to them, usually the category or mood, the formats, and whether to reuse a Look. Default the rest.

## Choosing a Canvas

- **`source`**: `ai-video` (default, a video model animates an image made from the cover's world), `ai-image`, `stock-photo`, `stock-video`.
- **`seconds`**: 5–8, default 8. Spotify loops it, so no part of the song is chosen.
- **Optional**: `style` (`cinematic`, `dreamy`, `abstract`, `surreal`, `retro`, `minimal`, `dark`, `vibrant`, `nature`, `urban`), `direction`, `feature_artist` (`auto`, `always`, `never`; `always` needs artist photos), `quality` (`standard` or `high`), and `resolution` (`1080p` or `720p`).
- **`variations`**: 1–4 different Canvases.

## When something goes wrong

- **A failed video returns its credits.**
- **`retry_video`** makes a video again. That is a new charge, so ask first, but it reuses the Look and media so nothing already made is paid for twice.
- **`cancel_video`** stops a queued or processing video and releases its credits.
- **Not enough credits**: tell the user their balance and the quote. Plans and credit packs are at https://kinetune.com/pricing. Don't shrink the request without asking.
- **CLI exit codes**:
  - `0`: done
  - `1`: API or network error
  - `2`: invalid usage, or the quote is over `--max-credits`
  - `3`: not signed in or not allowed
- **Unsure of an input?** `kinetune schema "quote canvas"` prints the exact JSON schema of any command.
- **Full reference**: every operation with its parameters, scopes, CLI command and MCP tool is at https://kinetune.com/llms-full.txt.
