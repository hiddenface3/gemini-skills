---
name: gemini-video-analyzer
description: Analyzes video clips using Gemini 2.5 Flash / 2.5 Pro native video understanding in interactive 20-minute user-gated chunks, generates timed 3rd-person documentary commentary benchmarked against mastercraft transcripts, audits scripts with an automated Inspector/QC pass, synthesizes studio voiceovers using Gemini 3.8 Flash Lite TTS with default Sadachbia voice, slices audio with Groq Whisper word boundaries, tracks quotas, and muxes video with narration via FFmpeg.
---

# Gemini Video Analyzer

Automated end-to-end pipeline for native video comprehension, real-time rate limit & token quota management, and documentary voiceover production using Google Gemini multimodal and TTS models.

## Default Configuration
- **Visual Model Hierarchy**:
  * **Primary**: `models/gemini-2.5-flash` (native multimodal video comprehension, fast processing, zero frame scraping).
  * **Deep Reasoning Alternate**: `models/gemini-2.5-pro` (for complex mechanical nuances).
  * ⛔ **STRICT PROHIBITION**: Do NOT use `gemini-3.1-pro-preview`.
- **TTS Model Hierarchy**:
  * **Primary**: `models/gemini-3.8-flash-lite-tts` (crisp, high-throughput studio speech synthesis).
  * **Alternate**: `models/gemini-3.8-flash-tts` (expressive speech synthesis).
  * ⛔ **STRICT PROHIBITION**: Do NOT use `gemini-2.5-flash-preview-tts`.
- **Default Voice**: `Sadachbia` (MANDATORY: Always use `Sadachbia`).

- **Video Pre-Processing**: FFmpeg downscale to `360p` (H.264, CRF 28, AAC 64k) for ultra-fast API uploads
- **Pacing & Cadence Architecture (Event-Driven, Anti-Metronome)**:
  - **The Anti-Metronome Principle**: Real human commentary is triggered by *visual events*, NOT a fixed stopwatch timer. Never force a robotic "speak 20s, wait 20s" rhythm.
  - **Adaptive Three-Tier Commentary Units**:
    * **Tier 1: Micro-Observations (2–5 seconds / 5–15 words)**: Rapid, punchy callouts to subtle techniques, micro-slips, or grip adjustments (*"Notice the bevel angle—he's rotating, not pushing."*).
    * **Tier 2: Core Engineering Loops (10–18 seconds / 20–35 words)**: Problem/shortcut/workaround breakdowns during active construction.
    * **Tier 3: Macro Retrospectives (20–30 seconds / 40–60 words)**: High-stakes intro, major pivots, and triumphant aesthetic outro.
  - **Dynamic Audio Breathing**:
    * Ambient silence between segments is variable (ranges from 5s to 35s depending on visual density).
    * **Safety Ceiling**: Ambient silence must NEVER exceed 40–45 seconds anywhere in the video (Zero Dead Air guardrail).
- **Audio Mix Standard**: Multi-input resampled stereo mix (`aformat=sample_rates=44100:channel_layouts=stereo`) with ambient background at `volume=0.25` and voiceover at `volume=1.40` mastered to EBU R128 (`-16 LUFS`).

---

## 📚 Gold-Standard Reference Examples (Mandatory Style Benchmarks)

Before generating or refining any documentary narration, the AI agent MUST view and study the style, vocabulary, sentence length, and rhythmic pacing in these master reference transcripts:

- **Reference 1 (Michael - Deep Wilderness Basecamp)**:
  Path: `/media/huzaifa/E0765FD1765FA756/next video 19 oct/my examples/first exmaple.txt`
- **Reference 2 (Max - Lawyer Turned Forest Craftsman)**:
  Path: `/media/huzaifa/E0765FD1765FA756/next video 19 oct/my examples/second example.txt`
- **Reference 3 (Janine - Earth & Timber Engineering)**:
  Path: `/media/huzaifa/E0765FD1765FA756/next video 19 oct/my examples/3rd exmaple.txt`

> [!IMPORTANT]
> **MANDATORY DIRECTIVE FOR AI AGENTS**:
> Whenever writing commentary, the AI must use `view_file` to inspect these reference files if guidance or style alignment is needed. Emulate their exact sentence rhythm (≤ 15 words/sentence), grounded physical vocabulary, material respect, and absence of generic emotional padding.



---

## 🚨 MANDATORY EXECUTION CONTRACT: INTERACTIVE 20-MINUTE CHUNK PIPELINE WITH HUMAN GATING & INSPECTOR QC

