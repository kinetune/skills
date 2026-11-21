---
name: kinetune
description: Make lyric videos (9:16, 16:9, 1:1), Spotify Canvas loops, AI music videos and cover art from a user's songs with Kinetune, through its MCP tools or the `kinetune` CLI. Use when the user wants a lyric video, a Spotify Canvas, a music video, a visualizer or promo video for a song, cover art for a song, wants to set up an artist's character or kit (outfits, places, props), wants to upload or import a song to Kinetune, fix its lyrics or their timing, or correct what it says a song is about, browse Looks or footage clips, set up an artist page, check their credits, or mentions Kinetune.
---

# Kinetune

Kinetune turns a song into release-ready video:

- **Lyric videos**: word-synced lyrics in one of 63 lyric styles, over short clips cut on the beat (Kinetune's Stock footage or the user's own) or a visualizer that moves with the music, in 9:16 (vertical), 16:9 (wide) and 1:1 (square). No AI designs them.
- **Spotify Canvas**: a 5–8 second vertical loop with its idea from the song's cover or from the song itself, uploaded by the artist in Spotify for Artists.
- **AI music videos**: the song's artist performs it, lip-synced, kept from their character, from 15 seconds to 10 minutes. See [Music videos](#music-videos).
- **Cover art**: Kinetune's cover art director makes 1 to 4 covers for a song (3000×3000), each a different idea, with the title and artist name lettered in if the user wants. See [Cover art](#cover-art).

Lyric videos use a **Look**, a complete reusable design. Reuse a saved Look, or let Kinetune pick one for the song, and change its lyric style, visual or picture.

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

1. **Check the account**: `get_account` / `kinetune account`. Note the credit balance, the workspace's `type` and whether its plan is `active`. Uploading and making videos need an active plan, with one exception:
   - **The Free plan.** Without a plan, a person gets one new free song and three free lyric videos a month. `free_song` (only when signed in as the person, never with an API key) says where they stand: `songs_per_month`, `videos_per_month` and `resets_at` (when the month starts again); `state` is this month's song: `available` (upload it now with `create_song`), `unverified` (they confirm their email first), `used` (its `song_id`) or `taken`; `video` gives `made` and `left` this month. A free video is a lyric video of one of their free songs, in one format, of at most 60 seconds of it (a `trim`, a part of a longer song), drawing its clips from up to 3 sources, with no watermark: quote and create it as usual; the quote says `free: true` and nothing is charged.
   - Anything else without a plan answers 402 `plan_required`: tell the user to choose a plan in the Kinetune app (Settings, Billing) before going further.
2. **Find the song**: `list_songs` with `q` (part of the title or artist) / `kinetune songs list --q "…"`. Only songs with `analysis.status` `"ready"` can be used. Lists return 50 per page; page with `limit`/`offset` rather than listing everything.
3. **Released songs need no upload.** When the song is out on the stores, link the artist once and import it: `find_artist_in_catalog` with their full name (or their Apple Music link) / `kinetune artists find --q "…"`, show the matches (song count, newest release) and let the user say which is theirs, then `link_artist` with that id and `catalog_confirmed: true` once they have / `kinetune artists link ARTIST_ID --catalog-artist-id … --catalog-confirmed`. Only in Artist and Label workspaces, and only on the user's word. After that, `list_released_songs` (`kinetune songs released ARTIST_ID`) lists their releases newest first (`song_id` set on the ones already in), and `import_songs` with the `track_ids` / `kinetune songs import --artist-id … --track-ids …` brings them in (releases flagged as possible duplicates go in `distinct_track_ids` once the user confirms they are different recordings): Kinetune fetches each song's audio (about half a minute), uses the release's title and cover, and analyzes it. Follow them with `list_song_imports` or `get_song_import`; one that fails (we couldn't fetch it) asks for the file: `add_song_import_audio` / `kinetune songs import-audio IMPORT_ID --audio ./song.wav`. Only the user's own songs, or songs they have the rights to.
4. **Unreleased song?** Find or create the artist (`list_artists` / `create_artist`), then upload: `create_song` / `kinetune songs create --artist-id … --title … --audio ./song.wav --cover-art ./cover.jpg`. In an artist workspace (`type` `"artist"`, one artist) leave `artist_id` out: its songs are its artist's. Audio is MP3, WAV or M4A up to 250 MB (the title may be left out: the file's tags name it; the same song twice answers 409 `duplicate` with its `song_id`); the cover is a square JPEG or PNG, 1000–6000 px (3000×3000 recommended). Analysis (lyrics, beats, sections) takes about a minute: poll `get_song` until `analysis.status` is `"ready"`. If it fails, `reanalyze_song`. If the user has the song's lyrics, pass them too (see [The song's own lyrics](#the-songs-own-lyrics)). The cover is optional: a song can be added without one and get one later (see [Cover art](#cover-art)).
   - **A file on the user's computer, but your tools only take URLs?** `create_upload` (`filename`, `content_type`, exact `size_bytes`) returns an `upload_url` to PUT the bytes to within an hour (`curl -X PUT -H "content-type: …" --upload-file song.wav "<upload_url>"`), and a `file_url` to pass in any `*_url` field (`audio_url`, `cover_art_url`, `photo_urls`, `image_url`, `file_url`) within a day. The CLI sends files itself.
5. **Choose what to make** (below), then **quote** it and tell the user the credits.
6. **Create** with the same input plus `quote_id`: `create_lyric_video` / `create_canvas`, or `kinetune create … --quote-id …`. It returns the video ids at once.
7. **Wait**: `wait_for_video` (call again while `status` is `queued` or `processing`) / `kinetune videos wait VIDEO_ID`. While it waits, `queue` says why: `plan_limit` (the workspace already makes as many videos at once as its plan allows; `position` is its place in line) or `render_capacity` (every render machine is busy; `estimated_wait_seconds` is about how long). Tell the user rather than calling it stuck.
8. **Deliver**: a completed video has one output per format with `download_url` (full quality), `web_url` (720p) and `poster_url`, signed for 7 days. Share the links, or `kinetune videos download VIDEO_ID --out ./videos`. A lyric video's poster is a frame with one whole lyric line on screen, made for the thumbnail where the user posts it; `poster_seconds` says where in the file it was cut, for platforms that take a cover frame offset instead of an image (TikTok).

## The song's own lyrics

Songs are transcribed to time every word, and a transcription can mishear one. When the user has the song's lyrics (pasted, Suno's lyrics, or an `.srt`, `.vtt` or `.lrc` file), give them to Kinetune: its videos then spell every word their way and break lines where they do, still timed from the audio.

