---
name: hitpop-shotstack
description: "Cloud video editing and rendering via Shotstack API. JSON-driven video assembly with concurrent rendering, transitions, text overlays, and effects. Free sandbox available. Best for enterprise-scale batch video production."
version: 0.1.0
metadata:
  openclaw:
    emoji: "☁️"
    requires:
      env:
        - SHOTSTACK_API_KEY
      bins:
        - curl
        - jq
    primaryEnv: SHOTSTACK_API_KEY
---

# Hitpop Shotstack — Cloud Video Editing API

Enterprise-grade cloud video rendering via Shotstack. JSON templates → rendered videos at scale.

## Setup

1. Get free API key at https://dashboard.shotstack.io/register
2. Set environment variable:

```bash
export SHOTSTACK_API_KEY="your-shotstack-api-key"
```

Use `stage` environment for testing (free), `v1` for production.

## Basic Video Assembly

```bash
curl -s -X POST 'https://api.shotstack.io/edit/stage/render' \
  -H 'Content-Type: application/json' \
  -H "x-api-key: $SHOTSTACK_API_KEY" \
  -d '{
    "timeline": {
      "tracks": [
        {
          "clips": [
            {
              "asset": {
                "type": "video",
                "src": "https://example.com/clip1.mp4"
              },
              "start": 0,
              "length": 5,
              "transition": {
                "in": "fade",
                "out": "fade"
              }
            },
            {
              "asset": {
                "type": "video",
                "src": "https://example.com/clip2.mp4"
              },
              "start": 4.5,
              "length": 5
            }
          ]
        },
        {
          "clips": [
            {
              "asset": {
                "type": "title",
                "text": "My Product",
                "style": "future",
                "size": "large"
              },
              "start": 0,
              "length": 3,
              "position": "center"
            }
          ]
        }
      ]
    },
    "output": {
      "format": "mp4",
      "resolution": "hd"
    }
  }' | jq .
```

## Poll for Result

```bash
RENDER_ID="your-render-id"
curl -s "https://api.shotstack.io/edit/stage/render/$RENDER_ID" \
  -H "x-api-key: $SHOTSTACK_API_KEY" | jq '{status: .response.status, url: .response.url}'
```

## Key Features
- Concurrent rendering (thousands of videos simultaneously)
- Transitions: fade, reveal, wipeLeft, wipeRight, slideLeft, slideRight, carouselLeft, carouselRight, zoom
- Text overlays with built-in styles
- Audio mixing (voice + BGM)
- Image and video overlays
- Webhooks for render completion
- Free sandbox for development

## When to Use Shotstack vs Rendervid

| Feature | Shotstack | Rendervid |
|---|---|---|
| Hosting | Cloud (managed) | Self-hosted |
| Pricing | Free sandbox + paid | Free forever |
| Scale | Thousands concurrent | Limited by your infra |
| Templates | JSON timeline | JSON scenes |
| Best for | Enterprise batch rendering | AI agent workflows |

## Tips
- Use `stage` endpoint for free testing, `v1` for production
- Shotstack renders in seconds, not minutes
- Combine with hitpop-gen-image to create assets, then assemble with Shotstack
- Use webhooks (`callback`) instead of polling for production
