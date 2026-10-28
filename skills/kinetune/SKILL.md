---
name: kinetune
description: Make lyric videos (9:16, 16:9, 1:1), Spotify Canvas loops and AI music videos from a user's songs with Kinetune, through its MCP tools or the `kinetune` CLI. Use when the user wants a lyric video, a Spotify Canvas, a music video, a visualizer or promo video for a song, wants to set up an artist's kit (outfits, places, props), wants to upload a song or artist photos to Kinetune, browse Looks, check their credits, or mentions Kinetune.
---

# Kinetune

Kinetune turns a song into release-ready video:

- **Lyric videos**: word-synced lyrics over a designed background with audio-reactive visuals, in 9:16 (vertical), 16:9 (wide) and 1:1 (square).
- **Spotify Canvas**: a 5–8 second vertical loop designed from the song's cover, uploaded by the artist in Spotify for Artists.
- **AI music videos**: the song's artist performs it, lip-synced, in a video made from their photos, from 15 seconds to 10 minutes. See [Music videos](#music-videos).

Lyric videos and Canvases use a **Look**, a complete reusable design. Reuse a saved Look, or have a new one designed from the song's cover art.

## The one rule: quote, tell, wait for the go

Creating a video spends the user's credits. Always:

1. Quote the exact request (`quote_lyric_video` / `quote_canvas` / `quote_music_video`, or `kinetune quote …`).
2. Tell the user the credits, what they get, and their balance.
3. Create only after they say yes, with the **same** input plus the `quote_id`. A quote is valid for 30 minutes.

Never create without a quote and the user's agreement, and never pass the CLI's `--yes` on your own. If the user has already agreed to a budget, pass `--max-credits N` so the CLI refuses anything pricier.

## Use the Kinetune tools

When Kinetune is installed as a plugin or connector (ChatGPT, Claude, Cursor, VS Code…), its tools are already available: `get_account`, `list_songs`, `quote_canvas` and the rest. Use them and nothing else.

Apps that draw cards (ChatGPT, Claude and others with MCP Apps) show Kinetune's results as cards, so show rather than describe:

- `show_looks` when the person wants to see or pick Looks; each plays as the real render. Pass `song_id` to preview them with their song. Their pick arrives as a message naming the Look.
- `show_songs` when they want to see or pick a song.
- A quote draws as an order ticket with a **Make it** button. Tapping it is their yes: the card creates the videos itself, so don't create them again.
- `show_video` right after `create_lyric_video`, `create_canvas` or `create_music_video`, with the ids it returned. The card follows the render and offers the downloads, so you don't need `wait_for_video` then.

If the tools aren't available in a chat app, or anywhere Kinetune was added as a plugin or connector, the user hasn't connected it yet. Ask them to connect Kinetune and sign in with their Kinetune account; their app shows a Connect or Sign in step for it. Don't install anything, and don't switch to the CLI or the API instead. When unsure, ask them to connect.

Without a plugin or connector, two other ways reach the same operations:

- **The CLI**, when the user works in a local coding agent with a terminal and installed only this skill, or asks for the CLI: `kinetune` (`npm install -g @kinetune/cli`, or `npx @kinetune/cli …` without installing). Check sign-in with `kinetune auth status`; if not signed in, ask the user to run `kinetune auth login` (it opens their browser) or set `KINETUNE_API_KEY`. Every command prints JSON when piped. See [references/cli.md](references/cli.md).
- **REST** at `https://kinetune.com/api/v1` with `Authorization: Bearer <key>`, only when writing code for the user. The spec is at `https://kinetune.com/api/v1/openapi.json`.

They take the same inputs as the tools, so the steps below name each tool with its CLI command.

## Make a video

