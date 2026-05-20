# Video Editing Studio

This directory is a conversation-driven video editing studio. Drop raw footage
into a subfolder, then ask Claude to edit it — cut, grade, subtitle, add motion
graphics, render.

## Installed pipeline

Two open-source toolkits are installed as Claude Code skills (in `~/.claude/skills/`):

| Skill | Repo | Role |
|-------|------|------|
| `video-use` | browser-use/video-use → `~/Developer/video-use` | Editing pipeline: transcribe → cut fillers/shorts → color grade → subtitles → render |
| `hyperframes` (+ `gsap`, `lottie`, `three`, `animejs`, `waapi`, `css-animations`, `tailwind`, `typegpu`, `hyperframes-cli`, `hyperframes-media`, `hyperframes-registry`, `website-to-hyperframes`, `remotion-to-hyperframes`) | heygen-com/hyperframes → `~/Developer/hyperframes` | Motion graphics: HTML/CSS/GSAP compositions rendered to MP4/WebM overlays |

Both are git clones; `cd` into the repo and `git pull --ff-only` to update.

## Running video-use helpers

The video-use Python deps live in a venv. **Always invoke helpers with the venv
interpreter**, not the system `python`:

```bash
~/Developer/video-use/.venv/bin/python ~/.claude/skills/video-use/helpers/<name>.py ...
```

Helpers: `transcribe.py`, `transcribe_batch.py`, `pack_transcripts.py`,
`timeline_view.py`, `render.py`, `grade.py`.

## Workflow (per the video-use skill)

1. Drop raw file(s) into a subfolder of this directory.
2. Ask: *"edit these into a launch video"* / *"cut a short from this"* /
   *"inventory these takes and propose a strategy."*
3. Claude inspects footage, transcribes, proposes a plan in plain language, and
   **waits for confirmation** before editing.
4. All outputs land in `<subfolder>/edit/` — sources and this directory stay clean.
5. Session memory persists in `<subfolder>/edit/project.md`.

Motion graphics are built via the `hyperframes` skill as overlay clips, composited
by `render.py` (subtitles applied last).

## Requirements (all verified present)

- ffmpeg + ffprobe (8.0) ✓
- Node.js 24 / bun (for HyperFrames) ✓
- yt-dlp (for URL sources) ✓
- `ELEVENLABS_API_KEY` in `/Users/chinmay/code/video/.env` (project root) — **required for transcription**. Helpers resolve the key from `~/Developer/video-use/.env`, then `./.env`, then the env var.
