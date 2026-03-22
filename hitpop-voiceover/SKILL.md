---
name: hitpop-voiceover
description: "AI voiceover and text-to-speech for video narration. Supports OpenAI TTS, ElevenLabs, and Edge TTS (free). Generate natural-sounding voiceovers in 50+ languages, then merge with video using FFmpeg."
version: 0.1.0
metadata:
  openclaw:
    emoji: "🎙️"
    requires:
      bins:
        - curl
        - ffmpeg
---

# Hitpop Voiceover — AI Text-to-Speech for Video

Generate voiceovers for your videos using multiple TTS providers.

## Option 1: Edge TTS (Free, No API Key)

Microsoft Edge TTS — free, 300+ voices, 75+ languages.

```bash
pip install edge-tts

# List available voices
edge-tts --list-voices | grep en-US

# Generate voiceover
edge-tts --voice "en-US-AriaNeural" --text "Welcome to our product showcase" --write-media voiceover.mp3

# Chinese voice
edge-tts --voice "zh-CN-XiaoxiaoNeural" --text "欢迎来到我们的产品展示" --write-media voiceover_cn.mp3
```

## Option 2: OpenAI TTS (Best Quality)

Requires `OPENAI_API_KEY`.

```bash
curl -s https://api.openai.com/v1/audio/speech \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "tts-1-hd",
    "input": "Welcome to our product showcase. Today we are introducing something incredible.",
    "voice": "nova"
  }' --output voiceover.mp3
```

Voices: `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer`

## Option 3: ElevenLabs (Most Natural)

Requires `ELEVENLABS_API_KEY`.

```bash
curl -s -X POST "https://api.elevenlabs.io/v1/text-to-speech/21m00Tcm4TlvDq8ikWAM" \
  -H "xi-api-key: $ELEVENLABS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Welcome to our product showcase.",
    "model_id": "eleven_multilingual_v2"
  }' --output voiceover.mp3
```

## Merge Voiceover with Video

```bash
# Replace audio
ffmpeg -i video.mp4 -i voiceover.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 -shortest output.mp4

# Mix with existing audio (voice at full volume, original at 20%)
ffmpeg -i video.mp4 -i voiceover.mp3 \
  -filter_complex "[0:a]volume=0.2[orig];[1:a]volume=1.0[voice];[orig][voice]amix=inputs=2:duration=first[aout]" \
  -map 0:v -map "[aout]" -c:v copy -c:a aac output.mp4
```

## Tips
- Use Edge TTS for free prototyping, OpenAI/ElevenLabs for production
- Generate voiceover first, then use its duration to set video length
- For long scripts, split into paragraphs and generate separately, then concat
