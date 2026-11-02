# Hookova for Claude

Your short-video reference library, inspiration boards and publishing queue, from the
conversation you're already in — on claude.ai, in Cowork and in Claude Code.

[Hookova](https://www.hookova.com) is where people who make Shorts, Reels and TikToks keep the
clips they want to learn from. Save a video or a page, get a synopsis and takeaways, analyse a
clip shot by shot, pin saves to a board for the video you're making, and schedule the finished
post to your connected accounts. This plugin brings that account into Claude.

## What's in the plugin

- **One connector** — the hosted Hookova MCP server at `https://www.hookova.com/mcp`. Thirty-five
  tools: `auth_status`; `save_link`, `list_saves`, `get_save`,
  `update_save`, `delete_save`, `export_saves`; `transcribe`, `get_transcription`;
  `list_projects`, `get_hook_guide`; `generate_image` (Pop-Cam's image model, 1 credit per image —
  credits provided by Pop-Cam); `make_promo_video`, `list_promo_videos` (a finished promotion video
  from a product brief in about 20 minutes, 10 credits, refunded if it fails); `list_publish_accounts`, `list_publish_groups`,
  `create_publish_group`, `update_publish_group`, `delete_publish_group`, `create_upload`,
  `publish_post`, `list_posts`, `get_post`, `update_post`, `schedule_draft`, `cancel_post` — sort
  your connected accounts into groups, and create, read, edit, schedule and cancel posts to them;
  and the free libraries: `search_sounds`, `list_sounds`, `list_sound_groups`, `get_sound` (about a
  thousand short sound effects) and `search_music`, `list_music`, `list_music_tags`,
  `recommend_music`, `get_track` (1,442 royalty-free tracks), each answering with the MP3's url. Sign-in is OAuth 2.1 in your own browser — Google-linked accounts work, and
  nothing is pasted into the chat. Revoke it any time under **Settings → MCP server** on
  hookova.com.
- **`hookova` skill** — teaches Claude the workflow: check `auth_status` first, read before
  retagging, confirm accounts and timing before anything posts, and say plainly what the
  connector can't do (start an analysis, receive a file pasted into the chat, or render any video but
  a promotion video from a product brief), and say the price before anything that spends credits.
- **`hook-from-project` skill** — turns a board into the opening seconds of *your* video with
  HyperFrames: reads the board as a hook guide, borrows the pattern (timing, shot sizes, camera,
  text placement) from the analysed clips and nothing else, and hands the build to the
  `hyperframes` skills. Needs those skills installed; it never spends a credit.
- **`popcam-images` skill** — makes images with `generate_image`: the three modes (prompt,
  reference + prompt, preset), the same character in every shot of a video, prompts that work, and
  the price said up front. 1 credit per image, credits provided by Pop-Cam.

## Install

- **claude.ai / Claude Desktop:** Customize → Plugins → Discover → Hookova, then connect the
  Hookova connector on the plugin's **Connectors** tab and sign in.
- **Claude Code:** `/plugin directory` → Hookova, or from this repository:

  ```
  claude plugin marketplace add petagit/hookova-plugin
  claude plugin install hookova@hookova-plugin
  ```

  then `/mcp` → hookova → **Authenticate**.

Try: *"What did I save about sourdough?"* · *"Save this link to Hookova and tag it Recipes."* ·
*"Transcribe this Short."* · *"Get the hook guide for my cats board."* · *"What's in my publishing
queue this week?"*

## What it sends and where

The plugin talks to `www.hookova.com` and nowhere else. Every tool call carries your OAuth
access token to `https://www.hookova.com/mcp`; the token is a one-hour credential minted by
hookova.com and refreshed by your Claude client. The skills are plain text and run no code.
Saves, boards, transcriptions and posts are your Hookova account's data, under the Hookova
[privacy policy](https://www.hookova.com/legal/privacy) and
[terms](https://www.hookova.com/legal/terms).

## Not in the plugin

Analysing a clip shot by shot and connecting a social account happen in the Hookova app.
Claude cannot hand a tool a file pasted into the chat, so `create_upload` gives you a hookova.com
page to pick the file from your own device (or, in Claude Code, a URL to PUT it to); `publish_post`
then posts it from there. A public https link to the file works too (up to 100 MB a post). The
`connect` / `sign_in` tools exist only in the [local stdio server](https://www.hookova.com/guides/mcp-server),
which Claude Code users can run alongside this plugin. The free
[sound-effects](https://www.hookova.com/sound-effects) and [music](https://www.hookova.com/music)
libraries are separate MCP servers.

## Support

Documentation: <https://www.hookova.com/guides/mcp-server> · Developers:
<https://www.hookova.com/guides/developers> · <https://www.hookova.com/support> ·
hookovasupport@gmail.com

Source of truth for this plugin is `apps/claude-plugin/` in Hookova's monorepo; this repository
is the published copy. MIT licensed.
