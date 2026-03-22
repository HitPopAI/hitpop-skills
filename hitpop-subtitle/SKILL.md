---
name: hitpop-subtitle
description: "Auto-generate subtitles/captions for videos using OpenAI Whisper. Transcribe audio to SRT/VTT, then burn subtitles into video with FFmpeg. Supports 50+ languages with word-level timestamps."
version: 0.1.0
metadata:
  openclaw:
    emoji: "💬"
    requires:
      bins:
        - ffmpeg
        - python3
---

# Hitpop Subtitle — Auto Subtitles with Whisper

Generate and burn subtitles into videos automatically.

## Setup

```bash
pip install openai-whisper
# or for faster inference:
pip install faster-whisper
```

## Generate Subtitles

### Using Whisper CLI

```bash
# Extract audio from video
ffmpeg -i video.mp4 -vn -acodec pcm_s16le -ar 16000 audio.wav

# Transcribe to SRT
whisper audio.wav --model base --output_format srt --language en
# Output: audio.srt
```

### Using faster-whisper (Python)

```python
from faster_whisper import WhisperModel
model = WhisperModel("base", compute_type="int8")
segments, info = model.transcribe("audio.wav")

with open("subs.srt", "w") as f:
    for i, seg in enumerate(segments, 1):
        start = f"{int(seg.start//3600):02d}:{int(seg.start%3600//60):02d}:{seg.start%60:06.3f}".replace(".", ",")
        end = f"{int(seg.end//3600):02d}:{int(seg.end%3600//60):02d}:{seg.end%60:06.3f}".replace(".", ",")
        f.write(f"{i}\n{start} --> {end}\n{seg.text.strip()}\n\n")
```

### Using OpenAI API (cloud, no GPU needed)

```bash
curl -s https://api.openai.com/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F file="@audio.mp3" \
  -F model="whisper-1" \
  -F response_format="srt" > subs.srt
```

## Burn Subtitles into Video

```bash
# Basic
ffmpeg -i video.mp4 -vf "subtitles=subs.srt" output.mp4

# Styled (white text, black outline, bottom-center)
ffmpeg -i video.mp4 -vf "subtitles=subs.srt:force_style='FontSize=22,PrimaryColour=&HFFFFFF&,OutlineColour=&H000000&,BorderStyle=3,Outline=2,Alignment=2'" output.mp4

# With semi-transparent background box
ffmpeg -i video.mp4 -vf "subtitles=subs.srt:force_style='FontSize=20,PrimaryColour=&HFFFFFF&,BackColour=&H80000000&,BorderStyle=4,Outline=0,Shadow=0,MarginV=30'" output.mp4
```

## Whisper Model Sizes

| Model | Size | Speed | Quality | VRAM |
|---|---|---|---|---|
| tiny | 39M | Fastest | Basic | ~1GB |
| base | 74M | Fast | Good | ~1GB |
| small | 244M | Medium | Better | ~2GB |
| medium | 769M | Slow | Great | ~5GB |
| large-v3 | 1.5G | Slowest | Best | ~10GB |

## Tips
- Use `base` model for quick drafts, `medium` or `large-v3` for production
- For Chinese content, Whisper works well but specify `--language zh`
- Always extract audio to WAV/MP3 before transcribing (Whisper handles video files poorly)
- OpenAI API Whisper is fast and cheap — best option if you have an API key
