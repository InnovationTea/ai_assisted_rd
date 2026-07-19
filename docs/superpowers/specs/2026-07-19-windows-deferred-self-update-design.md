# Windows Deferred Self-Update Design

## Goal

Allow an approved Agent Seed self-update to complete automatically on Windows
when the current agent host has locked the installed skill directory. The
current session must continue safely; the replacement may occur only after the
lock is released.

## Scope

- Keep the existing synchronous replacement behaviour on Linux and macOS.
- On Windows, stage a downloaded update outside the skill directory before
  attempting replacement.
- If replacement fails with a Windows sharing or busy error, transfer the
  replacement operation to a detached helper outside the skill directory.
- Record durable, user-visible pending, success, and terminal failure states.
- Preserve replacement semantics: the final skill directory contains exactly
  the released package, without stale files.

This does not install a Windows scheduled task, modify Claude Code, or force a
host application to exit.

## Alternatives Considered

1. Retry the existing in-process deletion. This cannot outlive the invoking
   session, and retries remain blocked while the host owns the directory.
2. Overlay new files into the installed directory. This can leave stale files
   and may still fail if a loaded file is locked; it violates the current
   replacement guarantee.
3. Stage first and use an external deferred helper. Recommended. The helper is
   independent from the locked skill root, can wait for the host to release its
   handles, and retains rollback behaviour.

## Architecture

`update-agent-seed.mjs --apply` remains the public entry point.

1. It downloads and extracts the release into a unique staging directory under
   the current user's local application-data directory. The staging record
   includes the target path, requested version, and a copy of the external
   helper script.
2. It attempts the existing replacement once.
3. A successful replacement removes the staging directory and records
   `last_check.status: "updated"`.
4. On Windows only, an `EBUSY`, `EPERM`, or sharing-violation error queues a
   detached Node helper from the staging directory. The command returns success
   with a distinct `queued` result rather than reporting a failed update.
5. The helper retries only the local replacement. It never re-downloads,
   re-resolves proxies, or depends on files in the old skill directory.
6. Once the target can be replaced, the helper copies a backup, replaces the
   directory, verifies the installed `VERSION.json`, writes `updated`, then
   removes its staging directory.

The helper is launched detached with all standard streams ignored and with its
working directory outside the target. It writes an append-only text log beside
the staging record for diagnostics.

## Persistent State

The project-local `.agents/agent-seed.json` remains the user-facing state
surface. The updater writes:

```json
{
  "self_update": {
    "last_check": {
      "status": "queued",
      "reason": "windows-directory-locked",
      "current_version": "v0.2.11",
      "latest_version": "v0.2.12",
      "checked_at": "2026-07-19T00:00:00.000Z"
    }
  }
}
```

The helper changes this to `updated` after it verifies the replacement. It uses
`failed` with a specific reason such as `lock-timeout`, `replacement-failed`,
or `version-verification-failed` after a terminal failure. A later `--apply`
may supersede an existing queued stage for the same target; stages for other
targets are independent.

The private staged files live under an Agent Seed-owned local-data root, not
under the project or the system temporary directory. The path is derived from
`LOCALAPPDATA` on Windows, with a deterministic per-user fallback where that
variable is unavailable.

## Retry And Recovery

- Retry after an initial short delay with capped exponential backoff.
- Stop after twelve hours or a bounded number of attempts, whichever occurs
  first. Do not leave an unbounded background process.
- Treat only Windows lock-related errors as retryable. Permission errors not
  attributable to sharing violations fail immediately.
- Before each replacement, copy the existing target to the stage backup. If
  copying the release fails, restore the backup and continue only when the
  original error was retryable.
- Retain a terminally failed stage and its log for seven days; successful stages
  are removed immediately. A subsequent updater invocation prunes expired
  failed stages.

## Integrity And Safety

- Validate the downloaded zip's SHA-256 when the GitHub release asset exposes
  a `sha256:` digest. Refuse an advertised digest mismatch before extraction.
- Use generated stage identifiers, never interpolate user-controlled paths into
  shell commands, and invoke the helper with `spawn` argument arrays.
- The helper is copied to the stage before the first replacement attempt, so it
  cannot lose its executable source when the installed directory changes.
- The release version in the final `VERSION.json` must equal the planned latest
  version before the state becomes `updated`.

## Interfaces

`applyUpdate` returns a structured result:

- `{ status: "updated", version }` for an immediate completed replacement.
- `{ status: "queued", version, stagePath }` when Windows defers replacement.

The CLI prints an explicit queued message and exits zero for `queued`; callers
can distinguish an accepted, deferred update from a terminal failed update.
Internal helper mode is intentionally not part of the user CLI contract. It
accepts only a generated stage-record path.

## Tests

- Existing replacement test still proves stale files are absent after a normal
  synchronous update.
- Digest verification accepts a matching release digest and rejects a mismatch.
- A simulated Windows `EBUSY` queues a helper, persists `queued`, and does not
  delete the staged release.
- Helper retry succeeds after a simulated lock release, verifies the version,
  and writes `updated`.
- A non-retryable replacement error records `failed` without spawning an
  unbounded retry loop.
- Retry deadline leaves a diagnostic log and a bounded retained stage.
- Linux/macOS retain the present immediate replacement path.

## Acceptance Criteria

On Windows, an approved update that encounters a directory lock returns with a
queued result, needs no further user command, and completes automatically after
the host releases the lock. It either installs the exact requested release and
records `updated`, or leaves the old installation intact and records a specific
terminal failure. No replacement path can leave a partially installed skill.