> [!CAUTION]
> **CRITICAL ARCHITECTURAL CONTRACT FOR ANY AI AGENT RUNNING THIS SKILL:**
> Whenever you generate ANY script to analyze video footage (e.g. `scratch/analyze_video.js`, `analyze_clip.js`, or any runtime pipeline):
>
> 1. ⛔ **ZERO INTERMEDIATE SCOUT JSON LAW**:
>    - You are **STRICTLY FORBIDDEN** from having the AI generate an intermediate JSON mapping of every physical action, tool, and second-by-second event.
>    - Writing out intermediate action-cataloging JSON is impractical, wastes massive API tokens, introduces latency, and adds zero creative quality to the final script.
> 2. 🛑 **MANDATORY HUMAN APPROVAL GATE (ZERO AUTONOMOUS CHUNK JUMPING)**:
>    - You are **STRICTLY FORBIDDEN** from running an autonomous loop that writes all chunks or generates TTS automatically.
>    - Process **ONE 20-minute chunk at a time** (`Chunk 1: 00:00–20:00`).
>    - After drafting and inspecting Chunk 1's script, you **MUST STOP AND PRESENT THE WRITTEN SCRIPT TO THE USER**.
>    - ONLY after the user explicitly reviews, approves, and commands you to proceed do you process `Chunk 2 (20:00–40:00)`.
> 3. 🔍 **MANDATORY SCRIPT INSPECTOR / QC AUDIT STEP**:
>    - Every drafted chunk script MUST pass through an automated **Script Inspector / QC Reviewer** pass before being shown to the user.
>    - The Inspector audits: 15-word sentence cap, anti-slop blacklist, 4-step engineering loop, and style alignment with the reference examples (`first exmaple.txt`, `second example.txt`, `3rd exmaple.txt`).
>    - The Inspector auto-corrects mistakes and outputs a clear summary of issues found and repaired.
> 4. 🎙️ **DECOUPLED TTS GENERATION**:
>    - Voiceover synthesis (`models/gemini-3.8-flash-lite-tts`, voice `Sadachbia`) is strictly decoupled.
>    - **NEVER** run TTS while drafting or chunking scripts. TTS runs **ONLY as the final stage** after ALL chunk scripts are approved by the user.
> 5. 🎯 **VISUAL MODEL ENFORCEMENT**:
>    - Always use `models/gemini-2.5-flash` (Primary) or `models/gemini-2.5-pro` (Deep Reasoning Alternate).
>    - ⛔ `gemini-3.1-pro-preview` is STRICTLY FORBIDDEN.

### Canonical Single-Chunk Scripting & QC Inspector Template (Verbatim Script Embedding)
Every analysis script must execute this single-chunk pattern with an integrated QC pass:

```javascript
// =========================================================================
// INTERACTIVE SINGLE-CHUNK SCRIPT GENERATION & INSPECTOR QC PASS
// Visual Model: gemini-2.5-flash (Alternate: gemini-2.5-pro)
// TTS Model: gemini-3.8-flash-lite-tts (DECOUPLED: Run ONLY after all chunks approved)
// Default Voice: Sadachbia (MANDATORY)
// Mode: Direct Multimodal Scripting + Automated Inspector QC Audit
// =========================================================================

// --- Step 1: Slice 20-Minute 360p Video Chunk via FFmpeg ---
const chunkIdx = 0; // 0 = 00:00-20:00, 1 = 20:00-40:00, etc.
const startSec = chunkIdx * 1200;
const endSec = Math.min((chunkIdx + 1) * 1200, totalDurationSec);
const chunkLen = endSec - startSec;
const chunkProxyPath = `scratch/chunk_${chunkIdx}_360p.mp4`;

console.log(`[Video Pipeline] Slicing Chunk ${chunkIdx + 1} (${formatMMSS(startSec)} - ${formatMMSS(endSec)})...`);
execSync(`ffmpeg -y -ss ${startSec} -t ${chunkLen} -i "${sourceProxyPath}" -c copy "${chunkProxyPath}"`, { stdio: 'inherit' });

// --- Step 2: Upload Chunk Proxy to Gemini Files API ---
const file = await uploadToGemini(chunkProxyPath);

// --- Step 3: Direct Multimodal Scripting Call ---
const chunkPrompt = `You are a master documentary director, veteran craft engineer, and voiceover writer.
You are watching footage from ${formatMMSS(startSec)} to ${formatMMSS(endSec)} (Duration: ${chunkLen}s).
Overall build context so far:
- Protagonist: ${accumulatedContext.protagonistName} (${accumulatedContext.civilianRole})
- Completed components: ${JSON.stringify(accumulatedContext.structuresBuilt)}
- Salvaged scrap materials from earlier: ${JSON.stringify(accumulatedContext.salvagedScraps)}

