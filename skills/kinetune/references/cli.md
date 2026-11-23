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
| `kinetune auth list` | The workspaces you are signed in to; `--organization` on any command acts for one of them |
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
| `kinetune options` | Video types, lyric styles, visuals, Stock collections (`clip_moods`), transitions, grades, title cards, formats, headlines, the plans and credit prices, and each music video model's guide (best for, strengths, weaknesses, scores) |

## Artists

| Command | What it does |
|---|---|
| `kinetune artists list [--q …]` | Artists by name, with song and photo counts |
| `kinetune artists create --name "…"` | Adds an artist |
| `kinetune artists get ARTIST_ID` | One artist |
| `kinetune artists rename ARTIST_ID [--name "…"] [--spotify-artist-id …] [--links '{…}']` | Renames an artist, or sets their Spotify id and links |
| `kinetune artists find --q "…"` | The artist in the music catalog by full name or Apple Music link, each match with its song count and newest release |
| `kinetune artists link ARTIST_ID --catalog-artist-id … --catalog-confirmed` | Links the artist to the match the user said is theirs, once (Artist and Label workspaces) |
| `kinetune artists archive ARTIST_ID` | Removes an artist that has no songs |
| `kinetune artists image set ARTIST_ID --image ./photo.jpg [--crop '{"x":0.1,"y":0,"size":0.8}']` | The artist's picture, kept as a 512 px square (a display picture only, never shown to a model) |
| `kinetune artists image remove ARTIST_ID` | Removes it: they show their initial |
| `kinetune artists profiles list ARTIST_ID` | Their links on other services, the search's state, and the ones removed |
| `kinetune artists profiles add ARTIST_ID --url …` | Adds a link by hand (the service is recognised from the address) |
| `kinetune artists profiles update ARTIST_ID PROFILE_ID [--confirmed] [--state shown\|hidden]` | Confirms, hides, shows or corrects one. A found profile is on the page only once confirmed |
| `kinetune artists profiles confirm-all ARTIST_ID` | Confirms every found profile at once. Only when the user says all are theirs |
| `kinetune artists profiles order ARTIST_ID --order PROFILE_ID …`, `kinetune artists profiles remove ARTIST_ID PROFILE_ID` | Reorders them, or removes one for good |
| `kinetune artists profiles discover ARTIST_ID` | Looks for their profiles again from their Apple Music connection |
| `kinetune artists page check HANDLE`, `kinetune artists page claim ARTIST_ID --handle …` | Whether kinetune.com/@handle is free, then claims it |
| `kinetune artists page get ARTIST_ID`, `kinetune artists page preview ARTIST_ID` | The page's status and draft, or the draft as fans will see it |
| `kinetune artists page update ARTIST_ID [--sections '[…]'] …` | Changes the draft (look, photo, genres, sections); send only what changes |
| `kinetune artists page publish ARTIST_ID`, `kinetune artists page unpublish ARTIST_ID` | Puts the draft live (ask first), or takes the page offline |
| `kinetune artists page stats ARTIST_ID` | Views, visitors, clicks and click rate |

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
| `kinetune songs released ARTIST_ID [--q …]` | A linked artist's released songs, newest first, with `song_id` on the ones already in |
| `kinetune songs import --artist-id … --track-ids … [--distinct-track-ids …]` | Brings released songs in without uploading: Kinetune fetches the audio and uses the release's title and cover |
| `kinetune songs imports`, `kinetune songs import-status IMPORT_ID` | Imports on their way, failed ones waiting for a file, and today's done ones |
| `kinetune songs import-audio IMPORT_ID --audio ./song.wav`, `kinetune songs import-cancel IMPORT_ID` | Gives the file of a song that couldn't be fetched, or drops the import |
| `kinetune songs get SONG_ID` | One song. Wait for `analysis.status` `"ready"` |
| `kinetune songs download SONG_ID` | Its cover art (full size) and audio file |
| `kinetune songs analysis SONG_ID [--min-seconds 15]` | Word-timed lyrics, tempo, beats, sections, hook and the parts a video can be cut to (`--min-seconds 15` for a music video) |
| `kinetune songs reanalyze SONG_ID` | Retries a failed analysis |
| `kinetune songs set-lyrics SONG_ID --lyrics-file FILE` | Gives the song its own lyrics (text, `.srt`, `.vtt` or `.lrc`; `--lyrics "…"` takes them inline): videos spell every word that way, timed from the audio |
| `kinetune songs clear-lyrics SONG_ID` | Goes back to the transcribed words |
| `kinetune songs set-words SONG_ID --input words.json` | Every word with its `start`, `end` and `line`, fixed by hand (`{"words":[…]}`, as `songs analysis` gives them) and saved exactly so |
| `kinetune songs transcription-price SONG_ID` | What listening to the song again costs |
| `kinetune songs transcribe SONG_ID --credits N [--language fr]` | Listens to it again with Kinetune's current transcription model. Ask first |
| `kinetune songs draft SONG_ID` | The Look and clips picture Kinetune picks for the song, at once (what a lyric video without a Look id gets) |
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
| `kinetune looks save --name "…" --from-id LOOK_ID [--lyric-style …] [--visual '{…}'] [--background '{…}']` | A private Look of your own from a saved one, with your changes |
| `kinetune looks clips LOOK_ID` | The clips a clips Look cuts its picture from |
| `kinetune looks music-video [--scope all\|community\|mine]` | Music video Looks: public ones and the workspace's own. Pass one's id as `--look-id` when planning or quoting |
| `kinetune looks music-video-get LOOK_ID`, `kinetune looks music-video-delete LOOK_ID` | One music video Look, or removes one of yours |

