# Watch

`watch` is an Agent Skill for evidence-based video analysis. It combines timestamped speech with representative frames instead of treating a transcript as the whole video.

## What it does

- accepts YouTube and other `yt-dlp` URLs, direct media links, Loom or Zoom shares, and local files;
- prefers existing subtitles and uses local Whisper when subtitles are unavailable;
- samples scene changes and periodic control frames;
- connects narration with slides, code, UI states, charts, diagrams, and demonstrations;
- produces timestamped summaries, reviews, tutorials, meeting notes, or Obsidian-ready notes;
- reports missing access, weak captions, unreadable visuals, and other evidence limits.

## Why speech and frames matter

A transcript misses silent screen activity. Fixed-interval screenshots can miss short events and over-sample static scenes. `watch` uses two evidence channels:

1. Timestamped subtitles or local speech-to-text.
2. Frames from scene changes, plus sparse control frames for long static segments.

The skill labels an analysis as partial when a required channel is unavailable. A narrow transcription or visual-only request uses only the channel it needs.

## Requirements

- A Codex-compatible agent that supports Agent Skills.
- [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) for web metadata, subtitles, and media downloads.
- [`FFmpeg`](https://ffmpeg.org/) for audio extraction and frame sampling.
- [`openai-whisper`](https://github.com/openai/whisper) or a compatible `whisper` CLI for local fallback transcription.
- Enough disk space and compute for the selected video.

For full YouTube support, current `yt-dlp` releases can also require an external JavaScript runtime and EJS components. The `yt-dlp` project recommends Deno. Node and QuickJS are supported alternatives. See the official [`yt-dlp` EJS guide](https://github.com/yt-dlp/yt-dlp/wiki/EJS).

The skill does not install dependencies automatically.

## Install

### Git

PowerShell:

```powershell
git clone https://github.com/lev4ik52/watch-skill.git "$env:USERPROFILE\.codex\skills\watch"
```

macOS or Linux:

```bash
git clone https://github.com/lev4ik52/watch-skill.git ~/.codex/skills/watch
```

Restart or reload the agent after installation.

### ZIP

1. Select **Code → Download ZIP** on GitHub.
2. Extract the archive.
3. Rename the extracted folder to `watch`.
4. Move it to `~/.codex/skills/watch`.
5. Restart or reload the agent.

On Windows, use `%USERPROFILE%\.codex\skills\watch`.

## Verify

The installed directory must contain:

```text
watch/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── pipeline.md
    ├── obsidian.md
    └── troubleshooting.md
```

Try this request:

```text
Analyze this video completely. Give me a timestamped summary, describe important on-screen content, and state any evidence limits: <URL>
```

Automatic selection is enabled. You can ask naturally without naming `$watch`.

## Examples

### Lecture

```text
Watch this lecture. Explain its argument, cite key timestamps, and describe the important diagrams.
```

### Product demo

```text
Review this screen recording. List the actions, UI states, results, errors, and unresolved questions.
```

### Meeting

```text
Analyze this meeting recording. Return decisions, owners, action items, and the timestamps that support them.
```

### Obsidian

```text
Turn this video into an Obsidian source note. Use my Research/Sources folder, keep it concise, and omit the raw transcript.
```

## Privacy and safety

- Media artifacts remain local unless the user asks to send or publish them.
- The skill does not bypass logins, private shares, DRM, paywalls, or site restrictions.
- Browser cookies are not imported without explicit permission.
- Missing dependencies are reported before any installation.
- Obsidian writing requires a known destination, note form, and transcript policy.
- Raw transcripts are omitted by default.

## Limitations

- Site changes or unsupported players can block extraction.
- Automatic captions and speech recognition can mishear names, jargon, and overlapping speech.
- Scene detection is heuristic. Static interfaces and gradual visual changes need control frames.
- Long videos can require substantial bandwidth, storage, compute, and context.
- A summary cannot recover segments that were unavailable or corrupted.

## Update

For a Git installation:

```powershell
git -C "$env:USERPROFILE\.codex\skills\watch" pull --ff-only
```

```bash
git -C ~/.codex/skills/watch pull --ff-only
```

For a ZIP installation, replace the folder after preserving intentional local changes.

## Uninstall

Remove only the `watch` skill directory, then restart or reload the agent. Verify the target path before deletion.

## Repository layout

- `SKILL.md` — runtime decisions and output contract.
- `references/` — conditional procedures and troubleshooting.
- `agents/openai.yaml` — display metadata and invocation policy.
- `AGENTS.md` — maintenance and release guidance.

## License

MIT — see [`LICENSE`](LICENSE).
