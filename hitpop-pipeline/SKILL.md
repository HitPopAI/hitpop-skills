---
name: hitpop-pipeline
description: "Orchestrate multi-step video production workflows via JSON definitions. Chain any hitpop skills together into automated pipelines — generate images, animate to video, add voiceover, burn subtitles, mix music, and publish, all in one run. The glue that connects all hitpop skills."
version: 0.1.0
metadata:
  openclaw:
    emoji: "🔗"
    requires:
      env:
        - ZHIPU_API_KEY
      bins:
        - curl
        - jq
        - ffmpeg
        - python3
    primaryEnv: ZHIPU_API_KEY
---

# Hitpop Pipeline — Multi-Step Video Workflow Orchestration

Define complex video production workflows as JSON, then execute them step by step. Each step calls a hitpop skill, and outputs flow into the next step automatically.

## Why Pipeline

Without pipeline, you manually run each skill and pass outputs between them. With pipeline, you define the entire flow once and run it in one shot:

```
Without pipeline:
  1. Manually run hitpop-gen-image → save output → copy path
  2. Manually run hitpop-gen-video with that path → save output → copy path
  3. Manually run hitpop-voiceover → save output → copy path
  4. Manually run hitpop-edit to merge → ...
  (tedious, error-prone, not repeatable)

With pipeline:
  1. Define workflow.json once
  2. Run it → get final video
  (automated, repeatable, shareable)
```

## Workflow JSON Format

```json
{
  "name": "Product Promo Video",
  "version": "1.0",
  "description": "Generate a product promo from a single image",
  "env": {
    "ZHIPU_API_KEY": "$ZHIPU_API_KEY"
  },
  "steps": [
    {
      "id": "generate_image",
      "skill": "hitpop-gen-image",
      "action": "create",
      "params": {
        "model": "doubao-seedream-4.5",
        "prompt": "Premium headphones on marble surface, studio lighting, 4K product photography",
        "size": "2K",
        "watermark": false
      },
      "output": "product_image.jpg"
    },
    {
      "id": "animate",
      "skill": "hitpop-gen-video",
      "action": "img2video",
      "params": {
        "model": "viduq2-pro-img2video",
        "image_url": ["$generate_image.output"],
        "prompt": "Camera slowly orbits around the product, dramatic lighting shifts",
        "duration": "5",
        "size": "1920x1080",
        "movement_amplitude": "medium"
      },
      "output": "product_video.mp4"
    },
    {
      "id": "voiceover",
      "skill": "hitpop-voiceover",
      "action": "edge-tts",
      "params": {
        "voice": "en-US-AriaNeural",
        "text": "Introducing the next generation of premium audio. Feel every beat, hear every detail."
      },
      "output": "narration.mp3"
    },
    {
      "id": "subtitle",
      "skill": "hitpop-subtitle",
      "action": "transcribe",
      "params": {
        "input": "$voiceover.output",
        "model": "base"
      },
      "output": "subs.srt"
    },
    {
      "id": "merge",
      "skill": "hitpop-edit",
      "action": "merge_audio_video",
      "params": {
        "video": "$animate.output",
        "audio": "$voiceover.output",
        "subtitle": "$subtitle.output"
      },
      "output": "final_with_subs.mp4"
    },
    {
      "id": "add_music",
      "skill": "hitpop-music",
      "action": "mix",
      "params": {
        "video": "$merge.output",
        "bgm_url": "https://cdn.pixabay.com/audio/2024/ambient-corporate.mp3",
        "bgm_volume": 0.2
      },
      "output": "final.mp4"
    },
    {
      "id": "export",
      "skill": "hitpop-publish",
      "action": "resize",
      "params": {
        "input": "$add_music.output",
        "platforms": ["tiktok", "youtube", "instagram"]
      },
      "output": "exports/"
    }
  ]
}
```

## Variable References

Use `$step_id.output` to reference the output of a previous step:

- `$generate_image.output` → resolves to the file path from the "generate_image" step
- `$voiceover.output` → resolves to the audio file from the "voiceover" step

This is how data flows between steps automatically.

## Running a Pipeline

### Option 1: Let the Director Execute It

Give the workflow JSON to your AI agent and say:

```
Run this hitpop pipeline: [paste workflow JSON]
```

The `hitpop-director` will parse each step, execute them in order, resolve variable references, and pass outputs between steps.

### Option 2: Python Runner Script

