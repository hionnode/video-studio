# Video Editing Studio

Conversation-driven video editing studio. Drop raw footage into a project
folder, describe the edit, and Claude runs the pipeline: cut, grade, subtitle,
add motion graphics, render.

**Read [`architecture.md`](architecture.md) for the full system design and the
standard operating procedure — follow it for every project.**

## System at a glance

10-stage pipeline: **Ingest → Inventory → Transcribe → Pack → Strategy
(user approval gate) → EDL → Animate → Render → Self-eval → Deliver.**

Two toolkits, installed as Claude Code skills:

| Skill | Repo → clone | Role |
|-------|--------------|------|
| `video-use` | browser-use/video-use → `~/Developer/video-use` | Editing pipeline: transcribe, cut, grade, subtitle, render |
| `hyperframes` (+ `gsap`, `lottie`, `three`, `animejs`, `waapi`, `css-animations`, `tailwind`, `typegpu`, `hyperframes-cli/-media/-registry`, `website-to-hyperframes`, `remotion-to-hyperframes`) | heygen-com/hyperframes → `~/Developer/hyperframes` | Motion graphics overlays |

The `video-use` skill (`~/.claude/skills/video-use/SKILL.md`) carries the
detailed editing methodology. Invoke it when starting any edit.

## Conventions

- **One folder per project** under `raw-files/<project>/`. Raw footage is never modified.
- **All outputs** go in `raw-files/<project>/edit/` — never the repo root.
- **Session memory** persists in each project's `edit/project.md`.
- Raw media, `edit/`, and `.env` are git-ignored. Only docs are committed.

## Running video-use helpers

Always use the venv interpreter — system `python` lacks the deps:

```bash
~/Developer/video-use/.venv/bin/python ~/.claude/skills/video-use/helpers/<name>.py ...
```

Helpers: `transcribe.py`, `transcribe_batch.py`, `pack_transcripts.py`,
`timeline_view.py`, `render.py`, `grade.py`.

## Hard rules (never violate)

Subtitles applied LAST · per-segment extract + lossless concat · 30ms audio
fades at boundaries · word-boundary cuts only · cache transcripts · parallel
animation sub-agents · **confirm strategy before executing**. Full list in
`architecture.md` §5.

## Requirements

- ffmpeg + ffprobe 8 ✓ · Node 22+ / bun ✓ · Python + uv ✓ · yt-dlp ✓
- `ELEVENLABS_API_KEY` in `/Users/chinmay/code/video/.env` (project root).
  Key resolution: `~/Developer/video-use/.env` → `./.env` → env var.
- **Transcription needs a *paid* ElevenLabs plan** — the Free Tier is disabled
  on newly-created accounts by the abuse detector. Any paid plan (Starter
  ~$5/mo) works; usage here is well within limits.
