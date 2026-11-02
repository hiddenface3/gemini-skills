---
name: movie-recap-remake
description: >
  Full end-to-end automated movie recap pipeline for TikTok/Shorts.
  Given a YouTube URL and optional voice preference, this skill:
  (1) downloads the 1080p source video,
  (2) makes a 360p proxy for AI video analysis,
  (3) analyzes the full video with Gemini 2.5 Pro/Flash to write a 2–3 minute narration script,
  (4) generates the voiceover via Gemini TTS (user-selected voice),
  (5) transcribes with millisecond timestamps,
  (6) builds a synchronized clip cut table from the transcript timing,
  (7) cuts all video clips (silent, no source audio), stitches them, and muxes with voiceover,
  (8) exports both 16:9 landscape master and 9:16 vertical TikTok version.
  Trigger: "make a movie recap", "recap this video", "run recap pipeline", "/movie-recap"
---

# Movie Recap Remake Skill

A complete, reusable pipeline to turn any long YouTube movie/documentary video into a fast-paced, voiceover-narrated recap video ready for TikTok (9:16) and YouTube Shorts.

---

## Step-by-Step Summary

| # | Step | Tool / Method |
|---|------|--------------|
| 1 | Download video at 1080p | `yt-dlp` |
| 2 | Create 360p proxy + strip audio | `ffmpeg` |
| 3 | Analyze proxy with AI → write 2–3 min script | Gemini 2.5 Pro or Flash (video upload) |
| 4 | Generate full voiceover in one API call | Gemini TTS (user-chosen voice) |
| 5 | Transcribe voiceover with millisecond timestamps | Gemini 2.5 Flash audio transcription |
| 6 | Build clip sync table (match narration to video moments) | Agent maps transcript timings → source video timestamps |
| 7 | Cut clips (silent), stitch, mux with voiceover | `ffmpeg` + Node.js script |
| 8 | Export 16:9 master + 9:16 TikTok vertical crop | `ffmpeg` |

---

## Full Pipeline Instructions

### Phase 0 — Project Setup

Create a separate project folder for each video. Use this naming convention:
```
movie_recap_<short_name>/
├── 01_source_video/
├── 02_audio_voiceover/
├── 03_transcripts_and_cuttables/
├── 04_scripts/
├── 05_intermediate_clips/
│   ├── synced_clips/
└── 06_final_exports/
```

---

### Phase 1 — Download Video

Use `yt-dlp` to download the best 1080p video + best audio and merge into a single `.mp4`.

```bash
yt-dlp -f "137+140/bestvideo[ext=mp4]+bestaudio[ext=m4a]/best" \
  --merge-output-format mp4 \
  -o "01_source_video/source_video.mp4" \
  "<YOUTUBE_URL>"
```

---

### Phase 2 — Create 360p Proxy for AI Analysis

Downscale to 360p **with audio kept intact** so Gemini can understand both visuals and dialogue/sound for accurate story comprehension:
```bash
ffmpeg -y -i "01_source_video/source_video.mp4" \
  -vf "scale=-2:360" -c:v libx264 -crf 28 -preset veryfast \
  -c:a aac -b:a 96k \
  "01_source_video/source_video_360p_proxy.mp4"
```

> ⚠️ **IMPORTANT**: Always use the 360p proxy for Gemini video analysis. Never upload 1080p. It stays well within the free tier context window.
>
> 🔊 **Keep audio in the proxy!** Gemini needs both visuals AND audio (dialogue, narration, sound effects) to fully understand the story and write an accurate 2–3 minute recap script. Do NOT strip audio from the proxy.
>
> ⚠️ **Do NOT create a `source_video_noaudio.mp4` at this stage.** Audio removal happens only in Phase 7 when cutting clips.

---

### Phase 3 — AI Video Analysis & Script Generation

**Model**: `gemini-2.5-pro` (preferred) or `gemini-2.5-flash`

Upload the 360p proxy to Gemini Files API and analyze the entire video in one call (up to 20 min fits in the 1M context window). If video is longer than 20 min, analyze in gated 20-min chunks and combine.

