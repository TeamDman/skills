---
name: windows-temporary-secret-cleanup
description: Safely stage Windows secrets from 1Password for scripts without exposing values to the conversation, work around sandbox visibility and elevation issues, and create reliable self-cleaning scheduled tasks. Use when Codex needs a temporary API key or credential file, encounters a missing op.exe in the sandbox, or must guarantee cleanup after a missed scheduled run.
---

# Windows Temporary Secret Cleanup

Use this skill when a Windows workflow needs a secret briefly but repeatedly, especially when 1Password unlock prompts would interrupt each command. Keep the secret out of chat output, source files, repositories, command arguments, logs, and task actions.

## Workflow

1. Locate the 1Password CLI.

   Check `Get-Command op` first. If it is missing from the sandbox, inspect the WinGet package locations, commonly:

   ```powershell
   Get-ChildItem "$env:LOCALAPPDATA\Microsoft\WinGet\Packages" -Filter op.exe -File -Recurse
   ```

   The sandbox may not see an application installed by WinGet. If the path exists but cannot be executed or read, rerun the narrowly scoped command elevated. Do not expose the CLI output when it contains a secret.

2. Materialize the secret only when necessary.

   Prefer 1Password’s output-file mode so the value never enters the model context:

   ```powershell
   & $opPath read $secretReference --no-newline --out-file $secretFile
   ```

   This lets the user unlock the vault once instead of being prompted for every API request. Never print the result, interpolate it into a tool description, pass it as a command-line argument, or commit the temporary file. If the CLI prompts, explain that one interactive unlock may be required; elevation can change which 1Password desktop session is visible.

3. Protect the temporary file.

   Store it outside the repository, preferably below `%LOCALAPPDATA%` or another user-private temporary directory. Confirm it is non-empty, disable inherited ACLs, and grant access only to the Windows user or service identity that must read it. Read it inside scripts with `Get-Content -Raw`; do not send its contents to stdout.

4. Schedule cleanup immediately.

   Create a one-shot Task Scheduler task whose action:

   - removes the secret file with `Remove-Item -LiteralPath ... -Force -ErrorAction SilentlyContinue`;
   - unregisters itself with `Unregister-ScheduledTask -Confirm:$false -ErrorAction SilentlyContinue`;
   - contains only the file path and task name, never the secret.

   Set these reliability settings:

   - `StartWhenAvailable = true` so a missed run starts as soon as possible;
   - `DeleteExpiredTaskAfter = P30D` so a non-recurring task does not remain forever;
   - allow start on battery and do not stop when switching to battery for cleanup-only tasks.

   For an older one-shot task whose XML lacks `EndBoundary`, update it through the Task Scheduler COM interface or add a valid boundary before re-registering. Use a boundary that leaves the task eligible during the intended cleanup window; do not copy an existing task XML blindly.

5. Verify without revealing the secret.

   Check the task settings and action:

   ```powershell
   $task = Get-ScheduledTask -TaskName $taskName
   $info = Get-ScheduledTaskInfo -TaskName $taskName
   $task.Settings.StartWhenAvailable
   $task.Settings.DeleteExpiredTaskAfter
   $info.NextRunTime
   ```

   Export or inspect the task XML and verify that the decoded action contains both file deletion and self-unregistration, while confirming that it does not contain the secret value. After use, remove the temporary file manually if it is no longer needed; the scheduled task is a fallback, not a reason to retain a secret longer than necessary.

## Safety rules

- Do not put secret values in chat, tool output, logs, scripts, repositories, or scheduled-task arguments.
- Do not use a broad recursive delete. Validate the exact temporary file path before removal.
- Do not modify media, source, or repository files when the request is only to stage a credential.
- Treat exported task XML as diagnostic input. It may be missing `StartWhenAvailable`, `DeleteExpiredTaskAfter`, or a required trigger boundary and is not automatically a correct template.
