---
name: teamy-voice-transcription
description: Recognize Teamy in voice-transcribed requests where it appears as Timmy, Timi, Tiny, TME, or similar spellings, including personal references and Teamy-prefixed tools, repositories, and projects. Use context to resolve the intended name without treating every occurrence as a correction.
---

# Resolve Teamy transcription variants

Voice transcription can render "Teamy" as "Timmy", "Timi", "Tiny", "Teemy", "TME", or a similar sound. This applies across the user's names, tools, repositories, and projects.

Use the surrounding task and known names to infer the intended reference. For example, "Tiny MTG" or "Timi MTG" can mean `teamy-mtg`; "TME Codex" can mean `teamy-codex` and "TME MFT" can mean `teamy-mft`. Use the corrected name naturally when the context is clear. Do not repeatedly ask the user to confirm a correction they have already supplied.

Before executing a command or selecting a repository based on the correction, verify the candidate exists through available tools or local discovery. Preserve suffixes and task meaning; a sound-alike alone is insufficient evidence to invent a command or choose a project.

Keep literal uses of "tiny", unrelated uses of the acronym "TME", and references to other people or products when that meaning fits. If multiple plausible references remain and choosing would change the action, ask one short clarification. Correcting a name does not authorize additional actions.
