<!-- markdownlint-disable -->

# Hardening Report: robinraju--release-downloader/v1.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **robinraju--release-downloader/v1.12** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference GitHub Actions using mutable version tags (@v4) instead of pinned full-length SHA commit hashes. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Failing references: ci.yml — `actions/checkout@v4` (line 22), `actions/setup-node@v4` (line 25); update-main-version.yml — `actions/checkout@v4` (line 24).

Locations:

- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:25`
- `.github/workflows/update-main-version.yml:24`

### script-injection (severity: high)

Rule (a) violation: GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing injection of arbitrary shell commands.

(1) In ci.yml, the 'Read output variable set by previous steps' step interpolates `${{steps.download-public.outputs.tag_name}}`, `${{steps.download-private.outputs.tag_name}}`, and `${{steps.download-public.outputs.downloaded_files}}` directly into shell commands (e.g., `if [ -n ${{steps.download-public.outputs.tag_name}} ]` and `echo ${{steps.download-public.outputs.downloaded_files}}`). These step outputs are derived from release asset data fetched from external repositories and could contain shell metacharacters.

(2) In update-main-version.yml, the 'Tag new target' and 'Push new tag' steps interpolate `${{ github.event.inputs.major_version }}` and `${{ github.event.inputs.target }}` directly into `git tag` and `git push` shell commands. These are user-supplied `workflow_dispatch` inputs and are fully attacker-controlled, enabling arbitrary command injection.

Locations:

- `.github/workflows/ci.yml:89`
- `.github/workflows/update-main-version.yml:30`
- `.github/workflows/update-main-version.yml:32`

### missing-permissions (severity: medium)

Neither workflow file declares a top-level `permissions:` key, and no job within either file declares job-level `permissions:`. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions, which may be overly broad (e.g., write access to contents, packages, etc.). Both `ci.yml` and `update-main-version.yml` should declare minimal required permissions (e.g., `permissions: contents: read` for the build/test job, and `contents: write` scoped only to the tagging job in update-main-version.yml).

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/update-main-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned `actions/checkout@v4` → `@11d5960a326750d5838078e36cf38b85af677262 # v4` and `actions/setup-node@v4` → `@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4` in both ci.yml and update-main-version.yml.

2. **script-injection**: In ci.yml, moved step output expressions (`steps.download-public.outputs.tag_name`, `steps.download-private.outputs.tag_name`, `steps.download-public.outputs.downloaded_files`) into the step's `env:` block and referenced them as quoted shell variables. In update-main-version.yml, moved `github.event.inputs.major_version` and `github.event.inputs.target` into `env:` blocks for the 'Tag new target' and 'Push new tag' steps, referencing them as `"$MAJOR_VERSION"` and `"$TARGET"`.

3. **missing-permissions**: Added `permissions: contents: read` to ci.yml (minimal for checkout/test) and `permissions: contents: write` to update-main-version.yml (required for pushing tags).

