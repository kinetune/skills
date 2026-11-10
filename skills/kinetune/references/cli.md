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
| `kinetune options` | Video types, Look categories, background sources, formats, headlines, the credit prices, and each music video model's guide (best for, strengths, weaknesses, scores) |

## Artists

| Command | What it does |
|---|---|
| `kinetune artists list [--q …]` | Artists by name, with song and photo counts |
| `kinetune artists create --name "…"` | Adds an artist |
| `kinetune artists get ARTIST_ID` | One artist |
| `kinetune artists rename ARTIST_ID --name "…"` | Renames an artist |
| `kinetune artists archive ARTIST_ID` | Removes an artist that has no songs |
| `kinetune artists image set ARTIST_ID --image ./photo.jpg [--crop '{"x":0.1,"y":0,"size":0.8}']` | The artist's picture, kept as a 512 px square (a display picture only, never shown to a model) |
| `kinetune artists image remove ARTIST_ID` | Removes it: they show their initial |

## Characters

| Command | What it does |
|---|---|
| `kinetune characters list [--artist-id …] [--for-artist-id …]` | The people videos can show: artists' characters and cast |
| `kinetune characters start --mode generate --photos a.jpg b.jpg --rights-confirmed` | Photos of a person (1 to 6) read for a sheet; then `characters draw` (priced). `--mode describe --description "…"` draws a made-up person from words; `--mode add --photos sheet.png` adds a finished sheet, free. The camera mode is Take from camera in the app |
| `kinetune characters get REQUEST_ID` | What was read, the outfit line, the price, and the sheet once drawn |
| `kinetune characters draw REQUEST_ID --credits N [--outfit "…"] [--note "…"]` | Draws the sheet for the price shown (again with a note to redraw). Ask the user first |
| `kinetune characters keep REQUEST_ID (--artist-id … \| --name "…") [--ai-made] [--shared]` | Keeps it as the artist or as named cast. Ask whether the person is AI-made; never guess |
| `kinetune characters waiting` | Requests read or drawn but not kept yet |
| `kinetune characters discard REQUEST_ID` | Deletes a request's uploads and sheet |
| `kinetune characters face-check get CHARACTER_ID --model …`, `kinetune characters face-check start CHARACTER_ID --model … --consent`, `kinetune characters face-check withdraw CHARACTER_ID --model …` | A real person's face check for a model that needs one (`start` gives a link to open on a phone, with their consent) |

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
| `kinetune songs set-cover SONG_ID --cover-art ./cover.jpg` | Adds a cover or replaces the one in use; the one before stays on the song |
| `kinetune songs edit-brief SONG_ID [--theme "…"] [--story "…"] [--setting "…"] [--tone "…"] [--mood-arc '[{"section":"Verse 1","feeling":"…"}]']` | Corrects what the song is about (`songs get` → `brief`), in the user's words; kept until `rewrite-brief` |
| `kinetune songs rewrite-brief SONG_ID` | Drops the user's correction and reads the song again (free). Ask first |
| `kinetune uploads create --filename … --content-type … --size-bytes N` | A signed URL to PUT a local file to, and the `file_url` to pass in any `*_url` field (for agents that can only pass URLs; the CLI sends files itself) |

## Cover art

| Command | What it does |
|---|---|
| `kinetune covers list SONG_ID` | Every cover the song has had (made or uploaded), the one in use, rounds being made, and the price of 1 to 4 covers and of a redraw |
| `kinetune covers quote --count N` | Credits for a round of N covers (1 to 4) and for a redraw |
| `kinetune covers make --song-id … --count N [--style auto] [--vibes sad dark] [--artist auto\|always\|never] [--words none\|title\|artist_title] [--idea "…"] --credits N` | The cover art director makes N covers, each a different idea (paid plan, analysed song). Quote and ask first |
| `kinetune covers get REQUEST_ID` | A round's covers as they are drawn |
| `kinetune covers redraw REQUEST_ID --cover-id … --note "…" --credits N` | Draws one cover again with the user's note (priced; ask first) |
| `kinetune covers use SONG_ID COVER_ID` | Puts a cover in use. Only on the user's pick |
| `kinetune covers delete SONG_ID COVER_ID` | Deletes a cover for good. Ask first |

## Looks