## Footage

| Command | What it does |
|---|---|
| `kinetune clips browse MOOD_ID [--shape tall\|wide] [--energy calm\|medium\|bold] [--sort best\|newest\|length] [--offset N]` | The clips inside one Stock collection, a page at a time (free) |
| `kinetune clips similar CLIP_ID [--limit 24]` | More like this: clips from Stock and your folders that look and move like one (free, no AI) |
| `kinetune clips list --moods nature,euphoria [--folders …]` | The clips of those sources, to plan a cut yourself (`edit.clips`) |
| `kinetune clips generate (--prompt "…" \| --song-id … --from song\|cover) --kind image\|video [--with artist\|kit_…] [--model …] [--style calm\|cinematic\|dreamy\|intense] [--loop] [--shape tall\|wide] [--count N] [--credits N]` | New pictures or clips into your AI folder, from words, the song or its cover. Without `--credits`, only the price (and, from the song, three ideas): ask first |
| `kinetune clips loop CLIP_ID [--span N] [--seam N]` | Best spot: where a clip loops best as a Canvas drift, and its loop score (free) |
| `kinetune clips rebound CLIP_ID [--at N] [--half N] [--speed N] [--steady]`, `kinetune clips steady CLIP_ID` | The file a Canvas rebound, or a Canvas with `steady`, will play (free) |
| `kinetune clips analyze --clip-ids … [--behind]` | Finds the subject and faces of clips a preview you planned shows (free, capped per day for your own clips) |
| `kinetune clips retry-import IMPORT_ID`, `kinetune clips dismiss-import IMPORT_ID` | Tries a failed clip import or generated clip again, or removes its tile |
| `kinetune clips move --clip-ids … --folder-id …`, `kinetune clips delete CLIP_ID` | Moves some of your clips to another folder, or removes one |
| `kinetune clip-folders list`, `kinetune clip-folders get FOLDER_ID` | Your folders, or one folder's clips and the imports on their way |
| `kinetune clip-folders create --name "…"`, `kinetune clip-folders rename FOLDER_ID --name "…"`, `kinetune clip-folders delete FOLDER_ID` | Makes, renames or removes a folder (and its clips) |
| `kinetune clip-folders import FOLDER_ID (--url … \| --file-url … \| --stock-id …)` | Cuts a YouTube, TikTok or Instagram link, a file or a stock video into clips at every scene change (free with a plan) |
| `kinetune stock search --kind photo\|video --q "…" [--orientation portrait\|landscape] [--seconds 8]` | Stock photos or clips, to import into a folder or use in a Canvas (credit the creator) |

