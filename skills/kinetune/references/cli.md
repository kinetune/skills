# The `kinetune` CLI

Install with `npm install -g @kinetune/cli` (Node 20+), or run without installing: `npx @kinetune/cli <command>`.

Conventions:

- Path ids are arguments and fields are flags.
- `--input file.json` (or `-` for stdin) takes the whole request; flags override it.
- Output is JSON when piped, or always with `--json`.
- `kinetune schema <command>` prints a command's JSON input schema. `kinetune openapi` prints the whole API.
- Environment: `KINETUNE_API_KEY` wins over the stored sign-in, which is useful in CI. Credentials live in `~/.config/kinetune/config.json`.

## Sign in

| Command | What it does |
|---|---|
| `kinetune auth login` | Opens the browser to sign in and pick the workspace (a new account sets it up and chooses a plan first). Add `--no-browser` to print the URL instead |
| `kinetune auth login --api-key kt_live_…` | Stores a workspace API key |
| `kinetune auth status` | The workspace, its plan, how you are signed in, and the credit balance |
| `kinetune auth token` | Prints the bearer token, for curl |
| `kinetune auth logout` | Forgets the credentials |

## Account and options

| Command | What it does |
|---|---|
| `kinetune account` | The workspace (`organization`, with its `type`), its plan (`active` says whether it can upload and make videos) and its credit balance |
| `kinetune organizations list` | Your workspaces, with type and role; `current` is the one you act for |
| `kinetune organizations create --name "…" --type artist\|label` | A new workspace you own: an artist's (one artist) or a label's (a roster). It starts with no plan |
| `kinetune organizations update [--name "…"] [--type artist\|label]` | Renames this workspace or switches its type (an artist workspace holds one artist) |
| `kinetune options` | Video types, Look categories, background sources, formats, headlines and the credit prices |

## Artists

| Command | What it does |
|---|---|
| `kinetune artists list [--q …]` | Artists by name, with song and photo counts |
| `kinetune artists create --name "…"` | Adds an artist |
| `kinetune artists get ARTIST_ID` | One artist |
| `kinetune artists rename ARTIST_ID --name "…"` | Renames an artist |
| `kinetune artists archive ARTIST_ID` | Removes an artist that has no songs |
| `kinetune artists picture ARTIST_ID --photo-id … [--crop '{…}']` | Chooses which photo is the artist's picture, and its square crop |
| `kinetune artists photos list\|add\|delete ARTIST_ID …` | Artist photos (up to 6, JPEG, PNG or WebP), which keep the artist recognizable when a video shows them |

## Songs

| Command | What it does |
|---|---|
| `kinetune songs list [--q …] [--status ready] [--sort newest] [--artist-id …] [--limit N --offset N]` | Songs with cover, duration and analysis status |
| `kinetune songs create [--artist-id …] --title "…" --audio FILE_OR_URL --cover-art FILE_OR_URL [--language en] [--lyrics-file FILE]` | Uploads a song, then analysis starts. `--lyrics-file` (text, `.srt`, `.vtt` or `.lrc`) gives it the song's own lyrics |
| `kinetune songs get SONG_ID` | One song. Wait for `analysis.status` `"ready"` |
| `kinetune songs analysis SONG_ID [--min-seconds 15]` | Word-timed lyrics, tempo, beats, sections, hook and the parts a video can be cut to (`--min-seconds 15` for a music video) |
| `kinetune songs reanalyze SONG_ID` | Retries a failed analysis |
| `kinetune songs set-lyrics SONG_ID --lyrics-file FILE` | Gives the song its own lyrics (text, `.srt`, `.vtt` or `.lrc`; `--lyrics "…"` takes them inline): videos spell every word that way, timed from the audio |
| `kinetune songs clear-lyrics SONG_ID` | Goes back to the transcribed words |
| `kinetune songs rename SONG_ID --title "…"` | Renames a song |
| `kinetune songs archive SONG_ID` | Removes a song. Its videos stay |

## Looks

