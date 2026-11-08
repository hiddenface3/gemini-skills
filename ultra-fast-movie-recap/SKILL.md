---
name: ultra-fast-movie-recap
description: "Ultra-fast automated movie recap pipeline from a YouTube URL. Extracts the source video's timestamped transcript/subtitles, writes a concise voiceover script mapped to source timestamp anchors, generates Edge TTS audio, measures exact spoken sentence durations with Groq Whisper, slices matching source clips aligned to natural reading times, stitches, muxes voiceover, crops for 9:16 TikTok vertical with 384px black bands, and mixes background music."
argument-hint: "[YouTube URL] [optional voice preference]"
user-invocable: true
disable-model-invocation: false
---

# Ultra Fast Movie Recap Skill

Create a fast-paced, high-retention movie recap directly from a YouTube URL without needing manual visual breakdowns or heavy video AI uploads.

## Core Architecture

Instead of receiving a pre-written voiceover script and cut table, or uploading heavy video proxies to multimodal LLMs:
1. **Zero Video Watching / Zero Visual AI**: Do NOT call Gemini, Claude, or any visual AI to watch or analyze video frames. The original narrator is already describing the actions on screen in real time. Their timestamped words ARE the visual ground truth.
2. **Source Transcript Extraction**: Fetch the YouTube video's timestamped subtitles/transcript (or transcribe source audio with Groq Whisper if subtitles are missing).
3. **Context-Mapped Script Writing**: Because the original narrator's commentary directly aligns with what appears on screen at those exact seconds, mapping our recap script to the source transcript timestamps ensures every visual cut matches the narrative beat.
4. **Exact Spoken-Duration Matching**: Every individual sentence of the script is paired with a specific video clip whose duration matches the natural reading/spoken time of that sentence (e.g., a 15-word sentence takes ~7 seconds to speak, so it is paired with a 7-second clip window).
5. **Autonomous Video Production**: Generate voiceover with Edge TTS, obtain exact word boundaries via Groq Whisper, build a contiguous 0-gap cut table, slice silent clips using NVENC on the GTX 750 Ti, mux audio, generate 9:16 vertical TikTok exports with black bands, and mix background music.

---

## GTX 750 Ti Encoding and Performance Rules

The host GPU is an NVIDIA GeForce GTX 750 Ti. It supports H.264 NVENC only:
- Encode video with `-c:v h264_nvenc`; do not use HEVC or AV1 NVENC.
- Always run FFmpeg at low priority with `nice -n 10` and limit processing with `-filter_threads 1 -filter_complex_threads 1 -threads 2`.
- Slicing parameters: `-preset p2 -rc vbr -cq 23 -b:v 0 -pix_fmt yuv420p`.
- For letterboxed sources (e.g. 1920x960), scale and pad into a standard 1920x1080 canvas:
  `-vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:black"`
- For audio-only changes, use `-c:v copy` and encode only the mixed audio.

---

## Required Inputs & Environment

- **YouTube URL**: URL of the movie recap / commentary video.
- **Voice Preference**: Default to `en-US-GuyNeural` unless specified.
- **Environment**: `GROQ_API_KEY` set in environment for Whisper transcription.
- **Tools**: `yt-dlp`, `ffmpeg`, `ffprobe`, `edge-tts`, `python3`.

---

## Standard Project Layout

Create a dated project folder per movie run:
```text
YYYY-MM-DD_video_X/
├── 01_source_video/
│   ├── source_video.mp4
│   └── source_subtitles.*
├── 02_audio_voiceover/
│   ├── combined_voiceover.mp3
│   └── combined_voiceover.wav
├── 03_transcripts_and_cuttables/
│   ├── source_timestamped_transcript.txt
│   ├── voiceover_script.txt
│   ├── script_source_mapping.json
│   ├── provided_cut_table.txt
│   ├── word_transcript.json
│   ├── word_transcript.txt
│   ├── transcript_with_timestamps.txt
│   └── synced_cut_table.txt
├── 04_video_clips/
│   ├── clip_01.mp4 ... clip_XX.mp4
│   ├── concat.txt
│   └── stitched_silent.mp4
└── 06_final_exports/
    ├── final_movie_recap_synced.mp4
    └── final_movie_recap_tiktok_vertical.mp4
```

---

## Workflow Steps

### Phase 1: Fetch Timestamped Transcript from Source Video

1. Try downloading English subtitles or auto-generated subtitles first:
```bash
yt-dlp --write-auto-sub --write-sub --sub-lang "en.*,en" --sub-format "vtt/json3/srt" --skip-download \
  -o "01_source_video/source_subtitles" "<YOUTUBE_URL>"
```
2. If subtitle files exist, parse them into a clean timestamped transcript file:
   `03_transcripts_and_cuttables/source_timestamped_transcript.txt`
   Format:
   `[MM:SS - MM:SS] Narrator text line`

