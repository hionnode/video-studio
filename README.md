# 🎬 Video Editing Studio

A **conversation-driven video editing studio**. Drop raw footage into a folder,
tell Claude what you want, and get back a finished, graded, subtitled cut —
complete with motion graphics. No timelines, no menus, no presets: you describe
the edit in plain language, Claude proposes a plan, you approve, it renders.

---

## What it can do

| Stage | Capability |
|-------|-----------|
| **Ingest** | Local files or URLs (`yt-dlp`); auto `ffprobe` inventory of every source |
| **Transcribe** | Word-level timestamps + speaker diarization via ElevenLabs Scribe |
| **Cut** | Remove fillers (`um`, `uh`), false starts, dead air; assemble best takes; extract shorts |
| **Color grade** | ASC CDL grading — cinematic presets or custom ffmpeg filter chains |
| **Subtitle** | Burn-in captions, customizable chunking/case/placement |
| **Motion graphics** | HTML/CSS/GSAP overlays — title cards, kinetic type, data viz, transitions |
| **Render** | Per-segment extract + lossless concat + overlay compositing, self-evaluated at every cut |

The whole loop is **audio-first**: cuts are reasoned from the transcript and
silence gaps, with visual inspection only at decision points.

---

## How it works

Two open-source toolkits power the studio, installed as Claude Code skills:

```
  raw footage ──▶  video-use  ──▶  cut · grade · subtitle  ──┐
                  (editing pipeline)                          ├──▶  final.mp4
                   hyperframes  ──▶  motion-graphics overlays ─┘
                  (HTML/CSS/GSAP render engine)
```

- **[video-use](https://github.com/browser-use/video-use)** — the editing
  pipeline. A `SKILL.md` plus Python `helpers/` (`transcribe`, `render`,
  `grade`, `timeline_view`, `pack_transcripts`).
- **[hyperframes](https://github.com/heygen-com/hyperframes)** — the motion
  graphics engine. HTML compositions rendered to MP4/WebM overlay clips, with
  13 animation-adapter skills (`gsap`, `lottie`, `three`, `animejs`, …).

Production-correctness rules (word-boundary cuts, 30ms audio fades, subtitles
applied last, no double-encoding) are enforced as hard rules; everything else —
pacing, look, style — is creative freedom.

---

## Quick start

> First time on this machine? Follow [`setup-guide.md`](setup-guide.md) to
> install the toolkits and configure the ElevenLabs key.

1. **Add footage** — drop file(s) into [`raw-files/`](raw-files/).
2. **Start Claude** in this directory.
3. **Ask** for what you want, e.g.:
   - *"inventory these takes and propose an edit strategy"*
   - *"cut three 30–60s shorts from the best moments"*
   - *"edit this into a 90s launch video with motion graphics for the key stats"*
4. **Review the plan** Claude proposes — it waits for your OK before editing.
5. **Collect output** from `<folder>/edit/final.mp4`.

---

## Project structure

```
video/
├── README.md          ← you are here
├── setup-guide.md     ← install + configuration
├── CLAUDE.md          ← context loaded by Claude every session
├── .gitignore         ← excludes raw-files/, edit/, media, .env
└── raw-files/         ← drop raw footage here (git-ignored)
    └── <project>/
        └── edit/      ← all outputs land here (git-ignored)
            ├── project.md        session memory
            ├── takes_packed.md   phrase-level transcript
            ├── edl.json          edit decision list
            ├── animations/       motion-graphics slots
            ├── preview.mp4
            └── final.mp4
```

Raw footage and rendered output are **never committed** — only docs, config, and
project notes are tracked.

---

## Requirements

ffmpeg 8 · Node.js 22+ · Python 3.10+ with uv · yt-dlp (optional) · an
ElevenLabs API key with **Speech-to-Text** access. See
[`setup-guide.md`](setup-guide.md) for details.

---

## Credits

Built on [browser-use/video-use](https://github.com/browser-use/video-use)
(Apache 2.0) and [heygen-com/hyperframes](https://github.com/heygen-com/hyperframes)
(Apache 2.0).