STYLE BENCHMARKS (Consult Reference Examples):
You are strictly modeled after the style, rhythm, and engineering depth in:
- Reference 1: /media/huzaifa/E0765FD1765FA756/next video 19 oct/my examples/first exmaple.txt
- Reference 2: /media/huzaifa/E0765FD1765FA756/next video 19 oct/my examples/second example.txt
- Reference 3: /media/huzaifa/E0765FD1765FA756/next video 19 oct/my examples/3rd exmaple.txt

YOUR MISSION: Watch this 20-minute video chunk and compose the broadcast-grade documentary voiceover script.
Write the narration directly. Do NOT output raw action summaries or intermediate logs.

STRICT MASTERCLASS SCRIPTING RULES:
1. 🚨 ZERO PURPLE PROSE / BANNED WORDS:
   NEVER use: testament, tapestry, alchemy, dance of fire, fortress from decay, sanctuary, nature's fury, masterclass in, silent sentinel, symphony, whisper of the forest, humble beginning, battle against the elements, relentless march of time, beacon of hope, unyielding resolve.
   Use concrete physical nouns and mechanics: adze, mortise, green sap, frost heave, capillary action, tensile shear, fulcrum, kerf, drawknife, pith, heartwood.

2. ⏱️ 15-WORD HARD CAP PER SENTENCE:
   Every sentence MUST be 15 words or fewer. Average 8-12 words. Punchy, crisp, breathless documentary cadence.

3. 👤 NAMED PROTAGONIST & CIVILIAN CONTRAST:
   Refer to the builder by name ("${accumulatedContext.protagonistName}"). Anchor the stakes against their civilian life.

4. ⚙️ PROBLEM-FIRST 4-STEP ENGINEERING LOOP:
   - Physical Problem: Moisture rot, frost heave, shear stress, wood checking, lever torque.
   - Why Amateur Shortcut Fails: Nails split frozen cedar, flat joints rot, green logs twist.
   - Ingenious Craft Workaround / Reversal: Interlocking saddle notch, counter-weighted lever, charred wood skin, stone dry-stacking.
   - ASMR Payoff Beat: Leave 15-25 seconds of silence so real tool acoustics (adze bite, chisel ring, wood crack) breathe.

5. ♻️ MATERIAL FORESHADOWING & SCRAP RECYCLING:
   Track off-cuts, shavings, and bent metal. Reference how earlier scraps solve new problems.

6. 🏛️ REGIONAL HERITAGE & MECHANICAL REVERSALS:
   Use authentic craft terms (Zamlanka, Slavic reed weaving, drawknife chamfering, dovetail scribing). Turn physical defects into structural assets.

7. 🎙️ EVENT-DRIVEN 3-TIER PACING:
   - Tier 1 (Micro-Observation, 2-5s / 5-15 words): Subtle grip, blade bevel, hairline risk.
   - Tier 2 (Core Loop, 10-18s / 20-35 words): Physical problem, shortcut failure, craft workaround.
   - Tier 3 (Macro Retrospective, 20-30s / 40-60 words): High-stakes intro, mid-build crisis, triumphant outro.
   - Silence Gap Guardrail: Ambient gap between narration segments must NEVER exceed 40 seconds.
   - Written for TTS voice 'Sadachbia'.

Return ONLY a valid JSON object:
{
  "chunkIndex": ${chunkIdx},
  "structuresBuiltInChunk": ["list components completed in this chunk"],
  "salvagedScrapsNew": ["list scraps/materials saved for later"],
  "segments": [
    {
      "segmentNumber": 1,
      "relativeStartTimeSec": 8.0,
      "globalStartTimeSec": ${startSec} + 8.0,
      "timestamp": "00:08",
      "tier": "Tier 1 (Micro-Observation) | Tier 2 (Core Loop) | Tier 3 (Macro Retrospective)",
      "phase": "High-Stakes Intro Hook | Engineering Body Loop | Crisis Pivot | Triumphant Outro",
      "visualAction": "What the viewer sees happening right now",
      "engineeringMechanism": "Hidden physics, material resistance, or technique",
      "narrationText": "Broadcast documentary narration text strictly following the 15-word sentence cap and anti-slop rules."
    }
  ]
}`;

const draftResponse = await ai.models.generateContent({
  model: 'models/gemini-2.5-flash', // Alternate: 'models/gemini-2.5-pro'
  contents: [
    { fileData: { fileUri: file.uri, mimeType: file.mimeType } },
    chunkPrompt
  ],
  config: { responseMimeType: 'application/json' }
});
const rawDraft = JSON.parse(draftResponse.text);

// --- Step 4: Automated Script Inspector / QC Review Pass ---
console.log(`[Script Inspector] Auditing Chunk ${chunkIdx + 1} script against quality doctrine...`);
const qcPrompt = `You are a strict, forensic Documentary Script Inspector and Quality Control Editor.
You are auditing the raw drafted narration script for Chunk ${chunkIdx + 1} (${formatMMSS(startSec)} - ${formatMMSS(endSec)}):

