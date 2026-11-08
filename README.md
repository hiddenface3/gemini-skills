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