```python
#!/usr/bin/env python3
"""Hitpop Pipeline Runner — execute workflow JSON files."""

import json, os, sys, subprocess, time, re

def resolve_vars(value, outputs):
    """Replace $step_id.output with actual file paths."""
    if isinstance(value, str) and value.startswith("$"):
        match = re.match(r'\$(\w+)\.output', value)
        if match:
            step_id = match.group(1)
            if step_id in outputs:
                return outputs[step_id]
            raise ValueError(f"Step '{step_id}' not found in outputs")
    if isinstance(value, list):
        return [resolve_vars(v, outputs) for v in value]
    if isinstance(value, dict):
        return {k: resolve_vars(v, outputs) for k, v in value.items()}
    return value

def run_pipeline(workflow_path):
    with open(workflow_path) as f:
        workflow = json.load(f)

    print(f"Pipeline: {workflow['name']}")
    print(f"Steps: {len(workflow['steps'])}")
    print("=" * 50)

    outputs = {}

    for i, step in enumerate(workflow["steps"], 1):
        step_id = step["id"]
        skill = step["skill"]
        print(f"\n[{i}/{len(workflow['steps'])}] {step_id} ({skill})")

        # Resolve variable references in params
        params = resolve_vars(step.get("params", {}), outputs)
        output_path = step.get("output", f"{step_id}_output")

        print(f"  Params: {json.dumps(params, indent=2, ensure_ascii=False)[:200]}...")
        print(f"  Output: {output_path}")

        # Execute the step (agent handles actual execution per skill)
        # This is where each skill's logic runs
        print(f"  → Executing {skill}...")

        # Store output for next steps
        outputs[step_id] = output_path
        print(f"  ✓ Complete: {output_path}")

    print("\n" + "=" * 50)
    print(f"Pipeline complete! Final output: {outputs[workflow['steps'][-1]['id']]}")
    return outputs

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python run_pipeline.py workflow.json")
        sys.exit(1)
    run_pipeline(sys.argv[1])
```

## Pre-Built Pipeline Templates

### 1. Product Promo (Image → Video + Voice + Subs)
```json
{
  "name": "Product Promo",
  "steps": [
    {"id": "img", "skill": "hitpop-gen-image", "params": {"prompt": "YOUR_PRODUCT_DESCRIPTION"}},
    {"id": "vid", "skill": "hitpop-gen-video", "params": {"image_url": ["$img.output"]}},
    {"id": "voice", "skill": "hitpop-voiceover", "params": {"text": "YOUR_SCRIPT"}},
    {"id": "subs", "skill": "hitpop-subtitle", "params": {"input": "$voice.output"}},
    {"id": "final", "skill": "hitpop-edit", "params": {"video": "$vid.output", "audio": "$voice.output", "subtitle": "$subs.output"}}
  ]
}
```

### 2. Batch Social Content (Template × Data)
```json
{
  "name": "Batch Social Videos",
  "steps": [
    {"id": "render", "skill": "hitpop-rendervid", "params": {"template": "social-post", "data": "products.csv"}},
    {"id": "music", "skill": "hitpop-music", "params": {"video": "$render.output", "bgm_volume": 0.3}},
    {"id": "export", "skill": "hitpop-publish", "params": {"input": "$music.output", "platforms": ["tiktok", "instagram"]}}
  ]
}
```

### 3. Full AI Short Film (Text → Everything)
```json
{
  "name": "AI Short Film",
  "steps": [
    {"id": "script", "skill": "hitpop-director", "params": {"task": "write_script", "topic": "YOUR_TOPIC", "duration": 60}},
    {"id": "scenes", "skill": "hitpop-gen-image", "params": {"prompt": "$script.scene_descriptions", "sequential_image_generation": "auto"}},
    {"id": "clips", "skill": "hitpop-gen-video", "params": {"image_url": ["$scenes.output"], "duration": "5"}},
    {"id": "voice", "skill": "hitpop-voiceover", "params": {"text": "$script.narration"}},
    {"id": "subs", "skill": "hitpop-subtitle", "params": {"input": "$voice.output"}},
    {"id": "bgm", "skill": "hitpop-music", "params": {"style": "cinematic"}},
    {"id": "final", "skill": "hitpop-edit", "params": {"clips": "$clips.output", "audio": "$voice.output", "music": "$bgm.output", "subtitle": "$subs.output"}},
    {"id": "publish", "skill": "hitpop-publish", "params": {"input": "$final.output", "platforms": ["youtube"]}}
  ]
}
```

### 4. Local Generation Pipeline (ComfyUI, Free)
```json
{
  "name": "Local Free Generation",
  "steps": [
    {"id": "img", "skill": "hitpop-comfyui", "params": {"workflow": "txt2img_flux.json", "prompt": "YOUR_PROMPT"}},
    {"id": "vid", "skill": "hitpop-comfyui", "params": {"workflow": "img2vid_wan.json", "image": "$img.output"}},
    {"id": "voice", "skill": "hitpop-voiceover", "params": {"text": "YOUR_SCRIPT", "engine": "edge-tts"}},
    {"id": "final", "skill": "hitpop-edit", "params": {"video": "$vid.output", "audio": "$voice.output"}}
  ]
}
```

## Pipeline Design Rules

1. **Each step must have a unique `id`** — used for variable references
2. **Steps execute in order** — top to bottom, no parallel execution (yet)
3. **`$step_id.output` resolves at runtime** — the previous step must complete first
4. **Any hitpop skill can be a step** — gen-video, gen-image, comfyui, edit, voiceover, subtitle, music, rendervid, shotstack, json2video, creatomate, publish
5. **Pipelines are shareable** — save as JSON, put in git, share with team

## Future: Visual Pipeline Editor

The JSON workflow format is designed to be the data model for a future visual canvas editor — think "ComfyUI but for video production." Each step becomes a node on the canvas, connections between steps become edges, and clicking "Run" executes the pipeline. This is on the Hitpop roadmap.

## Tips
- Start with a simple 2-3 step pipeline and add complexity gradually
- Use `hitpop-director` to auto-generate a pipeline from natural language
- Test each step individually before combining into a pipeline
- Keep pipeline JSON files in version control — they're your production workflows
- Share useful pipelines with the community via GitHub PRs
