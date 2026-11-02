---
name: hookova
description: Work with the user's Hookova account through the Hookova connector — save a TikTok, Shorts, Reels or web link, search and read back what they saved, retag or annotate a save, transcribe a video, read an inspiration board as a hook guide, and list, schedule or cancel posts on their connected social accounts. Use whenever the user mentions Hookova, their saves, their boards, "my references", or asks to schedule a post to YouTube, TikTok, Instagram, Facebook, X, LinkedIn, Pinterest, Threads or Bluesky.
---

# Hookova

Hookova (hookova.com) is a short-video reference library for people who make Shorts, Reels and
TikToks. They save clips and pages they want to learn from, analyse a clip shot by shot, pin saves
to a board for a video they are making, and schedule finished posts to their connected accounts.
This plugin connects that account to the conversation through the hosted Hookova MCP server at
`https://www.hookova.com/mcp`. Signing in is OAuth in the user's own browser; no token is ever
pasted into the chat.

## Start with `auth_status`

Every session, before anything else, call `auth_status`. It says whether the connector is signed
in, as whom, on which plan, and what is left today (analysis credits, saves against the daily
allowance). If it fails with a sign-in error, tell the user to connect the Hookova connector in
their Claude client (the plugin's **Connectors** tab on claude.ai, or `/mcp` → hookova →
Authenticate in Claude Code) and stop — there is no tool that signs in for them.

## What each tool is for

**The library.** `save_link(url, title?)` queues any public URL and returns at once with
`status: queued`; the title, synopsis, takeaways and category land a few seconds later, so read
them back with `get_save(id)`. Saving the same URL twice returns the existing save.
`list_saves(q?, tag?, limit?, cursor?)` browses newest first, searches title, synopsis, URL,
saved text, notes and tags, and returns every tag with a count. `get_save(id)` is the full
record — synopsis, excerpt, takeaways, tags, notes, and whether a shot-by-shot analysis exists for
the same URL. `update_save(id, tags?, notes?, title?, liked?)` — **`tags` replaces the whole
list**, so read first, then write. `delete_save(id)` is permanent; confirm before calling it.
`export_saves(limit?)` is the library in bulk and the one paid-plan tool; on a free plan use
`list_saves` + `get_save`, which read the same rows.

**Transcription.** `transcribe(url? | saveId?)` queues a video and returns an id;
`get_transcription(id?, text?)` reports `queued`, `running` (with stage and percentage), `done`,
or `error`, and with `text: true` returns the transcript as one paragraph; without an id it lists
recent ones. Transcription spends credits (a saved clip costs 1; a link costs 1 per started three
minutes, charged only on success), so state the price and wait for a yes before calling
`transcribe`. The worker takes 30 seconds to two minutes; poll `get_transcription` a few times
rather than promising a time.

**Boards.** `list_projects()` lists the user's boards — id, name, the brief in their words, how
many saves are pinned. `get_hook_guide(projectId, full?, stills?)` writes one board out as a
HOOK-GUIDE: the brief, every pinned save, and for each analysed clip its hook *pattern*, the
opening shot by shot. `stills` (0–12) returns reference frames as images. It is free and generates
nothing. For building the user's own opening from a board, the `hook-from-project` skill in this
plugin is the full workflow.

**Images — credits provided by Pop-Cam.** `generate_image(prompt?, reference_image_url?, preset_id?,
include_image?)` makes one image with Pop-Cam's image model (Nano Banana 2.1). Pop-Cam is the same
maker's image app, bundled with Hookova: there is no Pop-Cam account or key to set up. It costs
**1 Hookova credit per image**, charged only when an image comes back, so say the price before making
several. A reference image (a public https link — the `image_url` of an earlier result works) keeps a
character, product or face the same across every shot, and a 9:16 reference gives a 9:16 image
(768×1376); with no reference the model picks the shape (landscape in testing). `image_url` lives for one hour: use or pass it on
right away. Images carry no text unless the prompt asks for it, so put titles and captions in the
edit, not in the picture.

**Publishing — create, read, update, delete.** `list_publish_accounts()` is the connected social
accounts (id, platform, handle, and `group`: the user's own group it is in, or null).
`list_publish_groups()` is those groups — a brand, a client, a language — each with its accounts;
`create_publish_group(name, accountIds?)`, `update_publish_group(group, name?, addAccountIds?,
removeAccountIds?)` and `delete_publish_group(group)` sort them (an account is in one group at
most; deleting a group never touches its accounts or posts). `publish_post(caption, mediaUrls,
accountIds?, group?, scheduledAt?, draft?, …)` creates a post — `group` (a name) sends it to every
connected account in that group: one video or one to ten photos, given as **public https links to the
files themselves** (up to 100 MB a post, all files together), posted now, at an ISO time *with an offset*
(`2026-09-18T11:00:00-07:00`), or saved as a draft. You cannot pass a file the user pasted into
this chat to a tool. For that, or any file on their device, call `create_upload()`: give them
`uploadPage` (a hookova.com page where they pick the file), or, when you have a shell and the file
is on this machine, PUT it to `putUrl` with its content-type. When they say it is uploaded, pass the
returned `mediaUrl` in `mediaUrls` — any size, since it is already in storage. One file per link;
call it once per photo. Never claim something was posted. `list_posts(status?, group?)` is the queue and the calendar — `draft`,
`scheduled`, `processing`, `processed`; `get_post(id)` is one post and where it landed.
`update_post(id, caption?, title?, accountIds?, scheduledAt?, draft?)` edits a draft (it stays a
draft) or a scheduled post (it keeps its time unless given one). `schedule_draft(id, scheduledAt?)`
sends a draft; `cancel_post(id)` withdraws a scheduled post or deletes a draft, with no undo.
**Before anything that posts or changes a scheduled post:** call `list_publish_accounts`, then
confirm media, caption, target accounts and time with the user in one message.

**Sound effects and music for a video** — free, no credits. `search_sounds(query, group?, max_seconds?)`
finds a hit, a whoosh, a sting in about a thousand short MP3s (tags are what a clip sounds like —
search "boom", not a meme name); `list_sound_groups()` names the sixteen kinds, `list_sounds` browses
one, `get_sound(id)` reads one. For a background bed, `search_music(query?, mood?, use?,
min_seconds?)` over 1,442 royalty-free tracks (`list_music_tags()` lists the moods and uses), or
`recommend_music(brief)` with the video's story, feeling and energy — leave out mixing notes like
"no vocals" or "under the voice", which it would read as the mood — and check `read_as` before
picking. `get_track(id)` reads one. Give the user each pick's title and MP3 url.

## What this connector cannot do

Say so plainly instead of improvising. It does not start a shot-by-shot analysis (that is the
Analyze page in the app, 1 credit, about two minutes), spends credits only on `transcribe` and
`generate_image`, does not connect a
social account (that is a browser sign-in on the platform), cannot receive a file pasted into the
chat (it uses `create_upload` instead), does not render a video,
and does not style subtitles. It cannot put a file on the user's disk: the sound and music tools
answer with each MP3's url, which is the download.

It does not generate video. Video — image-to-video, motion control, lip-sync — is in Pop-Cam's web
app at pop-cam.com/video-gen. Point the user there when they ask for one, and never ask for a
Pop-Cam key in chat.

## Habits

- Quote a save by its title and URL, not its id, unless the user asks for the id.
- When a search returns nothing, say what you searched and offer the tag list from `list_saves`.
- A save's `notes` belong to the user: append, never overwrite, unless told to replace.
- Never reproduce another creator's caption, speech or on-screen text as the user's own copy.