1. **Check the account**: `get_account` / `kinetune account`. Note the credit balance, the workspace's `type` and whether its plan is `active`. Uploading and making videos need an active plan: without one, tell the user to choose a plan in the Kinetune app (Settings, Billing) before going further.
2. **Find the song**: `list_songs` with `q` (part of the title or artist) / `kinetune songs list --q "…"`. Only songs with `analysis.status` `"ready"` can be used. Lists return 50 per page; page with `limit`/`offset` rather than listing everything.
3. **No song yet?** Find or create the artist (`list_artists` / `create_artist`), then upload: `create_song` / `kinetune songs create --artist-id … --title … --audio ./song.wav --cover-art ./cover.jpg`. In an artist workspace (`type` `"artist"`, one artist) leave `artist_id` out: its songs are its artist's. Audio is MP3, WAV or M4A up to 250 MB; the cover is a square JPEG or PNG, 1000–6000 px (3000×3000 recommended). Analysis (lyrics, beats, sections) takes about a minute: poll `get_song` until `analysis.status` is `"ready"`. If it fails, `reanalyze_song`. If the user has the song's lyrics, pass them too (see [The song's own lyrics](#the-songs-own-lyrics)).
4. **Choose what to make** (below), then **quote** it and tell the user the credits.
5. **Create** with the same input plus `quote_id`: `create_lyric_video` / `create_canvas`, or `kinetune create … --quote-id …`. It returns the video ids at once.
6. **Wait**: `wait_for_video` (call again while `status` is `queued` or `processing`) / `kinetune videos wait VIDEO_ID`. While it waits, `queue` says why: `plan_limit` (the workspace already makes as many videos at once as its plan allows; `position` is its place in line) or `render_capacity` (every render machine is busy; `estimated_wait_seconds` is about how long). Tell the user rather than calling it stuck.
7. **Deliver**: a completed video has one output per format with `download_url` (full quality), `web_url` (720p) and `poster_url`, signed for 7 days. Share the links, or `kinetune videos download VIDEO_ID --out ./videos`. A lyric video's poster is a frame with one whole lyric line on screen, made for the thumbnail where the user posts it; `poster_seconds` says where in the file it was cut, for platforms that take a cover frame offset instead of an image (TikTok).

## The song's own lyrics

Songs are transcribed to time every word, and a transcription can mishear one. When the user has the song's lyrics (pasted, Suno's lyrics, or an `.srt`, `.vtt` or `.lrc` file), give them to Kinetune: its videos then spell every word their way and break lines where they do, still timed from the audio.

- With the upload: `lyrics` on `create_song` / `kinetune songs create … --lyrics-file ./song.txt`.
- Later, or to fix them: `set_song_lyrics` / `kinetune songs set-lyrics SONG_ID --lyrics-file ./song.srt`. `clear_song_lyrics` / `kinetune songs clear-lyrics SONG_ID` goes back to the transcribed words.
- Pass the text as the user has it: section tags like `[Chorus]` are fine, and a part sung again can be written once.
- `get_song` → `analysis.lyrics.status`: `transcribed` (as heard), `checked` (theirs, timed from the audio; `corrected` and `added` say what changed), `waiting` (checked once the analysis finishes) or `mismatch`.
- Lyrics that don't match what's sung, or cover only part of the song, are refused with the reason and the song stays as it was: tell the user what it says.
- Videos made from then on use the lyrics. Videos already made stay as they are; making one again picks them up.

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
- **`trim`**: render only part of the song (at least 8 seconds). Take it from the song's **parts** (`get_song_analysis` / `kinetune songs analysis SONG_ID`): each section (Verse 1, Chorus 1…), clips of about 15 seconds cut between the lines of long ones, and each verse with its chorus, every one starting and ending between lines so no word is cut. Pass a part's `start` and `end` as `start_seconds` and `end_seconds`; `default_part` is the one the app starts on (the sung part nearest 15 seconds, the cheapest clip). Suggest a part by its label and first words ("Chorus 1, “Didn't I tell you…”, 15 s") rather than raw times.

Ask the user only what matters to them, usually the category or mood, the formats, and whether to reuse a Look. Default the rest.

## Choosing a Canvas

- **`source`**: `ai-video` (default, a video model animates an image made from the cover's world), `ai-image`, `stock-photo`, `stock-video`.
- **`seconds`**: 5–8, default 8. Spotify loops it, so no part of the song is chosen.
- **Optional**: `style` (`cinematic`, `dreamy`, `abstract`, `surreal`, `retro`, `minimal`, `dark`, `vibrant`, `nature`, `urban`), `direction`, `feature_artist` (`auto`, `always`, `never`; `always` needs artist photos), `quality` (`standard` or `high`), and `resolution` (`1080p` or `720p`).
- **`variations`**: 1–4 different Canvases.

## Music videos

The song's artist performs it, lip-synced, in a video made by **MiniMax H3 Max** (model id `one-take`) from the artist's photos, in shots cut between the song's lines and edited like an official video. A music video is one file in one format (`aspect_ratio`), of the whole song or a `trim` of 15 seconds to 10 minutes (best a part from `get_song_analysis` with `min_seconds` 15), priced by its length. `quality` is one of the qualities `get_options` lists for the model: `768p` by default, and `1080p` is the same video sharper for about twice the credits. `get_options` lists each model with its kinds, looks, formats, lengths and kit limits.

It is planned before it is made, and planning is free:

1. **Photos**: `list_artist_photos` / `kinetune artists photos list ARTIST_ID`. No photos: ask the user for a few (a sharp face with the mouth visible) and add them with `add_artist_photos`. A photo meant for one video only goes up with `hidden` true.
2. **Plan**: `plan_music_video` / `kinetune plan music-video --song-id … [--category …] [--aspect-ratio 9:16] [--trim '{"start_seconds":32,"end_seconds":96}'] [--direction "…"]`. It takes about a minute and returns the director's concept and its shots (time, lyrics, summary, and `transition`: how each comes in, a cut or the director's dissolve, dip to black, flash or whip). Show it to the user. `takes` is `cuts` (the default, edited shots) or `continuous` (the director may carry a shot on into the next through a picture drawn at the join, priced per picture). For transitions, or none, say so in `direction` ("only hard cuts", "dissolve between scenes").
3. **Revise** until they like it: `plan_music_video` with `revise` `{plan_id, note}`.
4. **Quote** with `song_id` and `plan_id`: `quote_music_video` / `kinetune quote music-video --song-id … --plan-id …`. Tell the user the credits and that it usually takes 5 to 20 minutes.
5. **Create** after their go: `create_music_video` / `kinetune create music-video --song-id … --plan-id … --quote-id …`. Check back with `get_video` rather than waiting in one call.

`cast` (`face_photo_id`, `outfit_photo_id`) picks the two photos the artist is kept from; left out, the director picks.

### The artist kit and @mentions

The kit holds what music videos are made from: each artist's **outfits**, and the workspace's **places** and **props**, shared by every artist. Each piece has an `@handle`.

- `list_kit` / `kinetune kit list [--artist-id …]`.
- `add_kit_piece` / `kinetune kit add --kind outfit|place|prop --name "…" …` from an image (`file`, or `file_url` with `rights_confirmed`) or a stock photo (`stock_id` from `search_stock`), with an optional `crop` (fractions of the image: `{"x","y","w","h"}`, at least 256 px kept). An outfit needs its `artist_id`.
- `update_kit_piece` renames it, changes its handle or description, or crops it again; `delete_kit_piece` removes it.

A piece mentioned in `direction` ("Nina in the @blue-set on the @rooftop") is used exactly as it is, and the director designs the rest. An unknown handle is refused with its name: offer to add it to the kit. For places the brief leaves open, `scenes` `invent` (default) has the director imagine them, and `stock` has it find real empty places and put the artist in them (`swap_music_video_place` changes one place's photo).

### Design the shoot (optional)

Before the plan, `design_music_video_shoot` / `kinetune lookbook design …` (free, about a minute) has the director design what the brief leaves open: outfits drawn on the artist, empty places, props. Show its pieces and `price.draw.credits`; drawing them costs those credits, so draw (`render_music_video_lookbook` with those credits) only after the user's go. They keep or redraw pieces (each redraw is priced: ask first), then `plan_music_video` with `lookbook_id`.

### Backing vocals

Words in (parentheses) in the song's lyrics are the backing vocals': they are sung off camera, and the artist doesn't lip-sync them. To mark them, set the song's lyrics with the parentheses (see [The song's own lyrics](#the-songs-own-lyrics)).

## When they wish Kinetune did something

If the user asks for something Kinetune can't do yet ("I wish it could…"), offer to suggest it as an idea: `suggest_idea` (`kinetune ideas suggest --title "…" [--body "…"] [--area lyric-video|canvas|music-video|everything]`). It goes on the Community board in the app under their name, where others vote on it, and they hear by email when it ships. Write the title in their words, in one line, with how they would use it in the body. Leave out private links and contact details, and ask before posting. At most 5 a day.

## When something goes wrong

- **A failed video returns its credits.**
- **`retry_video`** makes a video again. That is a new charge, so ask first, but it reuses the Look and media so nothing already made is paid for twice.
- **`cancel_video`** stops a queued or processing video and releases its credits.
- **"plan it again"**: a music video plan whose photo or kit piece was deleted, or made with choices no longer offered, can't be quoted. Plan it again.
- **Not enough credits**: tell the user their balance and the quote. Plans and credit packs are at https://kinetune.com/pricing. Don't shrink the request without asking.
- **`plan_required`** (402): the workspace has no active plan, so it can't upload or make videos. Tell the user to choose one in the app; don't retry.
- **`storage_full`** (402): the plan's storage is full (`get_account`: `storage`). Tell the user; deleting videos (`delete_video`) or archiving songs (`archive_song`) they no longer need frees space at once, or they can move up a plan. Ask before deleting anything.
- **Workspaces**: the app calls them workspaces; tools and fields say organization (`list_organizations`, `create_organization` with a `type`, `update_organization`). One connection acts for one workspace: to use another, the user reconnects and picks it.
- **CLI exit codes**:
  - `0`: done
  - `1`: API or network error
  - `2`: invalid usage, or the quote is over `--max-credits`
  - `3`: not signed in or not allowed
- **Unsure of an input?** `kinetune schema "quote canvas"` prints the exact JSON schema of any command.
- **Full reference**: every operation with its parameters, scopes, CLI command and MCP tool is at https://kinetune.com/llms-full.txt.
