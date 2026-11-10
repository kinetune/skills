---
name: kinetune
description: Make lyric videos (9:16, 16:9, 1:1), Spotify Canvas loops, AI music videos and cover art from a user's songs with Kinetune, through its MCP tools or the `kinetune` CLI. Use when the user wants a lyric video, a Spotify Canvas, a music video, a visualizer or promo video for a song, cover art for a song, wants to set up an artist's character or kit (outfits, places, props), wants to upload a song to Kinetune or correct what it says a song is about, browse Looks, check their credits, or mentions Kinetune.
---

# Kinetune

Kinetune turns a song into release-ready video:

- **Lyric videos**: word-synced lyrics over a designed background with audio-reactive visuals, in 9:16 (vertical), 16:9 (wide) and 1:1 (square).
- **Spotify Canvas**: a 5–8 second vertical loop with its idea from the song's cover or from the song itself, uploaded by the artist in Spotify for Artists.
- **AI music videos**: the song's artist performs it, lip-synced, kept from their character, from 15 seconds to 10 minutes. See [Music videos](#music-videos).
- **Cover art**: Kinetune's cover art director makes 1 to 4 covers for a song (3000×3000), each a different idea, with the title and artist name lettered in if the user wants. See [Cover art](#cover-art).

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

1. **Check the account**: `get_account` / `kinetune account`. Note the credit balance, the workspace's `type` and whether its plan is `active`. Uploading and making videos need an active plan, with one exception:
   - **The free song.** Without a plan, `free_song` (only when signed in as the person, never with an API key) says where their one free song stands: `available` (upload one song now with `create_song`), `unverified` (they confirm their email first), `used` (its `song_id`; `video.available` while its one free video is still to make) or `taken`. The free video is a lyric video of that song with an existing Look (`look.mode` `existing`) in one format: quote and create it as usual; the quote says `free: true` and nothing is charged.
   - Anything else without a plan answers 402 `plan_required`: tell the user to choose a plan in the Kinetune app (Settings, Billing) before going further.