Raw Draft:
${JSON.stringify(rawDraft.segments, null, 2)}

AUDIT CRITERIA:
1. Anti-Slop Blacklist: Scan for ANY instance of: testament, tapestry, alchemy, dance of fire, fortress from decay, sanctuary, nature's fury, masterclass in, silent sentinel, symphony, whisper of the forest, humble beginning, battle against the elements, relentless march of time. If found, REWRITE with concrete craft terms.
2. 15-Word Sentence Cap: Scan EVERY sentence in every narrationText. If any sentence exceeds 15 words, SPLIT or TIGHTEN it into crisp, punchy sentences under 15 words.
3. Obvious Visual Narration: Check if narrator states the obvious (e.g., "He picks up the axe"). Replace with hidden mechanics and physical stakes.
4. Style Consistency: Ensure voice matches the master craft reference transcripts (Michael, Max, Janine).

Return ONLY a valid JSON object:
{
  "issuesFound": [
    { "segmentNumber": 1, "issue": "Sentence length 18 words / Blacklisted word 'sanctuary'", "fixApplied": "Shortened to 11 words and replaced with 'interlocking timber frame'" }
  ],
  "qcPassed": true,
  "verifiedSegments": [
    // Array of all segments with all fixes applied
  ]
}`;

const qcResponse = await ai.models.generateContent({
  model: 'models/gemini-2.5-flash',
  contents: [qcPrompt],
  config: { responseMimeType: 'application/json' }
});
const qcResult = JSON.parse(qcResponse.text);

// Clean up remote file
await ai.files.delete({ name: file.name });

// --- Step 5: Save Script & AWAIT USER APPROVAL (Do NOT auto-advance!) ---
fs.writeFileSync(`scratch/chunk_${chunkIdx}_script_verified.json`, JSON.stringify(qcResult.verifiedSegments, null, 2));
console.log(`[Video Pipeline] Chunk ${chunkIdx + 1} script completed and verified by Inspector.`);
console.log(`[QC Summary] Issues fixed: ${qcResult.issuesFound.length}`);
console.log(`⚠️ STOPPING: Presenting written script to USER for inspection. Awaiting user command to proceed to Chunk ${chunkIdx + 2}.`);
```

---

## The Master 4-Phase Documentary Formula & Craft Doctrine

Extracted from empirical analysis of 95+ minutes of top-tier bushcraft, mechanical builds, and architectural documentaries (e.g., Michael, Max the lawyer, Janine):

### 🚨 1. The Anti-Slop Law & Blacklisted AI Vocabulary
- **STRICT PURPLE PROSE BAN**: You are strictly forbidden from using generic AI fluff, poetic padding, or emotional clichés.
  * ❌ **BLACKLISTED WORDS & PHRASES**:
    `testament`, `tapestry`, `alchemy`, `dance of fire`, `fortress from decay`, `sanctuary`, `nature's fury`, `masterclass in`, `silent sentinel`, `symphony`, `whisper of the forest`, `humble beginning`, `battle against the elements`, `relentless march of time`, `beacon of hope`, `unyielding resolve`.
- **COMMENTATOR PERSONA**: You are an **Observant Veteran Craftsman / Mechanical Engineer** standing over the builder's shoulder. Unflinching, practical, curious, grounded.
- **NEVER NARRATE WHAT EYES ALREADY SEE**: Never say *"Now he picks up the axe and cuts wood"*. Reveal hidden physics, material tension, split vectors, moisture drainage, and microscopic trade-offs.
- **THE 4 SECRET COMMENTARY ELEMENTS**:
  1. **Hidden Stakes (The "Why")**: What structural, aerodynamic, or mechanical failure occurs if he is off by 2mm or 2 degrees?
  2. **Directed Attention Hooks**: Actively guide the viewer's gaze (*"Notice where his thumb braces..."*, *"Watch the grain right at the bevel..."*, *"Look at the flex before it shears..."*).
  3. **Material Friction & Behavior**: Respect physical reality (cardboard fluting direction, green timber checking, glue cure speed, blade dulling).
  4. **The ASMR Silence Beat**: 15–25 seconds of pure, unfiltered tool acoustics between commentary bursts.

### ⏱️ 2. The 15-Word Hard Cap per Sentence
- **UNBREAKABLE RULE**: Every spoken sentence MUST contain **15 words or fewer**.
- **Target Average**: 8 to 12 words per sentence.
- **Cadence**: Short, breathless, punchy declaratives. Eliminates AI drone and keeps TTS cadence gripping and cinematic.

