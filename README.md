# 🎬 Hitpop Skills

**The most comprehensive Video AI skills suite for OpenClaw, Claude Code, and Cursor.**

One toolkit to generate, edit, voice, subtitle, and publish videos — all from your terminal.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/skills-15-blue.svg)](#skills)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-compatible-orange.svg)](https://docs.openclaw.ai/tools/skills)

---

## What is this?

Hitpop Skills is a collection of 10 OpenClaw-compatible AI skills that cover the **entire video production pipeline** — from AI generation to social media publishing. All skills use the standard `SKILL.md` format and work with:

- **OpenClaw** (clawhub install)
- **Claude Code** (copy to `.claude/skills/`)
- **Cursor** (copy to project skills folder)

## Skills

| Skill | What it does | Backend | Cost |
|---|---|---|---|
| `hitpop-director` | 🧠 Routing brain — analyzes intent, picks skills | GLM-5-Turbo | ~$0.001/call |
| `hitpop-gen-video` | 🎥 AI video generation | Vidu Q2 (6 models) | ¥0.20-0.40/video |
| `hitpop-gen-image` | 🖼️ AI image generation | Seedream 4.0/4.5 | ¥0.20-0.25/image |
| `hitpop-edit` | ✂️ Trim, merge, resize, watermark | FFmpeg | Free |
| `hitpop-rendervid` | 🎞️ Template-based programmatic video | Rendervid (OSS) | Free |
| `hitpop-shotstack` | ☁️ Cloud video rendering at scale | Shotstack API | Free sandbox |
| `hitpop-voiceover` | 🎙️ AI narration / TTS | Edge TTS / OpenAI | Free+ |
| `hitpop-subtitle` | 💬 Auto-generate & burn subtitles | Whisper | Free (local) |
| `hitpop-music` | 🎵 Background music & mixing | FFmpeg | Free |
| `hitpop-publish` | 📤 Multi-platform export & upload | FFmpeg | Free |
| `hitpop-json2video` | 📋 JSON-to-video with text animations + TTS | JSON2Video API | Free tier |
| `hitpop-creatomate` | 🎨 Template video automation at scale | Creatomate API | Free trial |
| `hitpop-twick` | 🎛️ Interactive React video editor SDK | Twick (OSS) | Free |
| `hitpop-comfyui` | 🖥️ Local open-source AI generation (Flux, Wan2.1) | ComfyUI | Free (local GPU) |
| `hitpop-pipeline` | 🔗 Multi-step workflow orchestration via JSON | All skills | Free |

## Quick Start

### 1. Get your API key

Sign up at [bigmodel.cn](https://www.bigmodel.cn/) and grab an API key.

```bash
export ZHIPU_API_KEY="your-key-here"
```

### 2. Install

**OpenClaw:**
```bash
cp -r hitpop-*/  ~/.openclaw/workspace/skills/
```

**Claude Code:**
```bash
cp -r hitpop-*/ .claude/skills/
```

**Cursor:**
```bash
cp -r hitpop-*/ .cursor/skills/
```

### 3. Use

Just describe what you want in natural language:

```
> Generate a 5-second product video from this image, add voiceover and subtitles

> Create a TikTok vertical video about our new feature launch

> Make 4 product images in consistent style, then turn them into a slideshow video with BGM
```

The `hitpop-director` skill automatically routes to the right tools.

## Architecture

```
User Request
     │
     ▼
┌─────────────────────────────────┐
│  hitpop-director (GLM-5-Turbo)  │
│  Analyzes intent, picks skills  │
└───────────┬─────────────────────┘
            │
  ┌─────────┼──────────┬──────────────┐
  ▼         ▼          ▼              ▼
Generate  Process    Enhance       Distribute
  │         │          │              │
  ├ gen-video  ├ edit      ├ voiceover   ├ publish
  ├ gen-image  ├ twick     ├ subtitle    │
  ├ rendervid  │          ├ music       │
  ├ shotstack  │          │             │
  ├ json2video │          │             │
  ├ creatomate │          │             │
  └ comfyui    │          │             │
  │         │          │              │
  ▼         ▼          ▼              ▼
Zhipu API  FFmpeg   Whisper/TTS   Platform APIs
+ ComfyUI
```

## API Backends

### Zhipu (Z.ai) — `open.bigmodel.cn/api/paas/v4/`

All AI generation goes through Zhipu's unified API:

| Capability | Model | Endpoint |
|---|---|---|
| Text → Video | `viduq2-text2video` | `/videos/generations` |
| Image → Video (quality) | `viduq2-pro-img2video` | `/videos/generations` |
| Image → Video (fast) | `viduq2-turbo-img2video` | `/videos/generations` |
| Reference → Video (1-7 imgs) | `viduq2-img2video` | `/videos/generations` |
| Start+End Frame → Video | `viduq2-pro-img2video-frame` | `/videos/generations` |
| Text → Image (best) | `doubao-seedream-4.5` | `/images/generations` |
| Text+Image → Image | `doubao-seedream-4.0` | `/images/generations` |
| Chat / Routing | `glm-5-turbo` | `/chat/completions` |

### Free / Open-Source Tools

| Tool | Used by | License |
|---|---|---|
| FFmpeg | edit, music, publish, subtitle | LGPL |
| Whisper | subtitle | MIT |
| Edge TTS | voiceover | Free (Microsoft) |
| Rendervid | rendervid | Apache 2.0 |

### Optional Paid APIs

| Service | Used by | Free tier? |
|---|---|---|
| Shotstack | shotstack | Yes (sandbox) |
| OpenAI TTS | voiceover | No |
| ElevenLabs | voiceover | Yes (limited) |

## Example Workflows

### Product Video (Image → Video + Voice + Subtitles)

```
hitpop-gen-image  →  hitpop-gen-video  →  hitpop-voiceover  →  hitpop-subtitle  →  hitpop-edit  →  hitpop-publish
  Seedream 4.5       Vidu Q2 Pro          Edge TTS             Whisper             FFmpeg merge     TikTok/YouTube
```

### Batch Marketing Videos (Template-Based)

```
hitpop-rendervid  →  hitpop-music  →  hitpop-publish
  JSON template       Add BGM          Multi-platform export
  100+ presets        FFmpeg mix        Auto-resize
```

## Requirements

| Requirement | Skills that need it |
|---|---|
| `ZHIPU_API_KEY` | gen-video, gen-image, director |
| `curl` + `jq` | All API skills |
| `ffmpeg` | edit, voiceover, subtitle, music, publish |
| `python3` + `pip` | subtitle (whisper), voiceover (edge-tts) |
| `SHOTSTACK_API_KEY` (optional) | shotstack |
| `JSON2VIDEO_API_KEY` (optional) | json2video |
| `CREATOMATE_API_KEY` (optional) | creatomate |
| `OPENAI_API_KEY` (optional) | voiceover (premium), subtitle (API) |

## Repo Structure

```
hitpop-skills/
├── README.md
├── LICENSE
├── .gitignore
├── hitpop-director/
│   └── SKILL.md          # GLM-5-Turbo routing brain
├── hitpop-gen-video/
│   └── SKILL.md          # Vidu Q2 video generation
├── hitpop-gen-image/
│   └── SKILL.md          # Seedream 4.0/4.5 image generation
├── hitpop-edit/
│   └── SKILL.md          # FFmpeg post-processing
├── hitpop-rendervid/
│   └── SKILL.md          # Rendervid template video
├── hitpop-shotstack/
│   └── SKILL.md          # Shotstack cloud rendering
├── hitpop-voiceover/
│   └── SKILL.md          # Edge TTS / OpenAI / ElevenLabs
├── hitpop-subtitle/
│   └── SKILL.md          # Whisper auto-subtitles
├── hitpop-music/
│   └── SKILL.md          # BGM & audio mixing
├── hitpop-publish/
│   └── SKILL.md          # Multi-platform export
├── hitpop-json2video/
│   └── SKILL.md          # JSON2Video cloud API
├── hitpop-creatomate/
│   └── SKILL.md          # Creatomate template automation
└── hitpop-twick/
    └── SKILL.md          # React video editor SDK
├── hitpop-comfyui/
│   └── SKILL.md          # Local ComfyUI open-source models
└── hitpop-pipeline/
    └── SKILL.md          # JSON workflow orchestration
```

## Contributing

1. Fork this repo
2. Create a new skill in `hitpop-<name>/SKILL.md`
3. Follow the [OpenClaw skill format](https://docs.openclaw.ai/tools/skills)
4. Submit a PR

## Roadmap

- [x] `hitpop-twick` — React video editor SDK integration
- [x] `hitpop-json2video` — JSON2Video API integration
- [x] `hitpop-creatomate` — Creatomate template rendering
- [x] `hitpop-comfyui` — Local open-source model generation
- [x] `hitpop-pipeline` — JSON workflow orchestration
- [ ] `hitpop-translate` — Multi-language video localization
- [ ] `hitpop-avatar` — AI talking head / digital human
- [ ] Visual pipeline editor (canvas UI)
- [ ] Publish all skills to ClawHub

## License

[MIT](LICENSE)

---

Built by [Hitpop](https://github.com/hitpop) · Video AI Infrastructure for Developers