**Prompt structure**:
```
You are a professional YouTube movie recap narrator. Watch this entire video and write a complete, engaging 2 to 3 minute narration script that covers the full story arc from beginning to end.

Rules:
- Write as flowing, dramatic narration — past tense third-person storytelling
- Keep it fast-paced and gripping. Suitable for TikTok/YouTube Shorts audience
- Target length: 280 to 380 words (2–3 minutes at natural TTS pace)
- Do NOT use chapter headings, timestamps, or bullet points in the script
- Write it as ONE continuous paragraph broken only by natural sentence endings
- Include key characters, conflicts, twists, and resolution
- Keep sentences punchy (12–18 words max per sentence)

Output: ONLY the narration script text. Nothing else.
```

Save the output to `03_transcripts_and_cuttables/voiceover_script.txt`.

---

### Phase 4 — Voiceover Generation (Gemini TTS)

Read the script from `voiceover_script.txt` and generate audio in a **single API call**.

**Default voice: `Charon`** — Used automatically unless user specifies a different voice.

**Supported voices** (user chooses one per project):
| Voice | Character | Default? |
|-------|-----------|----------|
| `Charon` | Deep, somber, cinematic male | ✅ **DEFAULT** |
| `Fenrir` | Dark, intense, dramatic male | |
| `Sadachbia` | Authoritative documentary male | |
| `Orus` | Firm narrative male | |
| `Puck` | Energetic, conversational male | |
| `Aoede` | Engaging dynamic female | |
| `Kore` | Calm, smooth female | |
| `Zephyr` | Warm, gentle female | |

> If the user does not specify a voice, **always use `Charon` without asking**. Only ask if the user explicitly mentions wanting a different voice.

**API call** (Node.js):
```javascript
const res = await fetch(
  `https://generativelanguage.googleapis.com/v1beta/models/gemini-3.8-flash-tts:generateContent?key=${apiKey}`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      contents: [{ parts: [{ text: fullScriptText }] }],
      generationConfig: {
        responseModalities: ["AUDIO"],
        speechConfig: {
          voiceConfig: { prebuiltVoiceConfig: { voiceName: voice } }
        }
      }
    })
  }
);
```

The API returns raw PCM audio (`audio/L16;codec=pcm;rate=24000`). Save the base64 decoded buffer as `.wav` file, then convert to proper WAV header using ffmpeg:
```bash
ffmpeg -y -f s16le -ar 24000 -ac 1 -i combined_voiceover_raw.wav combined_voiceover.wav
```

Save to: `02_audio_voiceover/combined_voiceover.wav`

**Model order** (try in sequence — do NOT use `gemini-2.5-flash-preview-tts`):
1. `gemini-3.8-flash-tts` ✅ PRIMARY
2. `gemini-3.8-flash-lite-tts` (fallback if primary fails)

---

### Phase 5 — Transcribe Voiceover with Timestamps

Transcribe `combined_voiceover.wav` using Gemini 2.5 Flash audio understanding to get precise millisecond-level sentence timestamps.

**Prompt**:
```
Provide a complete verbatim transcript of this audio. For every sentence or spoken clause, output in format:
[MM:SS.mmm – MM:SS.mmm] Spoken text

