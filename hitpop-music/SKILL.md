---
name: hitpop-music
description: "Add background music to videos. Supports AI music generation via Suno/Udio APIs, royalty-free music libraries, and FFmpeg audio mixing with volume control."
version: 0.1.0
metadata:
  openclaw:
    emoji: "🎵"
    requires:
      bins:
        - ffmpeg
        - curl
---

# Hitpop Music — Background Music for Video

Add background music to videos using AI-generated or royalty-free tracks.

## Option 1: Royalty-Free Music (No API Key, Instant)

Download from free libraries, then mix with FFmpeg:

- **Pixabay Music**: https://pixabay.com/music/ (free, no attribution)
- **Free Music Archive**: https://freemusicarchive.org/
- **Incompetech**: https://incompetech.com/music/ (CC BY)

```bash
# Download a track
curl -L "https://cdn.pixabay.com/audio/2024/01/01/audio_example.mp3" -o bgm.mp3

# Mix into video at 30% volume
ffmpeg -i video.mp4 -i bgm.mp3 \
  -filter_complex "[1:a]volume=0.3[music];[0:a][music]amix=inputs=2:duration=first[aout]" \
  -map 0:v -map "[aout]" -c:v copy -c:a aac output.mp4
```

## Option 2: Vidu Q2 Built-in BGM (Simplest)

When generating video with `hitpop-gen-video`, set `with_audio: true`:

```json
{
  "model": "viduq2-text2video",
  "prompt": "...",
  "with_audio": true
}
```

The system auto-selects appropriate BGM from a preset library.

## Option 3: AI Music Generation (Suno API)

```bash
# Via unofficial Suno API (check current availability)
curl -s -X POST "https://api.suno.ai/v1/generate" \
  -H "Authorization: Bearer $SUNO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "upbeat corporate background music, positive energy, no vocals",
    "duration": 30
  }' | jq .
```

## Audio Mixing with FFmpeg

```bash
# Add BGM to video (no existing audio)
ffmpeg -i video.mp4 -i bgm.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 -shortest output.mp4

# Mix BGM with voiceover (voice loud, music quiet)
ffmpeg -i video.mp4 -i voice.mp3 -i bgm.mp3 \
  -filter_complex "[1:a]volume=1.0[v];[2:a]volume=0.2[m];[v][m]amix=inputs=2:duration=first[aout]" \
  -map 0:v -map "[aout]" -c:v copy -c:a aac output.mp4

# Fade music in/out
ffmpeg -i bgm.mp3 -af "afade=t=in:st=0:d=2,afade=t=out:st=28:d=2" bgm_faded.mp3

# Loop short music to match video length
ffmpeg -stream_loop -1 -i short_bgm.mp3 -i video.mp4 -c:v copy -c:a aac -map 1:v:0 -map 0:a:0 -shortest output.mp4
```

## Tips
- Always fade BGM in/out for professional feel
- Keep BGM at 20-30% volume when mixing with voiceover
- Use `-shortest` flag to auto-trim audio to video length
- For production, use royalty-free tracks to avoid copyright issues