- With the upload: `lyrics` on `create_song` / `kinetune songs create … --lyrics-file ./song.txt`.
- Later, or to fix them: `set_song_lyrics` / `kinetune songs set-lyrics SONG_ID --lyrics-file ./song.srt`. `clear_song_lyrics` / `kinetune songs clear-lyrics SONG_ID` goes back to the transcribed words.
- Pass the text as the user has it: section tags like `[Chorus]` are fine, and a part sung again can be written once.
- `get_song` → `analysis.lyrics.status`: `transcribed` (as heard), `checked` (theirs, timed from the audio; `corrected` and `added` say what changed), `waiting` (checked once the analysis finishes) or `mismatch`.
- Lyrics that don't match what's sung, or cover only part of the song, are refused with the reason and the song stays as it was: tell the user what it says.
- Videos made from then on use the lyrics. Videos already made stay as they are; making one again picks them up.
- **A word or its timing is off** (a word too early, a line broken in the wrong place): `get_song_analysis` gives every word with its `start`, `end` and `line` (true on the first word of a line). Change what is wrong (the text, the times, `line`, a word removed or added) and send every word back with `set_song_words` / `kinetune songs set-words SONG_ID --input words.json` (`{"words":[…]}`). It is saved exactly as sent, nothing timed again (`analysis.lyrics.edited_at` says when); `set_song_lyrics` or `clear_song_lyrics` later times every word from the audio again.
- **Heard badly, or in the wrong language**: `transcribe_song` listens to the song again with Kinetune's current transcription model, for a few credits. Get the price (`get_transcription_price` / `kinetune songs transcription-price SONG_ID`), tell the user, and after their go `kinetune songs transcribe SONG_ID --credits N [--language fr]`. Their own lyrics stay, timed to the new hearing; words fixed by hand are replaced.
- A video can hide swear words on its own: `display.censor` shows their vowels as `*` (the song is untouched).

## What the song is about (its brief)

Once a song is analysed, Kinetune writes its **brief**: what it is about, as every director reads it before making anything for the song (videos, Canvases, covers). `get_song` → `brief`: `theme` (one line), `story`, `mood_arc` (the feeling section by section), `setting`, `tone`; `status` `writing` for about half a minute after the analysis (get the song again); `edited` true once the user corrected it.

- Show it when the user asks what Kinetune understood, or before a music video or cover if the song's meaning matters to them.
- **It read the song wrong** ("it's about my father, not a breakup"): `edit_song_brief` / `kinetune songs edit-brief SONG_ID --theme "…" [--story "…"] [--setting "…"] [--tone "…"] [--mood-arc '[{"section":"Verse 1","feeling":"…"}]']`, in the user's words. Only the fields sent change; `mood_arc` replaces the whole list; `theme` can't be emptied. Plain text, one line each. Their correction is kept: new lyrics or a new analysis don't replace it.
- `rewrite_song_brief` / `kinetune songs rewrite-brief SONG_ID` drops their correction and reads the song again (free, a few times a day). Ask first, since their words are lost; then get the song until `brief.status` is `ready`.

## Choosing a lyric video

Read `get_options` / `kinetune options` for the current lyric styles, visuals, Stock collections, transitions, grades, headlines and credit prices. Don't guess ids. No AI designs a lyric video: it costs its setup and render only.

