# Gemini Skills

Custom AI engineering skills and pipelines for Google Antigravity & Gemini multimodal models.

## Contained Skills

### 1. [Gemini Video Analyzer](./SKILL.md)
* **Visual Model**: `models/gemini-2.5-flash` (Primary) / `models/gemini-2.5-pro` (Deep Reasoning).
* **Audio TTS**: `models/gemini-3.8-flash-lite-tts` (Default Voice: `Sadachbia`).
* **Workflow**: Interactive 20-minute user-gated chunks with automated Script Inspector / QC Review pass and decoupled single-call speech synthesis.
* **Synchronization**: Groq Whisper `whisper-large-v3` word boundary matching + FFmpeg broadcast audio mixing.
