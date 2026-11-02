---
name: popcam-images
description: Make images with Hookova's generate_image tool — Pop-Cam's image model, 1 Hookova credit per image, credits provided by Pop-Cam. Use when the user asks for an image, a set of stills for a video (a character in every scene, a mascot, a product shot, a thumbnail, a storyboard), or a photo restyled ("make my logo 3D", "turn this photo into a cartoon", "the same bird in five scenes"). Needs the Hookova connector (generate_image).
---

# popcam-images

Hookova and Pop-Cam (pop-cam.com) are one maker's bundle. `generate_image` makes an image with
Pop-Cam's model (Nano Banana 2.1) and pays for it from the user's **Hookova credits: 1 per image,
charged only when an image comes back**. Credits are provided by Pop-Cam. Nobody needs a Pop-Cam
account or key, so never ask for one.

## Before you spend

- Say the price in the same message as the plan: "10 images, 10 credits." Wait for a yes before you
  make more than one. `auth_status` shows the balance.
- Plan every still on paper first (what each one shows, which reference it uses). Retries cost the
  same as first tries, so a list you already agreed on is the cheap path.
- A failed or refused image costs nothing. If the model declines a prompt, reword it. Don't retry
  the same words.

## The three modes

| Mode | Pass | Use it for |
|---|---|---|
| Text → image | `prompt` | A new scene, a logo, a product shot |
| Image → image | `reference_image_url` + `prompt` | Restyling a photo; keeping a character, product or face identical |
| Preset | `preset_id` (+ optional reference) | A Pop-Cam house style |

**Shape follows the reference.** A 9:16 reference gives a 9:16 image (768×1376), which is right for
Reels, Shorts and TikTok. With no reference the image is square. To get a vertical frame of a
character, first make (or upload) the character on a 9:16 canvas and use that as the reference.

## Same character in every shot

1. Make the **anchor** first: the character large in frame, in the costume and style of the whole
   set. Check it before going on.
2. Pass the anchor's `image_url` as `reference_image_url` for every later still. In the prompt, say
   "the same character as in the reference image", then repeat its key features word for word
   every time (e.g. "round pink bird, big glossy purple eyes, small orange beak, tiny pink instant
   camera on a strap").
3. For a scene with two recurring characters, make each one's anchor first. Then use a reference
   that shows both: a still where they already appear together, or a side-by-side sheet.
4. Use one style sentence and end every prompt with it, e.g. "Cute 3D animated storybook
   illustration, soft clay-render look, pastel colors, warm cinematic lighting. Vertical 9:16
   full-bleed frame. No text, no letters, no borders, no watermark."
5. `image_url` lasts one hour. Download each still or pass it on before then.

## Prompts that work

- Describe what is in the frame: who, where, their pose and expression, and the camera ("close-up
  at the bird's eye level", "wide shot, character in the lower third").
- Keep words out of the picture. Titles, captions and prices belong in the edit, where they can be
  fixed. Ask for text in the image only when it *is* the product (a logo, a sign).
- Never name a studio's style or characters ("Pixar style", a Disney princess). Describe the look
  itself instead. Restyle only photos the user owns or has the rights to.
- One extreme close-up gives a video two or three shots (eyes, mouth, a reaction) when you frame it
  differently in the edit.

## Turning stills into a video

The images are the material, not the film. Hookova's music and sound-effect libraries (separate MCP
servers: hookova.com/music, hookova.com/sound-effects) and HyperFrames (an HTML-to-video renderer)
make the edit. When it's finished, `publish_post` takes it to the user's accounts after they confirm
the caption, the accounts and the time. Video generation itself (image-to-video, lip-sync) is in
Pop-Cam's web app at pop-cam.com/video-gen.