Helper python snippet to parse `.vtt`:
```python
import re, glob
vtt_files = glob.glob("01_source_video/source_subtitles*.vtt")
if vtt_files:
    lines = open(vtt_files[0]).read().splitlines()
    out = []
    for i, line in enumerate(lines):
        m = re.match(r"(\d{2}:\d{2}:\d{2}\.\d{3}|\d{2}:\d{2}\.\d{3})\s*-->\s*(\d{2}:\d{2}:\d{2}\.\d{3}|\d{2}:\d{2}\.\d{3})", line)
        if m and i + 1 < len(lines):
            text = re.sub(r"<[^>]+>", "", lines[i+1]).strip()
            if text:
                out.append(f"[{m.group(1)[:5]} - {m.group(2)[:5]}] {text}")
    open("03_transcripts_and_cuttables/source_timestamped_transcript.txt", "w").write("\n".join(out))
```
3. Fallback: If no subtitles exist on YouTube, download audio only (`-f 140`) and transcribe with Groq Whisper verbose JSON to extract source timestamps.

### Phase 2: Author Voiceover Script with Source Timestamp Anchors

Analyze `source_timestamped_transcript.txt` and write a punchy 2–2.5 minute recap script (~300–400 words, ~15–22 sentences):
- **Core Principle**: Because the original narrator's commentary directly aligns with what appears on screen at those exact seconds, map every sentence of the new script to the exact timestamp in the source video where that event occurred.
- For each sentence, identify:
  - `sentence`: The concise recap sentence text.
  - `source_timestamp`: The exact source start timestamp (e.g., `01:44`) from the source transcript where this scene is shown.
  - `visual_description`: Short 1-line note of what is happening on screen.
- Save:
  - `03_transcripts_and_cuttables/voiceover_script.txt` (clean text paragraphs)
  - `03_transcripts_and_cuttables/provided_cut_table.txt` (Sentence | Source Range | Visual Description)

### Phase 3: Generate Voiceover via Edge TTS

Generate narration audio using the requested voice (default `en-US-GuyNeural`):
```bash
edge-tts --voice "en-US-GuyNeural" \
  --file "03_transcripts_and_cuttables/voiceover_script.txt" \
  --write-media "02_audio_voiceover/combined_voiceover.mp3"

ffmpeg -y -i "02_audio_voiceover/combined_voiceover.mp3" \
  "02_audio_voiceover/combined_voiceover.wav"
```

### Phase 4: Exact Spoken-Duration Transcribe via Groq Whisper

Transcribe the generated WAV to obtain word-level timestamps:
```bash
curl https://api.groq.com/openai/v1/audio/transcriptions \
  -H "Authorization: Bearer $GROQ_API_KEY" \
  -F file=@02_audio_voiceover/combined_voiceover.wav \
  -F model=whisper-large-v3 \
  -F response_format=verbose_json \
  -F 'timestamp_granularities[]=word' \
  > 03_transcripts_and_cuttables/word_transcript.json
```

### Phase 5: Build Synced Contiguous Cut Table

Run `build_timeline.py` to match the exact spoken sentence durations:
1. Sentence 1 starts at visual timeline `00:00.000`.
2. Each visual interval extends through the start of the next sentence, covering natural narration pauses so that zero gaps or black frames exist.
3. Final visual interval ends exactly at total WAV duration.
4. Each row specifies:
   - `Visual timeline`: `[v_start – v_end]`
   - `Visual duration`: `v_end - v_start`
   - `Source start`: `src_start` anchored to the matching source transcript moment.
   - Slicing will take `v_end - v_start` seconds starting at `src_start` from the 1080p source video.

