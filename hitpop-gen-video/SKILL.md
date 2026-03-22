---
name: hitpop-gen-video
description: "AI video generation using Vidu Q2 models via Zhipu (Z.ai) API. Supports text-to-video, image-to-video, reference-based video, and start/end frame interpolation. Up to 1080p, 8 seconds, with optional BGM."
version: 0.1.0
metadata:
  openclaw:
    emoji: "🎥"
    requires:
      env:
        - ZHIPU_API_KEY
      bins:
        - curl
        - jq
    primaryEnv: ZHIPU_API_KEY
---

# Hitpop Gen Video — AI Video Generation (Vidu Q2)

Generate videos using Vidu Q2 models through the Zhipu (Z.ai) open platform API.

## Setup

```bash
export ZHIPU_API_KEY="your-api-key-from-bigmodel.cn"
```

## Available Models

| Model | Mode | Max Duration | Default Resolution | Price |
|---|---|---|---|---|
| `viduq2-text2video` | Text → Video | 8s | 1280x720 | ~¥0.40 |
| `viduq2-pro-img2video` | Image → Video (high quality) | 8s | 1280x720 | ~¥0.40 |
| `viduq2-turbo-img2video` | Image → Video (fast) | 8s | 1920x1080 | ~¥0.20 |
| `viduq2-img2video` | Reference images → Video (subject consistency) | 8s | 1280x720 | ~¥0.40 |
| `viduq2-pro-img2video-frame` | Start+End frame → Video (high quality) | 8s | 1280x720 | ~¥0.40 |
| `viduq2-turbo-img2video-frame` | Start+End frame → Video (fast) | 8s | 1920x1080 | ~¥0.20 |

## API Reference

- **Base URL**: `https://open.bigmodel.cn/api/paas/v4/videos/generations`
- **Method**: POST (async — returns task ID, poll for result)
- **Auth**: `Authorization: Bearer $ZHIPU_API_KEY`

## Usage Examples

### 1. Text to Video

```bash
curl -s -X POST 'https://open.bigmodel.cn/api/paas/v4/videos/generations' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $ZHIPU_API_KEY" \
  -d '{
    "model": "viduq2-text2video",
    "prompt": "A girl holding a fox, slowly opening her eyes and gently looking at the camera, hair blowing in the wind",
    "duration": "5",
    "size": "1920x1080",
    "aspect_ratio": "16:9",
    "with_audio": false,
    "watermark": false
  }' | jq .
```

### 2. Image to Video (High Quality)

```bash
curl -s -X POST 'https://open.bigmodel.cn/api/paas/v4/videos/generations' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $ZHIPU_API_KEY" \
  -d '{
    "model": "viduq2-pro-img2video",
    "image_url": ["https://example.com/product-photo.jpg"],
    "prompt": "Camera slowly zooms out revealing the full product, soft lighting",
    "duration": "5",
    "size": "1920x1080",
    "movement_amplitude": "medium",
    "watermark": false
  }' | jq .
```

### 3. Reference-Based Video (Subject Consistency, 1-7 images)

```bash
curl -s -X POST 'https://open.bigmodel.cn/api/paas/v4/videos/generations' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $ZHIPU_API_KEY" \
  -d '{
    "model": "viduq2-img2video",
    "image_url": [
      "https://example.com/character-ref1.jpg",
      "https://example.com/character-ref2.jpg"
    ],
    "prompt": "The character walks through a sunlit forest, looking around curiously",
    "duration": "5",
    "size": "1280x720",
    "watermark": false
  }' | jq .
```

### 4. Start & End Frame Interpolation

```bash
curl -s -X POST 'https://open.bigmodel.cn/api/paas/v4/videos/generations' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $ZHIPU_API_KEY" \
  -d '{
    "model": "viduq2-pro-img2video-frame",
    "image_url": [
      "https://example.com/frame-start.jpg",
      "https://example.com/frame-end.jpg"
    ],
    "prompt": "Smooth transition with a flash of white light",
    "duration": "5",
    "size": "1920x1080",
    "movement_amplitude": "auto",
    "watermark": false
  }' | jq .
```

### 5. Poll for Result

All video generation is async. Extract the `id` from the creation response, then poll:

```bash
TASK_ID="your-task-id-from-creation-response"

# Poll until SUCCESS or FAIL
while true; do
  RESULT=$(curl -s -X GET "https://open.bigmodel.cn/api/paas/v4/async-result/$TASK_ID" \
    -H "Authorization: Bearer $ZHIPU_API_KEY")
  STATUS=$(echo "$RESULT" | jq -r '.task_status')
  echo "Status: $STATUS"
  if [ "$STATUS" = "SUCCESS" ]; then
    echo "$RESULT" | jq -r '.video_result[0].url'
    break
  elif [ "$STATUS" = "FAIL" ]; then
    echo "Generation failed"
    echo "$RESULT" | jq .
    break
  fi
  sleep 5
done
```

## Parameters Reference

### Common Parameters (all models)
| Parameter | Type | Required | Description |
|---|---|---|---|
| `model` | string | Yes | Model ID (see table above) |
| `prompt` | string | Optional | Text description, max 2000 chars (512 for img2video) |
| `duration` | string | Optional | Video length: "1" to "8" seconds, default "5" |
| `size` | string | Optional | "1280x720" or "1920x1080" |
| `with_audio` | bool | Optional | Auto BGM from preset library. Default: false |
| `watermark` | bool | Optional | Add "AI generated" watermark. Default: false |

### Image-Specific Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `image_url` | array | Yes (for img models) | Array of image URLs or base64. Formats: png, jpeg, jpg, webp |
| `aspect_ratio` | string | Optional | "16:9", "9:16", "4:3", "3:4", "1:1" |
| `movement_amplitude` | string | Optional | "auto", "small", "medium", "large" |

### Image Constraints
- **img2video (reference)**: 1-7 images, each < 50MB, min 128x128, ratio < 4:1
- **pro/turbo-img2video**: Single image only, < 50MB, ratio < 4:1
- **frame models**: Exactly 2 images (start + end), similar resolution (ratio 0.8-1.25)
- Base64 format: `data:image/png;base64,{base64_data}` (decoded < 10MB)

## Tips
- Use `viduq2-turbo-*` for fast iterations, `viduq2-pro-*` for final output
- `movement_amplitude: "small"` for product shots, `"large"` for action scenes
- Video URLs expire after 24 hours — download immediately
- For best results, write prompts describing camera movement + subject action
