# Guide for AI agents and maintainers

This repository contains the `watch` agent skill. Treat `SKILL.md` as the normative runtime instruction set. This guide explains the reasoning behind it, the expected behavior, and how to modify or port it without weakening its guarantees.

## Mission

The skill converts a video into grounded, timestamped understanding. A successful run answers both:

- **What was said?** Captured through subtitles or speech-to-text.
- **What was shown?** Captured through representative frames selected around visual transitions.

The central failure the skill prevents is transcript-only analysis being presented as full video review.

## Activation

Use this skill for requests involving a complete video, including watching, transcribing, summarizing, reviewing, extracting lessons, documenting a demo, analyzing a lecture, or preparing an Obsidian note. Supported inputs include:

- URLs supported by `yt-dlp`;
- YouTube, Loom, and Zoom share links when access permits;
- direct media URLs;
- local video or screen-recording files.

Do not activate it for a static image, an audio-only request that does not need video semantics, or a request that supplies only a transcript and does not ask for video inspection.

## Non-negotiable invariants

1. **Evidence before claims.** Never say the video was watched unless timestamped speech/text and representative visuals were both inspected.
2. **No access circumvention.** Respect authentication, privacy, DRM, paywalls, and share permissions.
3. **Local by default.** Keep downloads, audio, transcripts, frames, and logs in a task-specific temporary folder. Do not upload or publish them without explicit user authorization.
4. **No surprise installs.** Detect missing dependencies and ask before installing them.
5. **No surprise knowledge-base writes.** Before writing to Obsidian, confirm the destination, note form/depth, and raw-transcript policy.
6. **Distill by default.** Return useful synthesis, not a transcript dump, unless the user asks for the raw transcript.
7. **Preserve uncertainty.** Call out missing segments, unreliable captions, inaccessible visuals, or claims that cannot be verified from the video.

## Pipeline and decision logic

### 1. Classify the source

Determine whether the input is a supported web URL, a direct media URL, or a local file. For a URL, inspect metadata before downloading the full video. Record the canonical page URL, title, creator/uploader, duration, upload date when available, and subtitle inventory.

If access fails, distinguish the cause: unsupported extractor, unavailable video, authentication requirement, private share, network failure, or media restriction. Report the specific obstacle rather than claiming the content is missing.

### 2. Establish a timestamped speech channel

Prefer creator-provided subtitles, then automatic subtitles, because they are usually cheaper and preserve timestamps. Prefer languages requested by the user; otherwise try the video's primary language and useful fallbacks.

If subtitles are absent or unusable, extract audio and run local Whisper. Do not hard-code Russian when the language is unknown. Retain SRT or an equivalent timestamped format. If recognition quality is poor, state why and avoid false precision in quotes.

### 3. Establish a visual channel

Download or use the local video, then select frames on scene changes. The default scene threshold in `SKILL.md` is a starting point:

- lower it for slowly changing slides or subtle UI transitions;
- raise it for noisy footage or rapid animation;
- reduce near-duplicates when the result is too large;
- preserve major transitions and visually information-dense frames.

Aim for a representative set, not an arbitrary fixed count. The `about 150` guidance is a practical ceiling for long videos, not a success criterion.

### 4. Fuse the channels

Align visual moments with nearby transcript timestamps. Look specifically for information that speech alone cannot express:

- code and terminal output;
- UI labels, states, and user actions;
- slide headings, diagrams, equations, tables, and charts;
- visual comparisons, before/after states, errors, warnings, and demonstrations;
- contradictions between narration and what is displayed.

Do not infer unreadable text. Mark uncertain interpretations as uncertain.

### 5. Produce the answer

The default report contains:

- a 3–5 line TL;DR;
- key concepts with timestamps;
- important on-screen content and why it matters;
- notable claims, warnings, or decisions with timestamps;
- gaps, uncertainties, and useful follow-up questions.

Adapt depth to the user's request. A tutorial may need ordered steps; a lecture may need argument structure; a product demo may need features, UI behavior, and limitations; a meeting may need decisions and action items. Preserve the evidence requirements in every mode.

## Obsidian behavior

Preparation and writing are separate permissions. The skill may propose a note structure without authorization, but it must not write to a vault until the user has answered:

1. Which vault, folder, or existing note?
2. What note type and depth?
3. Should the raw transcript be included, linked, or omitted?

After confirmation, create concise Markdown with source metadata, timestamps, synthesis, and relevant internal links. Prefer distilled knowledge over archival transcript unless the user explicitly wants the transcript.

## Resource and privacy considerations

Video processing can consume bandwidth, disk space, CPU/GPU time, and context. Inspect metadata first, prefer subtitles over transcription, cap resolution when full quality is unnecessary, and clean temporary artifacts only when deletion is authorized and safe. Never expose private media or transcript content in logs, public repositories, or third-party services without permission.

## Porting to another agent framework

The behavior is framework-agnostic even if packaging differs. A port must preserve:

- precise activation criteria;
- the transcript-first/fallback decision tree;
- scene-change visual sampling;
- dual-channel evidence before a "watched" claim;
- local/private defaults and authorization boundaries;
- timestamped, evidence-linked output.

Map `agents/openai.yaml` to the target framework's UI metadata and invocation policy. Keep `SKILL.md` or its equivalent as the concise runtime entrypoint; keep this guide as maintainer context rather than injecting it into every run.

## Maintenance checklist

When changing the skill:

- keep the YAML frontmatter valid and the folder/name as `watch`;
- make the description discriminating enough for automatic discovery;
- verify example commands against current `yt-dlp`, FFmpeg, and Whisper CLIs;
- keep operating-system-specific commands clearly labeled;
- do not replace a real safety invariant with vague advice;
- do not add tools, uploads, installs, or external writes as implicit permissions;
- validate the skill package and test at least one realistic URL and one local-file path when the environment permits;
- document material behavior changes in the repository or release notes.

## Definition of done for a real video task

A run is complete when the source and access status are known, a timestamped speech representation exists, representative visuals were inspected, the two evidence channels were synthesized, limitations are disclosed, and the requested report or confirmed knowledge-base artifact is delivered. Metadata-only inspection is not completion.
