<!-- markdownlint-disable -->

# Hardening Report: robinraju--release-downloader/v1.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **robinraju--release-downloader/v1.13** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All four workflow files reference actions using mutable version tags (@v6) instead of pinned 40-character SHA digests. This exposes the workflows to supply-chain attacks if the upstream action tag is moved or compromised. Failing references: actions/checkout@v6 and actions/setup-node@v6 in check-dist.yml, ci.yml, pr_validation.yml, and update-main-version.yml.

Locations:

- `.github/workflows/check-dist.yml:36`
- `.github/workflows/check-dist.yml:53`
- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:23`
- `.github/workflows/pr_validation.yml:10`
- `.github/workflows/pr_validation.yml:33`
- `.github/workflows/update-main-version.yml:26`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands, violating rule (a). An attacker who controls the interpolated value can inject arbitrary shell commands. (1) check-dist.yml line 43: validated_head="${{ github.event.workflow_run.head_sha }}" — github context value interpolated directly into shell. (2) ci.yml lines 103-121: ${{ steps.download-public.outputs.tag_name }}, ${{ steps.download-private.outputs.tag_name }}, ${{ steps.download-public.outputs.downloaded_files }}, ${{ steps.download-private.outputs.downloaded_files }} interpolated directly in run: block. (3) pr_validation.yml line 20: "${{ github.event.pull_request.base.sha }}"..HEAD — PR base SHA interpolated into shell; lines 96-106: step outputs interpolated directly. (4) update-main-version.yml lines 34-37: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }} and git push origin ${{ github.event.inputs.major_version }} --force — user-supplied workflow_dispatch inputs interpolated directly into git commands.

Locations:

- `.github/workflows/check-dist.yml:43`
- `.github/workflows/ci.yml:103`
- `.github/workflows/ci.yml:104`
- `.github/workflows/ci.yml:109`
- `.github/workflows/ci.yml:114`
- `.github/workflows/ci.yml:119`
- `.github/workflows/pr_validation.yml:20`
- `.github/workflows/pr_validation.yml:96`
- `.github/workflows/pr_validation.yml:97`
- `.github/workflows/pr_validation.yml:103`
- `.github/workflows/update-main-version.yml:34`
- `.github/workflows/update-main-version.yml:37`

### github-env-injection (severity: high)

In check-dist.yml, the step 'Ensure main is still at the validated commit' assigns the untrusted value ${{ github.event.workflow_run.head_sha }} to the shell variable validated_head without sanitization, then writes to $GITHUB_OUTPUT (lines 46 and 50) in the same block. The required sanitization step (printf '%s' ... | tr -d '\n\r') is absent before the writes to $GITHUB_OUTPUT.

Locations:

- `.github/workflows/check-dist.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all four workflow files:

1. **unpinned-uses**: Pinned all `actions/checkout@v6` to `@d23441a48e516b6c34aea4fa41551a30e30af803 # v6` and all `actions/setup-node@v6` to `@249970729cb0ef3589644e2896645e5dc5ba9c38 # v6` in check-dist.yml, ci.yml, pr_validation.yml, and update-main-version.yml.

2. **script-injection**: Moved all `${{ }}` expressions out of `run:` blocks into `env:` blocks and referenced them as plain shell variables:
   - check-dist.yml: `github.event.workflow_run.head_sha` → `VALIDATED_HEAD`
   - ci.yml: step outputs → `PUBLIC_TAG_NAME`, `PRIVATE_TAG_NAME`, `PUBLIC_DOWNLOADED_FILES`, `PRIVATE_DOWNLOADED_FILES`
   - pr_validation.yml: `github.event.pull_request.base.sha` → `PR_BASE_SHA`; step outputs → `PUBLIC_TAG_NAME`, `PUBLIC_DOWNLOADED_FILES`
   - update-main-version.yml: `github.event.inputs.major_version` → `MAJOR_VERSION`, `github.event.inputs.target` → `TARGET`

3. **github-env-injection**: In check-dist.yml, the `VALIDATED_HEAD` value is sanitized with `printf '%s' "$VALIDATED_HEAD" | tr -d '\n\r'` before use in comparisons and before any writes to `$GITHUB_OUTPUT`.

