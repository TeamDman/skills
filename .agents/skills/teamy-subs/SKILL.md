---
name: teamy-subs
description: Use the teamy-subs CLI to find and download subtitles for local media through SubDL, or to extract, convert, and download other subtitles when requested.
---

# Teamy Subs

Use the installed `teamy-subs` CLI for subtitle work. Check `teamy-subs help list --short` or a command's `--help` when syntax is uncertain. If the installed command is absent from the sandbox's `PATH`, locate the user's Cargo installation or the local `TeamDman/teamy-subs` checkout. Do not reinstall it unless the task calls for an update.

## SubDL for a local video

1. Confirm the video exists. Choose the requested subtitle directory; otherwise use the established ad hoc location `%USERPROFILE%\Downloads\subs`. Pass `--output-dir` explicitly because the CLI otherwise writes to its current working directory.
2. Run `teamy-subs subdl status`. If the lease is expired, run `teamy-subs subdl --op-ref 'op://Private/Subdl/API key' login` once. This is the user-authorized 1Password item reference, not the API key value. The default lease lasts 15 minutes across CLI processes and schedules its own cleanup. Do not pass `--op-ref` on every search or download: that would read 1Password again.
3. Run `teamy-subs subdl subtitle search --file '<video-file>' --language en` to inspect candidates. Compare title, year, source, and release group as well as the score. Candidate numbers are one-based. A unique score of at least 0.8 can be downloaded automatically; lower scores and ties require `--pick <number>`.
4. Run `teamy-subs subdl subtitle download --file '<video-file>' --language en --output-dir '<directory>'` with `--pick` when needed. Avoid `--force` unless replacement is requested. Confirm the output file exists and report the selected release and any meaningful sync uncertainty. For a weak match, compare subtitle timing with video duration when practical.

The CLI reads a cached key without contacting 1Password on each command. Never print or persist the API key value or a SubDL download URL, and do not read the encrypted lease directly. If Codex's sandbox identity cannot access the user-private lease, retry the narrow CLI command under the user's Windows identity through the normal escalation path; do not loosen the lease ACL.

## Other commands

- `teamy-subs subdl account show` checks quota; `teamy-subs subdl logout` removes cached leases early.
- `teamy-subs yt-dlp download <url>` downloads subtitles for remote media.
- `teamy-subs extract <media-file> <output-dir>` extracts embedded streams.
- `teamy-subs convert <subtitle-file> <output-file-or-format>` converts supported subtitle formats.

Use the user's requested language and output location when provided. Keep individual media paths out of the skill. The 1Password reference above is deliberately recorded at the user's request; do not copy the underlying key into any file or tool output.