### 👤 3. Named Protagonist & Civilian Contrast
- **Human Identity**: Name the builder immediately (e.g., *Michael*, *Max*, *Janine*).
- **Civilian Contrast**: Anchor the physical struggle against their background:
  > *"Michael spent forty hours at an office desk this week."*
  > *"Now he faces fifteen tons of unseasoned pine."*
  > *"Max is a corporate tax lawyer. Out here, legal briefs don't stop ground frost."*

### ⚙️ 4. The Problem-First 4-Step Engineering Loop
Every 60–90 seconds of build footage executes this four-step micro-arc:
1. **The Physical Problem**: What physical force threatens the build? (moisture rot, frost heave, shear stress, wind deflection, lever torque).
2. **Why Amateur Shortcut Fails**: What would a beginner do, and why does physics forbid it?
   * *Amateur shortcut*: *"Nailing the green cedar directly to the sill."*
   * *Physical failure*: *"Green timber shrinks across its diameter. The nails will tear loose within six months."*
3. **The Ingenious Craft Workaround / Reversal**: How does technique and geometry solve it?
   * *Craft workaround*: *"Instead, he carves an interlocking saddle notch with a continuous relief groove to trap air and shed moisture."*
4. **The ASMR Payoff Beat (15–25s)**: Complete narration silence while the tool bites (adze shaving, axe splitting, cold chisel ringing, clay smearing).

### ♻️ 5. Material Foreshadowing & Scrap Recycling
- **Zero Waste Law**: In high-production craft films, materials have memory. Scraps cut early in the build are never forgotten.
- **Recycling Tracking**:
  * Off-cut log ends become foundation footings, mallet heads, or wedges.
  * Shavings and curls become fire starter or insulation chinking.
  * Bent rebar or metal scraps become hinge pins or door latches.
- **Narrative Callbacks**: Highlight how scraps from earlier scenes solve current engineering problems.

### 🏛️ 6. Regional Heritage & Mechanical Reversals
- **Authentic Craft Lexicon**: Use authentic traditional terminology (*Zamlanka*, *Slavic reed weaving*, *drawknife chamfering*, *dovetail scribing*, *counter-levered ridge beam*).
- **Mechanical Reversals**: Turn apparent defects into structural assets (e.g., using a curved, crooked tree trunk as a natural arched roof truss).

