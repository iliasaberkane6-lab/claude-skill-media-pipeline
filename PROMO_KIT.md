# Promo Kit — media-pipeline

Repo: https://github.com/iliasaberkane6-lab/claude-skill-media-pipeline

---

## 1. Reddit — r/ClaudeAI

**Title:** I built a Claude Code skill that produces entire faceless YouTube videos from a script

**Body:**

I got tired of the manual pipeline for my history YouTube channel — generate images, synthesize voice, stitch in ffmpeg, make shorts, upload. So I packaged the whole thing as a Claude Code skill/plugin.

**What it does (one command):**
- Generates images (fal/flux or Pollinations)
- Synthesizes voiceover (ElevenLabs, edge-tts, or Kokoro)
- Assembles 1080p video with Ken Burns + ffmpeg
- Adds music bed + end card + LUFS mastering
- Auto-generates 9:16 shorts with burned captions
- Uploads to YouTube via Data API v3
- Tracks cost (~$0.75 per 6-min video)

**Install:**
```
/plugin marketplace add iliasaberkane6-lab/claude-skill-media-pipeline
/plugin install media-pipeline@media-pipeline
```

Then just tell Claude: "render my video"

It's a single SKILL.md file (~480 lines) that Claude Code follows as a step-by-step pipeline. No extra dependencies beyond Python + ffmpeg.

Full repo with the skill, prerequisites, and cost breakdown: https://github.com/iliasaberkane6-lab/claude-skill-media-pipeline

Feedback welcome, especially on the visual style prompts (flux keeps adding noses lol).

---

## 2. Reddit — r/facelessyoutube

**Title:** Open-source Claude Code skill for full faceless video pipeline (images → voice → ffmpeg → shorts → upload)

**Body:**

Same content as above, but emphasize:
- Cost: ~$0.75/video vs $20+/video on other tools
- No subscription, no SaaS
- Works with your own fal.ai key
- MIT licensed

---

## 3. Twitter/X

**Tweet 1:**

I built a Claude Code skill that turns a script into a finished faceless YouTube video — images, voiceover, ffmpeg assembly, shorts, upload — in one command.

~$0.75 per video. No SaaS. Open source.

Install:
```
/plugin marketplace add iliasaberkane6-lab/claude-skill-media-pipeline
/plugin install media-pipeline@media-pipeline
```

https://github.com/iliasaberkane6-lab/claude-skill-media-pipeline

**Tweet 2 (reply):**

Pipeline:
→ Images: fal/flux or Pollinations
→ Voice: ElevenLabs / edge-tts / Kokoro
→ Assembly: ffmpeg Ken Burns
→ Polish: music bed + end card + LUFS
→ Shorts: 9:16 with ASS captions
→ Upload: YouTube Data API v3

All from one SKILL.md file. Claude Code follows it as a recipe.

**Tweet 3 (reply):**

Cost breakdown for a 6-min video (30 scenes):
- Images: ~$0.60 (fal/flux)
- Voice: ~$0.15 (ElevenLabs via fal)
- Music: free (YouTube Audio Library)
- Upload: free

Total: ~$0.75

vs. $20-50/video on most SaaS tools.

---

## 4. Hacker News

**Title:** Show HN: I built a Claude Code skill for full faceless YouTube video production

**Body:**

I run a history YouTube channel and was spending hours on the manual pipeline: generate images, synthesize voiceover, stitch everything in ffmpeg, make shorts, upload.

So I packaged the entire workflow as a Claude Code skill — a single 480-line SKILL.md file that Claude Code follows as a step-by-step pipeline.

The skill covers:
- Image generation (fal/flux, Pollinations)
- TTS voiceover (ElevenLabs, edge-tts, Kokoro ONNX)
- ffmpeg assembly with Ken Burns effect
- Music bed + end card + LUFS mastering
- 9:16 shorts with burned ASS captions
- YouTube Data API v3 upload
- Budget tracking (~$0.75 per 6-min video)

It's a plugin, so install is two commands:
```
/plugin marketplace add iliasaberkane6-lab/claude-skill-media-pipeline
/plugin install media-pipeline@media-pipeline
```

Then just tell Claude "render my video" and it follows the skill.

MIT licensed, no SaaS, works with your own API keys.

Repo: https://github.com/iliasaberkane6-lab/claude-skill-media-pipeline

---

## 5. Discord servers to post in

- **Claude Code Discord** (if one exists) — #showcase or #plugins channel
- **Anthropic Discord** — #community-projects
- **Faceless YouTube Discord servers** — search on disboard.org for "faceless youtube"
- **AI video creator Discords**

---

## 6. Other platforms

- **Product Hunt**: Submit as a free open-source tool
- **Dev.to / Medium**: Write a short tutorial "How I automate my faceless YouTube channel with Claude Code"
- **YouTube**: Make a faceless video ABOUT the tool (dogfooding)
- **Awesome lists**: Submit to awesome-claude-code, awesome-ai-video, awesome-youtube lists on GitHub

---

## 7. GitHub discovery tips

- Star other repos in the claude-code / ai-video space
- Open issues on related repos (politely) mentioning compatibility
- Submit to: https://github.com/hesreallyhim/awesome-claude-code

---

## Timing

- Post on Tuesday-Thursday morning (US time) for max visibility
- Cross-post Reddit + Twitter within 1 hour
- HN: post morning US time (8-10am PT)
