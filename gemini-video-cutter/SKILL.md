---
name: gemini-video-cutter
description: Professional video editing pipeline that turns long, slow footage into fast-paced summary edits using Gemini in interactive 20-minute user-gated chunks. Strictly enforces syncopated event-driven pacing, anti-metronome laws (banning uniform loops), 4–7s varied keeps, 2–20s dynamic cuts (big skips 15–20s for tedious labor, tight skips 2–4s for detail work), 50–60% footage retention, 360p FFmpeg proxies, and chronological cut tables.
---

# Gemini Video Cutter (Professional Dynamic Syncopated Editor)

An automated professional video editing skill designed to transform raw, long, or slow footage (bushcraft, construction, carpentry, restoration, outdoor survival, gaming, vlogs) into a tight, cinematic, fast-paced summary edit while preserving 100% of the narrative and build progression from start to finish.

---

## 🎬 Master Video Editor Doctrine: Dynamic Syncopation & Event-Driven Pacing

### 🚨 1. The Anti-Metronome Law (Zero Arithmetic Loops)
- **STRICT PROHIBITION**: You are strictly forbidden from cutting on a robotic stopwatch timer or repeating uniform numbers (e.g. falling into a repetitive `6s keep, 7s cut` loop).
- **Consecutive Contrast Rule**: Two consecutive clips must **NEVER** have identical keep or cut durations. The edit must feel alive, natural, and syncopated.
- **Event-Driven Cutting**: Cuts must trigger on **physical motion and scene changes** (an axe strike, a log setting into a notch, a tool swap, a camera angle change), NOT on arithmetic clock ticks.

### ⏱️ 2. Keep Clips: Dynamic 4 to 7 Seconds
Vary the clip duration intentionally to fit the energy of the action:
- **Snappy Impact Beat (4.0s)**: High-speed action, quick blade chops, single stone placements, fast transitions.
- **Complete Physical Loop (5.0s – 6.0s)**: Full mechanical movements (lifting and setting a post, driving a wooden stake, mortise scribing).
- **Milestone Reveal / Scenic Beat (7.0s)**: Major structural completions, wide scenic views, embers bursting into flame, sunset shelter reveals.

### ✂️ 3. Cuts / Skips: Dynamic 2 to 20 Seconds
Rhythm comes from contrasting short and long jumps across time:
- **Big Skips for Tedious Labor (15s – 20s cuts)**:
  * Repetitive heavy digging, chopping 20 axe strokes into a tree, prolonged sawing of a massive trunk, clearing mounds of dirt.
  * Show the start of the action (4–5s), skip 15–20s of grueling repetition, show the mid-progress (4–5s), skip 15–20s, then show the finished result (5–6s).
- **Tight Skips for Rapid Detail Work (2s – 4s cuts)**:
  * Fine wood carving, feathering birch bark, striking ferro rods, pouring tea into a kuksa, lighting a lantern wick.
  * Keep jumps micro-short (2–4s) so the audience never loses the intricate craft technique.
- **Medium Transitions (5s – 12s cuts)**:
  * Walking between trees, carrying logs to the build site, changing camera positions.
- **Overall Retention Target**:
  * **Cut down**: **30% to 40%** of raw footage.
  * **Keep**: **50% to 60%** of the original video duration.

### 🗺️ 4. Story Arc & Narrative Continuity
- **Zero Milestone Skipping**: Never skip critical engineering, survival, or narrative steps.
- The viewer must see the entire build arc logically flow from ground clearing to completion:
  * *Site selection -> Trench excavation -> Foundation posts -> Wall timber layering -> Ridge beams -> Rafters -> Moss & soil roof -> Reindeer fur bed -> Hearth fire & tea -> Night rest -> Wildlife / morning departures*.

### 🔀 5. Optional Creative Rearrangement Suggestions
- If logical and does not damage the authentic progression or physical cause-and-effect, suggest 2–3 creative rearrangements (e.g., a cold-open hook using the evening campfire or badger sighting at 00:00:00 before jumping back to digging).

---

## 🚨 Mandatory Execution Contract: 20-Minute Chunks & 360p Proxies

> [!IMPORTANT]
> **CRITICAL EXECUTION CONTRACT FOR AI AGENTS:**
> 1. **Batching Law (Interactive 20-Minute Gating)**:
>    - Process the video in **20-minute user-gated chunks** (`Chunk 1: 00:00–20:00`, `Chunk 2: 20:00–40:00`, etc.).
>    - Run **one chunk per Gemini API call**.
>    - **DO NOT** process the entire 40+ minute video in one go.
>    - **STOP** immediately after generating Chunk 1's cut table and present it to the user for review.
>    - Only proceed to Chunk 2 when the user explicitly commands: *"Approved, do Chunk 2"*.
> 2. **360p Proxy Law**:
>    - Always downscale source video to a lightweight `360p` proxy (`scale=-2:360`, CRF 28, AAC 64k) via FFmpeg before calling Gemini Files API.
> 3. **Model Tier**:
>    - Primary Vision Model: `models/gemini-2.5-flash` or `models/gemini-2.5-pro` (if available to account).

---

## 📊 Required Output Format: Chronological Cut Table

Every chunk must output this exact Markdown table structure:

### Pass 1: Chunk 1 ([00:00:00] to [00:20:00])
- **Raw Duration**: 20:00 (1,200s)
- **Total KEEP Duration**: ~11:00 (~660s / 55%)
- **Total CUT Duration**: ~09:00 (~540s / 45%)
- **Average Keep Clip Length**: Dynamic (~4.0s – 7.0s varied)
- **Average Cut Length**: Dynamic (~2.0s – 20.0s varied)

| Clip # | Keep Time (Start - End) | Clip Length (4-7s) | Cut / Skipped Time (2-20s) | What happens in the kept clip |
| :---: | :---: | :---: | :---: | :--- |
| **01** | `[00:00:00] - [00:00:04]` | 4.0s | `[00:00:04] - [00:00:18] (14.0s)` | Walking into the hollow and driving surveyor stake into soil. |
| **02** | `[00:00:18] - [00:00:24]` | 6.0s | `[00:00:24] - [00:00:27] (3.0s)` | Digging first shovel strike and peeling turf layer back. |
| **03** | `[00:00:27] - [00:00:31]` | 4.0s | `[00:00:31] - [00:00:49] (18.0s)` | Shoveling loose clay (big 18s cut skips repetitive dirt throwing). |

#### 💡 Creative Rearrangement Suggestions (Optional):
- **Cold Open Hook**: Place a 5-second teaser of the illuminated night shelter at `00:00:00` before showing the raw digging.
