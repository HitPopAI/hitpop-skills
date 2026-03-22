---
name: hitpop-rendervid
description: "Programmatic template-based video generation using Rendervid. Free, open-source, AI-agent-native. Create marketing videos, social content, and data-driven videos from JSON templates. Built-in MCP server with 100+ templates."
version: 0.1.0
metadata:
  openclaw:
    emoji: "🎞️"
    requires:
      bins:
        - npx
    install:
      - kind: node
        package: rendervid
        bins: [npx]
---

# Hitpop Rendervid — Programmatic Template Video (Free)

Create template-based videos using Rendervid — a free, open-source video rendering engine designed for AI agents. Perfect for marketing videos, social media content, product showcases, and data-driven videos.

## Why Rendervid

- **Free**: Apache 2.0 license, no commercial license fees (unlike Remotion)
- **AI-native**: Built-in MCP server with 11 tools for AI agent integration
- **100+ templates**: Instagram, TikTok, YouTube, product showcase, ads, etc.
- **JSON-driven**: No React code needed — describe videos in JSON

## Setup

```bash
# Clone and install
git clone https://github.com/nickhudkins/rendervid.git
cd rendervid
npm install

# Or use npx directly
npx rendervid render --template templates/social-post.json --output video.mp4
```

## MCP Server (for Claude Code / Cursor / Windsurf)

Add to your MCP config:

```json
{
  "mcpServers": {
    "rendervid": {
      "command": "npx",
      "args": ["rendervid", "mcp"]
    }
  }
}
```

The MCP server exposes 11 tools:
1. `discover_capabilities` — What can Rendervid do
2. `browse_templates` — List 100+ templates by category
3. `get_template` — Get a specific template's JSON
4. `generate_template` — Create template from natural language
5. `validate_template` — Check JSON structure + media URLs
6. `render_video` — Render template to video file
7. `render_image` — Render single frame as image
8. `list_fonts` — Available fonts
9. `list_effects` — Available effects and transitions
10. `get_render_status` — Check render progress
11. `get_output` — Retrieve rendered file

## JSON Template Example

```json
{
  "width": 1080,
  "height": 1920,
  "fps": 30,
  "duration": 5,
  "scenes": [
    {
      "duration": 5,
      "elements": [
        {
          "type": "video",
          "src": "https://example.com/background.mp4",
          "fit": "cover"
        },
        {
          "type": "text",
          "text": "YOUR PRODUCT NAME",
          "style": {
            "fontSize": 72,
            "fontWeight": "bold",
            "color": "#ffffff",
            "textAlign": "center"
          },
          "position": { "x": "center", "y": "40%" },
          "animations": [
            { "type": "fadeIn", "duration": 0.5 }
          ]
        }
      ]
    }
  ]
}
```

## Template Categories

- Social media (Instagram Reels, TikTok, YouTube Shorts)
- Product showcases and flash sales
- News and announcements
- Data visualizations
- Educational content
- Real estate listings
- Event promotions

## When to Use Rendervid vs hitpop-gen-video

| Use Case | Rendervid | hitpop-gen-video |
|---|---|---|
| Template-based, data-driven | ✅ Best choice | ❌ |
| Batch generate variations | ✅ Best choice | ❌ |
| AI-generated creative content | ❌ | ✅ Best choice |
| Text-to-video from description | ❌ | ✅ Best choice |
| Branded marketing with consistent layout | ✅ Best choice | ❌ |
| Product photo animation | Could work | ✅ Better results |

## Tips
- Use Rendervid for structured, repeatable content (e-commerce, social templates)
- Use hitpop-gen-video (Vidu Q2) for creative, AI-generated content
- Combine both: generate AI images with hitpop-gen-image, then assemble with Rendervid templates
