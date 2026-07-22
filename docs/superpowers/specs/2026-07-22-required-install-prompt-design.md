# Required Install Prompt Design

## Goal

On every Agent Seed activation, require an explicit install decision for the
applicable, missing `git-code-tracker` bundled package and OpenCLI external
tool. A declined install must not block onboarding, but it must be recorded
for the current activation and must not suppress the prompt on a later
activation.

## Policy

The activation preflight identifies the active or detected platform, checks
the two configured tools, and handles each applicable missing tool as follows:

1. Prompt the owner before onboarding continues.
2. If approved, run the existing platform-specific installation flow and
   verify it before continuing.
3. If declined, request and record a reason, then continue onboarding.
4. Repeat the same prompt on every later activation while the tool remains
   missing.

Installed and successfully verified tools are not prompted again. OpenCLI
continues to require per-run approval because its installation is networked
and global. `git-code-tracker` continues to require approval because its
installer modifies the target project.

## Rejection Record

Agent Seed writes a best-effort activation record to
`.agents/agent-seed.json` in the target repository. Each declined install
adds an entry with the activation time, detected platform, tool name, and
owner-provided reason. Entries are audit history only: they must never be read
as an opt-out or used to skip a future install prompt.

If writing the local record is unavailable, the agent reports that limitation
and continues onboarding after capturing the reason in its current result.

## Configuration And Guidance

The external and bundled-package manifests gain an explicit recurring-prompt
policy. The Agent Seed instructions and default prompt state that the two
tools are checked at every activation and that missing applicable tools are
always offered before onboarding. They distinguish this from an automatic
installation: approval is still required before any project or global change.

## Verification

Release tests validate that the package includes the policy fields and that
the instructions describe recurring prompts rather than persistent skips.
Manual dry-run review validates these paths: installed, approved, declined,
and a second activation after a previous decline.
