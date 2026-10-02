# Maintainer guide

This repository contains the `watch` Agent Skill. Treat `SKILL.md` as the runtime contract. Keep conditional commands and recovery procedures in `references/`.

## Mission

Convert a video into grounded, timestamped understanding of what was said and shown. Prevent transcript-only work from being presented as a complete audiovisual review.

## Invariants

1. Require timestamped speech evidence when speech exists.
2. Require representative visual evidence for a complete video review.
3. Label the result as partial when required evidence is unavailable.
4. Respect authentication, privacy, DRM, paywalls, and share permissions.
5. Keep media local unless the user authorizes an external transfer.
6. Ask before installing software or importing browser cookies.
7. Ask only for missing Obsidian details before writing.
8. Return synthesis instead of a transcript dump unless the user requests the transcript.
9. Preserve uncertainty in captions, quotes, names, timestamps, and unreadable visuals.

## Documentation ownership

- Put runtime routing, invariants, and completion criteria in `SKILL.md`.
- Put user installation and examples in `README.md`.
- Put extraction commands in `references/pipeline.md`.
- Put knowledge-base writing rules in `references/obsidian.md`.
- Put failure recovery in `references/troubleshooting.md`.
- Put maintenance and release rules in this file.

Do not copy the full workflow into more than one file.

## Change rules

- Keep the skill name and directory name as `watch`.
- Keep the frontmatter description precise enough for automatic selection.
- Use platform-neutral instructions in `SKILL.md`.
- Label PowerShell and POSIX commands in references.
- Do not add personal absolute paths.
- Use the actual downloaded media path. Do not assume an `.mp4` extension.
- Prefer current official documentation for `yt-dlp`, FFmpeg, and Whisper commands.
- Do not add uploads, installs, cookie access, or external writes as implicit permissions.
- Add a reference file only when it prevents conditional detail from bloating `SKILL.md`.

## Review matrix

Test these cases when the environment permits:

1. Public URL with creator subtitles.
2. Public URL with automatic subtitles only.
3. Public URL with no subtitles and local Whisper fallback.
4. Local video file.
5. Silent or nearly silent video.
6. Long video with static slides or UI.
7. Private or login-gated link.
8. Failed codec or unsupported container.
9. Obsidian request with all details supplied.
10. Obsidian request with one or more details missing.

For each case, verify the source status, timestamps, visual coverage, limitation wording, and absence of unauthorized external actions.

## Release checklist

- Validate YAML frontmatter and repository paths.
- Check internal Markdown links.
- Search for personal paths, placeholders, and stale filenames.
- Verify commands against current upstream documentation.
- Run a no-network smoke review of the skill package.
- Test at least one realistic URL and one local file when tools and access are available.
- Confirm that `agents/openai.yaml` matches the skill behavior.
- Review the final diff before committing.

## Definition of done

A release is ready when the package validates, references are reachable, and commands use current interfaces. The review matrix must have no known critical gap. The documentation must preserve every invariant above.
