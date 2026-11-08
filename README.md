# Gemini Skills

Custom AI engineering skills and pipelines for Google Antigravity & Gemini multimodal models.

## Contained Skills

### 1. [Gemini Video Analyzer](./SKILL.md)
* **Visual Model**: `models/gemini-2.5-flash` (Primary) / `models/gemini-2.5-pro` (Deep Reasoning).
* **Audio TTS**: `models/gemini-3.8-flash-lite-tts` (Default Voice: `Sadachbia`).
* **Workflow**: Interactive 20-minute user-gated chunks with automated Script Inspector / QC Review pass and decoupled single-call speech synthesis.
* **Synchronization**: Groq Whisper `whisper-large-v3` word boundary matching + FFmpeg broadcast audio mixing.

### 2. [Movie Recap Remake](./movie-recap-remake/SKILL.md)
* **Visual Understanding**: `models/gemini-2.5-flash` with 360p audio/video proxy to write punchy 2–3 min recaps.
* **Voiceover**: `gemini-3.8-flash-tts` single-pass synthesis with default **Charon** voice.
* **Visual Matching**: Automated Gemini video understanding API pass to identify real scene timestamps matching transcript sentences.
* **Render Pipeline**: FFmpeg silent clip cutting (`-an`), stitching, audio muxing, and 9:16 vertical TikTok/Shorts center-crop export.

### 3. [Fast Recap Maker](./fast-recap-maker/SKILL.md)
* **Inputs**: User-provided narration script, timestamped visual breakdown, and source video.
* **Synchronization**: Edge TTS narration with Groq Whisper word-level alignment.
* **Rendering**: Exact 1080p source requirement, H.264 NVENC guidance, landscape and 1080×1920 vertical exports with 384-pixel black bands.
* **Audio**: Voiceover-led mix with configurable background music at approximately **-10 dB**.

### 4. [Ultra Fast Movie Recap](./ultra-fast-movie-recap/SKILL.md)
* **Inputs**: YouTube URL only (no manual script, no manual cut table, no multimodal video AI upload).
* **Visual Anchor Methodology**: Extracts source video's timestamped subtitles/transcript; because original commentary directly describes on-screen events in real-time, new recap sentences are mapped directly to source transcript moments.
* **Exact Spoken-Duration Matching**: Edge TTS voiceover with Groq Whisper word boundaries; slices clips to match exact natural reading/spoken durations with contiguous 0-gap timeline.
* **Render Pipeline**: GTX 750 Ti H.264 NVENC slicing, stitching, narration muxing, 9:16 vertical TikTok/Shorts export with 384px black bands, and -10 dB background music blend.