Complete reference `build_timeline.py`:
```python
import json, re
from difflib import SequenceMatcher
from pathlib import Path

root = Path(".")
data = json.loads((root / "03_transcripts_and_cuttables/word_transcript.json").read_text())
provided = []
for line in (root / "03_transcripts_and_cuttables/provided_cut_table.txt").read_text().splitlines()[1:]:
    if not line.strip():
        continue
    sentence, source, description = line.split(" | ", 2)
    provided.append((sentence.strip(), source.strip(), description.strip()))
sentences = [sentence for sentence, _, _ in provided]

def normalize(value):
    return re.sub(r"[^a-z0-9]+", "", value.lower())

words = data["words"]
(root / "03_transcripts_and_cuttables/word_transcript.txt").write_text(
    "\n".join(f"{w['start']:.3f} - {w['end']:.3f}: {w['word']}" for w in words) + "\n"
)

starts, position = [], 0
for s_idx, sentence in enumerate(sentences):
    target = [normalize(x) for x in re.findall(r"[A-Za-z0-9]+(?:['’-][A-Za-z0-9]+)?", sentence)]
    found, best_score = None, 0.0
    for index in range(position, len(words)):
        score = 0.0
        for offset, token in enumerate(target[:4]):
            if index + offset >= len(words):
                break
            candidate = normalize(words[index + offset]["word"])
            score += max(SequenceMatcher(None, candidate, token).ratio(), 1.0 if candidate == token else 0.0)
        score /= min(4, len(target))
        if score > best_score:
            best_score, found = score, index
    if best_score < 0.55:
        found = None
    if found is None:
        raise SystemExit(f"Could not locate sentence {s_idx + 1}: {sentence}")
    starts.append(found)
    position = found + 1

timings = []
for index, start_index in enumerate(starts):
    end_index = starts[index + 1] if index + 1 < len(starts) else len(words)
    timings.append((float(words[start_index]["start"]), float(words[end_index - 1]["end"])))

(root / "03_transcripts_and_cuttables/transcript_with_timestamps.txt").write_text(
    "\n".join(f"{i:02d}. [{s:.3f} - {e:.3f}] {sent}" for i, ((s, e), sent) in enumerate(zip(timings, sentences), 1)) + "\n"
)

total_duration = float(data["duration"])
lines = ["# | Script line | Visual timeline | Audio range | Provided Video Range | Visual description"]
for index, ((sentence, source, description), (a_start, a_end)) in enumerate(zip(provided, timings)):
    v_start = 0.0 if index == 0 else timings[index][0]
    v_end = timings[index + 1][0] if index + 1 < len(timings) else total_duration
    lines.append(
        f"{index + 1} | {sentence} | "
        f"[{v_start // 60:02.0f}:{v_start % 60:06.3f} – {v_end // 60:02.0f}:{v_end % 60:06.3f}] | "
        f"[{a_start // 60:02.0f}:{a_start % 60:06.3f} – {a_end // 60:02.0f}:{a_end % 60:06.3f}] | "
        f"{source} | {description}"
    )

(root / "03_transcripts_and_cuttables/synced_cut_table.txt").write_text("\n".join(lines) + "\n")
print(f"SUCCESS: synced {len(sentences)} sentences, total audio duration {total_duration:.3f}s")
```

### Phase 6: Download 1080p Source Video

Download the full video at 1080p:
```bash
yt-dlp -f "137+140-1/137+140-0/137+140/bestvideo[height<=1080]+bestaudio/best[height<=1080]" \
  --merge-output-format mp4 \
  -o "01_source_video/source_video.mp4" "<YOUTUBE_URL>"
```

### Phase 7: Cut Silent Clips, Stitch, and Export Complete Package

Create and run `render_recap.py` to automate clip rendering, stitching, muxing, vertical formatting, and music mixing:

