# Architecture — Video Editing Studio

The system design and standard operating procedure for this studio. Read this
before editing; follow it for every project.

---

## 1. Philosophy

The studio is **conversation-driven** and **audio-first**:

- **Audio is primary.** Cuts are reasoned from the transcript and silence gaps.
  Video is inspected only at decision points, never scanned frame-by-frame.
- **LLM reasons from text.** The only derived artifact that earns its keep is a
  phrase-level packed transcript. Filler tagging, take selection, emphasis
  scoring — all derived at decision time, never precomputed.
- **Ask → confirm → execute → iterate → persist.** No cut is touched until the
  user approves a plain-English strategy.
- **Outputs are isolated.** Every artifact lands in a project's `edit/` folder.
  Raw sources are never modified; the repo root stays clean.

---

## 2. The pipeline

Ten stages. Stages 1–4 are mechanical (run them every time). Stage 5 is the
human gate. Stages 6–10 execute the approved plan.

```
 ┌─────────┐  ┌───────────┐  ┌────────────┐  ┌──────┐  ┌──────────┐
 │1 INGEST │─▶│2 INVENTORY│─▶│3 TRANSCRIBE│─▶│4 PACK│─▶│5 STRATEGY│
 └─────────┘  └───────────┘  └────────────┘  └──────┘  └────┬─────┘
                                                       user approves
 ┌──────────┐  ┌────────────┐  ┌────────┐  ┌─────────┐      │
 │10 DELIVER│◀─│9 SELF-EVAL │◀─│8 RENDER│◀─│6 EDL +  │◀─────┘
 └──────────┘  └────────────┘  └────────┘  │7 ANIMATE│
                                           └─────────┘
```

| # | Stage | What happens | Tooling |
|---|-------|--------------|---------|
| 1 | **Ingest** | Place raw file(s) in `raw-files/<project>/`. URLs pulled via yt-dlp. | `yt-dlp` |
| 2 | **Inventory** | `ffprobe` every source — codec, resolution, fps, duration, audio layout. | `ffprobe` |
| 3 | **Transcribe** | ElevenLabs Scribe → word-level JSON with timestamps + diarization. Cached per source. | `transcribe.py` / `transcribe_batch.py` |
| 4 | **Pack** | JSON → `takes_packed.md`, phrase-level transcript broken on silence ≥0.5s. | `pack_transcripts.py` |
| 5 | **Strategy** | Pre-scan transcript for slips; converse; propose a 4–8 sentence plan. **Wait for user approval.** | conversation |
| 6 | **EDL** | Editor sub-agent picks takes/cuts → `edl.json` (word-boundary, padded). | `Agent` tool |
| 7 | **Animate** | Motion graphics built in parallel sub-agents → overlay clips. | `hyperframes` skill, PIL, Manim, Remotion |
| 8 | **Render** | Per-segment extract → grade → lossless concat → overlays → subtitles LAST. | `render.py`, `grade.py` |
| 9 | **Self-eval** | `timeline_view` every cut boundary on the *rendered output*; check cuts, pops, hidden subs, overlay sync. ≤3 passes. | `timeline_view.py` |
| 10 | **Deliver** | Present `preview.mp4`; on approval render `final.mp4`; append to `project.md`. | `render.py` |

---

## 3. Per-project layout

One folder per project under `raw-files/`. All outputs go in its `edit/`:

```
raw-files/<project>/
├── <source files>            ← raw footage, never modified
└── edit/                     ← all outputs (git-ignored)
    ├── project.md             session memory — appended every session
    ├── takes_packed.md        phrase-level transcript (primary reading view)
    ├── edl.json               edit decision list — cut decisions
    ├── transcripts/<name>.json cached raw Scribe output
    ├── animations/slot_<id>/   per-animation source + render
    ├── clips_graded/           per-segment extracts with grade + fades
    ├── master.srt              output-timeline subtitles
    ├── verify/                 self-eval timeline PNGs
    ├── preview.mp4             720p draft
    └── final.mp4               delivered output
```

---

## 4. Helper reference

Always run with the venv interpreter:
`~/Developer/video-use/.venv/bin/python ~/.claude/skills/video-use/helpers/<name>.py`

| Helper | Purpose |
|--------|---------|
| `transcribe.py <video>` | Single-file Scribe transcription, cached |
| `transcribe_batch.py <dir>` | 4-worker parallel transcription for multi-take |
| `pack_transcripts.py --edit-dir <dir>` | JSON transcripts → `takes_packed.md` |
| `timeline_view.py <video> <start> <end>` | Filmstrip + waveform PNG at a decision point |
| `render.py <edl.json> -o <out>` | Extract → concat → overlays → subtitles |
| `grade.py <in> -o <out>` | ASC CDL color grade (presets or custom filter) |

Motion graphics use the `hyperframes` skill (HTML/CSS/GSAP → MP4/WebM) and its
adapters; Manim ships bundled in video-use at `skills/manim-video/`.

---

## 5. Hard rules (production correctness — non-negotiable)

Deviation here causes silent failure, not bad taste:

1. **Subtitles applied LAST**, after all overlays.
2. **Per-segment extract → lossless `-c copy` concat** — never single-pass filtergraph with overlays.
3. **30ms audio fades** at every segment boundary.
4. **Overlays use `setpts=PTS-STARTPTS+T/TB`** to align frame 0 to window start.
5. **Master SRT uses output-timeline offsets.**
6. **Never cut inside a word** — snap to transcript word boundaries.
7. **Pad every cut edge** 30–200ms (absorbs ASR drift).
8. **Word-level verbatim ASR only** — never phrase mode / normalized fillers.
9. **Cache transcripts** — never re-transcribe an unchanged source.
10. **Parallel sub-agents** for multiple animations — never sequential.
11. **Strategy confirmed before execution.**
12. **All outputs in `edit/`** — never write to the repo root.

Everything outside these rules — pacing, look, style, length — is a taste call
driven by the material and the user.

---

## 6. Standard operating procedure

For any new project:

1. Drop footage in `raw-files/<project>/`.
2. Run inventory (`ffprobe`) and transcription on every source.
3. Pack transcripts; pre-scan for verbal slips.
4. Describe what you see; ask material-specific questions; propose a strategy.
5. **Stop. Get explicit approval.**
6. Build `edl.json`, animations (parallel), grade, render a preview.
7. Self-evaluate against the hard rules; fix; re-render (≤3 passes).
8. Present preview → iterate on feedback → render `final.mp4`.
9. Append the session to `edit/project.md`.

---

## 7. Project log

| Project | Source | Duration | Status |
|---------|--------|----------|--------|
| `dossier/andaman` | Great Nicobar Project (Hindi doc, 4K AV1, Opus stereo) | 27.6 min | Master delivered (`edit/master.mp4`, 15:52, English subs + 4 map overlays); 4 vertical shorts in progress. See its `edit/project.md`. |

> **ElevenLabs account note:** transcription requires a *paid* ElevenLabs plan.
> The Free Tier is disabled on freshly-created accounts by the abuse detector
> (`detected_unusual_activity`). Any paid subscription (Starter ~$5/mo) lifts it.
> Disable any VPN before subscribing.

Per-project decisions and session history live in each project's
`edit/project.md`.
