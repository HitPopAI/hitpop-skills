---
name: hitpop-twick
description: "React-based video editor SDK for building interactive editing experiences. Timeline editing, AI captions, canvas tools, drag-and-drop, and serverless MP4 export. Use when users need to manually edit or fine-tune AI-generated videos."
version: 0.1.0
metadata:
  openclaw:
    emoji: "🎛️"
    requires:
      bins:
        - npx
        - node
    install:
      - kind: node
        package: "@twick/video-editor"
        bins: [npx]
---

# Hitpop Twick — Interactive Video Editor SDK

React-based video editing SDK for building timeline editors, adding AI captions, and exporting MP4. Use when AI-generated videos need manual fine-tuning or when building user-facing editing interfaces.

## When to Use

- User wants to **edit** an AI-generated video (trim, rearrange, add text overlays)
- Building an app where **end users** need to edit videos themselves
- Need **AI captions** auto-synced to video
- Need **programmatic editing** via code (add elements, set timelines)

## Setup

```bash
# Install packages
npm install @twick/video-editor @twick/timeline @twick/live-player @twick/canvas

# Or clone the full repo
git clone https://github.com/ncounterspecialist/twick.git
cd twick
pnpm install
pnpm build
```

## Quick Start — Programmatic Video Creation

Launch the example editor, then use the browser console:

```bash
cd packages/examples
pnpm install && pnpm build && pnpm preview
# Open http://localhost:4173/demo
```

In browser console (F12):

```javascript
// Get editor instance
const editor = twickTimelineEditors.get("my-video-project");

// Add a video track
const videoTrack = editor.addTrack("video");

// Add a video clip
const video = new Twick.VideoElement(
  "https://example.com/ai-generated-video.mp4",
  { width: 1080, height: 1920 }
);
video.setStart(0).setEnd(8);
await editor.addElementToTrack(videoTrack, video);

// Add text overlay track
const textTrack = editor.addTrack("text");
const title = new Twick.TextElement("Your Product Name")
  .setStart(0.5)
  .setEnd(3)
  .setFill("#ffffff")
  .setStrokeColor("#000000")
  .setLineWidth(0.2)
  .setFontSize(64)
  .setPosition({ x: 0, y: 200 });
await editor.addElementToTrack(textTrack, title);

// Export as MP4
await editor.export("mp4");
```

## React Component Integration

```jsx
import React from 'react';
import VideoEditor from "@twick/video-editor";
import { LivePlayerProvider } from "@twick/live-player";
import { TimelineProvider } from "@twick/timeline";
import "@twick/video-editor/dist/video-editor.css";

function App() {
  return (
    <LivePlayerProvider>
      <TimelineProvider contextId="my-project">
        <VideoEditor
          editorConfig={{
            canvasMode: true,
            videoProps: { width: 1080, height: 1920 }
          }}
        />
      </TimelineProvider>
    </LivePlayerProvider>
  );
}
```

## Packages

| Package | Purpose |
|---|---|
| `@twick/video-editor` | Complete editing UI |
| `@twick/timeline` | Timeline CRUD operations |
| `@twick/live-player` | Video playback with controls |
| `@twick/canvas` | Drag-and-drop canvas editing |
| `@twick/visualizer` | Animations and real-time preview |
| `@twick/media-utils` | Audio/video metadata extraction |
| `@twick/studio` | Full studio interface |

## Element Types

- `VideoElement` — Video clips with timing and playback rate
- `TextElement` — Text overlays with font styling and positioning
- `ImageElement` — Image overlays
- `AudioElement` — Audio tracks
- `RectangleElement`, `CircleElement` — Shape overlays
- `CaptionElement` — AI-synced captions

## Twick vs Other Hitpop Skills

| Need | Use |
|---|---|
| Generate video from scratch (AI) | `hitpop-gen-video` |
| Template-based batch videos | `hitpop-rendervid` or `hitpop-shotstack` |
| **Edit/fine-tune a video interactively** | **`hitpop-twick`** |
| Simple trim/merge/watermark | `hitpop-edit` (FFmpeg, faster) |

## License

Sustainable Use License (SUL) v1.0 — free for commercial use in your app. Not for reselling as an SDK.

## Tips
- Use Twick when you need a visual editor UI; use FFmpeg (`hitpop-edit`) for headless processing
- AI captions use Google Vertex AI (Gemini) — requires separate setup
- Serverless export uses AWS Lambda + S3 — configure in `twick.config.ts`
- Combine: generate with `hitpop-gen-video` → fine-tune with `hitpop-twick` → publish with `hitpop-publish`
