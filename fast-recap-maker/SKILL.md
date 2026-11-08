---
name: fast-recap-maker
description: "Fast recap finishing workflow. Use when the user provides a voiceover script, timestamped cut-table clips, and a downloaded source video and asks to finish the recap, transcribe the voiceover, match clip timing, render the recap, or create TikTok exports."
argument-hint: "[voiceover script] [cut table] [source video]"
user-invocable: true
disable-model-invocation: false
---

# Fast Recap Maker

Finish a movie recap when the user already provides:

- a voiceover script;
- a cut table containing one visual range per script line;
- a downloaded source video, or a source path/URL;
- optional voice preference, defaulting to `en-US-GuyNeural`.

This skill does not analyze the movie or rewrite the user's script. It performs the remaining production steps: voiceover generation, timing transcription, timing alignment, clip rendering, muxing, and exports.

## GTX 750 Ti encoding and performance rules

The host GPU is an NVIDIA GeForce GTX 750 Ti. It supports H.264 NVENC only:

- Encode H.264 video with `-c:v h264_nvenc`; do not use HEVC or AV1 NVENC.
- FFmpeg must run directly on the Linux Mint host, or in an execution environment exposing `/dev/nvidia0` and `/dev/nvidiactl`. If FFmpeg reports `no CUDA-capable device`, do not reinstall drivers; rerun with host GPU access.
- Run FFmpeg at low priority with `nice -n 10` and limit processing with `-filter_threads 1 -filter_complex_threads 1 -threads 2`.
- Prefix every video-encoding command shown below with `nice -n 10` (for example, `nice -n 10 ffmpeg ...`); audio-only and stream-copy commands should still avoid unnecessary video encoding.
- Prefer one-pass filters. Do not create one full-video trim branch per silence or edit point.
- Keep simple silence detection, audio analysis, trimming decisions, and concat operations on the CPU.
- For pause removal, cut identical time ranges from audio and video so they remain synchronized.
- For audio-only changes, use `-c:v copy` and encode only the mixed audio.
- Try a short sample encode first, confirm it completes while the computer remains responsive, then render the full output.
- Verify finished files with `ffprobe`; do not reinstall the working NVIDIA 580 driver.

## Required Inputs

1. **Voiceover script**: preserve the user's wording exactly.
2. **Cut table**: preserve the provided source ranges and visual descriptions. A row may contain multiple source ranges separated by semicolons.
3. **Source video**: always download the source at exactly 1080p when YouTube provides it; never intentionally select a lower resolution. Use `-f "137+140"` (or the equivalent 1920x1080 video plus M4A audio format) and merge to MP4. If exact 1080p is unavailable, stop and report that rather than silently falling back.
4. **Project folder**: create a dated project folder if one does not already exist.

Never hardcode API keys. Use the existing environment configuration for Groq.

## Workflow

### Phase 1: Confirm and organize inputs

Create or use this layout:

```text
project/
├── 01_source_video/source_video.mp4
├── 02_audio_voiceover/
├── 03_transcripts_and_cuttables/
├── 04_video_clips/
└── 06_final_exports/
```

Save:

- `voiceover_script.txt`
- the original supplied cut table;
- `word_transcript.json`
- `word_transcript.txt`
- `transcript_with_timestamps.txt`
- `synced_cut_table.txt`

Do not overwrite the original supplied cut table. Copy it before adapting timing.

### Phase 2: Generate Edge TTS voiceover

Use the requested voice, or default to `en-US-GuyNeural`:

```bash
edge-tts \
  --voice "en-US-GuyNeural" \
  --file "03_transcripts_and_cuttables/voiceover_script.txt" \
  --write-media "02_audio_voiceover/combined_voiceover.mp3"
ffmpeg -y \
  -i "02_audio_voiceover/combined_voiceover.mp3" \
  "02_audio_voiceover/combined_voiceover.wav"
```

Verify the generated WAV duration with `ffprobe`.

### Phase 3: Transcribe the generated audio

Use Groq Whisper word timestamps:

```bash
curl https://api.groq.com/openai/v1/audio/transcriptions \
  -H "Authorization: Bearer $GROQ_API_KEY" \
  -F file=@02_audio_voiceover/combined_voiceover.wav \
  -F model=whisper-large-v3 \
  -F response_format=verbose_json \
  -F 'timestamp_granularities[]=word'
```

Save the JSON response and readable word transcript. Build sentence-level timestamps from the word timestamps without changing the script wording.

If Whisper normalizes names or punctuation, preserve the authored script text and use the aligned word positions for timing.

### Phase 4: Adapt the cut table to narration timing

For every script sentence:

1. Match it to the corresponding supplied cut-table row.
2. Keep the supplied source range as the visual source of truth.
3. Set the sentence audio range from the transcript.
4. Extend the visual timeline interval through the next sentence's audio start to cover natural pauses.
5. Start the first visual interval at `00:00.000`.
6. Make every next visual interval start exactly where the previous one ends.
7. End the final visual interval at the full WAV duration.
8. Adjust the selected source clip duration longer or shorter as needed to fill its visual interval.
9. Never insert black gap clips, blank frames, or artificial silence.