- **Look** (`look`):
  - Left out, Kinetune picks a Look and a clips picture for the song by rules, at once: `get_song_draft` / `kinetune songs draft SONG_ID` shows which (`look_id`, `background`, `edit.seed`), so you can show it and change any part. A video made without `look.id` is exactly that.
  - `{"id":"look_…"}` reuses a saved Look. Browse with `list_looks` (`scope`: `official`, `community`, `mine`; filter by `lyric_style` or `category`, or `q`; pass the song's `artist_id` to leave out Looks that show another artist).
  - Change any part beside it: `lyric_style`, `visual` and `background`. `get_options` → `lyric_styles` lists the 63 styles, each with its name and what it does (among them Slab, the line packed into a solid block of heavy caps, and Stamp, one heavy word at a time).
- **The picture** (`look.background`; left out, the Look keeps its own):
  - **Clips**: `{"kind":"clips","sources":{"moods":["city"],"folders":["folder_…"]}}`, short clips cut on the beat. `moods` are Kinetune's Stock collections, free for everyone (`get_options` → `clip_moods`, under Feelings or Places & textures); `folders` are the user's own footage (see [Footage](#footage-stock-and-the-users-own-clips)). How many sources one video may draw from is the plan's (`get_account` → `limits.clip_sources`; more answers 402 `plan_required`). Optional: `density` (`one`, `fewer`, `even`, `more`: how often it cuts), `transitions` (`style` `auto`, `impact`, `flash`, `smooth` or `none`; `amount` and `feel`, 0–1), `grade` (`clean`, `vivid`, `warm`, `noir`…), `shape` (`both`, `tall`, `wide`), `fill` (`cover`, `blur`, `bars`: how a clip of the other shape fills the frame) and `speed` (0.5 to 2).
  - **Visualizer**: `{"kind":"visualizer"}`, no picture: the visual that moves with the music fills the frame behind the words.
- **`edit`** (a clips picture): `{"seed":7}` draws its clips (the same seed, the same video; another seed, other clips on the same cuts). `clips` instead lists them exactly as a studio previewed them (`list_clips` gives the pool), each clip's join set on its own if wanted (`transition_seconds`, `direction`, `intensity`).
- **The visual** (the audio-reactive layer, 3D or Liquid): left out, the Look's own. When the user wants a particular one, `look.visual`: `{"preset":"<key>"}` for the whole song, plus `"sections":{"chorus":"<key>"}` for another in some sections, or `{"preset":"none"}` for no visual. Keys come from `get_options` / `kinetune options` (`visualizers`: each visual's `key`, `family`, `name`, `energy`, and for Liquid its `moods`, `colors` and a `preview_url` still). Over a picture, a visual stays light so the picture shows.
- **`variations`** (1–4, with a clips picture): several videos, each drawing other clips.
- **`aspect_ratios`**: any of `"9:16"`, `"16:9"`, `"1:1"`, each its own file. Each extra format adds credits.
- **`display`**: which elements show (`title`, `artist`, `cover`, `lyrics`, `badges`, `headline`) and their sizes; `censor` hides swear words; `title_card` says when a kinetic lyric style shows its big title card instead of the small label (`ends`, `breaks` or `label`; `get_options` → `title_cards`).
- **`trim`**: render only part of the song (at least 8 seconds). Take it from the song's **parts** (`get_song_analysis` / `kinetune songs analysis SONG_ID`): each section (Verse 1, Chorus 1…), clips of about 15 seconds cut between the lines of long ones, and each verse with its chorus, every one starting and ending between lines so no word is cut. Pass a part's `start` and `end` as `start_seconds` and `end_seconds`; `default_part` is the one the app starts on (the sung part nearest 15 seconds, the cheapest clip). Suggest a part by its label and first words ("Chorus 1, “Didn't I tell you…”, 15 s") rather than raw times.

Ask the user only what matters to them, usually the mood of the footage, the formats, and whether to reuse a Look. Default the rest.

### Footage: Stock and the user's own clips

- **Look inside a Stock collection**: `browse_clip_mood` / `kinetune clips browse MOOD_ID [--shape tall|wide] [--energy calm|medium|bold] [--sort best|newest|length]`, a page at a time (`next_offset`). Each clip has its shape, length, energy, colour, a poster, a light preview and its credit. Free.
- **More like this**: `similar_clips` / `kinetune clips similar CLIP_ID` finds up to 48 clips from Stock and the user's folders that look and move like one clip, each with its `source`. Free, no AI.
- **Their own footage** (My footage): `list_clip_folders`, `create_clip_folder` (`kinetune clip-folders create --name "On stage"`), then `import_clips` / `kinetune clip-folders import FOLDER_ID` with one of `--url` (a YouTube, TikTok or Instagram link, up to 20 minutes), `--file-url` (a file sent with `create_upload`) or `--stock-id` (a stock video from `search_stock` with `kind` video). Kinetune cuts it into clips of 1–5 seconds at every scene change, in the background; `get_clip_folder` shows them arriving. Free with a plan; it counts toward storage. Only footage the user has the rights to. Shortly after import each clip is analysed (`list_clips` then shows its `subject`: faces, and a silhouette when it has a `matte_url`; a photo clip gets `still_url` and `depth_url`): words keep clear of faces, the **Behind** lyric style stacks words behind the singer of clips that have a matte (Kinetune puts those clips under the sung lines), and photo clips move in depth. `move_clips`, `delete_clip`, `rename_clip_folder` and `delete_clip_folder` tidy them (ask before deleting).
- **New clips from a few words**: `generate_clips` / `kinetune clips generate --prompt "…" --kind image|video [--shape tall|wide] [--count N]`. Without `credits` it only prices them: tell the user, then send the same with `--credits N` after their go. They land in the AI folder, each charged once it is a clip. The only paid part of a lyric video's picture; needs a plan.
- **Keep the result as their Look**: `save_look` / `kinetune looks save --name "…" --from-id LOOK_ID [--lyric-style …] [--background '{…}']` saves a private Look from a saved one with their changes; then use its id.

## Choosing a Canvas

- **`source`**: `ai-video` (default, a video model animates an image made for the idea), `ai-image`, `stock-photo`, `stock-video`.
- **`inspiration`**: `song` (default: its words, story and sound lead; the colors stay close to the cover so the two sit together in the player) or `cover` (the cover art's world, colors and motifs, brought to life). Spotify asks for a Canvas tied to the release, so offer both.
- **`seconds`**: 5–8, default 8. Spotify loops it, so no part of the song is chosen.
- **Optional**: `style` (`cinematic`, `dreamy`, `abstract`, `surreal`, `retro`, `minimal`, `dark`, `vibrant`, `nature`, `urban`), `direction`, `feature_artist` (`auto`, `always`, `never`, AI sources only; `always` needs a character: `list_characters`, `start_character`), `performer` (a Creator workspace only: the cast character who may appear), `quality` (`standard` or `high`), and `resolution` (`1080p` or `720p`).
- **`variations`**: 1–4 different Canvases.
- **A stock pick of the user's own** (a stock source): `search_stock` / `kinetune stock search --kind photo|video --q "…" --seconds 8`, then its id as `stock_id`. Credit its creator wherever you show it.

## Cover art

Every cover a song has had, made or uploaded, stays on it until deleted; one is **in use** (videos, Looks, Canvases and plans made from then on use it; videos already made keep theirs).

- **See them**: `list_song_covers` / `kinetune covers list SONG_ID`: each cover (`in_use`, `download_url` at full size, a made one's idea), rounds being made, `can` (`plan`: making covers needs a paid plan; `ready`: the song is analysed; `artist`: the artist has a character, so they can be on a cover), `price.counts` (credits for 1 to 4) and `price.redraw`.
- **Their own file**: `set_song_cover` / `kinetune songs set-cover SONG_ID --cover-art ./cover.jpg` adds one or replaces the one in use (a square JPEG or PNG, 1000–6000 px). The one before stays on the song. No plan needed.
- **Make covers** (paid plan, analysed song): ask what they want rather than guessing, then quote.
  - `count` 1–4 (each a different idea; ask how many).
  - `style` (`auto`: the director picks) and up to 2 `vibes` (dark, sad, romantic, aggressive, luxury…): `get_options` → `cover_art` lists the styles (each with a picture), vibes and words. Don't guess ids.
  - `artist`: `auto`, `always` (needs their character) or `never`.
  - `words`: `none`, `title` or `artist_title`, lettered into the cover in the artist's typeface and read back letter by letter, always spelled as the song is.
  - `idea`: the user's own words, if they have one.
  - Quote with `quote_cover` / `kinetune covers quote --count N` and tell the user the credits, then `make_cover` / `kinetune covers make --song-id … --count N [--style …] [--vibes sad dark] [--words title] [--idea "…"] --credits N` after their go. Covers appear one by one in about a minute (`get_cover_request` / `kinetune covers get REQUEST_ID`); only covers that pass their check are charged. One round per song at a time.
- **Redraw one** with the user's note ("warmer, the moon bigger"): `redraw_cover` / `kinetune covers redraw REQUEST_ID --cover-id … --note "…" --credits N`, priced at `redraw_credits` (ask first). The cover it came from stays.
- **Use one**: `use_cover` / `kinetune covers use SONG_ID COVER_ID`. The song keeps its cover until the user picks one; never put one in use on your own. A made cover's colours and typeface become the artist's look for their next covers.
- **Delete one**: `delete_cover` / `kinetune covers delete SONG_ID COVER_ID`, for good. Ask first. Deleting the one in use leaves the song without a cover.
- While a video of the song is being made, changing or deleting its cover answers 409 `song_busy`: try again once it's done.

## Music videos

The song's artist performs it, lip-synced, kept from their **character** (a sheet in the kit, drawn once from a few photos), in shots cut between the song's lines and edited like an official video. A music video is one file in one format (`aspect_ratio`), of the whole song or a `trim` of 15 seconds to 10 minutes (best a part from `get_song_analysis` with `min_seconds` 15), priced by its length.

**The model**: Kinetune has five music video models: `one-take` (MiniMax H3 Max, the default, `768p` by default, up to `1080p`), `seedance-2-0` (Seedance 2.0, up to `4k`), `seedance-2-0-fast` (Seedance 2.0 Fast, up to `720p`), `seedance-2-0-mini` (Seedance 2.0 Mini, up to `720p`) and `seedance-2-5` (Seedance 2.5, up to `1080p`); the Seedance models are `720p` by default. `get_options` lists the ones offered to this user (some are offered to staff first), each with its kinds, visual styles, formats, qualities, lengths and kit limits, and its `guide`: `best_for` (one line), `strengths`, `weaknesses`, the kinds it `suits`, `max_quality`, public `benchmarks` (board, score, source, date) and `page_url` (kinetune.com/ai-music-video-models compares them). In short: MiniMax H3 Max for close-up singing and performance, for less; Seedance for movement (dance, action, live, band, cinematic, one take, places). `quality` is one of the model's qualities; a sharper one is the same video for more credits. A model with `face_check` true shows a real person only once they have passed a face check.

**Choosing the model**: when the user hasn't named one, don't silently use the default. Recommend the kind's `recommended_model` (`get_options` → `music_video.categories[]`), say why in one line from its `guide.best_for`, and show the price difference: quote the same brief with the recommended model and with the other offered models (`quote_music_video` is free) and give each one's credits. Then let them choose any offered model. A plan keeps the model it was planned with: quoting it with `plan_id` alone uses that model and its default quality.

It is planned before it is made. Writing the shot list costs credits, and so does each rewrite from a note:

1. **Character**: `list_characters` with the artist's `artist_id`. None yet: make one with `start_character` / `kinetune characters start --mode …`, in one of four ways:
   - `generate`: 1 to 6 photos of the person (a sharp face with the mouth visible, and one full length). They are read and given a plain base outfit line the user may edit; drawing the sheet is priced (`draw_character_sheet` / `kinetune characters draw REQUEST_ID --credits N [--outfit "…"] [--note "…"]`). Paid plan.
   - `describe`: a made-up person in words (`description`: looks, age, build, hair, style; no photos), drawn the same way and kept as AI-made. Never a real or famous person (422). Paid plan.
   - `add`: one finished character sheet, read and kept as it is, free.
   - `camera`: the six shots the app's **Take from camera** takes (or the three head shots when the body is skipped), laid out as a sheet, free. The user does this in Kinetune itself (Take from camera, guided turn by turn); point them there rather than asking for the shots yourself.

   Then follow the request (`get_character_request` / `kinetune characters get REQUEST_ID`): tell the user the credits before drawing, show the sheet, redraw it with a `note` if they ask (priced again), and `keep_character` / `kinetune characters keep REQUEST_ID --artist-id …` keeps it (or `--name "…"` for cast). Ask whether the person is AI-made (`ai_made`) or real; never guess it. A drawn sheet waits 30 days to be kept (`list_character_requests` / `kinetune characters waiting` finds it; `discard_character_request` deletes it). Photos, sheets and shots need the rights to use them (`rights_confirmed`). Other people in a video are the kit's cast characters, by `@handle` in `direction`.
2. **Face check** (a real person, on a model with `face_check`): `get_face_check`. Not passed: with the person's consent, `start_face_check` gives a link (good for 30 minutes) they open on a phone with a camera; then `get_face_check` until it says `passed`. One check covers every model that shares it; `withdraw_face_check` withdraws it if the person asks.
   **Who else is in it** (optional, before the shot list): nobody appears on screen without a character sheet. `cast_mode` `director` (the default) lets the director cast up to 3 more people; `exact` keeps it to who the user picked (`performer`, `cast`, `people`), and the song's other people are only suggested by the scene. With `director`, `cast_music_video` / `kinetune casting propose --song-id … [same brief]` (free, a few seconds) proposes them, each with a role, a first name, what they do, a description and about how many shots. Show each with `price.per_person`; the user may drop anyone, edit a description, or put a kit character in that role instead (free). After their yes, `draw_music_video_cast` / `kinetune casting draw CASTING_ID --people '[…]' --credits N` draws the sheets (one made-up adult each, kept in the artist's kit as AI-made); poll `get_music_video_casting` until `ready`, then add its `people_for_brief` to `people`. Or order the video with `casting_id` to draw them inside the order. `delete_music_video_casting` drops a casting (people already kept stay).
3. **Shot list** (with the model the user chose): `list_music_video_drafts` / `kinetune plans list --song-id …` gives `price.credits`, what writing one costs now. Tell the user, and after their go `plan_music_video` / `kinetune plan music-video --song-id … --credits N [--model …] [--category …] [--aspect-ratio 9:16] [--trim '{"start_seconds":32,"end_seconds":96}'] [--direction "…"]`. The credits are held while it is written, charged once it is saved and released if it fails; another price answers 409 with the current one. A video made from it has no creative direction line in its quote. It takes a minute or more and returns the director's concept and its shots (time, lyrics, summary). Show it to the user. With `wait: false` it answers at once: `get_music_video_plan_write` / `kinetune plans writing WRITE_ID` reads the shots as they are written until `status` is `ready` (its `plan_id`). It is written even if the request is cut off, and `list_music_video_drafts` lists it under `writing`. `takes` is `cuts` (the default, edited shots) or `continuous` (the director may carry a shot on into the next through a picture drawn at the join, priced per picture).
4. **Revise** until they like it: `plan_music_video` with `revise` `{plan_id, note}` and the same `credits` (a rewrite costs the same; say so first). Each rewrite is a new version of the same shot list: `list_music_video_versions` / `kinetune plans versions PLAN_ID` lists them (oldest first, the note each was written from); any version's `id` can be opened, quoted and made, and `delete_music_video_version` / `kinetune plans delete-version PLAN_ID VERSION_ID` deletes one a video wasn't made from (its pictures stay).
   - **Opening frames** (optional, for users who want control): offer a frame for the key shots, the opening and the hook. `list_music_video_frames` / `kinetune plans frames PLAN_ID` gives `price.credits` (per picture) and the shots that can't have one (`unavailable`, with why). Say the price and draw only after their go: `draw_music_video_frame` / `kinetune plans draw-frame PLAN_ID --shot N --credits N`, about a minute. Show each picture (`image_url`) and draw it again if they ask (priced again). The video then opens that shot on it, on every model; frames are never part of the quote.
   - **The shot list's pictures**: everything its shots are made from that is drawn. The director covers every place from the angles its shots need (a plan's `places[].angle`: `kind` from a photographer's range, wide, three-quarter, side, reverse, corner, low, high, overhead, detail, framed, and `camera`, where it stands), so no video is one viewpoint of a set; it draws the outfits on its people; and it may design up to two props a shot turns on (`designed_props`, numbered after `props`). `list_music_video_pictures` / `kinetune plans pictures PLAN_ID` lists them by key (`place:3`, `outfit:1`, `outfit:world`, `person:1`, `prop:2`), each with its price and its picture once drawn. To see them before the video, say the total and, after the user's go, `draw_music_video_pictures` / `kinetune plans draw-pictures PLAN_ID --confirm --credits N` (about a minute; without `--confirm` it only prices them; `pictures` and `redraw` draw chosen ones again). Otherwise they are drawn with the video, in its quote (Set angles, Outfits on the character, Props). Every drawn picture also waits in the artist's kit, `pending`: tell the user they can keep it (`update_kit_piece` with `kept: true`) or leave it out (`delete_kit_piece`). `drop_music_video_angle` / `kinetune plans drop-angle PLAN_ID PLACE` leaves an angle out (the next version, free).
   - **Another model** (free): `check_music_video_model` / `kinetune plans check-model PLAN_ID --model …` says whether the shot list can be kept with another model (`can_switch`, or `problems`: it has to be written again), what that model needs before paying (`needs`: a face check, a new character sheet) and how the video is made differently (`changes`). Tell the user, then `switch_music_video_model` / `kinetune plans switch-model PLAN_ID --model …` after their yes: the next version, with the same shots, idea, people and frames. Quote it with the new plan id.
   - **Pick up later**: `list_music_video_drafts` / `kinetune plans list --song-id …` lists a song's shot lists not made into a video yet (last 30 days); `get_music_video_plan` opens one, `delete_music_video_plan` drops one (its opening frames are kept).
5. **Quote** with `song_id` and `plan_id`: `quote_music_video` / `kinetune quote music-video --song-id … --plan-id …`. Tell the user the credits and that it usually takes 5 to 20 minutes.
   - **Test it first** (optional): the same quote with `test: true` (`--test`) prices only the plan's test shots (its first chorus, about 20 seconds at most, at 480p) for a fraction of the price. Offer it beside the whole video's quote; then order the whole video from the same `plan_id` without `test`.
   - **Outro** (optional, free): `outro` ends on a release card after the last shot, the song playing on under it (up to 5 seconds longer).
6. **Create** after their go: `create_music_video` / `kinetune create music-video --song-id … --plan-id … --quote-id …`. Check back with `get_video` rather than waiting in one call.
7. **One shot again**: while `revisable_until` (`get_video`) is ahead, `revise_music_video_shot` rewrites one shot for free; quote it with the video and make it after their go. The other shots are kept, not paid for twice.
   - **Find the shot they mean.** The user names a time ("at 0:12 her hand looks wrong", "the last few seconds") or describes the moment. Read `get_video` → `music_video.shots[]` (`index`, `start_seconds`, `end_seconds`, `summary`, `lyrics`) and pick the shot whose span holds that time, counted from the start of the delivered video (an outro card comes after the last shot, so "the last few seconds" may be the card, not a shot). For a described moment, match it to the `summary` and `lyrics`.
   - **Confirm it in one line** before revising: the shot's summary and its seconds ("Shot 4, 0:10–0:15: she reaches for the rail on the rooftop. That one?").
   - Then `revise_music_video_shot` (`kinetune videos revise VIDEO_ID --shot N --note "…"`) with that `index` and the user's own words as the note. Show the rewritten shot, quote it with `song_id`, `plan_id` and the video's `quality` (only the shots made again are charged), and make it only after their go.
   - A shot may hold several camera cuts (Seedance makes multi-shot clips): revising it makes that whole shot again.
   - **Several shots at once**: `shots` (up to 12, each `{shot, note}`) in one call gives one new version. A shot that the next one carries straight on from makes that one again too.
   - **What it costs before rewriting**: `price_music_video_revision` / `kinetune videos revision-prices VIDEO_ID`: each shot's credits, plus `base_credits` paid once per revision.
   - **Another outro, or none**: `create_music_video` with `refinish` `{video_id}` and the new `outro` (or none) finishes it again without making any shot again; only setup and delivery are charged.
   - The original video never changes: each revision is a new video.
8. **Keep its direction as a Look** (free): `save_music_video_look` / `kinetune videos save-look VIDEO_ID` saves a delivered music video's direction (kind, style, edit, treatment, AI-drawn places and props, never the people) as a reusable Look, private until the video is made public. Use it on any song with `look_id` in `plan_music_video` or `quote_music_video`. `list_music_video_looks` / `kinetune looks music-video [--scope all|community|mine]` lists the workspace's own and the public ones; `get_music_video_look` shows one (with its video and story while that video is public); `delete_music_video_look` removes one of theirs.

### The artist kit and @mentions

The kit holds what music videos are made from: **characters**, **outfits**, **places** and **props**. Each is in one artist's kit (`owner_artist_id`; a project in a Creator workspace) and serves that artist's songs; `shared: true` offers it to every other artist's songs too. Each piece has an `@handle`.

- `list_kit` / `kinetune kit list [--artist-id …]`: with `artist_id`, what a brief for that artist's song can mention (theirs, then what others shared); `owner_artist_id` lists only that artist's kit. `list_characters` takes `for_artist_id` the same way.
- `add_kit_piece` / `kinetune kit add --kind outfit|place|prop --name "…" …` from an image (`file`, or `file_url` with `rights_confirmed`) or a stock photo (`stock_id` from `search_stock`), with an optional `crop` (fractions of the image: `{"x","y","w","h"}`, at least 256 px kept). An outfit needs its `artist_id`; a place or prop takes `owner_artist_id`, the artist whose kit it goes in (ask whose it is in a workspace with several artists). `keep_character` takes `owner_artist_id` for someone who isn't an artist.
- `update_kit_piece` renames it, changes its handle or description, crops it again or shares it (`shared`); `delete_kit_piece` removes it. `get_kit_piece` / `kinetune kit get KIT_ID` shows one.
- **More angles of a place**: `draw_place_angles` / `kinetune kit angles KIT_ID --angles reverse low overhead` draws the same set from the angles picked (one or more of wide, three-quarter, side, reverse, corner, low, high, overhead, detail, framed; three-quarter, side and low when none), each a new place in the kit, from a camera spot worked out for that place. Without `confirm` it only prices them: tell the user, then send `confirm: true` with the `credits` shown. A new angle waits for the user to keep it (`update_kit_piece` with `kept: true`) or leave it out (`delete_kit_piece`). A brief that mentions places of one set has the director cut between the angles.

A piece mentioned in `direction` ("Nina in the @blue-set on the @rooftop") is used exactly as it is, and the director designs the rest. An unknown handle is refused with its name: offer to add it to the kit. Another artist's piece is refused until they share it: offer to share it (`update_kit_piece` with `shared: true`) once the user agrees. For places the brief leaves open, `scenes` `invent` (default) has the director imagine them, and `stock` has it find real empty places and put the artist in them (`swap_music_video_place` changes one place's photo).

### Backing vocals

Words in (parentheses) in the song's lyrics are the backing vocals': they are sung off camera, and the artist doesn't lip-sync them. To mark them, set the song's lyrics with the parentheses (see [The song's own lyrics](#the-songs-own-lyrics)).

## Artists, songs and Looks

- **Artist picture** (only a display picture, never shown to a model): `set_artist_image` / `kinetune artists image set ARTIST_ID --image ./photo.jpg [--crop '{"x":0.1,"y":0,"size":0.8}']` (JPEG, PNG or WebP, at least 512 px), `remove_artist_image` to go back to their initial. Works without a plan.
- `get_artist`, `rename_artist`, `archive_artist` (an artist with no songs); `rename_song`, `archive_song` (its videos stay); `list_videos` (by `song_id` or `type`).
- Lyric video Looks the workspace made: `get_look`, `rename_look`, `set_look_visibility` (`public` or `private`), `delete_look` (videos made with it stay).
- **Drafts**: a video started in the studio (web or app) and not made yet is a draft, one per song and kind, kept 30 days. `list_video_drafts` / `kinetune drafts list` lists them; `delete_video_drafts` / `kinetune drafts delete --id …` deletes one, or `--all` every one (narrowed by `--song-id` or `--type`). Only when the user asks: tell them how many first. Videos already made are never touched.
- Ask before archiving or deleting anything.

## Artist pages

An Artist or Label workspace gives each artist a page at kinetune.com/@handle (their music, videos and links).

- **Address**: `check_handle` / `kinetune artists page check HANDLE`, then `claim_artist_handle` / `kinetune artists page claim ARTIST_ID --handle …`.
- **Their links on other services**: `list_artist_profiles` / `kinetune artists profiles list ARTIST_ID`. Linking the artist to Apple Music (`link_artist`) searches for their profiles by itself (`discover_artist_profiles` looks again). A profile found that way is on the page only once the user confirms it is theirs: show them, then `update_artist_profile` per profile, or `confirm_all_artist_profiles` only when they say every one is theirs. `add_artist_profile` adds a link by hand, `reorder_artist_profiles` and `remove_artist_profile` arrange them (a removed one is never found again).
- **The page**: `get_artist_page` (its status and draft), `update_artist_page` / `kinetune artists page update ARTIST_ID …` (look, photo, genres, sections, featured music and videos; send only what changes), `preview_artist_page` to show the draft as fans will see it. Fans see changes only once published: `publish_artist_page` after the user's go (`unpublish_artist_page` takes it offline, everything kept). `get_artist_page_stats` gives views, visitors and clicks.

## Making a video public

Only when the user asks, in their own words: `share_video` (`kinetune videos share VIDEO_ID`) puts a delivered video in the app's Community, Kinetune may feature it on kinetune.com, and its Look becomes reusable by anyone (a music video's concept: kind, look, edit, places and props, never the people in it). Ask before calling it and never make a video public on your own. Its two choices are the user's alone, so never set them yourself or carry them over from another video:

- `rights_confirmed: true` (`--rights-confirmed`, required for a music video): they have the right to show the artist and everyone in it publicly.
- `socials: true` with `credit_handle` (`--socials --credit-handle @name`): Kinetune may also post it on its socials, crediting them.

`unshare_video` (`kinetune videos unshare VIDEO_ID`) makes it private again, its Look too. `get_video` shows `sharing` (null while private).

## When they wish Kinetune did something

If the user asks for something Kinetune can't do yet ("I wish it could…"), look for it first: `list_ideas` with `q` (`kinetune ideas list --q "…"`), and if it is already there, offer to vote for it (`vote_for_idea`, `kinetune ideas vote IDEA_ID`; `follow_idea` for an email when it ships; `get_idea` shows one with its comments; `remove_idea_vote` and `unfollow_idea` undo them). Otherwise offer to suggest it as an idea: `suggest_idea` (`kinetune ideas suggest --title "…" [--body "…"] [--area lyric-video|canvas|music-video|everything]`). It goes on the Community board in the app under their name, where others vote on it, and they hear by email when it ships. Write the title in their words, in one line, with how they would use it in the body. Leave out private links and contact details, and ask before posting. At most 5 a day.

## When something goes wrong

- **A failed video returns its credits.**
- **`retry_video`** makes a video again. That is a new charge, so ask first, but it reuses the Look and media so nothing already made is paid for twice.
- **`cancel_video`** stops a queued or processing video and releases its credits.
- **"plan it again"**: a music video plan whose photo or kit piece was deleted, or made with choices no longer offered, can't be quoted. Write the shot list again (`plan_music_video`).
- **Not enough credits**: tell the user their balance and the quote. Plans and credit packs are at https://kinetune.com/pricing. Don't shrink the request without asking.
- **`plan_required`** (402): the workspace has no active plan, so it can't upload or make this (only the Free plan's song and lyric videos work without one). Tell the user to choose one in the app; don't retry.
- **`storage_full`** (402): the plan's storage is full (`get_account`: `storage`). Tell the user; deleting videos (`delete_video`) or archiving songs (`archive_song`) they no longer need frees space at once, or they can move up a plan. Ask before deleting anything.
- **Workspaces**: the app calls them workspaces; tools and fields say organization (`list_organizations`, `create_organization` with a `type`, `update_organization`). One connection acts for one workspace: to use another, the user reconnects and picks it.
- **CLI exit codes**:
  - `0`: done
  - `1`: API or network error
  - `2`: invalid usage, or the quote is over `--max-credits`
  - `3`: not signed in or not allowed
- **Unsure of an input?** `kinetune schema "quote canvas"` prints the exact JSON schema of any command.
- **Full reference**: every operation with its parameters, scopes, CLI command and MCP tool is at https://kinetune.com/llms-full.txt.
