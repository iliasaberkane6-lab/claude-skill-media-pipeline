---
name: media-pipeline
description: >
  Produce faceless videos, shorts, and podcasts with AI-generated images, TTS voiceover,
  and ffmpeg assembly. Use when the user wants to create YouTube videos, automate video
  production, generate voiceover, add music beds, make vertical shorts, generate thumbnails,
  or build a content pipeline. Covers image generation (fal/flux, Pollinations), voice synthesis
  (ElevenLabs, edge-tts, Kokoro), audio mastering (LUFS/compression/limiting), video assembly
  (Ken Burns, concat demuxer, burned captions), 9:16 shorts, and YouTube upload automation.
when_to_use: >
  video, TTS, voiceover, ffmpeg, shorts, thumbnail, YouTube, faceless channel, content pipeline,
  image generation, audio mastering, Ken Burns, captions, render, media production, podcast
allowed-tools: Bash Read Write Edit Glob Grep
---

# Media Pipeline

Produce complete faceless videos — images, voiceover, music, assembly, shorts, upload — in one workflow.

## When to use this skill

Load this skill when the user wants to:
- Create a YouTube video from a script or topic
- Generate AI images for video scenes
- Synthesize voiceover (TTS)
- Assemble video clips with ffmpeg
- Add music beds or end cards
- Make 9:16 vertical shorts with burned captions
- Upload to YouTube programmatically
- Build an automated content pipeline

Do not load for: video editing of existing footage, live-action recording, streaming setup.

## Pipeline overview

```
Script -> Images -> Voice -> Assembly -> Polish -> Shorts -> Upload
```

Each phase is independent and cacheable. Re-running only re-executes the phase that changed.

---

## Phase 1: Script data

Store the script as a Python file with a list of tuples:

```python
S = [
    ("", "Hook narration text here.", ["image prompt 1", "image prompt 2"]),
    ("LEVEL 1   RANK NAME", "Narration for this rank.", ["prompt 1", "prompt 2", "prompt 3"]),
    ("LEVEL 2   RANK NAME", "Narration for this rank.", ["prompt 1", "prompt 2"]),
    ("CLIMAX", "Final narration.", ["prompt 1"]),
]
```

- First element: section label (empty string for intro/outro). Used as YouTube chapter.
- Second element: narration text. Split into sentences for per-clip TTS.
- Third element: image prompts. Each prompt generates one image shown for ~3-5 seconds.

### Scriptwriting rules

- Second person: "You're six years old when..."
- 750-1100 words for a 5-7 minute video.
- Each section reveals a new ability, danger, or rank.
- End sections with a micro-cliffhanger.
- Image prompts describe one clear visual scene each.

---

## Phase 2: Image generation

### Style prompt prefixes

Choose one style and prepend it to every image prompt:

**Minimalist white background (simplest, avoids face issues):**
```
Minimalist 2D cartoon illustration on a clean solid off-white background.
One central subject per image. Simple round-headed stick figures with tiny
black dot eyes, small simple mouth, no nose, thin stick limbs, simple clothing.
Flat solid colors, bold black outlines, no gradients, no photorealism.
No text, watermark, signature. Scene:
```

**Rich atmospheric background (detailed, Mr-Sticky style):**
```
2D cartoon history explainer with detailed hand-drawn background.
Main character: round off-white head, simple emoji face with two black dot
eyes and one mouth line, no nose, thin stick limbs. Background crowd as small
silhouettes or hooded figures seen from behind. Bold black outlines, flat cel
shading, muted earthy palette, torches, stone, wood, depth, atmosphere.
No text, watermark, signature. Scene:
```

**Silhouette style (zero face risk):**
```
Minimalist 2D cartoon on off-white background. Characters appear only as
black silhouettes or shown from behind — no visible face, no eyes, no nose.
Focus on objects, tools, weapons, action. Flat colors, bold outlines.
No text, watermark, signature. Scene:
```

### Provider: fal.ai (flux)

```python
import json, urllib.request, os

FAL_KEY = open(os.path.expanduser("~/.config/fal/key")).read().strip()  # adjust to your fal API key path

def generate_image(prompt, out_path, style_prefix="", model="fal-ai/flux/schnell", w=1920, h=1080):
    full = style_prefix + prompt
    body = {"prompt": full, "image_size": {"width": w, "height": h}}
    req = urllib.request.Request(
        f"https://fal.run/{model}",
        data=json.dumps(body).encode(),
        headers={"Authorization": f"Key {FAL_KEY}", "Content-Type": "application/json"}
    )
    resp = json.loads(urllib.request.urlopen(req, timeout=120).read())
    url = resp["images"][0]["url"]
    data = urllib.request.urlopen(url, timeout=90).read()
    open(out_path, "wb").write(data)
    return len(data)
```