## Quote and create

| Command | What it does |
|---|---|
| `kinetune quote lyric-video --song-id … [--look '{"id":"look_…","background":{"kind":"clips","sources":{"moods":["nature"]}}}'] [--aspect-ratios 9:16 16:9] [--variations N] [--display '{…}'] [--trim '{…}'] [--edit '{"seed":7}']` | Exact credits for a lyric video (`--look` left out: the one `songs draft` shows) |
| `kinetune quote canvas --song-id … [--picture '{"kind":"clips","sources":{"moods":["nature"]}}'] [--loop drift\|rebound\|cuts] [--seconds 8] [--edit '{…}'] [--light '{"preset":"…"}'] [--atmosphere dust] [--effects '[{"id":"film-grain","amount":0.5}]'] [--variations N]` | Exact credits for a Canvas: a picture (`clips`, `cover`, `visualizer` or `generate`), a loop and a length |
| `kinetune create lyric-video …same flags… --quote-id …` | Makes the lyric video the quote priced |
| `kinetune create canvas …same flags… --quote-id …` | Makes the Canvas the quote priced |
| `kinetune canvas-presets list` | Your saved Canvas looks, the latest first |
| `kinetune canvas-presets save --name "…" --look '{"loop":"drift","seconds":8,"grade":"noir","atmosphere":"dust"}'` | Keeps a Canvas look (no clips or song) under a name; the same name replaces it, so ask first |
| `kinetune canvas-presets rename PRESET_ID --name "…"`, `kinetune canvas-presets delete PRESET_ID` | Renames or removes one |

`kinetune create …` without `--quote-id` quotes first, shows the credits and asks. Use `--max-credits N` to refuse anything pricier. Pass `--yes` only when the user has agreed to that amount.

## Music videos

