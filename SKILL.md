---
name: watch
description: Watch and analyze videos from URLs or local files, including YouTube, Loom, Zoom, screen recordings, demos, lectures, tutorials, and meetings. Use for transcription, timestamped summaries, visual review, or Obsidian intake. For a complete review, combine timestamped speech with representative frames.
---

# Watch

Analyze a video as a time-based audiovisual source. Ground the result in timestamps and visible evidence.

## Required behavior

- Match the evidence channels to the request.
- Inspect every relevant channel before claiming a complete review.
- When speech exists, inspect timestamped subtitles or a timestamped transcription.
- Inspect representative frames, including important visual changes.
- If a required channel is unavailable, label the result as partial and state the limitation.
- Keep downloaded media and derived files in a task-specific temporary directory.
- Do not upload, publish, or send media unless the user asks.
- Respect authentication, privacy, DRM, paywalls, and share permissions.
- Check dependencies before processing. Ask before installing missing software.
- Ask only for Obsidian details that the user has not supplied.
- Return synthesis by default. Include a raw transcript only when requested.

A transcript-only request needs the speech channel but not frame analysis. A visual-only request needs frames but not transcription. Do not describe either narrow result as a complete audiovisual review.

## Workflow

1. Classify the source as a supported page URL, direct media URL, or local file.
2. Determine whether the request needs speech, visuals, or both.
3. Inspect access and metadata before downloading full media.
4. Obtain timestamped subtitles when speech evidence is needed.
5. If subtitles are absent or unusable, transcribe the audio locally.
6. Inspect scene-change frames and sparse control frames when visual evidence is needed.
7. Align visual events with nearby transcript timestamps for an audiovisual review.
8. Produce the format that matches the request.

Read [references/pipeline.md](references/pipeline.md) when extraction, transcription, or frame sampling is required.
Read [references/troubleshooting.md](references/troubleshooting.md) only after an access, extraction, or tool failure.

## Evidence rules

A complete audiovisual review requires both when speech exists:

- timestamped speech evidence when the video contains speech;
- representative visual evidence.

Inspect the beginning and end of the video. Add periodic control frames for long static segments. Do not infer unreadable text.

State these limitations when they occur:

- missing or unreliable subtitles;
- inaccessible or corrupted segments;
- visuals that cannot be decoded or read;
- uncertain names, quotations, or timestamps;
- incomplete access caused by login or site restrictions.

## Output

Adapt the report to the video type. Include:

- a short summary;
- key points with timestamps;
- important on-screen evidence;
- decisions, warnings, or claims;
- uncertainty and useful follow-up questions.

For tutorials, return ordered steps. For demos, return UI states, actions, results, and failures. For meetings, return decisions, owners, and action items. For lectures, return the argument, evidence, and important diagrams.

Do not paste a full transcript into chat unless the user asks.

## Obsidian

Writing a note is a separate external action. Confirm only the details that are missing:

- destination;
- note type and depth;
- transcript policy.

Read [references/obsidian.md](references/obsidian.md) before writing the note.

## Completion

Finish when the source status is known and the required evidence channels were inspected. Combine the evidence, disclose limitations, and deliver the requested result.