Models:
- `fal-ai/flux/schnell` — fast, ~$0.006/img. Default.
- `fal-ai/flux/dev` — better prompt adherence, ~$0.03/img. Use if schnell ignores style rules.
- `fal-ai/flux/dev` supports `negative_prompt` field — suppress unwanted features (blush, nose, text).

### Provider: Pollinations (free, no key)

```python
def generate_image_pollinations(prompt, out_path, w=1920, h=1080):
    import urllib.parse
    url = ("https://image.pollinations.ai/prompt/" + urllib.parse.quote(prompt)
           + f"?width={w}&height={h}&nologo=true&model=flux")
    data = urllib.request.urlopen(url, timeout=90).read()
    if len(data) > 10000:
        open(out_path, "wb").write(data)
        return len(data)
    return 0
```

### QA: Always test one image first

Before generating all 30+ images, generate ONE test image and have the user verify the style. This prevents burning budget on a wrong prompt. Only proceed after visual confirmation.

---

## Phase 3: Voice synthesis

### ElevenLabs via fal (recommended)

```python
# FAL_KEY must be defined (see Phase 2)
TTS_ENDPOINT = "https://fal.run/fal-ai/elevenlabs/tts/multilingual-v2"

def synth_voice(text, out_path, voice="Brian"):
    body = {"text": text, "voice": voice}
    req = urllib.request.Request(TTS_ENDPOINT,
        data=json.dumps(body).encode(),
        headers={"Authorization": f"Key {FAL_KEY}", "Content-Type": "application/json"})
    resp = json.loads(urllib.request.urlopen(req, timeout=60).read())
    audio = resp.get("audio") or resp.get("audio_url") or {}
    url = audio.get("url") if isinstance(audio, dict) else audio
    if url:
        data = urllib.request.urlopen(url, timeout=60).read()
        open(out_path, "wb").write(data)
```

Voices: `Brian` (documentary), `George` (wise), `Adam` (raspy), `Daniel` (British).
Cost: ~$0.10 per 1,000 characters. ~$0.55 for a 6-minute video.

### edge-tts (free fallback)

```bash
edge-tts --voice de-DE-ConradNeural --rate=-5% --text "TEXT" --write-media out.mp3
```

Voices: `de-DE-ConradNeural` (deep German), `en-US-GuyNeural` (deep English).
Quality: robotic but free. Use for test renders or when budget is exhausted.

### Kokoro ONNX (local, free)

```python
from kokoro_onnx import Kokoro
k = Kokoro("kokoro-v1.0.onnx", "voices-v1.0.bin")
audio, sr = k.create(text, voice="am_onyx", speed=1.0, lang="en-us")
```

Quality: good, but English-only voices. No German narrator voice available.

### Voice caching

Cache by text hash to avoid re-paying on re-renders:

```python
import hashlib
def cache_path(text, work_dir):
    h = hashlib.md5(text.encode()).hexdigest()[:12]
    return f"{work_dir}/vo_{h}.mp3"
```

Always check cache before synthesizing. Skip if file exists and is > 1500 bytes.

---

## Phase 4: Video assembly

### Per-clip rendering

Each sentence = one image + one voice clip = one video clip:

```bash
ffmpeg -y -loop 1 -i image.jpg -i voice.mp3 \
  -vf "zoompan=z='min(zoom+0.0015,1.12)':d=150:x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)',format=yuv420p" \
  -c:v libx264 -preset veryfast -crf 22 -pix_fmt yuv420p \
  -c:a aac -b:a 160k -shortest clip.mp4
```

### Concatenation

Use concat demuxer (reliable, preserves quality, fast):

```bash
# list.txt:
# file 'clip_000.mp4'
# file 'clip_001.mp4'
# ...
ffmpeg -y -f concat -safe 0 -i list.txt -c copy assembly.mp4
```

Do NOT use the concat filter (`[v0][v1]concat=...`) -- it re-encodes and can double duration on some ffmpeg versions.

### Chapter generation

```python
def chapters(sections, durations):
    t = 0; out = ["0:00 Intro"]
    for i, (title, _, _) in enumerate(sections):
        if title:
            m, s = divmod(int(t), 60)
            out.append(f"{m}:{s:02d} {title}")
        t += durations[i]
    return "\n".join(out)
```