### 🔍 7. The Automated Script Inspector & QC Review Protocol
Immediately after drafting each 20-minute chunk script, the AI runs an automated forensic quality audit before presenting the text to the user:
1. **Blacklist Scan**: Flags any blacklisted word (*testament, tapestry, alchemy, dance of fire, fortress from decay, sanctuary, nature's fury, masterclass in, silent sentinel, symphony, whisper of the forest, humble beginning, battle against the elements, relentless march of time*). Replaces with technical physical craft terms.
2. **15-Word Sentence Cap**: Flags and splits any sentence > 15 words into punchy 8–12 word sentences.
3. **Engineering Rigor**: Ensures the Problem $\rightarrow$ Shortcut Failure $\rightarrow$ Craft Workaround $\rightarrow$ ASMR Payoff pattern is followed. Eliminates stating obvious visual actions.
4. **Reference Alignment**: Evaluates style against the master benchmarks (`first exmaple.txt`, `second example.txt`, `3rd exmaple.txt`).
5. **Pacing & Silence Safety**: Checks segment gaps; confirms no silent interval exceeds 40 seconds.
6. **QC Summary Generation**: Summarizes all detected issues and corrected lines in a structured report presented alongside the script.

---

### The 4 Narrative Phases Across the Video Timeline
- **Phase 1: High-Stakes Intro Hook (First 45–90s)**:
  * Name protagonist and establish civilian contrast.
  * Contrast basic survival shelter vs. permanent architectural mastery.
  * Forecast physical enemies: frost heave, timber rot, roof load, caloric depletion.
- **Phase 2: Engineering Body Loops (Minutes 2–18 / 22–38 / 42–58)**:
  * Rapid sequence of 4-Step Engineering Loops.
  * 3-tier pacing (Micro-Observations, Core Loops, Macro Retrospectives).
- **Phase 3: Mid-Build Crisis & Field Pivots**:
  * Frame tool slips, split logs, or sudden downpours as engineering challenges.
  * Quantify the cost (lost calories, daylight, ruined material).
  * Showcase improvised field recovery.
- **Phase 4: Triumphant Outro & Aesthetic Tour (Final 2–3 mins)**:
  * Tempo drops 50% (measured, reflective).
  * Contrast opening mud and rubble with finished architectural marvel.
  * Extended 20–30 second ambient lingering shots with zero narration.

---

## Linguistic Signatures & Sentence Bridges
- **Punchy Declaratives (≤ 15 words)**:
  - *"The geometry is unforgiving."*
  - *"A shelter is just survival."*
  - *"Physics offers no second chances."*
  - *"Good news fades fast."*
  - *"Every millimeter off center compounds under load."*
- **Directed Attention Hooks**:
  - *"Notice where his thumb anchors..."*
  - *"Watch the grain right at the bevel..."*
  - *"Look closely at the flex before the cut shears..."*
  - *"Observe the relief angle on the blade..."*
- **Action & Engineering Verbs**: *harness, anchor, excavate, interlock, notch, mortise, funnel, counter-balance, mitigate, score, bevel, crimp, brace, relieve, scribe*.

---

## Gemini Quota Architecture & Capacity Limits

| Model | Role | Free Tier RPM | Free Tier TPM | Free Tier RPD | Max Output Tokens | Single-Call Max Capacity |
|---|---|---|---|---|---|---|
| **`gemini-2.5-flash`** | Video Analysis (Primary) | **15 RPM** | **1,000,000 TPM** | **1,500 RPD** | 65,536 tokens | ~1 Hour Video Input (1M Context) |
| **`gemini-2.5-pro`** | Video Analysis (Deep Reasoning) | **15 RPM** | **1,000,000 TPM** | **1,500 RPD** | 65,536 tokens | ~1 Hour Video Input (1M Context) |
| **`gemini-3.8-flash-lite-tts`** | Speech Synthesis (Sadachbia) (Primary) | **15 RPM** | **1,000,000 TPM** | **10 RPD (Strict)**| 16,384 tokens | ~655 Seconds (~10.9 Min Audio) |
| **`gemini-3.8-flash-tts`** | Speech Synthesis (Sadachbia) (Alternate) | **10 RPM** | **1,000,000 TPM** | **10 RPD (Strict)**| 16,384 tokens | ~655 Seconds (~10.9 Min Audio) |

### Accurate Token Consumption Formulas
- **Video Input Perception (`gemini-2.5-flash` / `gemini-2.5-pro`)**:
  * ~263 video tokens/sec + ~32 audio tokens/sec ≈ **~295 tokens / second**.
  * 1-minute video ≈ 17,700 tokens | 20-minute chunk ≈ 354,000 tokens (safely within 1M window).
- **Audio Output Synthesis (`gemini-3.8-flash-lite-tts` / `gemini-3.8-flash-tts`)**:
  * Output acoustic compression rate ≈ **~25 audio tokens / second** of speech.
  * 16,384 max output tokens ÷ 25 tokens/sec ≈ **655 seconds (~10.9 minutes)** of continuous audio in **1 API call**.
  * Input text capacity: **8,192 tokens** (~6,000 words).

---

## Audio Muxing Standard (Audible & Balanced)

To guarantee that speech is never muffled, dropped, or silenced by sample rate mismatches:
```bash
ffmpeg -y -i source_clip_360p.mp4 \
  -i seg_1.wav -i seg_2.wav ... \
  -filter_complex "\
    [0:a]volume=0.25[bg]; \
    [1:a]aformat=sample_rates=44100:channel_layouts=stereo,adelay=15000|15000,volume=1.40[v1]; \
    [2:a]aformat=sample_rates=44100:channel_layouts=stereo,adelay=45000|45000,volume=1.40[v2]; \
    [bg][v1][v2]amix=inputs=3:duration=first:dropout_transition=300:normalize=0[aout]" \
  -map 0:v -map "[aout]" -c:v copy -c:a aac -b:a 192k final_output.mp4
```

---

## Single-Call Unified Audio Synthesis & Groq Whisper Split/Sync Pipeline

To minimize API consumption, preserve RPM limits, and maintain frame-accurate audio sync:

### 1. Timestamps & Script Export
When Gemini analyzes each 20-minute chunk and produces the commentary segments, compile them into an ordered list with global starting seconds:
```text
00:08 - Segment 1: High-Stakes Intro Hook (Duration: ~18s)
00:45 - Segment 2: Engineering Body Loop 1 (Duration: ~14s)
01:30 - Segment 3: Engineering Body Loop 2 (Duration: ~15s)
```

### 2. Single-API-Call TTS Synthesis (Default Voice: `Sadachbia`)
Concatenate all segment narration texts into **one single prompt** separated by a silence pause marker (e.g. `\n\n...\n\n`) and call Gemini 3.8 Flash Lite TTS with mandatory voice `Sadachbia` **in a single API call**:
```javascript
const fullNarration = segments.map(s => s.narrationText).join('\n\n...\n\n');
const response = await ai.models.generateContent({
  model: 'models/gemini-3.8-flash-lite-tts', // Alternate: models/gemini-3.8-flash-tts
  contents: fullNarration,
  config: {
    responseModalities: ['AUDIO'],
    speechConfig: {
      voiceConfig: {
        prebuiltVoiceConfig: { voiceName: 'Sadachbia' }
      }
    }
  }
});
fs.writeFileSync('master_commentary_tts.wav', Buffer.from(response.candidates[0].content.parts[0].inlineData.data, 'base64'));
```

### 3. Groq Whisper Word-Boundary Slicing (`whisper-large-v3`)
Rather than relying on fragile silence detection or duration guessing, transcribe `master_commentary_tts.wav` using Groq Whisper with word-level timestamps (`timestamp_granularities[]="word"`):
- Match the first and last words of each segment in the transcript.
- Extract the exact start time (`start_sec`) and end time (`end_sec`) down to millisecond accuracy.
- Slice each segment cleanly via FFmpeg:
```bash
ffmpeg -y -i master_commentary_tts.wav -ss <start_seg1> -to <end_seg1> -c copy seg_1.wav
ffmpeg -y -i master_commentary_tts.wav -ss <start_seg2> -to <end_seg2> -c copy seg_2.wav
ffmpeg -y -i master_commentary_tts.wav -ss <start_seg3> -to <end_seg3> -c copy seg_3.wav
```

### 4. Frame-Accurate Video Synchronization
Using the video start timestamps recorded in the text file, each sliced segment is delayed to its precise start point via FFmpeg `adelay` (where `adelay = timestamp_seconds * 1000`) and mixed with the video background audio (`amix`):
```bash
ffmpeg -y -i input_video.mp4 \
  -i seg_1.wav -i seg_2.wav -i seg_3.wav \
  -filter_complex "\
    [0:a]volume=0.25[bg]; \
    [1:a]aformat=sample_rates=44100:channel_layouts=stereo,adelay=8000|8000,volume=1.40[v1]; \
    [2:a]aformat=sample_rates=44100:channel_layouts=stereo,adelay=45000|45000,volume=1.40[v2]; \
    [3:a]aformat=sample_rates=44100:channel_layouts=stereo,adelay=90000|90000,volume=1.40[v3]; \
    [bg][v1][v2][v3]amix=inputs=4:duration=first:dropout_transition=300:normalize=0[aout]" \
  -map 0:v -map "[aout]" -c:v copy -c:a aac -b:a 192k final_synced_video.mp4
```
This guarantees zero rate-limit waste, exactly 1 TTS call, and perfect millisecond-level sync across the entire video.

---

## Tool Specifications, Capacity Limits & Agent Playbook (For AI Agents)

This dedicated section educates and instructs any AI agent executing this workflow on the exact capabilities, hardware/API boundaries, and strict operational guidelines for every tool in the pipeline.

### Tool 1: Video Analysis Engine (`models/gemini-2.5-flash` | Alternate: `models/gemini-2.5-pro`)
- **Primary Purpose**: Native multimodal video perception, scene transition analysis, and documentary voiceover scripting in user-gated 20-minute chunks.
- **Specifications & Hardware Boundaries**:
  * **Primary Visual Model**: `models/gemini-2.5-flash` (fast multimodal video comprehension).
  * **Deep Reasoning Alternate**: `models/gemini-2.5-pro` (for complex mechanical nuances).
  * ⛔ **PROHIBITED MODEL**: Do NOT use `gemini-3.1-pro-preview`.
  * **Input Context Window**: `1,048,576 tokens` (~1 Million tokens).
  * **Max Output Tokens**: `65,536 tokens`.
  * **Video Token Consumption**: ~263 video tokens/sec + ~32 audio tokens/sec ≈ **~295 tokens / second**.
  * **Chunk Window**: Process in **20-minute chunks** (`00:00-20:00`, `20:00-40:00`, etc.) with user inspection gates between chunks.
  * **Free Tier Quotas**: `15 RPM` (Requests/Min), `1,000,000 TPM` (Tokens/Min), `1,500 RPD` (Requests/Day).
  * **Paid Tier Quotas**: `1,000+ RPM`, `4,000,000 TPM`, unlimited RPD.
- **Agent Guidelines & Playbook**:
  * **DO**: ALWAYS downscale source videos to a `360p` proxy (`scale=-2:360`, CRF 28) using FFmpeg before uploading to Gemini Files API.
  * **DO**: Slice the proxy into 20-minute chunks via FFmpeg (`-ss -t -c copy`), upload chunk to Gemini Files API, analyze with `gemini-2.5-flash` (or `gemini-2.5-pro`), and delete remote file after parsing.
  * **DO**: Run the Script Inspector / QC Reviewer step immediately after drafting each chunk.
  * **DO**: STOP after each chunk and present the written script and QC audit to the user. Await explicit user command before moving to the next chunk.
  * **DO NOT**: Never advance to Chunk $N+1$ without user approval.
  * **DO NOT**: Never scrape individual frames or dump image sequences when Gemini natively ingests video and audio tracks together.
  * **🚨 STRICT MANDATE: ZERO INTERMEDIATE SCOUT JSON**:
    Do NOT ask the model to produce an intermediate JSON log mapping every single physical action and tool second-by-second. Run **Direct Multimodal Script Generation** per 20-minute chunk, embedding the 15-word cap, named protagonist, and 4-step engineering loop directly into the prompt.

---

### Tool 2: Audio Synthesis Engine (`models/gemini-3.8-flash-lite-tts` / `models/gemini-3.8-flash-tts`)
- **Primary Purpose**: High-definition (24 kHz) studio speech synthesis with prebuilt character profiles and multi-speaker capabilities.
- **Specifications & Hardware Boundaries**:
  * **Primary Model**: `models/gemini-3.8-flash-lite-tts` (crisp, high-throughput voice generation).
  * **Alternate Model**: `models/gemini-3.8-flash-tts` (expressive speech synthesis).
  * ⛔ **PROHIBITED MODEL**: Do NOT use `gemini-2.5-flash-preview-tts`.
  * **Max Output Tokens**: `16,384 tokens` per response.
  * **Audio Generation Token Rate**: ~**25 audio tokens per second** of spoken speech.
  * **Single-Call Maximum Audio Length**: `16,384 ÷ 25` ≈ **655 seconds (~10.9 minutes / ~11 minutes)** of continuous audio in **1 API call**.
  * **Max Input Text Tokens**: `8,192 tokens` (~6,000 words).
  * **Free Tier Quotas**: `10 RPM`, `1,000,000 TPM`, **10 RPD (Requests Per Day - Strict Ceiling)**.
  * **Paid Tier Quotas**: `1,000 RPM`, `4,000,000 TPM`, unlimited RPD.
- **Voice Catalog (30 HD Prebuilt Voices)**:
  * **Default Project Voice**: **`Sadachbia`** (MANDATORY: Deep, calm, authoritative, broadcast documentary cadence).
  * **Alternative Voices**: `Puck`, `Fenrir`, `Orus`, `Charon`, `Aoede`, `Kore`, `Zephyr`.
- **Agent Guidelines & Playbook**:
  * **DO**: Generate the ENTIRE video's audio commentary segments in **ONE single API call** (concatenate all segment texts separated by `\n\n...\n\n`) **ONLY AFTER all chunk scripts are reviewed and approved by the user**.
  * **DO**: Keep track of the 10 RPD Free Tier limit. Single-call synthesis prevents key lockouts.
  * **DO NOT**: Never run TTS during chunk scriptwriting. TTS is strictly decoupled until the user finalizes the complete script.
  * **DO NOT**: Never send text scripts exceeding 655 seconds (~1,500 spoken words) in a single request. If total narration exceeds 10.5 minutes, split into 2 requests (Chunk 1 and Chunk 2) and concatenate with FFmpeg.
  * **DO NOT**: Never pass invalid or lowercase voice names. Names are strictly case-sensitive.

---

### Tool 3: Groq Whisper & FFmpeg Synchronization Engine
- **Primary Purpose**: Word-boundary audio slicing, sub-millisecond audio alignment, and balanced multi-track volume ducking.
- **Specifications & Standard Parameters**:
  * **Word-Boundary Slicing**: Use Groq Whisper (`whisper-large-v3`) with `timestamp_granularities[]="word"` on `master_commentary_tts.wav` to extract exact segment millisecond boundaries.
  * **360p Proxy Command**:
    `ffmpeg -y -i input.mp4 -vf "scale=-2:360" -c:v libx264 -crf 28 -preset veryfast -c:a aac -b:a 64k proxy_360p.mp4`
  * **Audio Segment Delay Formula**: `adelay = timestamp_in_seconds * 1000` (e.g. `00:45` start = `45000|45000` ms).
  * **Multi-Input Resampling**: `aformat=sample_rates=44100:channel_layouts=stereo`.
  * **Balanced Audio Levels**:
    - Ambient/Original video background: **`volume=0.25`** (ducked so tools/nature remain audible but never drown out the narrator).
    - Synthesized Commentary Voiceover: **`volume=1.40`** (amplified for broadcast clarity, -16 LUFS target).
  * **Mixing Filter**: `amix=inputs=N:duration=first:dropout_transition=300:normalize=0`.
- **Agent Guidelines & Playbook**:
  * **DO**: Always use `-c:v copy` when muxing final audio back into the original video to guarantee zero visual generation loss and instant rendering speed.
  * **DO**: Suffix delays with `|delay` (e.g., `adelay=4000|4000`) to delay both left and right stereo channels equally.
  * **DO NOT**: Never re-encode 1080p/4K video tracks during the final audio muxing stage (`-c:v copy` only).
  * **DO NOT**: Never mute the background video audio entirely (`volume=0.0`); authentic documentaries rely heavily on tactile ASMR tool sounds between commentary intervals.


