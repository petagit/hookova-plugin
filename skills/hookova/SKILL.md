---
name: hookova
description: Work with the user's Hookova account through the Hookova connector — save a TikTok, Shorts, Reels or web link, search and read back what they saved, retag or annotate a save, transcribe a video, read an inspiration board as a hook guide, and list, schedule or cancel posts on their connected social accounts. Use whenever the user mentions Hookova, their saves, their boards, "my references", or asks to schedule a post to YouTube, TikTok, Instagram, Facebook or X.
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

**Publishing — create, read, update, delete.** `list_publish_accounts()` is the connected social
accounts (id, platform, handle). `publish_post(caption, mediaUrls, accountIds?, scheduledAt?,
draft?, …)` creates a post: one video or one to ten photos, given as **public https links to the
files themselves** (up to 100 MB a post, all files together), posted now, at an ISO time *with an offset*
(`2026-09-18T11:00:00-07:00`), or saved as a draft. You cannot pass an image the user pasted into
this chat to a tool — say so, and ask for a link to the file or point them to hookova.com/publish;
never claim it was posted. `list_posts(status?)` is the queue and the calendar — `draft`,
`scheduled`, `processing`, `processed`; `get_post(id)` is one post and where it landed.
`update_post(id, caption?, title?, accountIds?, scheduledAt?, draft?)` edits a draft (it stays a
draft) or a scheduled post (it keeps its time unless given one). `schedule_draft(id, scheduledAt?)`
sends a draft; `cancel_post(id)` withdraws a scheduled post or deletes a draft, with no undo.
**Before anything that posts or changes a scheduled post:** call `list_publish_accounts`, then
confirm media, caption, target accounts and time with the user in one message.

## What this connector cannot do

Say so plainly instead of improvising. It does not start a shot-by-shot analysis (that is the
Analyze page in the app, 1 credit, about two minutes), does not spend credits, does not connect a
social account (that is a browser sign-in on the platform), cannot receive a file pasted into the
chat (only a link to one), does not render a video,
and does not style subtitles. The sound-effect and music libraries are separate MCP servers
documented at hookova.com/sound-effects and hookova.com/music.

## Habits

- Quote a save by its title and URL, not its id, unless the user asks for the id.
- When a search returns nothing, say what you searched and offer the tag list from `list_saves`.
- A save's `notes` belong to the user: append, never overwrite, unless told to replace.
- Never reproduce another creator's caption, speech or on-screen text as the user's own copy.