---

## Phase 5: Audio mastering and polish

### Mastering chain (ffmpeg)

Apply to the final mixed audio (voice + music bed):

```
highpass=f=75,
acompressor=threshold=-18dB:ratio=3:attack=20:release=220,
loudnorm=I=-16:TP=-1.5:LRA=11,
alimiter=limit=0.95,
aformat=sample_rates=44100:channel_layouts=mono
```

Targets: -16 LUFS integrated, -1.5 dBTP true peak, LRA 11.
YouTube recommends -14 LUFS but -16 gives headroom for music bed.

### Music bed

Download royalty-free tracks from YouTube Audio Library. Loop to video duration:

```bash
ffmpeg -y -stream_loop -1 -i music.m4a -t $DURATION \  # DURATION must be integer seconds
  -af "afade=t=in:d=3,afade=t=out:st=$((DURATION-3)):d=3,volume=0.25" \
  -c:a aac -b:a 128k bed.m4a
```

Mix with voice:

```bash
ffmpeg -y -i video.mp4 -i bed.m4a -filter_complex \
  "[0:a]aformat=sample_fmts=fltp:sample_rates=44100[va]; \
   [1:a]aformat=sample_fmts=fltp:sample_rates=44100[ma]; \
   [va][ma]amix=inputs=2:normalize=0:duration=first,alimiter=limit=0.92[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 160k final.mp4
```

### End card (6 seconds)

```bash
ffmpeg -y -f lavfi -i "color=c=0x1a1a1a:s=1920x1080:r=30:d=6" \
  -vf "drawtext=fontfile=font.ttf:text='SUBSCRIBE':fontsize=120:fontcolor=white:x=(w-text_w)/2:y=280, \
       drawtext=fontfile=font.ttf:text='for more':fontsize=54:fontcolor=0xcccccc:x=(w-text_w)/2:y=440, \
       drawtext=fontfile=font.ttf:text='WATCH NEXT':fontsize=72:fontcolor=0xFFD23F:x=(w-text_w)/2:y=640, \
       format=yuv420p" \
  -c:v libx264 -preset veryfast -crf 23 -pix_fmt yuv420p -shortest endcard.mp4
```

Concat end card to final video.

### Font file

ffmpeg `drawtext` and PIL both need a TTF font. Download one (e.g. Arial Bold) and place it as `font.ttf` in the working directory, or reference a system font like `/Library/Fonts/Arial Bold.ttf` (macOS) or `/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf` (Linux).

---

## Phase 6: Shorts (9:16 vertical)

### Selection

Pick 3 high-impact sections: the hook, the most surprising rank, and the climax.

### 9:16 conversion with blurred background

```bash
ffmpeg -y -i long_video.mp4 -vf "
  split=2[bg][fg];
  [bg]scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,boxblur=40:8[b];
  [fg]scale=1080:-2[f];
  [b][f]overlay=(W-w)/2:(H-h)/2-200,format=yuv420p
" -c:v libx264 -crf 20 -b:v 10M -c:a aac -b:a 160k short.mp4  # 10M for max quality; 5M is sufficient for most shorts
```

### Burned captions (ASS subtitles)

Use ASS format for word-by-word pop-in animation:

```ini
[Script Info]
ScriptType: v4.00+
PlayResX: 1080
PlayResY: 1920

[V4+ Styles]
Format: Name,Fontname,Fontsize,PrimaryColour,OutlineColour,BackColour,Bold,Italic,Underline,StrikeOut,ScaleX,ScaleY,Spacing,Angle,BorderStyle,Outline,Shadow,Alignment,MarginL,MarginR,MarginV,Encoding
Style: Cap,Arial,86,&H00FFFFFF,&H00141414,&H00000000,-1,0,0,0,100,100,0,0,1,8,3,2,60,60,520,1
```

Split narration into 2-3 word chunks, time each to its audio segment, add `{\fad(40,30)}` for pop-in.

Apply:
```bash
ffmpeg -y -i short.mp4 -vf "subtitles=captions.ass" -c:v libx264 -crf 20 -c:a copy final_short.mp4
```

---

## Phase 7: Thumbnail

Generate a hero image in the same art style, then overlay bold text:

```python
from PIL import Image, ImageDraw, ImageFont

def make_thumbnail(bg_path, line1, line2, out_path):
    bg = Image.open(bg_path).resize((1280, 720))
    draw = ImageDraw.Draw(bg)
    font_big = ImageFont.truetype("font.ttf", 72)
    font_huge = ImageFont.truetype("font.ttf", 120)
    for dx, dy in [(2,2),(-2,2),(2,-2),(-2,-2)]:
        draw.text((640+dx, 200+dy), line1, font=font_big, fill="black", anchor="mm")
        draw.text((640+dx, 400+dy), line2, font=font_huge, fill="black", anchor="mm")
    draw.text((640, 200), line1, font=font_big, fill="white", anchor="mm")
    draw.text((640, 400), line2, font=font_huge, fill="#FFD23F", anchor="mm")
    bg.save(out_path)
```

Title patterns: "JEDER RANG EINES NINJA", "WHY IT SUCKS TO BE X", "YOUR LIFE AS [TOPIC]".

---

## Phase 8: YouTube upload

Use YouTube Data API v3 with OAuth 2.0 refresh token.

```python
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload

def upload_video(video_path, title, description, tags, thumbnail_path=None):
    creds = Credentials.from_authorized_user_file("token.json")
    yt = build("youtube", "v3", credentials=creds)
    body = {
        "snippet": {
            "title": title,
            "description": description,
            "tags": tags,
            "categoryId": "27",
        },
        "status": {"privacyStatus": "public", "selfDeclaredMadeForKids": False}
    }
    media = MediaFileUpload(video_path, resumable=True)
    request = yt.videos().insert(part="snippet,status", body=body, media_body=media)
    response = None
    while response is None:
        response = request.next_chunk()
    video_id = response["id"]
    if thumbnail_path:
        yt.thumbnails().set(videoId=video_id, media_body=MediaFileUpload(thumbnail_path)).execute()
    return video_id
```

OAuth scopes: `youtube.upload`, `youtube.force-ssl` (for privacy changes).
Token refresh: store refresh_token in token.json, auto-refreshes via google-auth.

---

## Budget tracking

Track all AI spend in a JSON file:

```python
import json, time, os

SPEND_FILE = "fal_spend.json"
MONTHLY_BUDGET = 21.0

def get_budget():
    month = time.strftime("%Y-%m")
    try: d = json.load(open(SPEND_FILE))
    except Exception: d = {}
    if d.get("month") != month: d = {"month": month, "spent": 0.0}
    return d

def charge(amount):
    d = get_budget()
    d["spent"] = round(d.get("spent", 0.0) + amount, 4)
    with open(SPEND_FILE, "w") as f:
        json.dump(d, f)
    return d["spent"]

def budget_remaining():
    d = get_budget()
    return MONTHLY_BUDGET - d.get("spent", 0.0)
```

Always check `budget_remaining()` before any paid API call. Fall back to free providers (edge-tts, Pollinations) when budget is exhausted.

### Cost per video estimate

| Item | Cost |
|---|---|
| 30 images (flux/schnell) | ~$0.18 |
| Voiceover (ElevenLabs, 6 min) | ~$0.55 |
| Music | $0 (YouTube Audio Library) |
| Upload | $0 |
| **Total** | **~$0.75** |

---

## Common pitfalls

| Problem | Fix |
|---|---|
| flux generates noses/blush on faces | Use silhouette style or flux/dev with negative_prompt |
| edge-tts sounds robotic | Switch to ElevenLabs via fal, or Kokoro for English |
| ffmpeg concat doubles duration | Use concat demuxer (-f concat -safe 0), not concat filter |
| Video has no audio after assembly | Ensure each clip has -shortest flag and voice file is valid |
| YouTube upload fails | OAuth token expired -- refresh via Google Cloud Console |
| Music bed too loud | Lower volume filter to 0.15-0.25 relative to voice |
| Shorts rejected as not Shorts | Ensure 9:16 aspect ratio and duration < 60s |
| zoompan zooms too fast | Lower zoom increment to 0.0015 and duration to 150 frames |

---

## Quick start

```bash
# 1. Create topic_data.py with script (S = [...])
# 2. Render:
python render_video.py topic    # images + voice + assembly
# 3. Polish:
python polish.py topic_HD.mp4 topic_FINAL.mp4
# 4. Shorts:
python make_short.py topic 0 "HOOK"
python make_short.py topic 4 "TWIST"
python make_short.py topic 10 "CLIMAX"
# 5. Upload:
python upload.py topic
```

Always test one image first before committing to a full render.
