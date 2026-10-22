---
name: gemini-video-cutter
description: Professional video editing pipeline that turns long, slow footage into fast-paced summary edits using Gemini 2.5 Pro in interactive 20-minute user-gated chunks. Strictly enforces 4–7s keeps, 2–20s skips (mostly 5–10s), 50–60% footage retention, 360p FFmpeg proxies, and chronological cut tables with optional pacing rearrangement suggestions.
---

# Gemini Video Cutter (Professional Dynamic Summary Editor)

An automated professional video editing skill designed to transform raw, long, or slow footage (bushcraft, construction, carpentry, restoration, outdoor survival, gaming, vlogs) into a tight, cinematic, fast-paced summary edit while preserving 100% of the narrative and build progression from start to finish.

---

## 🎬 Core Editing Doctrine & Strict Cutting Rules

### 1. ⏱️ Keep Clips: Strictly 4 to 7 Seconds
- **Hard Boundaries**: Every kept clip MUST be continuous and between **4 and 7 seconds long** (minimum: 4.0s, maximum: 7.0s).
- **Zero Linger Law**: Never exceed 7.0 seconds on any single angle or action. If an action takes longer, cut intermediate repetitive motion and take two distinct 4–6s beats.
- **Dynamic Action Focus**: Every kept clip must contain a decisive visual beat (e.g., blade striking wood, notch sliding together, mortar placed, stone lifted, ember catching fire, mud smoothed).

### 2. ✂️ Cuts & Skips: 2 to 20 Seconds (Target 5 to 10s)
- **Cut Duration Range**: Every cut between kept clips must skip between **2 and 20 seconds**.
- **Preferred Cadence**: Whenever possible, keep cuts short (between **5 and 10 seconds**). This maintains natural scene rhythm without causing jarring visual teleports.
- **What to Cut Out**:
  * Boring, repetitive actions (e.g., swinging an axe 15 times in a row, repetitive sawing strokes, scraping dirt for 30s).
  * Static shots with no visible change in progress.
  * Awkward pauses, camera readjustments, and prolonged hesitation.
- **Overall Retention Target**: 
  * **Cut down**: **30% to 40%** of the raw footage.
  * **Keep**: **50% to 60%** of the original video (e.g., a 20-minute chunk becomes a tight ~10 to 12-minute edit).

### 3. 🗺️ Story Arc & Narrative Continuity
- **Zero Milestone Skipping**: You are strictly forbidden from skipping key architectural, survival, or narrative steps.
- The viewer must see the entire build arc logically flow from ground clearing to completion:
  * *Site selection -> Excavation -> Foundation posts -> Wall framing -> Ridge beam -> Rafters -> Roofing -> Chinking/Insulation -> Interior/Furniture -> Fireplace/Bed -> Final Tour*.

### 4. 🔀 Optional Pacing Rearrangements
- If logical and does not damage the authentic progression or physical cause-and-effect, you may suggest re-ordering or cross-cutting segments to make the edit even more dynamic and distinct from the original video (e.g., tease a 5s flash of the completed cozy fireplace at the very start as a cold open before jumping back to digging).
- These must always be presented as **Optional Suggestions** immediately beneath the cut table.

---

## 🚨 Mandatory Execution Contract: 20-Minute Chunks & 360p Proxies

> [!IMPORTANT]
> **CRITICAL EXECUTION CONTRACT FOR AI AGENTS:**
> 1. **Batching Law (Interactive 20-Minute Gating)**:
>    - Process the video in **20-minute user-gated chunks** (`Chunk 1: 00:00–20:00`, `Chunk 2: 20:00–40:00`, etc.).
>    - Run **one chunk per Gemini 2.5 Pro API call**.
>    - **DO NOT** process the entire 40+ minute video in one go.
>    - **STOP** immediately after generating Chunk 1's cut table and present it to the user for review.
>    - Only proceed to Chunk 2 when the user explicitly commands: *"Approved, do Chunk 2"*.
> 2. **360p Proxy Law**:
>    - **NEVER** upload raw 1080p or 4K files directly to the Gemini API.
>    - Always downscale source video to a lightweight `360p` proxy (`scale=-2:360`, CRF 28, AAC 64k) via FFmpeg before calling Gemini Files API.
> 3. **Model Tier Enforcement**:
>    - Use **`models/gemini-2.5-pro`** (Primary) for fine-grained mechanical comprehension and sub-second cut boundary accuracy.
>    - Alternate: `models/gemini-2.5-flash` for high-throughput draft passes.

---

## 📊 Required Output Format: Chronological Cut Table

Every chunk must output this exact Markdown table structure:

### Pass 1: Chunk 1 ([00:00:00] to [00:20:00])
- **Raw Duration**: 20:00 (1,200s)
- **Total KEEP Duration**: ~11:30 (~690s / 57.5%)
- **Total CUT Duration**: ~08:30 (~510s / 42.5%)
- **Average Keep Clip Length**: ~5.2s (Range: 4.0s – 7.0s)
- **Average Cut Length**: ~7.8s (Range: 2.0s – 20.0s)

