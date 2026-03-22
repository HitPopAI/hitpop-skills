---
name: hitpop-edit
description: "Video post-processing using FFmpeg. Trim, merge, resize, add watermarks, burn subtitles, convert formats, extract audio, create thumbnails, and combine video+audio tracks."
version: 0.1.0
metadata:
  openclaw:
    emoji: "✂️"
    requires:
      bins:
        - ffmpeg
        - ffprobe
    install:
      - kind: brew
        formula: ffmpeg
        bins: [ffmpeg, ffprobe]
---

# Hitpop Edit — Video Post-Processing (FFmpeg)

All video editing and post-processing operations using FFmpeg.

## Common Operations

### Trim a video
```bash
ffmpeg -i input.mp4 -ss 00:00:02 -to 00:00:07 -c copy trimmed.mp4
```

### Merge multiple videos
```bash
# Create file list
echo "file 'intro.mp4'" > list.txt
echo "file 'main.mp4'" >> list.txt
echo "file 'outro.mp4'" >> list.txt

ffmpeg -f concat -safe 0 -i list.txt -c copy merged.mp4
```

### Resize / change resolution
```bash
ffmpeg -i input.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" -c:a copy resized.mp4
```

### Add logo/watermark overlay
```bash
ffmpeg -i input.mp4 -i logo.png -filter_complex "overlay=W-w-20:H-h-20" -c:a copy watermarked.mp4
```

### Burn subtitles into video
```bash
ffmpeg -i input.mp4 -vf "subtitles=subs.srt:force_style='FontSize=24,PrimaryColour=&HFFFFFF&'" -c:a copy subtitled.mp4
```

### Extract audio from video
```bash
ffmpeg -i input.mp4 -vn -acodec libmp3lame -q:a 2 audio.mp3
```

### Add audio track to video
```bash
ffmpeg -i video.mp4 -i voiceover.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 -shortest output.mp4
```

### Mix multiple audio tracks (voice + music)
```bash
ffmpeg -i video.mp4 -i voice.mp3 -i bgm.mp3 \
  -filter_complex "[1:a]volume=1.0[voice];[2:a]volume=0.3[music];[voice][music]amix=inputs=2:duration=first[aout]" \
  -map 0:v -map "[aout]" -c:v copy -c:a aac output.mp4
```

### Convert format
```bash
ffmpeg -i input.webm -c:v libx264 -c:a aac output.mp4
```

### Create thumbnail from video
```bash
ffmpeg -i input.mp4 -ss 00:00:03 -frames:v 1 thumbnail.jpg
```

### Create GIF from video
```bash
ffmpeg -i input.mp4 -ss 0 -t 3 -vf "fps=15,scale=480:-1:flags=lanczos" -loop 0 output.gif
```

### Get video info
```bash
ffprobe -v quiet -print_format json -show_format -show_streams input.mp4 | jq '{duration: .format.duration, width: .streams[0].width, height: .streams[0].height, codec: .streams[0].codec_name}'
```

### Speed up / slow down
```bash
# 2x speed
ffmpeg -i input.mp4 -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]" fast.mp4

# 0.5x speed (slow motion)
ffmpeg -i input.mp4 -filter_complex "[0:v]setpts=2.0*PTS[v];[0:a]atempo=0.5[a]" -map "[v]" -map "[a]" slow.mp4
```

### Add fade in/out
```bash
ffmpeg -i input.mp4 -vf "fade=t=in:st=0:d=1,fade=t=out:st=4:d=1" -c:a copy faded.mp4
```

## Tips
- Always use `-c copy` when possible (no re-encoding = fast + lossless)
- For concat, all videos must have same codec, resolution, and frame rate
- Use `ffprobe` to inspect files before processing
- `-shortest` flag cuts output to shortest input stream length
