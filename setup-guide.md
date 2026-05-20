# Video Editing Studio — Setup Guide

A conversation-driven video editing studio: drop in raw footage, ask Claude to
edit it, and get back a finished cut with motion graphics. This guide covers a
fresh setup on a new machine and day-to-day usage. For an overview of the
project, see [`README.md`](README.md).

---

## 1. What's installed

Two open-source toolkits, registered as Claude Code skills in `~/.claude/skills/`:

| Skill(s) | Source repo | Role |
|----------|-------------|------|
| `video-use` | [browser-use/video-use](https://github.com/browser-use/video-use) | Editing pipeline: transcribe → cut fillers/shorts → color grade → subtitles → render |
| `hyperframes` + 13 adapters | [heygen-com/hyperframes](https://github.com/heygen-com/hyperframes) | Motion graphics: HTML/CSS/GSAP compositions rendered to MP4/WebM overlays |

The hyperframes adapters: `gsap`, `lottie`, `three`, `animejs`, `waapi`,
`css-animations`, `tailwind`, `typegpu`, `hyperframes-cli`, `hyperframes-media`,
`hyperframes-registry`, `website-to-hyperframes`, `remotion-to-hyperframes`.
Manim (for diagrams/equations) ships bundled inside video-use at
`skills/manim-video/`.

---

## 2. Prerequisites

All verified present on this machine; install only what's missing.

| Tool | Why | Install (macOS) |
|------|-----|-----------------|
| ffmpeg + ffprobe | All cutting, grading, rendering | `brew install ffmpeg` |
| Node.js 22+ | HyperFrames render engine | `brew install node` |
| Python 3.10+ + uv | video-use helpers | `brew install uv` |
| yt-dlp | Pull sources from URLs (optional) | `brew install yt-dlp` |
| ElevenLabs API key | Scribe transcription (required) | See §5 below |

---

## 3. Fresh-machine install

```bash
# 1. Clone both toolkits to a stable location
mkdir -p ~/Developer
git clone https://github.com/browser-use/video-use   ~/Developer/video-use
git clone https://github.com/heygen-com/hyperframes   ~/Developer/hyperframes

# 2. Install video-use Python deps into a venv
cd ~/Developer/video-use
uv venv
uv pip install requests librosa matplotlib pillow numpy

# 3. Register skills with Claude Code
mkdir -p ~/.claude/skills
ln -sfn ~/Developer/video-use ~/.claude/skills/video-use
for d in ~/Developer/hyperframes/skills/*/; do
  n=$(basename "$d")
  [ "$n" = "contribute-catalog" ] && continue
  ln -sfn "$d" ~/.claude/skills/"$n"
done

# 4. Add the ElevenLabs API key (see §5 for how to create the key)
printf 'ELEVENLABS_API_KEY=%s\n' 'YOUR_KEY_HERE' > /Users/chinmay/code/video/.env
chmod 600 /Users/chinmay/code/video/.env
```

> Symlink the **whole** video-use repo (not just `SKILL.md`) — its `helpers/`
> scripts must sit beside `SKILL.md`.

---

## 4. Running video-use helpers

Python deps live in the venv, **not** system Python. Always invoke helpers with
the venv interpreter:

```bash
~/Developer/video-use/.venv/bin/python ~/.claude/skills/video-use/helpers/<name>.py ...
```

Helpers: `transcribe.py`, `transcribe_batch.py`, `pack_transcripts.py`,
`timeline_view.py`, `render.py`, `grade.py`.

---

## 5. ElevenLabs API key

Transcription runs on ElevenLabs **Scribe**. Without a key, nothing transcribes.

### Creating the key

At <https://elevenlabs.io/app/settings/api-keys> → **Create API Key**:

| Setting | Value |
|---------|-------|
| **Restrict Key** | ON |
| **Speech to Text** | **Access** ✅ — the only endpoint this studio needs |
| All other endpoints (TTS, Speech to Speech, Dubbing, Voices, Projects, History, …) | **No Access** |
| **Monthly limit** | Set a credit cap rather than Unlimited — limits blast radius if the key leaks |

> If you later use the `hyperframes-media` skill for AI **voiceover/narration**,
> also grant **Text to Speech → Access**. Not needed for editing/transcription.

### Storing the key — security

Never paste the key into a chat or commit it. Write it to a `.env` file
yourself, from your own terminal. Put it in **this project root** — the
video-use helpers check `./.env` (current working directory) in addition to the
toolkit repo, so a project-local key keeps it scoped to this studio:

```bash
printf 'ELEVENLABS_API_KEY=%s\n' 'sk_your_key_here' > /Users/chinmay/code/video/.env
chmod 600 /Users/chinmay/code/video/.env
```

`.env` is git-ignored. Claude verifies the key with a quota-free `GET /v1/user`
call (expects HTTP `200`) without ever reading the value.

> Key resolution order (`helpers/transcribe.py`): `~/Developer/video-use/.env`
> → `./.env` (project root) → `ELEVENLABS_API_KEY` env var.

---

## 6. Daily usage

1. Drop raw file(s) into `raw-files/` (or any subfolder of this directory).
2. Start Claude here and say what you want:
   - *"inventory these takes and propose an edit strategy"*
   - *"cut three 30–60s shorts from the best moments"*
   - *"edit this into a 90s launch video with motion graphics for the key stats"*
3. Claude inspects the footage, transcribes, and proposes a plan in plain
   language — **it waits for your confirmation before editing**.
4. Outputs land in `<subfolder>/edit/` — raw sources and the repo stay clean.
5. Session memory persists in `<subfolder>/edit/project.md` across sessions.

Motion graphics are built via the `hyperframes` skill as overlay clips and
composited by `render.py` (subtitles are always applied last).

---

## 7. Repo conventions

This directory is a git repo. `.gitignore` excludes:

- `raw-files/` — all raw footage
- `edit/` (anywhere) — transcripts, EDLs, renders, previews
- media files (`*.webm/.mp4/.mov/.mkv/.mp3/.wav`)
- `.env` and `.DS_Store`

Commit docs, config, and project notes — never footage or rendered output.

---

## 8. Keeping toolkits current

```bash
cd ~/Developer/video-use  && git pull --ff-only   # re-run uv pip install if deps changed
cd ~/Developer/hyperframes && git pull --ff-only
```

The symlinks pick up updates automatically on the next run.