| Command | What it does |
|---|---|
| `kinetune looks list [--scope official\|community\|mine] [--category …] [--q …] [--sort newest\|popular]` | Browses Looks |
| `kinetune looks get LOOK_ID` | One Look with its preview images |
| `kinetune looks rename LOOK_ID --name "…"` | Renames one of your Looks |
| `kinetune looks visibility LOOK_ID --visibility public\|private` | Makes one of your Looks public or private |
| `kinetune looks delete LOOK_ID` | Removes one of your Looks. Videos made with it stay |
| `kinetune looks music-video [--scope all\|community\|mine]` | Music video Looks: public ones and the workspace's own. Pass one's id as `--look-id` when planning or quoting |
| `kinetune looks music-video-get LOOK_ID`, `kinetune looks music-video-delete LOOK_ID` | One music video Look, or removes one of yours |
| `kinetune stock search --q "…"` | Stock photos or clips for a background (credit the photographer) |
| `kinetune backgrounds upload --file ./clip.mp4 --rights-confirmed` | Your own photo or video (or `--video-id` of one of your music videos) to design a New Look around |

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
| `kinetune plan music-video --song-id … [--model one-take\|seedance-2-0\|seedance-2-0-fast\|seedance-2-0-mini\|seedance-2-5] [--category …] [--aspect-ratio 9:16] [--trim '{"start_seconds":32,"end_seconds":96}'] [--direction "…"] [--takes cuts\|continuous]` | The director's plan, free (about a minute): its shots and how each comes in (a cut or a transition) |
| `kinetune plan music-video --song-id … --revise '{"plan_id":"plan_…","note":"…"}'` | Rewrites a plan from a note (the next draft), free |
| `kinetune plans get PLAN_ID` | A plan made earlier |
| `kinetune plans list --song-id …` | A song's shot lists and lookbooks not made into a video yet (last 30 days) |
| `kinetune plans delete PLAN_ID` | Drops a shot list |
| `kinetune plans swap-place PLAN_ID PLACE --stock-id …` | Another stock photo for one of a plan's places (free) |
| `kinetune casting propose --song-id … [same brief as plan]` | The director proposes up to 3 more people, free; each shown with its price |
| `kinetune casting get CASTING_ID` | The casting, then the people as drawn (`people_for_brief` once ready) |
| `kinetune casting draw CASTING_ID --people '[{"index":1},{"index":2,"character":"mia-torres"}]' --credits N` | Draws the people the user kept (or puts a kit character in a role, free). Ask first |
| `kinetune casting delete CASTING_ID` | Drops a casting; people already kept stay |
| `kinetune plans frames PLAN_ID` | Opening frames drawn for a plan's shots, the price of one (`price.credits`) and the shots that can't have one |
| `kinetune plans draw-frame PLAN_ID --shot N --credits N` | Draws the picture a shot opens on, for the price shown. Ask the user first, then show it |
| `kinetune quote music-video --song-id … --plan-id … [--quality 768p]` | Exact credits for the planned video, in one of the qualities `kinetune options` lists for the model |
| `kinetune quote music-video --song-id … --model … [--category …]` | Credits for the same brief with another model, to compare models before planning (free) |
| `kinetune create music-video …same flags… --quote-id …` | Makes the music video the quote priced |
| `kinetune lookbook design --song-id … [--direction "…"]` | A lookbook (optional, free, before the shot list): the director's outfits, places and props, not drawn yet |
| `kinetune lookbook render LOOKBOOK_ID --credits N` | Draws the pieces for the credits the lookbook shows (`price.draw.credits`). Ask the user first |
| `kinetune lookbook get LOOKBOOK_ID` | Its pieces and pictures as they are drawn, and the prices |
| `kinetune lookbook keep LOOKBOOK_ID PIECE --no-kept` | Leaves a piece out of the video (free) |
| `kinetune lookbook redraw LOOKBOOK_ID PIECE [--note "…"] --credits N` | Draws one of the director's pieces again. Ask first |
| `kinetune lookbook redesign LOOKBOOK_ID --note "…" --credits N` | Redesigns the director's pieces from a note. Ask first |
| `kinetune lookbook update LOOKBOOK_ID --no-save-to-kit` | Uses the kept pieces for that video only, not the kit |
| `kinetune lookbook delete LOOKBOOK_ID` | Deletes one nothing was made from |
| `kinetune quote music-video --song-id … --plan-id … --test` | Prices testing the plan first: its first chorus at 480p, for a fraction of the price |

