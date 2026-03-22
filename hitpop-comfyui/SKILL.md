---
name: hitpop-comfyui
description: "Run open-source AI models locally via ComfyUI. Text-to-image (Flux, SDXL), text-to-video (Wan2.1, HunyuanVideo), image-to-video, and more. Free, no API key needed — just a local GPU. Submit workflow JSON to ComfyUI's REST API and poll for results."
version: 0.1.0
metadata:
  openclaw:
    emoji: "🖥️"
    requires:
      bins:
        - curl
        - jq
        - python3
---

# Hitpop ComfyUI — Local Open-Source AI Generation (Free)

Run open-source AI models locally via ComfyUI. No API keys, no cost per generation — just your GPU.

## When to Use

- You have a local GPU (RTX 3090/4090 or better)
- You want **free** generation with no per-image/video cost
- You need to run **open-source models** (Flux, SDXL, Wan2.1, HunyuanVideo)
- You want full control over model weights and parameters
- Privacy matters — nothing leaves your machine

**No GPU?** Use `hitpop-gen-video` (Vidu Q2) and `hitpop-gen-image` (Seedream) instead — they run on Zhipu's cloud.

## Setup

### 1. Install ComfyUI

```bash
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
pip install -r requirements.txt

# Start the server
python main.py --listen 0.0.0.0 --port 8188
```

ComfyUI runs at `http://127.0.0.1:8188`.

### 2. Install Models

Download models into `ComfyUI/models/checkpoints/`:

| Model | Type | VRAM | Download |
|---|---|---|---|
| Flux.1-dev | Text→Image | ~12GB | huggingface.co/black-forest-labs/FLUX.1-dev |
| SDXL 1.0 | Text→Image | ~8GB | huggingface.co/stabilityai/stable-diffusion-xl-base-1.0 |
| Wan2.1-A14B | Text→Video | ~24GB | huggingface.co/Wan-AI/Wan2.1-T2V-A14B |
| HunyuanVideo | Text→Video | ~24GB | huggingface.co/tencent/HunyuanVideo |

### 3. Install Video Nodes (for video generation)

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/kijai/ComfyUI-WanVideoWrapper.git
git clone https://github.com/kijai/ComfyUI-HunyuanVideoWrapper.git
# Restart ComfyUI after installing nodes
```

## API Reference

ComfyUI exposes a REST API at `http://127.0.0.1:8188`:

| Endpoint | Method | Purpose |
|---|---|---|
| `/prompt` | POST | Submit a workflow for execution |
| `/history/{prompt_id}` | GET | Get results for a completed job |
| `/queue` | GET | Check queue status |
| `/view?filename=...` | GET | Download output image/video |
| `/upload/image` | POST | Upload input image |
| `/object_info` | GET | List all available nodes |

## Usage

### Submit a Text-to-Image Workflow

First, create your workflow in ComfyUI's UI, then export it as API format (Settings → Dev Mode → Save API Format). Then submit via curl:

```bash
COMFYUI_URL="http://127.0.0.1:8188"

# Submit workflow
RESPONSE=$(curl -s -X POST "$COMFYUI_URL/prompt" \
  -H 'Content-Type: application/json' \
  -d "{\"prompt\": $(cat workflow_api.json)}")

PROMPT_ID=$(echo "$RESPONSE" | jq -r '.prompt_id')
echo "Submitted: $PROMPT_ID"
```

### Poll for Results

```bash
# Poll until complete
while true; do
  HISTORY=$(curl -s "$COMFYUI_URL/history/$PROMPT_ID")
  if echo "$HISTORY" | jq -e ".[\"$PROMPT_ID\"]" > /dev/null 2>&1; then
    echo "Done!"
    # Extract output filename
    FILENAME=$(echo "$HISTORY" | jq -r ".[\"$PROMPT_ID\"].outputs | to_entries[0].value.images[0].filename")
    echo "Output: $FILENAME"
    # Download the result
    curl -s "$COMFYUI_URL/view?filename=$FILENAME&type=output" -o output.png
    break
  fi
  sleep 2
done
```

### Python Helper Script