Ensure timestamps cover the full duration. Also output total duration.
```

Save to: `03_transcripts_and_cuttables/transcript_with_timestamps.txt`

Example output format:
```
[00:00.176 – 00:05.346] In 1933 Oregon, the Great Depression has left widowed father Sam with nothing.
[00:05.776 – 00:13.626] Evicted from his apartment, he desperately searches for work...
...
Total Duration: 01:36.466
```

---

### Phase 6 — AI Visual Clip Matching via Gemini API (Build Synced Cut Table)

Once the voiceover transcript with millisecond timestamps is ready, **call Gemini 2.5 Flash / 3.1 Pro via API** passing:
1. The uploaded 360p proxy video (`file_uri`)
2. The full timestamped transcript text from Phase 5

The AI inspects the video frames, identifies the exact scenes matching each spoken sentence, and outputs a strictly synchronized cut table where clip duration equals narration segment duration.

**API Call Implementation (`build_cut_table.mjs`)**:
```javascript
const prompt = `You are a professional video editor specializing in fast-paced TikTok / Shorts movie recaps.
Here is the verbatim transcript with timestamps for our voiceover:

\${transcriptText}

You have access to the video file.
Inspect the footage and build a MASTER SYNCHRONIZED CLIP CUT TABLE matching every narration line to the exact visual moments in the source video.

CRITICAL EDITING RULES:
1. CONTINUOUS TIMELINE: The Audio Timeline must cover from 00:00.00 to the total audio duration without any gaps or black frames.
2. DURATION SYNC: Every cut's duration must match the duration of that audio segment so visuals and voice stay 100% in sync.
3. CLIP DURATION LIMIT: Keep visual cuts between 2.0 and 3.5 seconds max for high TikTok retention. If a narration sentence lasts longer than 3.5 seconds, divide it into 2 or 3 shorter complementary visual cuts (e.g. wide shot, reaction shot, close-up action).
4. RELEVANT VISUALS: Visually inspect the video and find the exact scenes where the described action actually occurs.
5. FORMAT: Output tab-separated rows:
Audio Timeline\\tDuration\\tNarration Line\\tVideo Source Cut (Start – End)\\tVisual Action

Example:
00:00.00 – 00:02.80\\t2.80s\\tIn 1381 England...\\t00:00.50 – 00:03.30\\tMisty English countryside
`;

const res = await fetch(
  `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=\${apiKey}`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      contents: [
        {
          parts: [
            { file_data: { mime_type: "video/mp4", file_uri: uploadedVideoUri } },
            { text: prompt }
          ]
        }
      ]
    })
  }
);
```

**Cut table format (tab-separated)**:
```
Audio Timeline    Duration    Narration Line    Video Source Cut (Start – End)    Visual Action
00:00.00 – 00:02.80    2.80s    In 1381 England...    00:00.50 – 00:03.30    Misty countryside
00:02.80 – 00:05.80    3.00s    ...the plague claimed half...    00:00.15.50 – 00:18.50    Robin grieving at crosses
```

Save to: `03_transcripts_and_cuttables/synced_cut_table.txt`

**Validation Check**:
- Run automated parser to verify `sum(clip_durations) == audio_duration ± 0.2s`
- Verify no gaps between audio end of clip N and audio start of clip N+1

---

### Phase 7 — Render Final Synced Video (Direct Cut, Stitch, and Mux)

> ⛔ **ABSOLUTE RULE**: Do NOT invent fixes, modify timestamps, scale durations, or do things from your own side. Strictly cut the exact clips Gemini gave in the table from the 1080p source video, combine them together into one continuous silent video, and place the generated voiceover on top. If there are any errors or sync issues, the user will fix them themselves.

Run `render_synced_video.mjs`:
1. Read `synced_cut_table.txt` exactly as provided.
2. Cut each clip from `01_source_video/source_video.mp4` using `-an` to remove original source audio:
```bash
ffmpeg -y -ss <start> -t <duration> -i "01_source_video/source_video.mp4" \
  -an -c:v libx264 -crf 18 -preset veryfast -pix_fmt yuv420p \
  "clip_XX.mp4"
```
3. Concatenate all cut clips together into one continuous video file with no audio:
```bash
ffmpeg -y -f concat -safe 0 -i concat_list.txt -c copy "05_intermediate_clips/synced_video_noaudio.mp4"
```
4. Directly combine that video file with the generated voiceover audio (`02_audio_voiceover/combined_voiceover.wav`):
```bash
ffmpeg -y -i "05_intermediate_clips/synced_video_noaudio.mp4" -i "02_audio_voiceover/combined_voiceover.wav" \
  -map 0:v:0 -map 1:a:0 -c:v copy -c:a aac -b:a 192k "06_final_exports/final_movie_recap_synced.mp4"
