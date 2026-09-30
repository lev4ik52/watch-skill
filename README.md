# Watch — a video analysis skill for AI agents

`watch` teaches Codex-compatible agents to analyze a video as a time-based, audiovisual source rather than merely summarize its title or transcript. It obtains metadata, prefers existing subtitles, falls back to local Whisper transcription, extracts representative frames at scene changes, and combines speech with what is visible on screen.

The repository is the complete distributable skill. The agent entrypoint is [`SKILL.md`](SKILL.md); [`AGENTS.md`](AGENTS.md) explains the design and invariants for maintainers and other AI systems.

## What it does

- Accepts YouTube and other `yt-dlp` URLs, Loom or Zoom shares, direct video links, and local video files.
- Preserves timestamps through subtitles or local Whisper transcription.
- Samples meaningful visual changes instead of blindly taking one frame every N seconds.
- Connects narration with slides, code, UI, charts, diagrams, and demonstrations.
- Produces a concise report with a TL;DR, timestamped concepts, visual evidence, notable claims, and open questions.
- Can prepare a distilled Obsidian note, but requires the user's confirmation before writing it.

## Why transcript plus frames matters

A transcript alone misses silent screen activity: a presenter can click through a UI, reveal code, show a chart, or demonstrate an error without narrating every detail. Fixed-interval screenshots can miss short but important moments and waste attention on static scenes. The skill therefore uses two evidence channels:

1. **Audio/text:** subtitles when available, local speech-to-text otherwise.
2. **Visuals:** frames selected around scene changes, then reduced to a representative set if necessary.

An agent may say it watched the video only after inspecting both the timestamped text and representative visuals.

## Requirements

- A Codex-compatible agent that supports skills stored as folders containing `SKILL.md`.
- [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) for URL metadata, subtitles, and downloads.
- [`FFmpeg`](https://ffmpeg.org/) for audio extraction and scene detection.
- [`openai-whisper`](https://github.com/openai/whisper) or a compatible `whisper` CLI for local fallback transcription.
- Enough disk space for temporary audio/video and enough compute for transcription.

The skill does not install dependencies automatically. The agent must report missing tools and ask the user before installing anything.

## Install

### Git clone

PowerShell:

```powershell
git clone https://github.com/lev4ik52/watch-skill.git "$env:USERPROFILE\.codex\skills\watch"
```

macOS/Linux:

```bash
git clone https://github.com/lev4ik52/watch-skill.git ~/.codex/skills/watch
```

Restart or reload your agent so it rediscovers installed skills.

### Download ZIP

1. On GitHub, choose **Code → Download ZIP**.
2. Extract the archive.
3. Rename the extracted folder to `watch`.
4. Move it to `~/.codex/skills/watch` (on Windows: `%USERPROFILE%\.codex\skills\watch`).
5. Restart or reload the agent.

### Verify

The installed path should contain at least:

```text
~/.codex/skills/watch/
├── SKILL.md
└── agents/
    └── openai.yaml
```

Then try:

```text
Use $watch to analyze https://example.com/video and give me a timestamped summary that includes what appears on screen.
```

## Usage examples

```text
Use $watch to watch this YouTube lecture. Explain its argument, cite key timestamps, and describe the important diagrams.
```

```text
Use $watch to review this local screen recording. Identify the steps demonstrated, UI states, errors, and unresolved questions.
```

```text
Use $watch to turn this video into an Obsidian source note. Ask me where and how to store it before writing anything.
```

The skill also allows automatic discovery because `agents/openai.yaml` sets `allow_implicit_invocation: true`.

## Privacy and safety

- Video artifacts stay in a temporary workspace unless the user explicitly asks to publish or send them.
- The skill does not bypass logins, private shares, paywalls, or site restrictions.
- It does not claim to have watched a video when only metadata or a transcript was available.
- It does not write to Obsidian until the user confirms the destination, note type/depth, and transcript policy.
- Raw transcripts are omitted from chat by default because they are often long and may contain sensitive material.

## How agent skills are structured

A portable skill is a directory whose required entrypoint is `SKILL.md`:

```text
watch/
├── SKILL.md              # YAML discovery metadata + operational instructions
├── agents/
│   └── openai.yaml       # UI metadata and invocation policy
├── AGENTS.md             # maintainer/AI implementation guide
├── README.md             # human-facing documentation and installation
└── LICENSE               # redistribution terms
```

Good skill packaging follows progressive disclosure:

- The frontmatter `name` and `description` make discovery cheap and precise.
- `SKILL.md` contains the decisions, constraints, workflow, and output contract the agent needs during a task.
- Optional supporting files hold detailed material that should not consume context on every invocation.
- Scripts belong in a skill only when deterministic, reusable automation materially improves reliability.
- Assets are for files copied into generated output, not extra prose.

Keep a skill focused. Do not repeat generic agent advice, do not silently broaden permissions, and make high-risk or irreversible steps explicit.

## Updating

If installed with Git:

```powershell
git -C "$env:USERPROFILE\.codex\skills\watch" pull --ff-only
```

If installed from ZIP, replace the folder with the new release after preserving any local edits.

## Uninstall

Remove the `~/.codex/skills/watch` folder and restart or reload the agent. Review the target path carefully before deleting it.

## Limitations

- Site changes, DRM, private access, or unsupported players may prevent extraction.
- Automatic subtitles and speech recognition can mishear names, jargon, or overlapping speech.
- Scene detection is a heuristic; very static videos or rapid edits may need a different threshold.
- Long videos can require substantial download time, disk space, transcription time, and model context.
- The commands in `SKILL.md` are practical defaults, not guarantees for every codec, operating system, or website.

## License

MIT — see [`LICENSE`](LICENSE).
