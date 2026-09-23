---
name: windows-temporary-secret-cleanup
description: Safely stage temporary Windows credentials and schedule reliable cleanup without exposing secrets. Use native Windows APIs through the windows crate in Rust applications; use shell tools for standalone scripts. Applies to temporary API keys, repeated 1Password access, private credential files and cleanup after missed scheduled runs.
---

# Windows Temporary Secret Cleanup

Use this skill when a Windows workflow needs a secret briefly but repeatedly, especially when 1Password unlock prompts would interrupt each command. Keep the secret out of chat output, source files, repositories, command arguments, logs, and task actions.

## Choose the implementation before the tools

When implementing this workflow in a Rust project, use native Windows APIs through the `windows` crate. Do not spawn PowerShell, `icacls.exe`, `schtasks.exe` or other Windows utilities to perform operations available through those APIs. This applies to encryption, ACLs, identity lookup, task registration, inspection and cleanup — not just secret storage.

Inspect the project's existing Windows bindings and dependency policy first. Reuse its pinned version and enable only the required features. This guidance does not authorize dependency upgrades.

| Operation in Rust | Implementation |
| --- | --- |
| Protect and unprotect cached credentials | User-scoped DPAPI through the Windows cryptography APIs |
| Identify the account and restrict file access | Windows token, SID and security descriptor/DACL APIs |
| Register, inspect and remove cleanup tasks | Task Scheduler COM APIs through `windows` |
| Read, create and delete exact files | Rust filesystem APIs; Windows file APIs where security attributes or handle checks are needed |

PowerShell examples below are for a standalone script or manual diagnostic, not an implementation template for a Rust subprocess. Do not add encoded PowerShell or a helper executable just to wrap these APIs. If a native implementation is genuinely unavailable, explain the limitation and obtain agreement before adding a subprocess fallback.

The 1Password CLI remains a valid external dependency for acquiring its secret. Invoke it directly with arguments and capture its output privately; do not route it through a shell. This exception does not extend to Windows operations that belong in the Rust application.

## Native Rust workflow

1. Acquire the secret only through the explicitly authorized authentication operation. Keep captured output out of logs, debug formatting and error messages. Do not create an intermediate plaintext file to pass it between processes.
2. Create a user-private directory with a restricted DACL before writing credentials. Use user-scoped DPAPI, not machine-wide protection. A cached lease needs its purpose and expiry checked on every use; scheduled deletion does not enforce validity.
3. Register cleanup through Task Scheduler COM, using the reliability settings below. Prefer a cleanup subcommand in the existing application. Give it a generated lease ID resolved beneath a fixed credential directory, not a secret or arbitrary deletion path. Unique file/task identities prevent an old task from deleting a renewed lease. Registration failure must not leave a usable credential behind.
4. Delete only the validated lease file and unregister that exact task through the native APIs. Use the bindings' ownership types where supported, including `windows::core::Owned<T>`, and balance COM initialization on the same thread. Keep guards for allocations or lifecycle rules not covered by those types.
5. Test with a dummy credential first: encrypted storage, restricted access, expiry, exact-file deletion, self-unregistration and renewal safety. Verify task settings through the COM API. Never print a real secret to prove that storage works.

A security-product block is not a reason to add exclusions or hide the same operation in another shell. Report the block and investigate the implementation and permissions.

## Standalone script workflow

1. Locate the 1Password CLI.

   Check `Get-Command op` first. If it is missing from the sandbox, inspect the WinGet package locations, commonly:

   ```powershell
   Get-ChildItem "$env:LOCALAPPDATA\Microsoft\WinGet\Packages" -Filter op.exe -File -Recurse
   ```

   The sandbox may not see an application installed by WinGet. If the path exists but cannot be executed or read, rerun the narrowly scoped command elevated. Do not expose the CLI output when it contains a secret.

2. Prepare the private location and materialize only when necessary.

   Create the user-private directory and restrict its ACL before writing the secret. For scripts, prefer 1Password’s output-file mode so the value never enters the model context:

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

   Apply the shared task reliability settings below.

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

## Task reliability settings for both implementations

- Set `StartWhenAvailable = true` so a missed run can start later. It does not guarantee immediate cleanup after expiry.
- Set `DeleteExpiredTaskAfter = P30D` so a non-recurring task does not remain forever.
- Supply an `EndBoundary` that leaves the task eligible during the intended cleanup window. Repair older task definitions that lack it; do not copy their XML blindly.
- Allow start on battery and do not stop when switching to battery for cleanup-only tasks.
- Keep secrets out of task actions. Deleting an expired task does not itself delete its credential file; the cleanup action must do both.

## Safety rules

- Do not put secret values in chat, tool output, logs, scripts, repositories, or scheduled-task arguments.
- Do not use a broad recursive delete. Validate the exact temporary file path before removal.
- Do not modify media, source, or repository files when the request is only to stage a credential.
- Treat exported task XML as diagnostic input. It may be missing `StartWhenAvailable`, `DeleteExpiredTaskAfter`, or a required trigger boundary and is not automatically a correct template.
