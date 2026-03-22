---
name: hitpop-gen-image
description: "AI image generation using Doubao Seedream 4.0/4.5 via Zhipu (Z.ai) API. Supports text-to-image, image+text-to-image, multi-image fusion, and sequential image sets. Up to 4K resolution."
version: 0.1.0
metadata:
  openclaw:
    emoji: "🖼️"
    requires:
      env:
        - ZHIPU_API_KEY
      bins:
        - curl
        - jq
    primaryEnv: ZHIPU_API_KEY
---

# Hitpop Gen Image — AI Image Generation (Seedream 4.0/4.5)

Generate images using Doubao Seedream models through the Zhipu open platform API.

## Setup

```bash
export ZHIPU_API_KEY="your-api-key-from-bigmodel.cn"
```

## Available Models

| Model | Capabilities | Min Resolution | Price |
|---|---|---|---|
| `doubao-seedream-4.5` | Text→Image, best quality, 2K-4K | 2560x1440 | ¥0.25/image |
| `doubao-seedream-4.0` | Text→Image, Image+Text→Image, multi-image fusion | 1280x720 | ¥0.20/image |

## API Reference

- **Base URL**: `https://open.bigmodel.cn/api/paas/v4/images/generations`
- **Method**: POST (async — returns task ID, poll for result)
- **Auth**: `Authorization: Bearer $ZHIPU_API_KEY`

## Usage Examples

### 1. Text to Image (Single Image)

```bash
curl -s -X POST 'https://open.bigmodel.cn/api/paas/v4/images/generations' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $ZHIPU_API_KEY" \
  -d '{
    "model": "doubao-seedream-4.5",
    "prompt": "A cinematic shot of a vintage train bursting out of a black hole, dramatic lighting, film grain, deep blue tones, ultra-realistic",
    "size": "2K",
    "watermark": false,
    "sequential_image_generation": "disabled"
  }' | jq .
```

### 2. Text to Image Set (Sequential/Group)

```bash
curl -s -X POST 'https://open.bigmodel.cn/api/paas/v4/images/generations' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $ZHIPU_API_KEY" \
  -d '{
    "model": "doubao-seedream-4.0",
    "prompt": "Generate 4 connected illustrations showing the same courtyard across four seasons, consistent style",
    "size": "2048x2048",
    "sequential_image_generation": "auto",
    "sequential_image_generation_options": {
      "max_images": 4
    },
    "watermark": false
  }' | jq .
```

### 3. Image + Text to Image (Reference-Based)

```bash
curl -s -X POST 'https://open.bigmodel.cn/api/paas/v4/images/generations' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $ZHIPU_API_KEY" \
  -d '{
    "model": "doubao-seedream-4.0",
    "prompt": "Generate the dog lying on a grass field, close-up shot",
    "images": "https://example.com/my-dog-photo.jpg",
    "size": "2K",
    "watermark": false
  }' | jq .
```

### 4. Multi-Image Fusion

```bash
curl -s -X POST 'https://open.bigmodel.cn/api/paas/v4/images/generations' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $ZHIPU_API_KEY" \
  -d '{
    "model": "doubao-seedream-4.0",
    "prompt": "Replace the clothing in image 1 with the outfit from image 2",
    "images": [
      "https://example.com/person.jpg",
      "https://example.com/outfit.jpg"
    ],
    "size": "2K",
    "watermark": false
  }' | jq .
```

### 5. Poll for Result

```bash
TASK_ID="your-task-id"
curl -s -X GET "https://open.bigmodel.cn/api/paas/v4/async-result/$TASK_ID" \
  -H "Authorization: Bearer $ZHIPU_API_KEY" | jq .
```

Response contains `data[].url` with image download links (valid 24 hours).

## Parameters Reference

| Parameter | Type | Required | Description |
|---|---|---|---|
| `model` | string | Yes | `doubao-seedream-4.5` or `doubao-seedream-4.0` |
| `prompt` | string | Yes | Image description. Recommended ≤300 Chinese chars or 600 English words |
| `size` | string | No | Method 1: "1K", "2K", "4K". Method 2: "2048x2048", "2560x1440", etc. |
| `images` | string/array | No | Reference image URL(s) or base64. Seedream 4.0 supports multi-image |
| `seed` | integer | No | Random seed [-1, 2147483647]. Same seed = similar results |
| `sequential_image_generation` | string | No | "auto" (model decides count) or "disabled" (single image). Default: disabled |
| `sequential_image_generation_options.max_images` | integer | No | Max images in a set [1, 15]. Only when sequential = "auto" |
| `response_format` | string | No | "url" (default) or "b64_json" |
| `watermark` | bool | No | Add "AI generated" watermark. Default: true — **set false for production** |
| `optimize_prompt_options.mode` | string | No | "standard" (better quality) or "fast" (quicker) |

## Recommended Sizes

| Aspect Ratio | Pixels |
|---|---|
| 1:1 | 2048×2048 |
| 4:3 | 2304×1728 |
| 3:4 | 1728×2304 |
| 16:9 | 2560×1440 |
| 9:16 | 1440×2560 |
| 3:2 | 2496×1664 |
| 21:9 | 3024×1296 |

## Tips
- Seedream 4.5 produces best quality but only supports text input (no reference images)
- Seedream 4.0 supports reference images and multi-image fusion
- Use `sequential_image_generation: "auto"` for storyboards, product sets, style guides
- Image URLs expire after 24 hours
- For video production pipeline: generate images here, then pass to `hitpop-gen-video`