```python
import json, requests, time, uuid

COMFYUI_URL = "http://127.0.0.1:8188"

def submit_workflow(workflow_json, modifications=None):
    """Submit a workflow and return prompt_id."""
    if modifications:
        for node_id, changes in modifications.items():
            if node_id in workflow_json:
                workflow_json[node_id]["inputs"].update(changes)

    # Randomize seed to get different results
    for node_id, node in workflow_json.items():
        if "inputs" in node and "seed" in node["inputs"]:
            node["inputs"]["seed"] = int(uuid.uuid4().int % 2**32)

    resp = requests.post(f"{COMFYUI_URL}/prompt",
                         json={"prompt": workflow_json})
    return resp.json().get("prompt_id")

def wait_for_result(prompt_id, timeout=300):
    """Poll until the workflow completes."""
    start = time.time()
    while time.time() - start < timeout:
        resp = requests.get(f"{COMFYUI_URL}/history/{prompt_id}")
        history = resp.json()
        if prompt_id in history:
            return history[prompt_id]
        time.sleep(2)
    raise TimeoutError(f"Workflow {prompt_id} did not complete in {timeout}s")

def download_output(result, output_dir="."):
    """Download all output files from a completed workflow."""
    files = []
    for node_id, output in result.get("outputs", {}).items():
        for img in output.get("images", []):
            filename = img["filename"]
            subfolder = img.get("subfolder", "")
            file_type = img.get("type", "output")
            url = f"{COMFYUI_URL}/view?filename={filename}&subfolder={subfolder}&type={file_type}"
            local_path = f"{output_dir}/{filename}"
            resp = requests.get(url)
            with open(local_path, "wb") as f:
                f.write(resp.content)
            files.append(local_path)
        for vid in output.get("gifs", output.get("videos", [])):
            filename = vid["filename"]
            subfolder = vid.get("subfolder", "")
            url = f"{COMFYUI_URL}/view?filename={filename}&subfolder={subfolder}&type=output"
            local_path = f"{output_dir}/{filename}"
            resp = requests.get(url)
            with open(local_path, "wb") as f:
                f.write(resp.content)
            files.append(local_path)
    return files

# Example usage:
# workflow = json.load(open("workflow_api.json"))
# pid = submit_workflow(workflow, {"6": {"text": "a cat in space"}})
# result = wait_for_result(pid)
# files = download_output(result)
```

### Upload Input Image (for img2img / img2video)

```bash
curl -s -X POST "$COMFYUI_URL/upload/image" \
  -F "image=@input_photo.jpg" \
  -F "type=input" | jq .
```

The returned filename can be used in workflow nodes that accept image input.

## Pre-Built Workflow Templates

Save these as `workflow_api.json` files and submit via the API. Find workflows at:

- **ComfyUI Examples**: github.com/comfyanonymous/ComfyUI_examples
- **OpenArt Workflows**: openart.ai/workflows
- **CivitAI Workflows**: civitai.com (filter by ComfyUI)
- **ComfyICU**: comfy.icu

Common workflow types:
- `txt2img_flux.json` — Flux text to image
- `txt2img_sdxl.json` — SDXL text to image
- `img2img.json` — Image to image with ControlNet
- `txt2vid_wan.json` — Wan2.1 text to video
- `img2vid_wan.json` — Wan2.1 image to video

## ComfyUI vs Cloud API (hitpop-gen-video / hitpop-gen-image)

| Feature | ComfyUI (local) | Zhipu Cloud API |
|---|---|---|
| Cost | Free (electricity only) | ¥0.20-0.40 per generation |
| Speed | Depends on your GPU | Fast (server-grade GPUs) |
| GPU needed | Yes (8GB+ VRAM) | No |
| Models | Any open-source model | Vidu Q2, Seedream only |
| Privacy | 100% local | Data sent to cloud |
| Customization | Full (LoRA, ControlNet, custom nodes) | Limited to API params |
| Setup | Complex (install models, nodes) | Just an API key |

## Tips
- Always randomize the seed before each submission to get unique outputs
- Export workflows in **API format** (not regular format) — regular format won't work with the API
- Use `--lowvram` flag when starting ComfyUI if you have limited GPU memory
- For video generation, you typically need 24GB+ VRAM (RTX 4090 or A100)
- ComfyUI caches models in memory — first run is slow, subsequent runs are fast
