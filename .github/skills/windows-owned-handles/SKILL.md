---
name: windows-owned-handles
description: 'Use windows-rs owned handle patterns in Rust Win32 code. Use when reviewing, writing, or refactoring Rust code that owns Windows handles such as HANDLE, service handles, registry handles, tokens, files, IOCP handles, or volume handles; especially when code defines custom OwnedHandle wrappers, manually calls CloseHandle in Drop, imports std::os::windows raw-handle ownership traits for Win32 values, or needs guidance on the windows::core::Owned type.'
---

# Windows Owned Handles

Prefer the ownership types that `windows-rs` already provides for Win32 handles.

## Outcome

Produce Rust code that:

- uses `windows::core::Owned<T>` for owned Win32 handle values when the handle type supports it
- avoids local `OwnedHandle` newtypes whose only job is calling `CloseHandle`
- keeps custom guards only for resources with different free functions or extra invariants
- searches for the antipattern before adding another wrapper
- preserves existing error handling and safety boundaries

## Audit First

Search the target codebase before editing:

```powershell
rg -n "OwnedHandle|struct Owned|impl Drop|CloseHandle|BorrowedHandle|AsRawHandle|FromRawHandle|IntoRawHandle|Owned<" .
```

Treat these as likely smells:

- `struct OwnedHandle(HANDLE);`
- an `impl Drop` that only checks `HANDLE::is_invalid()` and calls `CloseHandle`
- a module importing `CloseHandle` only for that wrapper
- direct raw-handle ownership traits used around Win32 `HANDLE` values

Treat these as likely good patterns:

- `use windows::core::Owned;`
- fields or parameters typed as `Owned<HANDLE>`
- `unsafe { Owned::new(api_that_returns_handle()?) }` immediately after successful handle creation
- passing `*owned_handle` to APIs that need `HANDLE`

## Replacement Pattern

For Win32 APIs returning a `HANDLE` that must be closed with `CloseHandle`, wrap the successful result immediately:

```rust
use windows::Win32::Foundation::HANDLE;
use windows::core::Owned;

let handle: Owned<HANDLE> = unsafe { Owned::new(CreateFileW(...)?) };
```

Use the owned value directly in structs:

```rust
struct Reader {
    handle: Owned<HANDLE>,
}
```

Pass the raw handle to Win32 calls with deref:

```rust
DeviceIoControl(*self.handle, ...)?;
```

When replacing a custom wrapper:

1. Remove the local `OwnedHandle` struct and its `Drop` impl.
2. Remove the `CloseHandle` import if it becomes unused.
3. Import `windows::core::Owned`.
4. Change fields, locals, and constructor parameters from the wrapper to `Owned<HANDLE>`.
5. Wrap newly-created handles with `unsafe { Owned::new(handle) }` after the API has reported success.
6. Replace `.0` raw-handle access with `*owned_handle`.

## Keep Custom Guards When Appropriate

Do not force `Owned<T>` onto resources that do not use normal windows-rs handle ownership semantics.

Keep or create a custom guard when:

- the resource is freed with `LocalFree`, `RegCloseKey`, `CloseServiceHandle`, `FindClose`, or another non-`CloseHandle` API and there is no suitable `windows-rs` owned type in use
- the value is a pointer allocation, security descriptor, SID buffer, or ACL allocation rather than a kernel handle
- the wrapper stores backing memory that must remain alive for a borrowed pointer
- the type encodes domain invariants beyond ownership, such as a validated NTFS volume handle

If a domain wrapper is still useful, store `Owned<HANDLE>` inside it rather than reimplementing handle closure.

## Verification

After edits:

- run `cargo fmt`
- run the narrowest relevant Rust tests or checks available in the repo
- re-run the audit search and confirm no simple custom `OwnedHandle` wrappers remain
