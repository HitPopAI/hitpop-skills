---
name: hitpop-json2video
description: "Cloud video editing API by JSON2Video. Create videos from JSON with scenes, elements, text animations, AI voiceovers, and transitions. Free tier available. Best for automated content pipelines and faceless video channels."
version: 0.1.0
metadata:
  openclaw:
    emoji: "📋"
    requires:
      env:
        - JSON2VIDEO_API_KEY
      bins:
        - curl
        - jq
    primaryEnv: JSON2VIDEO_API_KEY
---

# Hitpop JSON2Video — JSON-to-Video Cloud API

Create videos programmatically from JSON definitions. Scenes, text animations, AI voiceovers, images, transitions — all in one API call.

## Setup

Get your free API key at https://json2video.com (no credit card required).

```bash
export JSON2VIDEO_API_KEY="your-api-key"
```

## Create a Video

```bash
curl -s -X POST 'https://api.json2video.com/v2/movies' \
  -H "x-api-key: $JSON2VIDEO_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
    "resolution": "full-hd",
    "quality": "high",
    "scenes": [
      {
        "comment": "Intro scene",
        "background-color": "#1a1a2e",
        "elements": [
          {
            "type": "text",
            "style": "003",
            "text": "Welcome to Hitpop",
            "duration": 5,
            "start": 1,
            "settings": {
              "font-size": "80",
              "color": "#ffffff"
            }
          }
        ]
      },
      {
        "comment": "Product scene",
        "elements": [
          {
            "type": "image",
            "src": "https://example.com/product.jpg",
            "duration": 5,
            "zoom": 120
          },
          {
            "type": "text",
            "style": "001",
            "text": "AI-Powered Video Creation",
            "duration": 4,
            "start": 1,
            "y": 800
          }
        ],
        "transition": "fade"
      }
    ]
  }' | jq .
```

Response returns a `project` ID:
```json
{ "project": "abc123", "status": "rendering" }
```

## Poll for Result

```bash
PROJECT_ID="abc123"
curl -s "https://api.json2video.com/v2/movies?project=$PROJECT_ID" \
  -H "x-api-key: $JSON2VIDEO_API_KEY" | jq '{status: .status, url: .url}'
```

## Add AI Voiceover

```json
{
  "scenes": [
    {
      "elements": [
        {
          "type": "audio",
          "src": "tts",
          "tts": {
            "text": "Welcome to our product showcase. Today we introduce something amazing.",
            "voice": "en-US-Neural2-F"
          },
          "duration": -1
        },
        {
          "type": "image",
          "src": "https://example.com/bg.jpg",
          "duration": -1
        }
      ]
    }
  ]
}
```

Setting `duration: -1` auto-matches to the TTS audio length.

## Add Background Music

```json
{
  "scenes": [...],
  "elements": [
    {
      "type": "audio",
      "src": "https://example.com/bgm.mp3",
      "volume": 0.2,
      "duration": -2
    }
  ]
}
```

`duration: -2` stretches audio to match total movie length.

## Element Types

| Type | Description |
|---|---|
| `text` | Animated text with 50+ built-in styles |
| `image` | Static images with zoom/pan effects |
| `video` | Video clips |
| `audio` | Audio tracks, BGM, or TTS voiceovers |
| `html` | Custom HTML5 content |
| `component` | Reusable template components |
| `shape` | Rectangles, circles, lines |
| `subtitles` | Auto-synced subtitle overlays |

## Transitions

`fade`, `crossfade`, `wipe-left`, `wipe-right`, `wipe-up`, `wipe-down`, `slide-left`, `slide-right`, `zoom-in`, `zoom-out`, `blur`

## JSON2Video vs Other Hitpop Skills

| Feature | JSON2Video | Rendervid | Shotstack |
|---|---|---|---|
| API style | REST + JSON scenes | JSON templates | REST + JSON timeline |
| Built-in TTS | ✅ Yes | ❌ | ✅ Yes |
| Text animations | ✅ 50+ styles | Basic | Basic |
| Pricing | Free tier + paid | Free forever | Free sandbox + paid |
| Best for | Content automation, faceless channels | AI agent workflows | Enterprise batch |
| Self-hostable | ❌ Cloud only | ✅ | ❌ Cloud only |

## Tips
- Use `draft: true` for quick previews (lower quality, faster rendering)
- Scenes render in parallel — more scenes ≠ slower
- Use `-1` duration to auto-match element to its content length
- Use `-2` duration for background audio that spans the full movie
- Combine: generate images with `hitpop-gen-image` → assemble with JSON2Video
- Great for YouTube Shorts, TikTok automation, news channels, quiz videos
