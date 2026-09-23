---
name: teamy-skill-creator
description: Create or update Teamy's personal Codex skills in the Skills repo and link them into the Windows user skills folder. Use for personal skill authoring on this machine.
---

# Teamy Skill Creator

Keep personal skill sources in `G:\Programming\Repos\skills\.agents\skills\<skill-name>`. Expose each skill with its own Windows junction at `%USERPROFILE%\.agents\skills\<skill-name>`. This local layout takes precedence over the bundled skill creator's default location in `$CODEX_HOME/skills`. Use the bundled `skill-creator` for general advice on writing a focused skill.

1. Inspect the Skills repo status and the destination before editing. Preserve unrelated work, especially an in-progress migration between skill directories. Use a lowercase, hyphenated folder name and create or update its `SKILL.md` in the repo. Include YAML frontmatter with matching `name` and a description that states when the skill applies. Add supporting files only when they have a concrete use.
2. For a new skill, confirm the user-folder destination does not exist, then create a junction to the repo folder. Follow the existing per-skill junction pattern; do not copy the skill or replace another link. If a destination already exists, inspect its `LinkType` and `Target` and update the source only when it points to the intended folder.
3. Validate manually: check the frontmatter and instructions, confirm the junction target, and read `SKILL.md` through the junction. **Skip the Python `quick_validate.py` check** and do not install Python packages just to run it.
4. Report the source file, junction, and any remaining repo changes. Commit or push only when the user asks. Keep raw secrets and unrelated machine details out of shared skills unless the user explicitly authorizes a specific reference.

PowerShell junction command after the checks in step 2:

```powershell
$skillName = '<skill-name>'
$source = Join-Path 'G:\Programming\Repos\skills\.agents\skills' $skillName
$destination = Join-Path (Join-Path $env:USERPROFILE '.agents\skills') $skillName
New-Item -ItemType Junction -Path $destination -Target $source
```
