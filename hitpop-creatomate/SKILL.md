---
name: hitpop-creatomate
description: "Cloud video and image automation via Creatomate API. Template-based rendering with dynamic data, RenderScript for code-only videos, and preview SDK for browser editing. Best for marketing teams and SaaS products needing branded video at scale."
version: 0.1.0
metadata:
  openclaw:
    emoji: "🎨"
    requires:
      env:
        - CREATOMATE_API_KEY
      bins:
        - curl
        - jq
    primaryEnv: CREATOMATE_API_KEY
---

# Hitpop Creatomate — Template Video Automation API

Cloud-based video and image generation using Creatomate. Design templates in a visual editor, then render variations via API with dynamic data.

## Setup

1. Create free account at https://creatomate.com
2. Get API key from Project Settings
3. Create a template in the online editor

```bash
export CREATOMATE_API_KEY="your-api-key"
```

## Render Video from Template

```bash
curl -s -X POST 'https://api.creatomate.com/v2/renders' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $CREATOMATE_API_KEY" \
  -d '{
    "template_id": "YOUR_TEMPLATE_ID",
    "modifications": {
      "Title": "Hitpop AI Video Suite",
      "Title.fill_color": "#4980f1",
      "Subtitle": "Generate, edit, publish — all from code",
      "Background-Video": "https://example.com/demo-footage.mp4",
      "Logo": "https://example.com/logo.png"
    }
  }' | jq .
```

Response:
```json
[{
  "id": "render-id",
  "status": "planned",
  "url": "https://cdn.creatomate.com/renders/..../output.mp4"
}]
```

## Check Render Status

```bash
RENDER_ID="your-render-id"
curl -s "https://api.creatomate.com/v2/renders/$RENDER_ID" \
  -H "Authorization: Bearer $CREATOMATE_API_KEY" | jq '{status: .status, url: .url}'
```

Status flow: `planned` → `rendering` → `succeeded` (or `failed`)

## Render Video from Code (RenderScript, No Template)

For full control without using the visual editor:

```bash
curl -s -X POST 'https://api.creatomate.com/v2/renders' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $CREATOMATE_API_KEY" \
  -d '{
    "source": {
      "output_format": "mp4",
      "width": 1080,
      "height": 1920,
      "duration": 10,
      "elements": [
        {
          "type": "video",
          "source": "https://example.com/background.mp4",
          "time": 0,
          "duration": 10
        },
        {
          "type": "text",
          "text": "PRODUCT LAUNCH",
          "y": "35%",
          "width": "80%",
          "x_alignment": "50%",
          "font_family": "Montserrat",
          "font_weight": "800",
          "font_size": "10 vmin",
          "fill_color": "#ffffff",
          "time": 1,
          "duration": 4,
          "animations": [
            {
              "type": "text-appear",
              "split": "word",
              "duration": 1
            }
          ]
        }
      ]
    }
  }' | jq .
```

## Modify Template Properties (Dot Notation)

```json
{
  "template_id": "YOUR_TEMPLATE_ID",
  "modifications": {
    "Title": "New Title Text",
    "Title.font_family": "Montserrat",
    "Title.font_size": "8 vmin",
    "Title.fill_color": "#ff5500",
    "Background-Color.fill_color": "#1a1a2e",
    "Outro-Scene": {}
  }
}
```

Setting an element to `{}` removes it entirely. Dot notation lets you change any property: colors, fonts, timing, positions, visibility.

## Batch Rendering

Render multiple variations in one call:

```bash
curl -s -X POST 'https://api.creatomate.com/v2/renders' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $CREATOMATE_API_KEY" \
  -d '[
    {
      "template_id": "YOUR_TEMPLATE_ID",
      "modifications": { "Title": "Version A — Blue", "Title.fill_color": "#0066ff" }
    },
    {
      "template_id": "YOUR_TEMPLATE_ID",
      "modifications": { "Title": "Version B — Red", "Title.fill_color": "#ff0000" }
    }
  ]' | jq .
```

## List Templates

```bash
curl -s 'https://api.creatomate.com/v2/templates' \
  -H "Authorization: Bearer $CREATOMATE_API_KEY" | jq '.[].name'
```

## SDKs

| Language | Package |
|---|---|
| Node.js | `npm install creatomate` |
| PHP | `composer require creatomate/creatomate` |
| Ruby | `gem install creatomate` |
| Python | REST API (no official SDK, use `requests`) |

## Creatomate vs Other Hitpop Skills

| Feature | Creatomate | JSON2Video | Shotstack | Rendervid |
|---|---|---|---|---|
| Visual template editor | ✅ Best in class | ❌ | ❌ | ❌ |
| Code-only (no editor) | ✅ RenderScript | ✅ JSON | ✅ JSON | ✅ JSON |
| Preview SDK (browser) | ✅ | ❌ | ✅ | ❌ |
| Pricing | $54/mo+ | Free tier | Free sandbox | Free |
| Best for | Marketing teams, SaaS | Content automation | Enterprise | AI agents |

## Tips
- Design once in the visual editor, then render thousands of variations via API
- Use `webhook_url` parameter instead of polling for production
- `render_scale: 0.5` for fast previews, `1.0` for final output
- Use `max_width`/`max_height` to auto-scale without changing template
- RenderScript = "HTML for video" — use it when templates are too rigid
- Combine: generate AI images with `hitpop-gen-image` → insert into Creatomate template → batch render
