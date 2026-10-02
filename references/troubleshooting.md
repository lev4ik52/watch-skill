# Troubleshooting

Read this file only after an access, extraction, transcription, or decoding failure.

## Classify the failure

Identify the failing stage before changing tools:

- source access;
- metadata extraction;
- subtitles;
- audio extraction;
- transcription;
- video decoding;
- frame sampling;
- output writing.

Report the specific stage and preserve useful error text without exposing secrets.

## YouTube JavaScript warning

Current `yt-dlp` releases can require an external JavaScript runtime and EJS components for full YouTube support. Do not use a personal absolute runtime path.

1. Check whether Deno, Node, or QuickJS is already installed.
2. Prefer Deno when choosing a new runtime because `yt-dlp` recommends it.
3. Enable an installed alternative with `--js-runtimes`, for example `--js-runtimes node`.
4. If a runtime path is required, discover it on the current machine.
5. Ask before installing a runtime or additional EJS components.

Follow the current official guide: <https://github.com/yt-dlp/yt-dlp/wiki/EJS>.

## Login or private share

Do not bypass access controls. Explain whether the source requires authentication, a different share permission, or a supported cookie flow.

Browser cookies are sensitive. Ask before importing them. Never print cookie contents or commit them to the repository.

## No usable subtitles

Confirm that creator and automatic subtitles were both checked. If none are usable, extract audio and run local Whisper. If local transcription is unavailable, report that the speech channel is missing and return only a clearly labeled partial visual analysis when useful.

## Weak transcription

- verify the language setting;
- use a stronger model when resources permit;
- keep timestamps but avoid false precision;
- mark uncertain names, jargon, and quotes;
- compare important terms with visible slides or captions.

## Unexpected media filename

Do not assume that the download ends in `.mp4`. Use the path emitted after post-processing. If that path is unclear, inspect only the current task directory and identify the new media file there.

## FFmpeg decode failure

Use `ffprobe` to inspect the container and streams. Preserve the original file. If conversion is needed, create a derived copy in the task directory and record the command used.

## Too many or too few frames

- lower the scene threshold when gradual changes are missed;
- raise the threshold when edits or camera noise create too many frames;
- add periodic control frames for long static segments;
- remove near-duplicates only after preserving major transitions.

## Large or long videos

Inspect metadata first. Prefer subtitles over local transcription. Cap download resolution when fine visual detail is unnecessary. Process transcript and visuals in segments when context or memory is limited.

## Partial completion

When a blocker affects only one channel, continue with the useful evidence that remains. Label the result as partial, name the missing channel, and explain how the limitation affects confidence.