| Command | What it does |
|---|---|
| `kinetune looks list [--scope official\|community\|mine] [--category …] [--q …] [--sort newest\|popular]` | Browses Looks |
| `kinetune looks get LOOK_ID` | One Look with its preview images |
| `kinetune looks rename LOOK_ID --name "…"` | Renames one of your Looks |
| `kinetune looks visibility LOOK_ID --visibility public\|private` | Makes one of your Looks public or private |
| `kinetune looks delete LOOK_ID` | Removes one of your Looks. Videos made with it stay |

## Quote and create

| Command | What it does |
|---|---|
| `kinetune quote lyric-video --song-id … --look '{…}' [--aspect-ratios 9:16 16:9] [--variations N] [--display '{…}'] [--trim '{…}']` | Exact credits for a lyric video |
| `kinetune quote canvas --song-id … [--source ai-video] [--seconds 8] [--style …] [--direction "…"]` | Exact credits for a Canvas |
| `kinetune create lyric-video …same flags… --quote-id …` | Makes the lyric video the quote priced |
| `kinetune create canvas …same flags… --quote-id …` | Makes the Canvas the quote priced |

`kinetune create …` without `--quote-id` quotes first, shows the credits and asks. Use `--max-credits N` to refuse anything pricier. Pass `--yes` only when the user has agreed to that amount.

## Music videos

| Command | What it does |
|---|---|
| `kinetune plan music-video --song-id … [--category …] [--aspect-ratio 9:16] [--trim '{"start_seconds":32,"end_seconds":96}'] [--direction "…"] [--takes cuts\|continuous]` | The director's plan, free (about a minute): its shots and how each comes in (a cut or a transition) |
| `kinetune plan music-video --song-id … --revise '{"plan_id":"plan_…","note":"…"}'` | Rewrites a plan from a note (the next draft), free |
| `kinetune plans get PLAN_ID` | A plan made earlier |
| `kinetune quote music-video --song-id … --plan-id … [--quality 768p]` | Exact credits for the planned video, in one of the qualities `kinetune options` lists for the model |
| `kinetune create music-video …same flags… --quote-id …` | Makes the music video the quote priced |
| `kinetune lookbook design --song-id … [--direction "…"]` | Design the shoot (optional, free): the director's outfits, places and props, not drawn yet |
| `kinetune lookbook render LOOKBOOK_ID --credits N` | Draws the pieces for the credits the lookbook shows (`price.draw.credits`). Ask the user first |

## Kit

| Command | What it does |
|---|---|
| `kinetune kit list [--artist-id …] [--kind outfit\|place\|prop]` | The workspace's places and props, and an artist's outfits |
| `kinetune kit add --kind outfit\|place\|prop --name "…" [--artist-id …] [--file ./piece.jpg --rights-confirmed \| --stock-id …] [--crop '{"x":0.1,"y":0.1,"w":0.8,"h":0.8}']` | Adds a piece, named with an @handle |
| `kinetune kit update KIT_ID [--name "…"] [--handle …] [--crop '…']` | Renames it, changes its handle or crops it again |
| `kinetune kit delete KIT_ID` | Removes it |

## Videos

| Command | What it does |
|---|---|
| `kinetune videos list [--song-id …] [--type lyric-video\|canvas\|music-video]` | Videos, newest first |
| `kinetune videos get VIDEO_ID` | Status, progress, credits and, when completed, the download links; while it waits, `queue` says why |
| `kinetune videos wait VIDEO_ID [--timeout 1800]` | Waits until completed, failed or cancelled, showing its progress or why it waits (its place in line, or busy machines) |
| `kinetune videos download VIDEO_ID [--out DIR] [--format 9:16] [--web]` | Downloads the files, one per format. `--web` gets the 720p copies |
| `kinetune videos cancel VIDEO_ID` | Stops a queued or processing video and releases its credits |
| `kinetune videos retry VIDEO_ID` | Makes a finished, failed or cancelled video again. This is a new charge, though its Look and media are reused |
| `kinetune videos delete VIDEO_ID` | Deletes a finished video and its files |

## Ideas

| Command | What it does |
|---|---|
| `kinetune ideas suggest --title "…" [--body "…"] [--area lyric-video\|canvas\|music-video\|everything]` | Suggests a feature on the app's Community board under your name, for others to vote on (5 a day). Ask the user first |
