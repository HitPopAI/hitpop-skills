---
name: hitpop-publish
description: "Publish videos to social platforms. Resize and optimize for TikTok, YouTube Shorts, Instagram Reels, and more. Platform-specific formatting, thumbnail generation, and upload via official APIs."
version: 0.1.0
metadata:
  openclaw:
    emoji: "📤"
    requires:
      bins:
        - ffmpeg
        - curl
---

# Hitpop Publish — Video Distribution to Social Platforms

Optimize and publish videos for different social media platforms.

## Platform Specs

| Platform | Aspect Ratio | Resolution | Max Duration | Max Size |
|---|---|---|---|---|
| TikTok | 9:16 | 1080x1920 | 10 min | 287 MB |
| YouTube Shorts | 9:16 | 1080x1920 | 60s | 256 MB |
| Instagram Reels | 9:16 | 1080x1920 | 90s | 250 MB |
| Instagram Feed | 1:1 or 4:5 | 1080x1080 | 60s | 250 MB |
| YouTube | 16:9 | 1920x1080 | 12h | 256 GB |
| X (Twitter) | 16:9 or 1:1 | 1920x1080 | 140s | 512 MB |

## Resize for Platform

```bash
# TikTok / Reels / Shorts (9:16 vertical)
ffmpeg -i input.mp4 -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2:black" -c:a copy tiktok.mp4

# Instagram Feed (1:1 square)
ffmpeg -i input.mp4 -vf "scale=1080:1080:force_original_aspect_ratio=decrease,pad=1080:1080:(ow-iw)/2:(oh-ih)/2:black" -c:a copy instagram.mp4

# YouTube (16:9)
ffmpeg -i input.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:black" -c:a copy youtube.mp4
```

## Compress for Upload

```bash
# Target ~10MB for fast upload
ffmpeg -i input.mp4 -c:v libx264 -crf 28 -preset fast -c:a aac -b:a 128k compressed.mp4

# Target specific file size (e.g., 50MB)
DURATION=$(ffprobe -v error -show_entries format=duration -of csv=p=0 input.mp4)
BITRATE=$(echo "50 * 8192 / $DURATION" | bc)
ffmpeg -i input.mp4 -c:v libx264 -b:v ${BITRATE}k -pass 1 -f null /dev/null
ffmpeg -i input.mp4 -c:v libx264 -b:v ${BITRATE}k -pass 2 -c:a aac -b:a 128k output.mp4
```

## Generate Thumbnail

```bash
# Best frame (scene change detection)
ffmpeg -i video.mp4 -vf "select='gt(scene,0.3)'" -frames:v 1 -vsync vfr thumb.jpg

# Specific timestamp
ffmpeg -i video.mp4 -ss 00:00:03 -frames:v 1 -q:v 2 thumb.jpg
```

## Upload APIs

### YouTube (via YouTube Data API v3)
Requires OAuth2 setup. Use `google-api-python-client`:
```bash
pip install google-api-python-client google-auth-httplib2 google-auth-oauthlib
```

### TikTok (via TikTok Content Posting API)
Requires TikTok Developer account: https://developers.tiktok.com/

### Instagram (via Instagram Graph API)
Requires Facebook/Meta Business account.

## Batch Export for All Platforms

```bash
#!/bin/bash
INPUT="$1"
BASENAME=$(basename "$INPUT" .mp4)

# TikTok / Reels / Shorts
ffmpeg -i "$INPUT" -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2" -c:a aac "${BASENAME}_tiktok.mp4"

# YouTube
ffmpeg -i "$INPUT" -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" -c:a aac "${BASENAME}_youtube.mp4"

# Instagram Square
ffmpeg -i "$INPUT" -vf "scale=1080:1080:force_original_aspect_ratio=decrease,pad=1080:1080:(ow-iw)/2:(oh-ih)/2" -c:a aac "${BASENAME}_instagram.mp4"

echo "Exported: ${BASENAME}_tiktok.mp4, ${BASENAME}_youtube.mp4, ${BASENAME}_instagram.mp4"
```

## Tips
- Always re-encode when changing resolution (can't use `-c copy`)
- CRF 23-28 is good quality/size balance for social platforms
- Generate thumbnails from the most visually interesting frame
- Most platforms prefer H.264 video + AAC audio in MP4 container
