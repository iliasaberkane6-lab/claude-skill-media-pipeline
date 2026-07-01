# media-pipeline

A Claude Code skill for producing faceless YouTube videos, shorts, and podcasts with AI-generated images, TTS voiceover, and ffmpeg assembly.

## What it does

- Images: Generate via fal/flux or Pollinations
- Voice: ElevenLabs, edge-tts, or Kokoro ONNX
- Assembly: ffmpeg Ken Burns + concat demuxer
- Polish: Music bed + end card + LUFS mastering
- Shorts: 9:16 vertical with burned ASS captions
- Upload: YouTube Data API v3 automation
- Budget: Built-in cost tracking

## Prerequisites

- Python 3.10+
- ffmpeg (bundled with imageio-ffmpeg or system-installed)
- A fal.ai API key (get one at https://fal.ai) — stored at `~/.config/fal/key`
- Optional: edge-tts (`pip install edge-tts`) for free voiceover
- Optional: Kokoro ONNX (`pip install kokoro-onnx`) for local voiceover
- Optional: google-api-python-client for YouTube upload
- A TTF font file (e.g. Arial Bold) for thumbnails and end cards

## Installation

Clone into your skills directory:

    git clone https://github.com/iliasaberkane/claude-skill-media-pipeline.git ~/.claude/skills/media-pipeline

Or copy the SKILL.md into an existing project:

    mkdir -p .claude/skills/media-pipeline
    cp SKILL.md .claude/skills/media-pipeline/

## Quick start

1. Create a topic_data.py with your script (list of tuples: label, narration, image prompts)
2. Ask Claude: render my video
3. Claude generates images, synthesizes voice, assembles the video, adds music, makes shorts

## Cost

~$0.75 per 6-minute video (30 images + voiceover). Music and upload are free.

## License

MIT
