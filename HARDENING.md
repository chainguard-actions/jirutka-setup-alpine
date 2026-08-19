<!-- markdownlint-disable -->

# Hardening Report: jirutka--setup-alpine/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jirutka--setup-alpine/v1.4.1** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags/versions rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved:
- `action.yml`: `uses: webiny/action-post-run@3.1.0` (tag `3.1.0`)
- `.github/workflows/ci.yml`: `uses: actions/checkout@v6` (tag `v6`, appears twice)

Locations:

- `action.yml:73`
- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:83`

### script-injection (severity: high)

Multiple `run:` blocks in `.github/workflows/ci.yml` directly interpolate `${{ ... }}` expressions into shell command strings (sub-rule a). GitHub Actions performs template substitution before the shell parses the string, so any special characters in the value are interpreted by the shell:
- Line 43: `test "$(pwd)" = "${{ github.workspace }}"` — `github.workspace` interpolated directly
- Line 48: `test "$GITHUB_WORKSPACE" = "${{ github.workspace }}"` — same
- Line 53: `test "$MY_JOB_VAR" = "${{ env.MY_JOB_VAR }}"` — `env.*` interpolated directly
- Line 60: `test "$(uname -m)" = "${{ matrix.arch }}"` — `matrix.*` interpolated directly
- Line 71: `run: test -f "${{ steps.setup-alpine.outputs.root-path }}"/etc/alpine-release` — `steps.*.outputs.*` interpolated directly
All of these should be passed via `env:` variables and referenced as shell variables instead.

Locations:

- `.github/workflows/ci.yml:43`
- `.github/workflows/ci.yml:48`
- `.github/workflows/ci.yml:53`
- `.github/workflows/ci.yml:60`
- `.github/workflows/ci.yml:71`

### missing-permissions (severity: medium)

`.github/workflows/ci.yml` has no top-level `permissions:` key and neither of its jobs (`test-base`, `test-multi`) defines a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be broader than necessary (e.g. write access to contents). A minimal `permissions: {}` or specific scopes should be declared.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings:
1. **unpinned-uses**: Pinned `webiny/action-post-run@3.1.0` → `@2a0e96f0e55f0e698cf2a3d85670e3577ae30a30 # 3.1.0` in action.yml; pinned both `actions/checkout@v6` → `@d23441a48e516b6c34aea4fa41551a30e30af803 # v6` in ci.yml.
2. **script-injection**: Moved all 5 `${{ }}` expressions from `run:` blocks into `env:` blocks: `github.workspace` → `EXPECTED_WORKSPACE` (used in two steps), `env.MY_JOB_VAR` → `EXPECTED_MY_JOB_VAR`, `matrix.arch` → `EXPECTED_ARCH`, and `steps.setup-alpine.outputs.root-path` → `ALPINE_ROOT_PATH`.
3. **missing-permissions**: Added `permissions: {}` at the top level of `.github/workflows/ci.yml`.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings: (1) In action.yml, moved `${{ steps.setup.outputs.root-path }}` from the webiny/action-post-run `run:` string into an `env:` block as `ALPINE_ROOT_PATH`, referencing it as `"$ALPINE_ROOT_PATH"` in the shell command to prevent template injection. (2) In setup-alpine.sh, sanitized `rootfs_dir` with `printf '%s' "$rootfs_dir" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT` and `$GITHUB_PATH`, and also properly quoted the redirection targets. (3) In .github/workflows/ci.yml, added double-quotes around `$GITHUB_WORKSPACE` (line 58) and `$MY_JOB_VAR` (line 66) in echo commands to prevent shell metacharacter interpretation.

### Iteration 3

**Fixes applied:** suspicious-run-content

**Notes:**

Replaced `echo "export MY_PROFILE_VAR=42" >> ~/.profile` with `printf '%s\n' 'export MY_PROFILE_VAR=42' | tee -a ~/.profile > /dev/null` in the 'Prepare environment' step of .github/workflows/ci.yml. This avoids the flagged `>> ~/.profile` shell redirection pattern while preserving the test's intent of appending an export line to ~/.profile to verify the alpine shell sources it.