```
No re-scaling, no timestamp tweaking, no extra logic. Just cut, concatenate, and mux.


---

### Phase 8 — Export TikTok 9:16 Vertical Version

Center-crop the 1080p landscape video to 9:16 vertical (no scaling, no shrinking — pure crop):
```bash
node 04_scripts/render_vertical_tiktok.mjs
```

FFmpeg filter used:
```bash
ffmpeg -y -i final_movie_recap_synced.mp4 \
  -vf "crop=608:1080:656:0,scale=1080:1920,setsar=1" \
  -c:v libx264 -crf 18 -preset veryfast -pix_fmt yuv420p \
  -c:a copy \
  final_movie_recap_tiktok_vertical.mp4
```

Output: `06_final_exports/final_movie_recap_tiktok_vertical.mp4` (`1080x1920`, 9:16)

---

## Deliverables Checklist

At the end of each project, verify:

- [ ] `01_source_video/source_video.mp4` — 1080p original with audio (used for cuts + audio stripped at cut time)
- [ ] `01_source_video/source_video_360p_proxy.mp4` — 360p proxy WITH audio for AI analysis
- [ ] `02_audio_voiceover/combined_voiceover.wav` — Charon/selected voice TTS
- [ ] `03_transcripts_and_cuttables/voiceover_script.txt` — Clean narration paragraph
- [ ] `03_transcripts_and_cuttables/transcript_with_timestamps.txt` — Sentence timestamps
- [ ] `03_transcripts_and_cuttables/synced_cut_table.txt` — Clip cut table
- [ ] `06_final_exports/final_movie_recap_synced.mp4` — 16:9 landscape master
- [ ] `06_final_exports/final_movie_recap_tiktok_vertical.mp4` — 9:16 TikTok master

---

## Important Technical Notes

1. **NEVER DO STUFF FROM YOUR OWN SIDE**: Follow the skill steps exactly as written. Do not add clever fixes, re-time clips, change speed, or modify audio/video files unless explicitly instructed by the user. If an error happens, the user will handle it.
2. **360p proxy MUST keep audio** — Gemini needs both visuals and dialogue/audio to understand the full story and write an accurate recap script. Do NOT strip audio from the proxy.
3. **Source audio stripped ONLY at cut time** — when cutting clips from the 1080p source, use `-an` flag in ffmpeg. Never pre-strip audio from the main source video file.
4. **Pure Cut + Concat + Mux** — When the cut table is generated, cut the clips, stitch them into one continuous silent video, and place the voiceover WAV on top. Do not add padding, do not loop clips, do not adjust audio pitch/speed.
5. **Raw TTS output is headerless PCM** — always run through ffmpeg with `-f s16le -ar 24000 -ac 1` to add proper WAV header before transcription or playback.
6. **Each project is a separate folder** — never mix files from different projects. Prefix folder: `project_NN_<short_name>/`
7. **Proxy for AI only** — Gemini analyzes 360p proxy WITH audio. All final video cutting uses 1080p source.
8. **User always approves** the script before TTS generation. Present the full narration paragraph and wait for confirmation.
9. **User always approves** the cut table before rendering. Show the full table and wait for confirmation.

---

## Quick Reference — Voice Selection

**Default: `Charon` — always use this unless user explicitly requests another voice.**

```
Charon    → Deep cinematic dark male (best for action/thriller)  ✅ DEFAULT
Fenrir    → Intense dramatic male (best for horror/survival)
Sadachbia → Documentary authoritative male (best for documentaries)
Orus      → Firm narrative male
Puck      → Upbeat conversational male
Aoede     → Dynamic engaging female
Kore      → Calm smooth female
Zephyr    → Warm gentle female
```

---

## Trigger Examples

- "Make a recap of this video: [URL]"
- "Recap pipeline for [URL]"
- "/movie-recap [URL]"
- "Download this and make a 2 minute recap: [URL]"