The adapted table must contain:

```text
Audio Range | Sentence Duration | Visual Timeline Range | Visual Duration | Narration | Provided Video Range | Selected Source Start | Visual Match
```

For multiple supplied source ranges, use the strongest range first and retain all relevant ranges in the row. If a single source range cannot fill the required duration, use adjacent matching source footage from the supplied table; do not invent unsupported timestamps.

Validate:

- the first visual start is `0.000`;
- every visual interval is contiguous;
- the final visual end equals audio duration within `0.2` seconds;
- the number of rows matches the number of script sentences.

### Phase 5: Render silent clips

For each adapted row, cut the source video for the **visual duration**, not merely the spoken sentence duration:

```bash
ffmpeg -y \
  -ss "<SELECTED_SOURCE_START>" \
  -i "01_source_video/source_video.mp4" \
  -t "<VISUAL_DURATION>" \
  -an -c:v h264_nvenc -preset p2 -rc vbr -cq 23 -b:v 0 \
  -filter_threads 1 -filter_complex_threads 1 -threads 2 \
  -pix_fmt yuv420p \
  "04_video_clips/clip_XX.mp4"
```

Use absolute paths in the concat manifest. Concatenate clips directly:

```bash
ffmpeg -y -f concat -safe 0 \
  -i "04_video_clips/concat.txt" \
  -c copy "04_video_clips/stitched_silent.mp4"
```

Do not generate or insert gap clips. The visual intervals already cover the narration pauses.

### Phase 6: Mux narration

```bash
ffmpeg -y \
  -i "04_video_clips/stitched_silent.mp4" \
  -i "02_audio_voiceover/combined_voiceover.wav" \
  -map 0:v:0 -map 1:a:0 \
  -c:v copy -c:a aac -b:a 192k \
  -shortest -movflags +faststart \
  "06_final_exports/final_movie_recap_synced.mp4"
```

### Phase 7: Create TikTok export with black bands

Export to `1080x1920` with solid black bands of `384` pixels at the top and bottom. The video image occupies the middle `1152` pixels (60% of the frame). For landscape input, center-crop the sides to the `1080:1152` image aspect ratio, then scale and pad:

```bash
nice -n 10 ffmpeg -y \
  -i "06_final_exports/final_movie_recap_synced.mp4" \
  -vf "crop=ih*1080/1152:ih:(iw-ih*1080/1152)/2:0,scale=1080:1152,pad=1080:1920:0:384:black" \
  -c:v h264_nvenc -preset p2 -rc vbr -cq 23 -b:v 0 \
  -filter_threads 1 -filter_complex_threads 1 -threads 2 \
  -c:a copy -movflags +faststart \
  "06_final_exports/final_movie_recap_tiktok_vertical.mp4"
```

Save vertical exports in the separate `06_final_exports/` output folder with clear filenames. Run a short sample with the same filter before the full render when practical.

### Phase 8: Add background music

Set `BACKGROUND_MUSIC_PATH` to the desired local background music file before running the audio-mix commands. Do not commit the music asset or machine-specific absolute paths to the skill repository.

```text
BACKGROUND_MUSIC_PATH=/path/to/background-music.mp3
```

Replace the final exports in `06_final_exports/` with the music-added versions. Keep the existing voiceover/movie audio prominent while making the music subtly hearable: always mix the music at `0.316` volume (approximately `-10 dB`), preserve its normal speed and pitch, loop it if needed, trim it to the video duration, and fade it out over the final two seconds. Do not slow down or time-stretch the music. Since this is audio-only processing, copy the video stream:

```bash
nice -n 10 ffmpeg -y \
  -stream_loop -1 \
  -i "$BACKGROUND_MUSIC_PATH" \
  -i "06_final_exports/final_movie_recap_synced.mp4" \
  -filter_complex "[0:a]volume=0.316,afade=t=out:st=$(ffprobe -v error -show_entries format=duration -of csv=p=0 '06_final_exports/final_movie_recap_synced.mp4')-2:d=2[music];[1:a][music]amix=inputs=2:duration=first:dropout_transition=2[a]" \
  -map 1:v:0 -map "[a]" -c:v copy -c:a aac -b:a 192k \
  -shortest -movflags +faststart \
  "06_final_exports/final_movie_recap_synced.mp4"
```

Apply the same audio-only mix to the vertical export, saving:

```text
06_final_exports/final_movie_recap_tiktok_vertical.mp4
```

Do not re-encode video for music-only changes.

## Verification Checklist

Before reporting completion:

- verify source video, MP3, WAV, clips, and both exports exist;
- verify landscape is 1920×1080 and vertical is 1080×1920;
- verify final video duration matches WAV duration within `0.2` seconds;
- verify audio is present and source clip audio was removed;
- verify the concat manifest has no gap files;
- verify the adapted visual timeline has no gaps or black padding;
- verify vertical output is exactly `1080x1920`, with the image in rows `384` through `1535` and black bands above and below;
- verify music-added files retain the original video stream and have mixed audio;
- verify music is trimmed/looped to the video length and fades out over the final two seconds;
- report exact output paths.

Do not add background music, subtitles, or extra processing unless requested.