2. **Find the song**: `list_songs` with `q` (part of the title or artist) / `kinetune songs list --q "…"`. Only songs with `analysis.status` `"ready"` can be used. Lists return 50 per page; page with `limit`/`offset` rather than listing everything.
3. **No song yet?** Find or create the artist (`list_artists` / `create_artist`), then upload: `create_song` / `kinetune songs create --artist-id … --title … --audio ./song.wav --cover-art ./cover.jpg`. In an artist workspace (`type` `"artist"`, one artist) leave `artist_id` out: its songs are its artist's. Audio is MP3, WAV or M4A up to 250 MB; the cover is a square JPEG or PNG, 1000–6000 px (3000×3000 recommended). Analysis (lyrics, beats, sections) takes about a minute: poll `get_song` until `analysis.status` is `"ready"`. If it fails, `reanalyze_song`. If the user has the song's lyrics, pass them too (see [The song's own lyrics](#the-songs-own-lyrics)). The cover is optional: a song can be added without one and get one later (see [Cover art](#cover-art)).
   - **A file on the user's computer, but your tools only take URLs?** `create_upload` (`filename`, `content_type`, exact `size_bytes`) returns an `upload_url` to PUT the bytes to within an hour (`curl -X PUT -H "content-type: …" --upload-file song.wav "<upload_url>"`), and a `file_url` to pass in any `*_url` field (`audio_url`, `cover_art_url`, `photo_urls`, `image_url`, `file_url`) within a day. The CLI sends files itself.
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

## What the song is about (its brief)

Once a song is analysed, Kinetune writes its **brief**: what it is about, as every director reads it before making anything for the song (videos, Canvases, covers). `get_song` → `brief`: `theme` (one line), `story`, `mood_arc` (the feeling section by section), `setting`, `tone`; `status` `writing` for about half a minute after the analysis (get the song again); `edited` true once the user corrected it.

- Show it when the user asks what Kinetune understood, or before a music video or cover if the song's meaning matters to them.
- **It read the song wrong** ("it's about my father, not a breakup"): `edit_song_brief` / `kinetune songs edit-brief SONG_ID --theme "…" [--story "…"] [--setting "…"] [--tone "…"] [--mood-arc '[{"section":"Verse 1","feeling":"…"}]']`, in the user's words. Only the fields sent change; `mood_arc` replaces the whole list; `theme` can't be emptied. Plain text, one line each. Their correction is kept: new lyrics or a new analysis don't replace it.
- `rewrite_song_brief` / `kinetune songs rewrite-brief SONG_ID` drops their correction and reads the song again (free, a few times a day). Ask first, since their words are lost; then get the song until `brief.status` is `ready`.

## Choosing a lyric video

Read `get_options` / `kinetune options` for the current categories, background sources, quality tiers, headlines and credit prices. Don't guess ids.

- **Look**:
  - `{"mode":"existing","id":"look_…"}` reuses a saved Look exactly. It is the cheapest choice. Browse with `list_looks` (`scope`: `official`, `community`, `mine`; filter by `category`; pass the song's `artist_id` to leave out Looks that show another artist).
  - `{"mode":"new","category":"…","background":{"source":"ai-image","quality":"high"},"direction":"…","visibility":"private"}` has a new Look designed from the cover.
    - `direction` is the user's mood or references in plain words.
    - The singer stays out of the background unless asked: `"feature_artist":"always"` shows them. It needs an AI background, the artist's character (`list_characters`; `start_character` makes one), `"display":{"cover":false}` (the cover would hide them) and a private Look, which then only serves that artist's songs.
- **Background sources** for a new Look: `ai-image`, `ai-video` (an AI image brought to life as a loop; costs more), `stock-photo`, `stock-video`. AI sources take a `quality` tier.
  - A stock pick of the user's own: `search_stock` / `kinetune stock search --q "…"`, then its id as `background.stock_id` (credit the photographer where you show it).
  - Their own photo or video, or one of their music videos: `upload_background` / `kinetune backgrounds upload --file ./clip.mp4 --rights-confirmed` (or `--video-id` of a delivered music video of the same song: the lyric video plays it in step with the lyrics, with its own sound), then `"background":{"source":"upload-photo"|"upload-video","upload_id":"…"}`.
- **`variations`** (1–4): several different New Looks, one video each.
- **`aspect_ratios`**: any of `"9:16"`, `"16:9"`, `"1:1"`, each its own file. Each extra format adds credits.
- **`display`**: which elements show (`title`, `artist`, `cover`, `lyrics`, `badges`, `headline`) and their sizes.
- **`trim`**: render only part of the song (at least 8 seconds). Take it from the song's **parts** (`get_song_analysis` / `kinetune songs analysis SONG_ID`): each section (Verse 1, Chorus 1…), clips of about 15 seconds cut between the lines of long ones, and each verse with its chorus, every one starting and ending between lines so no word is cut. Pass a part's `start` and `end` as `start_seconds` and `end_seconds`; `default_part` is the one the app starts on (the sung part nearest 15 seconds, the cheapest clip). Suggest a part by its label and first words ("Chorus 1, “Didn't I tell you…”, 15 s") rather than raw times.

Ask the user only what matters to them, usually the category or mood, the formats, and whether to reuse a Look. Default the rest.

## Choosing a Canvas

- **`source`**: `ai-video` (default, a video model animates an image made for the idea), `ai-image`, `stock-photo`, `stock-video`.
- **`inspiration`**: `cover` (default: the cover art's world, colors and motifs, brought to life) or `song` (its words, story and sound lead; the colors stay close to the cover so the two sit together in the player). Spotify asks for a Canvas tied to the release, so offer both.
- **`seconds`**: 5–8, default 8. Spotify loops it, so no part of the song is chosen.
- **Optional**: `style` (`cinematic`, `dreamy`, `abstract`, `surreal`, `retro`, `minimal`, `dark`, `vibrant`, `nature`, `urban`), `direction`, `feature_artist` (`auto`, `always`, `never`, AI sources only; `always` needs a character: `list_characters`, `start_character`), `performer` (a Creator workspace only: the cast character who may appear), `quality` (`standard` or `high`), and `resolution` (`1080p` or `720p`).
- **`variations`**: 1–4 different Canvases.

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

It is planned before it is made, and planning is free:

1. **Character**: `list_characters` with the artist's `artist_id`. None yet: make one with `start_character` / `kinetune characters start --mode …`, in one of four ways:
   - `generate`: 1 to 6 photos of the person (a sharp face with the mouth visible, and one full length). They are read and given a plain base outfit line the user may edit; drawing the sheet is priced (`draw_character_sheet` / `kinetune characters draw REQUEST_ID --credits N [--outfit "…"] [--note "…"]`). Paid plan.
   - `describe`: a made-up person in words (`description`: looks, age, build, hair, style; no photos), drawn the same way and kept as AI-made. Never a real or famous person (422). Paid plan.
   - `add`: one finished character sheet, read and kept as it is, free.
   - `camera`: the six shots the app's **Take from camera** takes (or the three head shots when the body is skipped), laid out as a sheet, free. The user does this in Kinetune itself (Take from camera, guided turn by turn); point them there rather than asking for the shots yourself.

   Then follow the request (`get_character_request` / `kinetune characters get REQUEST_ID`): tell the user the credits before drawing, show the sheet, redraw it with a `note` if they ask (priced again), and `keep_character` / `kinetune characters keep REQUEST_ID --artist-id …` keeps it (or `--name "…"` for cast). Ask whether the person is AI-made (`ai_made`) or real; never guess it. A drawn sheet waits 30 days to be kept (`list_character_requests` / `kinetune characters waiting` finds it; `discard_character_request` deletes it). Photos, sheets and shots need the rights to use them (`rights_confirmed`). Other people in a video are the kit's cast characters, by `@handle` in `direction`.
2. **Face check** (a real person, on a model with `face_check`): `get_face_check`. Not passed: with the person's consent, `start_face_check` gives a link (good for 30 minutes) they open on a phone with a camera; then `get_face_check` until it says `passed`. One check covers every model that shares it; `withdraw_face_check` withdraws it if the person asks.
   **Who else is in it** (optional, before the lookbook and the shot list): nobody appears on screen without a character sheet. `cast_mode` `director` (the default) lets the director cast up to 3 more people; `exact` keeps it to who the user picked (`performer`, `cast`, `people`), and the song's other people are only suggested by the scene. With `director`, `cast_music_video` / `kinetune casting propose --song-id … [same brief]` (free, a few seconds) proposes them, each with a role, a first name, what they do, a description and about how many shots. Show each with `price.per_person`; the user may drop anyone, edit a description, or put a kit character in that role instead (free). After their yes, `draw_music_video_cast` / `kinetune casting draw CASTING_ID --people '[…]' --credits N` draws the sheets (one made-up adult each, kept in the artist's kit as AI-made); poll `get_music_video_casting` until `ready`, then add its `people_for_brief` to `people`. Or order the video with `casting_id` to draw them inside the order. `delete_music_video_casting` drops a casting (people already kept stay).
3. **Shot list** (with the model the user chose): `plan_music_video` / `kinetune plan music-video --song-id … [--model …] [--category …] [--aspect-ratio 9:16] [--trim '{"start_seconds":32,"end_seconds":96}'] [--direction "…"]`. It takes about a minute and returns the director's concept and its shots (time, lyrics, summary). Show it to the user. `takes` is `cuts` (the default, edited shots) or `continuous` (the director may carry a shot on into the next through a picture drawn at the join, priced per picture).
4. **Revise** until they like it: `plan_music_video` with `revise` `{plan_id, note}`.
   - **Opening frames** (optional, for users who want control): offer a frame for the key shots, the opening and the hook. `list_music_video_frames` / `kinetune plans frames PLAN_ID` gives `price.credits` (per picture) and the shots that can't have one (`unavailable`, with why). Say the price and draw only after their go: `draw_music_video_frame` / `kinetune plans draw-frame PLAN_ID --shot N --credits N`, about a minute. Show each picture (`image_url`) and draw it again if they ask (priced again). The video then opens that shot on it, on every model; frames are never part of the quote.
   - **Pick up later**: `list_music_video_drafts` / `kinetune plans list --song-id …` lists a song's shot lists and lookbooks not made into a video yet (last 30 days); `get_music_video_plan` opens one, `delete_music_video_plan` drops one.
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
- **More angles of a place**: `draw_place_angles` / `kinetune kit angles KIT_ID --angles three-quarter side low` draws the same set from other sides, each a new place in the kit. Without `confirm` it only prices them: tell the user, then send `confirm: true` with the `credits` shown. A new angle waits for the user to keep it (`update_kit_piece` with `kept: true`) or leave it out (`delete_kit_piece`). A brief that mentions places of one set has the director cut between the angles.

A piece mentioned in `direction` ("Nina in the @blue-set on the @rooftop") is used exactly as it is, and the director designs the rest. An unknown handle is refused with its name: offer to add it to the kit. Another artist's piece is refused until they share it: offer to share it (`update_kit_piece` with `shared: true`) once the user agrees. For places the brief leaves open, `scenes` `invent` (default) has the director imagine them, and `stock` has it find real empty places and put the artist in them (`swap_music_video_place` changes one place's photo).

### Lookbook (optional, before the shot list)

Before the shot list, `design_music_video_shoot` / `kinetune lookbook design …` (free, about a minute) has the director design what the brief leaves open: outfits drawn on the artist, empty places, props. Show its pieces and `price.draw.credits`; drawing them costs those credits, so draw (`render_music_video_lookbook` with those credits) only after the user's go. Poll `get_music_video_lookbook` / `kinetune lookbook get LOOKBOOK_ID` until `ready` and show the pictures. Then, each priced and only after the user's go:

- `keep_music_video_lookbook_piece` / `kinetune lookbook keep LOOKBOOK_ID PIECE --no-kept` leaves a piece out of the video (free).
- `redraw_music_video_lookbook_piece` / `kinetune lookbook redraw LOOKBOOK_ID PIECE [--note "…"] --credits N` draws one of the director's pieces again (`price.credits_per_picture`).
- `redesign_music_video_lookbook` / `kinetune lookbook redesign LOOKBOOK_ID --note "…" --credits N` redesigns all the director's pieces from a note (`price.redesign.credits`); the user's own pieces stay.
- `update_music_video_lookbook` with `save_to_kit` false uses the kept pieces for that video only (by default they join the artist's kit once a video is planned from it).
- `delete_music_video_lookbook` deletes one nothing was made from (drawn pictures aren't refunded).

Then `plan_music_video` with `lookbook_id`.

### Backing vocals

Words in (parentheses) in the song's lyrics are the backing vocals': they are sung off camera, and the artist doesn't lip-sync them. To mark them, set the song's lyrics with the parentheses (see [The song's own lyrics](#the-songs-own-lyrics)).

## Artists, songs and Looks

- **Artist picture** (only a display picture, never shown to a model): `set_artist_image` / `kinetune artists image set ARTIST_ID --image ./photo.jpg [--crop '{"x":0.1,"y":0,"size":0.8}']` (JPEG, PNG or WebP, at least 512 px), `remove_artist_image` to go back to their initial. Works without a plan.
- `get_artist`, `rename_artist`, `archive_artist` (an artist with no songs); `rename_song`, `archive_song` (its videos stay); `list_videos` (by `song_id` or `type`).
- Lyric video Looks the workspace made: `get_look`, `rename_look`, `set_look_visibility` (`public` or `private`), `delete_look` (videos made with it stay).
- Ask before archiving or deleting anything.

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
- **`plan_required`** (402): the workspace has no active plan, so it can't upload or make this (only the free song and its one free lyric video work without one). Tell the user to choose one in the app; don't retry.
- **`storage_full`** (402): the plan's storage is full (`get_account`: `storage`). Tell the user; deleting videos (`delete_video`) or archiving songs (`archive_song`) they no longer need frees space at once, or they can move up a plan. Ask before deleting anything.
- **Workspaces**: the app calls them workspaces; tools and fields say organization (`list_organizations`, `create_organization` with a `type`, `update_organization`). One connection acts for one workspace: to use another, the user reconnects and picks it.
- **CLI exit codes**:
  - `0`: done
  - `1`: API or network error
  - `2`: invalid usage, or the quote is over `--max-credits`
  - `3`: not signed in or not allowed
- **Unsure of an input?** `kinetune schema "quote canvas"` prints the exact JSON schema of any command.
- **Full reference**: every operation with its parameters, scopes, CLI command and MCP tool is at https://kinetune.com/llms-full.txt.