| Clip # | Keep Time (Start - End) | Clip Length (4-7s) | Cut / Skipped Time (2-20s) | What happens in the kept clip |
| :---: | :---: | :---: | :---: | :--- |
| **01** | `[00:00:00] - [00:00:05]` | 5.0s | `[00:00:05] - [00:00:10] (5.0s)` | Stepping through pine needles and discovering the deep natural hollow. |
| **02** | `[00:00:10] - [00:00:16]` | 6.0s | `[00:00:16] - [00:00:22] (6.0s)` | Driving the wooden surveyor stake into the clay soil to establish footing. |
| **03** | `[00:00:22] - [00:00:27]` | 5.0s | `[00:00:27] - [00:00:34] (7.0s)` | Powerful adze blow peeling thick spruce bark from the primary sill log. |

#### 💡 Creative Rearrangement Suggestions (Optional):
- **Cold Open Hook**: Take Clip #84 (lighting the stone fireplace at night, `[00:18:20]`) and place a 4-second tease at `00:00:00` before returning to the daytime excavation.

---

## 🛠️ Step-by-Step Automation Pipeline

### Step 1: Generate Fast 360p Proxy
Run FFmpeg to create an ultra-compact proxy for API ingestion:
```bash
ffmpeg -y -i input_video.mp4   -vf "scale=-2:360"   -c:v libx264 -crf 28 -preset veryfast   -c:a aac -b:a 64k   proxy_360p.mp4
```

### Step 2: Slice 20-Minute Chunk Proxy
```bash
# Chunk 1 (0 to 1200 seconds)
ffmpeg -y -ss 0 -t 1200 -i proxy_360p.mp4 -c copy chunk_0_360p.mp4

# Chunk 2 (1200 to 2400 seconds)
ffmpeg -y -ss 1200 -t 1200 -i proxy_360p.mp4 -c copy chunk_1_360p.mp4
```

### Step 3: Run Gemini 2.5 Pro Multimodal Script
```python
import os, json, time
import google.genai as genai
from google.genai import types

client = genai.Client()

# 1. Upload 360p chunk
print("[Upload] Uploading 360p chunk to Gemini Files API...")
chunk_file = client.files.upload(file="chunk_0_360p.mp4")

while chunk_file.state.name == "PROCESSING":
    time.sleep(2)
    chunk_file = client.files.get(name=chunk_file.name)

# 2. Multimodal Editor Prompt
prompt = """Act as a master professional video editor.
Your mission is to turn this 20-minute raw footage chunk (00:00:00 to 00:20:00) into a fast-paced, engaging summary edit while keeping the complete build and story arc intact from start to finish.

STRICT CUTTING RULES:
1. Keep Clips: Every clip you keep must be strictly 4 to 7 seconds long (never shorter than 4s, never longer than 7s).
2. Cuts / Skips: Every cut between kept clips must be between 2 and 20 seconds long. Keep cuts short whenever possible (typically 5 to 10 seconds) so the video does not feel over-chopped.
3. Cut out boring, repetitive parts (endless sawing, digging strokes, camera adjustments, static dead air).
4. Retention Target: Cut 30% to 40% of the raw footage, keeping 50% to 60% of the duration.
5. Story Arc: Do not skip important engineering steps or milestones.
6. Output format: Return a valid JSON list of clips, each with:
   - clipNumber (integer)
   - keepStart (MM:SS)
   - keepEnd (MM:SS)
   - clipLengthSeconds (float, 4.0-7.0)
   - cutStart (MM:SS)
   - cutEnd (MM:SS)
   - cutLengthSeconds (float, 2.0-20.0)
   - description (clear explanation of visual action kept)
   - reasonForCut (why the skipped portion was removed)
Include an optional 'rearrangementSuggestions' array at the root.
"""

response = client.models.generate_content(
    model="models/gemini-2.5-pro",
    contents=[chunk_file, prompt],
    config=types.GenerateContentConfig(
        response_mime_type="application/json"
    )
)

data = json.loads(response.text)
# Clean up file
client.files.delete(name=chunk_file.name)
with open("chunk_0_cuts.json", "w") as f:
    json.dump(data, f, indent=2)
```

### Step 4: Lossless Video Slicing & Concatenation (`cut_and_stitch.py`)
Once the user reviews and approves the cut table, execute the physical cuts on the full-resolution original video using FFmpeg:
```python
import subprocess, os, json

def execute_cuts(source_video, cuts_json, output_edited_video):
    with open(cuts_json) as f:
        data = json.load(f)
    
    clips = data.get("clips", data) if isinstance(data, dict) else data
    temp_dir = "temp_clips"
    os.makedirs(temp_dir, exist_ok=True)
    concat_list_path = os.path.join(temp_dir, "concat.txt")
    
    with open(concat_list_path, "w") as concat_file:
        for idx, clip in enumerate(clips):
            start = clip["keepStart"]
            dur = clip["clipLengthSeconds"]
            clip_path = os.path.join(temp_dir, f"clip_{idx:03d}.mp4")
            
            cmd = [
                "ffmpeg", "-y",
                "-ss", str(start),
                "-t", str(dur),
                "-i", source_video,
                "-c:v", "libx264", "-preset", "veryfast", "-crf", "18",
                "-c:a", "aac", "-b:a", "192k",
                clip_path
            ]
            subprocess.run(cmd, check=True)
            concat_file.write(f"file '{os.path.abspath(clip_path)}'\n")
    
    # Concatenate all clips into final video
    concat_cmd = [
        "ffmpeg", "-y",
        "-f", "concat", "-safe", "0",
        "-i", concat_list_path,
        "-c", "copy",
        output_edited_video
    ]
    subprocess.run(concat_cmd, check=True)
    print(f"[Success] Fast-paced summary edit generated: {output_edited_video}")
```