```python
import re, subprocess
from pathlib import Path

root = Path(".")
source = root / "01_source_video/source_video.mp4"
table = root / "03_transcripts_and_cuttables/synced_cut_table.txt"
audio = root / "02_audio_voiceover/combined_voiceover.wav"
music = Path("/media/huzaifa/06CC0A9BCC0A8561/mcp with antigravity/Subsurface Pulse (1).mp3")
clips = root / "04_video_clips"
exports = root / "06_final_exports"
clips.mkdir(exist_ok=True)
exports.mkdir(exist_ok=True)

def parse_time(val):
    val = val.strip()
    m, s = val.split(":")
    return int(m) * 60 + float(s)

def run(cmd):
    print("Running:", " ".join(cmd))
    subprocess.run(cmd, check=True)

def get_duration(fpath):
    res = subprocess.run(
        ["ffprobe", "-v", "error", "-show_entries", "format=duration", "-of", "csv=p=0", str(fpath)],
        capture_output=True, text=True, check=True
    )
    return float(res.stdout.strip())

# Read cut table rows
rows = []
for line in table.read_text().splitlines()[1:]:
    if not line.strip():
        continue
    parts = line.split(" | ")
    v_match = re.search(r"\[(\d{2}:\d{2}\.\d{3})\s*–\s*(\d{2}:\d{2}\.\d{3})\]", parts[2])
    if not v_match:
        raise ValueError(f"Could not parse visual range in line: {line}")
    v_start = parse_time(v_match.group(1))
    v_end = parse_time(v_match.group(2))
    
    src_parts = parts[4].split(" – ")
    src_start = parse_time(src_parts[0])
    
    rows.append({
        "index": int(parts[0]),
        "v_start": v_start,
        "v_end": v_end,
        "v_dur": v_end - v_start,
        "src_start": src_start,
        "desc": parts[5]
    })

print(f"Parsed {len(rows)} rows.")

# Render silent clips
manifest_lines = []
for r in rows:
    clip_path = clips / f"clip_{r['index']:02d}.mp4"
    run([
        "nice", "-n", "10", "ffmpeg", "-y",
        "-ss", f"{r['src_start']:.3f}",
        "-i", str(source),
        "-t", f"{r['v_dur']:.3f}",
        "-an",
        "-vf", "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:black",
        "-c:v", "h264_nvenc", "-preset", "p2", "-rc", "vbr", "-cq", "23", "-b:v", "0",
        "-filter_threads", "1", "-filter_complex_threads", "1", "-threads", "2",
        "-pix_fmt", "yuv420p",
        str(clip_path)
    ])
    manifest_lines.append(f"file '{clip_path.resolve()}'\n")

manifest = clips / "concat.txt"
manifest.write_text("".join(manifest_lines))

# Stitch silent
silent = clips / "stitched_silent.mp4"
run(["ffmpeg", "-y", "-f", "concat", "-safe", "0", "-i", str(manifest), "-c", "copy", str(silent)])

# Mux narration
raw_sync = exports / "raw_synced.mp4"
run([
    "ffmpeg", "-y",
    "-i", str(silent),
    "-i", str(audio),
    "-map", "0:v:0", "-map", "1:a:0",
    "-c:v", "copy", "-c:a", "aac", "-b:a", "192k",
    "-shortest", "-movflags", "+faststart",
    str(raw_sync)
])

# Create TikTok vertical raw
raw_tiktok = exports / "raw_tiktok.mp4"
run([
    "nice", "-n", "10", "ffmpeg", "-y",
    "-i", str(raw_sync),
    "-vf", "crop=ih*1080/1152:ih:(iw-ih*1080/1152)/2:0,scale=1080:1152,pad=1080:1920:0:384:black",
    "-c:v", "h264_nvenc", "-preset", "p2", "-rc", "vbr", "-cq", "23", "-b:v", "0",
    "-filter_threads", "1", "-filter_complex_threads", "1", "-threads", "2",
    "-c:a", "copy", "-movflags", "+faststart",
    str(raw_tiktok)
])

# Blend background music into landscape
final_landscape = exports / "final_movie_recap_synced.mp4"
dur_landscape = get_duration(raw_sync)
fade_start_l = max(0.0, dur_landscape - 2.0)
run([
    "nice", "-n", "10", "ffmpeg", "-y",
    "-stream_loop", "-1", "-i", str(music),
    "-i", str(raw_sync),
    "-filter_complex",
    f"[0:a]volume=0.316,afade=t=out:st={fade_start_l:.3f}:d=2[music];[1:a][music]amix=inputs=2:duration=first:dropout_transition=2[a]",
    "-map", "1:v:0", "-map", "[a]",
    "-c:v", "copy", "-c:a", "aac", "-b:a", "192k",
    "-shortest", "-movflags", "+faststart",
    str(final_landscape)
])

# Blend background music into TikTok vertical
final_tiktok = exports / "final_movie_recap_tiktok_vertical.mp4"
dur_tiktok = get_duration(raw_tiktok)
fade_start_t = max(0.0, dur_tiktok - 2.0)
run([
    "nice", "-n", "10", "ffmpeg", "-y",
    "-stream_loop", "-1", "-i", str(music),
    "-i", str(raw_tiktok),
    "-filter_complex",
    f"[0:a]volume=0.316,afade=t=out:st={fade_start_t:.3f}:d=2[music];[1:a][music]amix=inputs=2:duration=first:dropout_transition=2[a]",
    "-map", "1:v:0", "-map", "[a]",
    "-c:v", "copy", "-c:a", "aac", "-b:a", "192k",
    "-shortest", "-movflags", "+faststart",
    str(final_tiktok)
])

# Cleanup raw intermediates
if raw_sync.exists():
    raw_sync.unlink()
if raw_tiktok.exists():
    raw_tiktok.unlink()

print("ALL PHASES COMPLETE:")
print("Landscape:", final_landscape)
print("TikTok Vertical:", final_tiktok)
```

---

## Verification Checklist

Before reporting completion:
- [ ] Source video (1080p stream), MP3, WAV, silent clips, and both exports exist.
- [ ] Landscape is 1920×1080 and vertical is 1080×1920.
- [ ] Final video duration matches WAV duration within 0.2s.
- [ ] Audio stream has prominent voiceover and subtly mixed background music (volume 0.316, 2s end fade).
- [ ] Visual timeline has zero gaps (first starts at 0.000, each next starts where previous ended).
- [ ] Concat manifest contains no gap files.
- [ ] Report exact paths of both exported files.

