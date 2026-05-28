# Hardening Report: jirutka--setup-alpine/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jirutka--setup-alpine/v1.4.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses `webiny/action-post-run@3.1.0`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. This means the referenced action could be silently replaced with a different (potentially malicious) version without any change to this file, creating a supply-chain risk.

Locations:

- `action.yml:86`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced mutable tag reference `webiny/action-post-run@3.1.0` with the immutable commit SHA `webiny/action-post-run@2a0e96f0e55f0e698cf2a3d85670e3577ae30a30 # 3.1.0` in actions/hardened/jirutka--setup-alpine/v1.4.1/action.yml at line 86.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in setup-alpine.sh at lines 239-240. The variable `rootfs_dir` (derived from attacker-controlled `$INPUT_BRANCH` and `$INPUT_ARCH`) was written directly to `$GITHUB_OUTPUT` and `$GITHUB_PATH` without sanitization. The fix introduces a `safe_rootfs_dir` variable that strips newlines and carriage returns using `printf '%s' "$rootfs_dir" | tr -d '\n\r'` before writing to the special environment files. Proper quoting was also added around `$GITHUB_OUTPUT` and `$GITHUB_PATH`.

