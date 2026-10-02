# Video processing pipeline

Read this file when the task requires metadata extraction, subtitles, transcription, media download, or frame sampling.

## 1. Create a task directory

Use a unique directory inside the current workspace or a system temporary directory. Do not reuse output from another video unless the user requests it.

PowerShell:

```powershell
New-Item -ItemType Directory -Force -Path "work/watch/<slug>"
```

POSIX shell:

```bash
mkdir -p "work/watch/<slug>"
```

## 2. Check tools

PowerShell:

```powershell
Get-Command yt-dlp, ffmpeg, ffprobe, whisper -ErrorAction SilentlyContinue
```

POSIX shell:

```bash
for tool in yt-dlp ffmpeg ffprobe whisper; do command -v "$tool" || echo "missing: $tool"; done
```

Ask before installing a missing tool. Whisper is optional when usable subtitles already exist.

## 3. Inspect metadata

For a supported URL, inspect metadata before downloading full media:

```bash
yt-dlp --no-playlist --skip-download --dump-single-json "<URL>"
```

Record the canonical URL, title, uploader, duration, upload date, language, and available subtitles. For a local file, inspect streams and duration:

```bash
ffprobe -v error -show_format -show_streams -of json "<VIDEO_FILE>"
```

## 4. Obtain subtitles

Prefer creator subtitles, then automatic subtitles:

```bash
yt-dlp --no-playlist --skip-download --write-subs --write-auto-subs --sub-langs "ru,en" --convert-subs srt -o "work/watch/<slug>/subs.%(ext)s" "<URL>"
```

Inspect the produced files. Do not assume that a requested language exists. Preserve timestamps.

## 5. Transcribe when needed

For a URL:

```bash
yt-dlp --no-playlist -x --audio-format wav -o "work/watch/<slug>/audio.%(ext)s" "<URL>"
whisper "work/watch/<slug>/audio.wav" --model small --output_format srt --output_dir "work/watch/<slug>"
```

For a local file:

```bash
ffmpeg -i "<VIDEO_FILE>" -vn -ar 16000 -ac 1 -c:a pcm_s16le "work/watch/<slug>/audio.wav"
whisper "work/watch/<slug>/audio.wav" --model small --output_format srt --output_dir "work/watch/<slug>"
```

Specify `--language` only when the language is known. State uncertainty when names, jargon, or overlapping speakers reduce accuracy.

## 6. Obtain the video path

For a URL, cap resolution unless the task requires higher visual fidelity:

```bash
yt-dlp --no-playlist -f "bv*[height<=720]+ba/b[height<=720]" --print after_move:filepath -o "work/watch/<slug>/video.%(ext)s" "<URL>"
```

Use the actual emitted path for later commands. The final container can be MP4, WebM, or MKV. If output is ambiguous, inspect the task directory instead of predicting the extension.

## 7. Sample scene changes

Use the actual media path:

```bash
ffmpeg -i "<VIDEO_FILE>" -vf "select='gt(scene,0.30)',showinfo" -fps_mode vfr "work/watch/<slug>/scene_%05d.png" 2> "work/watch/<slug>/scenes.log"
```

Treat `0.30` as a starting point. Lower it for subtle slide or UI changes. Raise it for noisy footage or rapid animation.

## 8. Add control frames

Scene detection can miss gradual changes and long static segments. Add sparse control frames:

```bash
ffmpeg -i "<VIDEO_FILE>" -vf "fps=1/120" "work/watch/<slug>/control_%05d.png"
```

Adjust the interval to the duration and visual density. Inspect the first and last portions of the video even when no scene change is detected.

## 9. Reduce and inspect

Remove near-duplicates from the review set. Prefer frames with readable slides, UI changes, code, diagrams, charts, warnings, errors, or before-and-after states.

Inspect the timestamped transcript and representative frames together. Align visual events with nearby transcript timestamps. Mark any alignment that is approximate.

## 10. Retain or clean outputs

Keep artifacts until the requested result is verified. Do not delete user files. Remove temporary artifacts only when deletion is authorized and the target path is verified.
