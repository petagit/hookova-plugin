---
name: hook-from-project
description: Turn a Hookova project — a board of saved clips, some of them analysed shot by shot — into the opening seconds of the user's own video, built with HyperFrames. Use when the user names a Hookova board and wants a hook, an opening, or a first cut of a Short made from it ("make the hook from my cats board", "/hookova:hook-from-project <board>"). Needs the Hookova connector (list_projects, get_hook_guide) and the hyperframes skills.
---

# hook-from-project

A Hookova project is a board: a name, a brief in the user's words, and the saves pinned to it.
Some of those saves are short videos Hookova has taken apart shot by shot. This skill turns
the board into a **hook guide** — `HOOK-GUIDE.md` plus reference stills — and hands it to
HyperFrames as the material for the *user's own* opening seconds.

Two rules come before the steps, and they are the whole point:

1. **Pattern and subject are separate layers.** The analysed clips are other people's videos.
   What is borrowed from them is the *skeleton* of the opening — timing, shot sizes, camera
   moves, where text lands, which hook technique each shot performs. What the video is *about*
   comes only from the board's brief and material. **Nothing from the source clips is reused:
   not the footage, not the speech, not the on-screen text.** Write fresh lines that do the same
   job the pattern's lines do at that moment. The reference stills are for framing and
   composition only.
2. **HyperFrames owns its plan files.** Its `BRIEF.md` and `STORYBOARD.md` are written by its own
   workflow, after `hyperframes init`. This skill never pre-writes them; the guide is source
   material the intent layer reads as a formed request.

## Steps

1. **Signed in?** Call `auth_status`. If not, stop and say so — the user connects the Hookova
   connector in their client (the plugin's **Connectors** tab on claude.ai, or `/mcp` → hookova
   → Authenticate in Claude Code); no tool signs in for them.
2. **Find the board.** Call `list_projects` and match the argument to a board by name
   (case-insensitive, prefix or substring). If nothing matches, list the names and ask. If two
   match, ask which — that is the one question worth asking here.
3. **Is there anything to build on?** The guide's frontmatter says `patterns: N`. With
   `patterns: 0` — no pinned clip has a finished analysis — tell the user plainly and offer the
   choice: analyse one of the pinned clips first in the Hookova web app (1 credit, about two
   minutes, then re-run this), or go on with the subject alone, which HyperFrames can still build
   as a typographic, motion-led hook without a pattern. Do not analyse anything yourself; this
   skill never spends a credit.
4. **`init` first.** `init` refuses a non-empty directory, so the project is created before
   anything is written into it. Use a fresh directory named after the board:

       npx hyperframes init <dir> --example blank --resolution portrait --non-interactive

   `--example blank` is the empty starter (non-interactive init needs an example, a video or an
   audio file named), `--resolution portrait` is 1080x1920, and `--non-interactive` keeps the
   scaffold from prompting — the interview is HyperFrames' own, later, not init's.
5. **Write the guide into the project.** Call `get_hook_guide` with the board's id, `full: true`,
   and `stills: 12`. The hosted connector returns the guide as markdown plus up to twelve
   reference stills as images, in the order the guide cites them. Write the markdown to
   `<dir>/HOOK-GUIDE.md` yourself and read it. The stills are for your eyes — composition and
   framing — and cannot be saved to disk from this connector; if the local Hookova MCP server is
   also installed, prefer *its* `get_hook_guide` with `dir: <dir>`, which writes the file and
   downloads every still into `<dir>/refs/`.
6. **Several patterns?** The guide letters them A, B, C… in board order. Recommend the one whose
   hook length and tone sit closest to the brief, name a runner-up, and offer a merge of two.
   One question, then on. Record the choice.
7. **Hand the build to HyperFrames.** Invoke `/hyperframes` with the guide as the formed input.
   Route: `general-video` as a companion run by default; `motion-graphics` when the hook is
   text-and-motion with no narration and under ten seconds. Pre-answer what the guide settles —
   `message` from the brief's first sentence, `aspect` from the guide's `format` (1080x1920 unless
   the brief says otherwise), `length` from `hook_length` — and list `HOOK-GUIDE.md` (and `refs/`
   when it exists) under `## Assets`. Every storyboard frame names the pattern row it borrows from
   (`borrows: A·2`), so the borrowing is legible and reviewable. Restate rule 1 to the
   workflow in one line.
8. **Leave.** Review, sketch passes and render belong to HyperFrames' own loop
   (`/hyperframes-core` → `review-loop.md`). This skill does not own the deliverable, and it
   does not come back after the hand-off.

## What to say when it is done

Name the board, the pattern chosen and why, where the project is, and the one thing the user
still has to supply if the build needs it: their own material — footage, product shots, a logo,
data. A hook built with none is typographic and motion-led, and that is a legitimate result.

## Never

- Reuse a caption, a spoken line or on-screen text from a source clip, even reworded lightly.
- Pre-write `BRIEF.md` or `STORYBOARD.md`, or call `hyperframes init` on a non-empty directory.
- Start an analysis, spend a credit, or pin and unpin saves — the board is the user's to arrange.
- Ask the user for a token or paste one anywhere; the connector signs in through OAuth.