| Command | What it does |
|---|---|
| `kinetune plans list --song-id …` | What writing a shot list costs now (`price.credits`), the song's shot lists not made into a video yet (last 30 days) and the ones being written |
| `kinetune plan music-video --song-id … --credits N [--model one-take\|seedance-2-0\|seedance-2-0-fast\|seedance-2-0-mini\|seedance-2-5] [--category …] [--aspect-ratio 9:16] [--trim '{"start_seconds":32,"end_seconds":96}'] [--direction "…"] [--takes cuts\|continuous]` | The director's shot list for the price shown (a minute or more): its shots and how each comes in (a cut or a transition). Ask first |
| `kinetune plan music-video --song-id … --revise '{"plan_id":"plan_…","note":"…"}' --credits N` | Rewrites it from a note (the next version), for the same price. Ask first |
| `kinetune plans writing WRITE_ID` | A shot list being written (asked with `"wait": false`), its shots as they are written |
| `kinetune plans get PLAN_ID` | A plan made earlier |
| `kinetune plans delete PLAN_ID` | Drops a shot list (its opening frames are kept) |
| `kinetune plans check-model PLAN_ID --model …` | Whether the shot list can be kept with another model, what it needs and what changes (free) |
| `kinetune plans switch-model PLAN_ID --model …` | The next version made with that model: same shots, idea, people and frames (free) |
| `kinetune plans swap-place PLAN_ID PLACE --stock-id …` | Another stock photo for one of a plan's places (free) |
| `kinetune casting propose --song-id … [same brief as plan]` | The director proposes up to 3 more people, free; each shown with its price |
| `kinetune casting get CASTING_ID` | The casting, then the people as drawn (`people_for_brief` once ready) |
| `kinetune casting draw CASTING_ID --people '[{"index":1},{"index":2,"character":"mia-torres"}]' --credits N` | Draws the people the user kept (or puts a kit character in a role, free). Ask first |
| `kinetune casting delete CASTING_ID` | Drops a casting; people already kept stay |
| `kinetune plans frames PLAN_ID` | Opening frames drawn for a plan's shots, the price of one (`price.credits`) and the shots that can't have one |
| `kinetune plans draw-frame PLAN_ID --shot N --credits N` | Draws the picture a shot opens on, for the price shown. Ask the user first, then show it |
| `kinetune plans draw-frame PLAN_ID --all --shots 1 4 7 --credits N` | Draw all: every frame not drawn yet (`batch_shots` from `plans frames`), in the background, for one frame's price × their number (CLI 4.2.0 or later) |
| `kinetune plans storyboard PLAN_ID` | The shot list as a storyboard: every shot with its picture so far, the outfits, and what drawing the rest costs (free) (CLI 4.2.0 or later) |
| `kinetune plans versions PLAN_ID` | Every version of a shot list, oldest first, with the note each was written from |
| `kinetune plans delete-version PLAN_ID VERSION_ID` | Deletes one version a video wasn't made from; its pictures stay |
| `kinetune plans pictures PLAN_ID` | A shot list's pictures (places and their angles, outfits on its people, designed props), by key, each with its price and picture once drawn |
| `kinetune plans draw-pictures PLAN_ID [--pictures place:3 outfit:1] [--redraw] [--confirm --credits N]` | Draws them now to see them before the video; each waits in the kit to be kept. Without `--confirm`, only the price: ask first |
| `kinetune plans drop-angle PLAN_ID PLACE` | The next draft without that angle (its shots back on the place's own picture), free |
| `kinetune quote music-video --song-id … --plan-id … [--quality 768p]` | Exact credits for the planned video, in one of the qualities `kinetune options` lists for the model |
| `kinetune quote music-video --song-id … --model … [--category …]` | Credits for the same brief with another model, to compare models before planning (free) |
| `kinetune create music-video …same flags… --quote-id …` | Makes the music video the quote priced |
| `kinetune quote music-video --song-id … --plan-id … --test` | Prices testing the plan first: its first chorus at 480p, for a fraction of the price |

## Kit

| Command | What it does |
|---|---|
| `kinetune kit list [--artist-id …] [--owner-artist-id …] [--kind outfit\|place\|prop]` | What an artist's songs can use (theirs, then what others shared), or one artist's kit |
| `kinetune kit add --kind outfit\|place\|prop --name "…" [--artist-id …] [--owner-artist-id …] [--shared] [--file ./piece.jpg --rights-confirmed \| --stock-id …] [--crop '{"x":0.1,"y":0.1,"w":0.8,"h":0.8}']` | Adds a piece, named with an @handle |
| `kinetune kit update KIT_ID [--name "…"] [--handle …] [--crop '…'] [--shared]` | Renames it, changes its handle, crops it again or shares it with every artist |
| `kinetune kit delete KIT_ID` | Removes it |
| `kinetune kit get KIT_ID` | One piece |
| `kinetune kit angles KIT_ID [--angles reverse low overhead] [--confirm --credits N]` | A place drawn again from the angles picked (wide, three-quarter, side, reverse, corner, low, high, overhead, detail, framed), each a new place. Without `--confirm`, only the price: ask first |

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
| `kinetune drafts list` | Videos started in the studio and not made yet, one per song and kind (kept 30 days) |
| `kinetune drafts get SONG_ID [--type lyric-video\|canvas]` | The person's draft for a song, as the editor keeps it: Look, sources, recipe, every cut's clip and transition, and the exact `request` to quote (free) (CLI 4.2.0 or later) |
| `kinetune drafts edit SONG_ID --input change.json` | Changes the draft as the editor would (`shuffle`, `replace`, `joins`, `starts`, `sources`, recipe, Look, `format`, `trim`; a Canvas's picture and loop). Free; the editor opens the same draft (CLI 4.2.0 or later) |
| `kinetune drafts delete (--id … \| --all [--song-id …] [--type lyric-video\|canvas\|music-video])` | Deletes one draft or every one. Only when the user asks |
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
