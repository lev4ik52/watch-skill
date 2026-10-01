---
name: watch
description: Watch and analyze full videos from URLs or local files. Use for YouTube, Loom, Zoom recordings, direct MP4 links, screen recordings, demos, lectures, tutorials, or when the user asks to watch, transcribe, summarize, analyze, review, or add a video to Obsidian. Prefer subtitles first, fall back to local Whisper, and sample frames by scene changes so visual content on screen is included.
---

# Watch

Use this skill when the user gives a video URL or local video and asks Codex to watch, transcribe, summarize, analyze, review, or prepare it for Obsidian.

## Rules

- Do not claim the video was watched unless transcript/subtitles and representative frames were actually inspected.
- Work in a temporary folder, preferably `work/watch/<slug>` in the current Codex workspace. Do not clutter the project.
- Never publish, upload, or send files outside the machine unless the user explicitly asks.
- Respect site access, privacy, and paywall/login limits. If access requires a login or permission, tell the user.
- For Obsidian, always ask before writing: where to place the note, what style/depth to use, and whether to include raw transcript. Do not write to Obsidian without confirmation.
- For most Obsidian video intake, store distilled knowledge and links, not raw transcript, unless the user asks for raw transcript.

## First-use dependency setup

Before processing the first video on a machine, verify all three required command-line tools. Do not skip this bootstrap step:

```powershell
Get-Command yt-dlp
Get-Command ffmpeg
Get-Command whisper
```

On macOS or Linux, use `command -v yt-dlp ffmpeg whisper` instead.

The required tools are:

- `yt-dlp`: video metadata, subtitles, and audio/video downloads.
- `ffmpeg`: audio extraction and scene-change frame sampling.
- `whisper`: local transcription fallback when subtitles are missing.

If any command is missing:

1. Tell the user exactly which tools are absent and ask permission to install them and, when necessary, persist their executable directories in the user's `PATH`.
2. Install only the missing tools using an appropriate trusted package manager or the tool's official distribution method.
3. Make each executable discoverable from future terminals. Prefer a persistent **user-level** `PATH` entry; do not silently change the system-wide `PATH`.
4. Start a fresh shell when required, rerun the checks above, and verify each tool with `yt-dlp --version`, `ffmpeg -version`, and `whisper --help`.
5. Do not begin video processing until all three commands resolve successfully. Once they pass, do not reinstall them on later invocations; only repeat the lightweight availability check.

If YouTube warns that no supported JavaScript runtime was found, use the bundled Codex Node runtime when available:

```powershell
yt-dlp --js-runtimes "node:$env:USERPROFILE\Documents\Codex\tools\node-v24.17.0-win-x64\node.exe" --skip-download --print "%(title)s | %(duration)s" "<URL>"
```

## Workflow

### 1. Prepare

Create a working folder:

```powershell
New-Item -ItemType Directory -Force -Path "work/watch/<slug>"
```

Parse the input as one of:

- YouTube or other URL supported by `yt-dlp`
- Loom or Zoom share link
- direct video URL
- local video file

### 2. Metadata

For URLs, inspect metadata before downloading the full video:

```powershell
yt-dlp --skip-download --dump-json "<URL>"
```

Use the title, uploader/channel, duration, upload date, webpage URL, and available subtitles if present.

### 3. Transcript, Preferred Path

Try subtitles first:

```powershell
yt-dlp --write-subs --write-auto-subs --sub-langs "ru,en" --skip-download --convert-subs srt -o "work/watch/<slug>/subs.%(ext)s" "<URL>"
```

If an `.srt` file appears, use it as the timestamped transcript.

### 4. Transcript, Whisper Fallback

If there are no usable subtitles, extract audio and transcribe locally:

```powershell
yt-dlp -x --audio-format mp3 -o "work/watch/<slug>/audio.%(ext)s" "<URL>"
whisper "work/watch/<slug>/audio.mp3" --model small --language ru --output_format srt --output_dir "work/watch/<slug>"
```

For a local video file:

```powershell
ffmpeg -i "<VIDEO_FILE>" -vn -acodec mp3 "work/watch/<slug>/audio.mp3"
whisper "work/watch/<slug>/audio.mp3" --model small --language ru --output_format srt --output_dir "work/watch/<slug>"
```

If the language is unknown, omit `--language ru`.

### 5. Frames By Scene Change

Sample frames by scene changes, not a fixed timer:

```powershell
yt-dlp -f "bv*[height<=720]+ba/b[height<=720]" -o "work/watch/<slug>/video.%(ext)s" "<URL>"
ffmpeg -i "work/watch/<slug>/video.mp4" -vf "select='gt(scene,0.3)',showinfo" -vsync vfr "work/watch/<slug>/frame_%04d.png" 2> "work/watch/<slug>/scenes.log"
```

For local files, run the same `ffmpeg` command against the local video.

If too many frames are produced, keep about 150 most informative frames by spacing nearby duplicates and preferring major scene changes from `showinfo`.

### 6. Analyze

Read the transcript with timestamps and inspect representative frames. Connect what is said with what is visible on screen:

- slides, code, UI, diagrams, demos, charts, terminal output
- key timestamps and visual moments
- important quotes or claims with timestamps
- gaps where visuals add information missing from speech

### 7. Report

Return:

- TL;DR in 3-5 lines
- key concepts with timestamps
- what is shown on screen and why it matters
- notable quotes, warnings, or decisions with timestamps
- open questions or follow-ups

Do not paste a full raw transcript into chat unless the user asks.

### 8. Obsidian Intake

Before writing anything to Obsidian, ask:

1. Which vault/folder or existing note should receive this?
2. Should this be a lesson, source note, prompt/checklist, tool note, comparison, project note, or something else?
3. Should raw transcript be included, linked, or omitted?

After confirmation, use the available Obsidian workflow. Write concise Markdown with source URL, timestamps, useful summary, links to related notes, and only the transcript detail the user requested.