## Kit

| Command | What it does |
|---|---|
| `kinetune kit list [--artist-id …] [--owner-artist-id …] [--kind outfit\|place\|prop]` | What an artist's songs can use (theirs, then what others shared), or one artist's kit |
| `kinetune kit add --kind outfit\|place\|prop --name "…" [--artist-id …] [--owner-artist-id …] [--shared] [--file ./piece.jpg --rights-confirmed \| --stock-id …] [--crop '{"x":0.1,"y":0.1,"w":0.8,"h":0.8}']` | Adds a piece, named with an @handle |
| `kinetune kit update KIT_ID [--name "…"] [--handle …] [--crop '…'] [--shared]` | Renames it, changes its handle, crops it again or shares it with every artist |
| `kinetune kit delete KIT_ID` | Removes it |
| `kinetune kit get KIT_ID` | One piece |
| `kinetune kit angles KIT_ID [--angles three-quarter side low] [--confirm --credits N]` | A place drawn again from other sides, each a new place. Without `--confirm`, only the price: ask first |

## Videos

| Command | What it does |
|---|---|
| `kinetune videos list [--song-id …] [--type lyric-video\|canvas\|music-video]` | Videos, newest first |
| `kinetune videos get VIDEO_ID` | Status, progress, credits and, when completed, the download links; while it waits, `queue` says why. A music video lists `music_video.shots[]` (`index`, `start_seconds`, `end_seconds`, `summary`, `lyrics`), timed from the start of the delivered video |
| `kinetune videos revise VIDEO_ID --shot N --note "…"` | Rewrites one shot of a delivered music video, free. Pick N from `videos get`: the shot whose `start_seconds`–`end_seconds` holds the time the user named (an outro card comes after the last shot). Confirm its summary and seconds with them first, and use their words as the note. Then quote with `--plan-id` (only the shots made again are charged) and create after their go. A shot may hold several camera cuts (Seedance), and the whole shot is made again |
| `kinetune videos revise VIDEO_ID --shots '[{"shot":2,"note":"…"},{"shot":5,"note":"…"}]'` | Several shots at once, each from its own note: one new version |
| `kinetune videos revision-prices VIDEO_ID` | What revising each shot costs, before rewriting anything |
| `kinetune videos save-look VIDEO_ID` | Saves a delivered music video's direction as a reusable Look (free; never the people in it) |
| `kinetune videos wait VIDEO_ID [--timeout 1800]` | Waits until completed, failed or cancelled, showing its progress or why it waits (its place in line, or busy machines) |
| `kinetune videos download VIDEO_ID [--out DIR] [--format 9:16] [--web]` | Downloads the files, one per format. `--web` gets the 720p copies |
| `kinetune videos cancel VIDEO_ID` | Stops a queued or processing video and releases its credits |
| `kinetune videos retry VIDEO_ID` | Makes a finished, failed or cancelled video again. This is a new charge, though its Look and media are reused |
| `kinetune videos delete VIDEO_ID` | Deletes a finished video and its files |
| `kinetune videos share VIDEO_ID [--rights-confirmed] [--socials --credit-handle @name]` | Makes a delivered video public in the Community, its Look reusable by anyone. Only on the user's word; `--rights-confirmed` (a music video) and `--socials` only when they said so |
| `kinetune videos unshare VIDEO_ID` | Makes it private again, its Look too |

## Ideas

| Command | What it does |
|---|---|
| `kinetune ideas list [--q "…"] [--sort top\|new] [--status open\|…] [--yours]` | The Community board's ideas, most voted first: search here before suggesting one |
| `kinetune ideas get IDEA_ID` | One idea with its comments |
| `kinetune ideas vote IDEA_ID` / `unvote IDEA_ID` | Votes for an open idea, or takes the vote back. Ask the user first |
| `kinetune ideas follow IDEA_ID` / `unfollow IDEA_ID` | An email when it ships, or no more |
| `kinetune ideas suggest --title "…" [--body "…"] [--area lyric-video\|canvas\|music-video\|everything]` | Suggests a feature on the app's Community board under your name, for others to vote on (5 a day). Ask the user first |
