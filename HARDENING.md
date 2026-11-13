<!-- markdownlint-disable -->

# Hardening Report: jirutka--setup-alpine/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jirutka--setup-alpine/v1.4.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action step references a mutable tag instead of a 40-character commit SHA: `uses: webiny/action-post-run@3.1.0`. Pin it to the full commit SHA, with the tag kept as a comment.

Locations:

- `action.yml:90`

### github-env-injection (severity: high)

The setup step (`run: sudo -E ./setup-alpine.sh`) writes a value built from user input to special environment files without the `printf '%s' ... | tr -d '\n\r'` sanitization step: `echo "root-path=$rootfs_dir" >> $GITHUB_OUTPUT` and `echo "$rootfs_dir/abin" >> $GITHUB_PATH`. rootfs_dir is built as "$ROOTFS_BASE_DIR/alpine-${INPUT_BRANCH%-stable}-$INPUT_ARCH". INPUT_BRANCH comes from inputs.branch, and its only check is the case glob `v[0-9].[0-9]*`. That glob accepts any trailing characters, newlines included. A branch value with an embedded newline can therefore add extra GITHUB_OUTPUT entries or put an arbitrary directory on PATH. Strip CR/LF from the value before both writes, and tighten the branch check.

Locations:

- `setup-alpine.sh:344`
- `setup-alpine.sh:345`
- `action.yml:84`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index a8e8ecb..cb6d808 100644
--- a/action.yml
+++ b/action.yml
@@ -1,6 +1,7 @@
 name: Setup Alpine Linux environment
 description: >
   Set up a chroot-based Alpine Linux environment and optionally QEMU user emulator for non-x86 architectures.
+
 author: Jakub Jirutka
 branding:
   color: blue
@@ -8,15 +9,14 @@ branding:
 inputs:
   apk-tools-url:
     description: >
-      URL of the apk-tools static binary to use. It must end with `#!sha256!` followed by a SHA-256
-      hash of the file.
-      This should normally be left at the default value.
+      URL of the apk-tools static binary to use. It must end with `#!sha256!` followed by a SHA-256 hash of the file. This should normally be left at the default value.
+
     required: false
     default: https://gitlab.alpinelinux.org/api/v4/projects/5/packages/generic/v2.14.10/x86_64/apk.static#!sha256!34bb1a96f0258982377a289392d4ea9f3f4b767a4bb5806b1b87179b79ad8a1c
   arch:
     description: >
-      CPU architecture to emulate using QEMU. Allowed values are: `x86_64` (native), `x86` (native),
-      `aarch64`, `armhf` (armv6 with hard-float), `armv7`, `loongarch64` (available since v3.21), `ppc64le`, `riscv64` (available since v3.20), and `s390x`
+      CPU architecture to emulate using QEMU. Allowed values are: `x86_64` (native), `x86` (native), `aarch64`, `armhf` (armv6 with hard-float), `armv7`, `loongarch64` (available since v3.21), `ppc64le`, `riscv64` (available since v3.20), and `s390x`
+
     required: false
     default: x86_64
   branch:
@@ -25,15 +25,14 @@ inputs:
     default: latest-stable
   extra-keys:
     description: >
-      A list of paths of additional trusted keys (for installing packages from the
-      extra-repositories) to copy into /etc/apk/keys/. The paths should be relative to the
-      workspace directory (the default location of your repository when using the checkout action).
+      A list of paths of additional trusted keys (for installing packages from the extra-repositories) to copy into /etc/apk/keys/. The paths should be relative to the workspace directory (the default location of your repository when using the checkout action).
+
     required: false
     default: ''
   extra-repositories:
     description: >
-      A list of additional Alpine repositories to add into /etc/apk/repositories (Alpine's official
-      main and community repositories are always added).
+      A list of additional Alpine repositories to add into /etc/apk/repositories (Alpine's official main and community repositories are always added).
+
     required: false
     default: ''
   mirror-url:
@@ -46,32 +45,44 @@ inputs:
     default: ''
   shell-name:
     description: >
-      Name of the script to run `sh` in the Alpine chroot that will be added to `GITHUB_PATH`.
-      This name should be used in `jobs.<job_id>.steps[*].shell` (e.g. `shell: alpine.sh {0}`) to
-      run the step's script in the chroot.
+      Name of the script to run `sh` in the Alpine chroot that will be added to `GITHUB_PATH`. This name should be used in `jobs.<job_id>.steps[*].shell` (e.g. `shell: alpine.sh {0}`) to run the step's script in the chroot.
+
     required: false
     default: alpine.sh
   volumes:
     description: >
-      A list of directories on the host system to mount bind into the chroot. You can specify the
-      source and destination path: `<src-dir>:<dest-dir>``. The `<src-dir>` is an absolute path of
-      existing directory on the host system, `<dest-dir>` is an absolute path in the chroot
-      (it will be created if doesn't exist). You can omit the latter if they're the same.
+      A list of directories on the host system to mount bind into the chroot. You can specify the source and destination path: `<src-dir>:<dest-dir>``. The `<src-dir>` is an absolute path of existing directory on the host system, `<dest-dir>` is an absolute path in the chroot (it will be created if doesn't exist). You can omit the latter if they're the same.
+
+      Please note that /home/runner/work (where's your workspace located) is always m
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed setup-alpine.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/setup-alpine.sh b/setup-alpine.sh
index 00b830a..970d1ab 100755
--- a/setup-alpine.sh
+++ b/setup-alpine.sh
@@ -341,5 +341,6 @@ rm .setup.sh
 endgroup
 #-----------------------------------------------------------------------
 
-echo "root-path=$rootfs_dir" >> $GITHUB_OUTPUT
-echo "$rootfs_dir/abin" >> $GITHUB_PATH
+safe_rootfs_dir=$(printf '%s' "$rootfs_dir" | tr -d '\n\r')
+echo "root-path=$safe_rootfs_dir" >> "$GITHUB_OUTPUT"
+echo "$safe_rootfs_dir/abin" >> "$GITHUB_PATH"

```

Locations:

- `setup-alpine.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, prior-fix

**Notes:**

action.yml: webiny/action-post-run is now pinned to commit 2a0e96f0e55f0e698cf2a3d85670e3577ae30a30 (tag 3.1.0 kept as a comment). setup-alpine.sh: added a bash regex check so the branch input must exactly match ^(v[0-9]+\.[0-9]+|edge|latest-stable)$. Upstream's case glob also accepted newlines and any trailing characters. Also re-applied the earlier fix: CR/LF are stripped from rootfs_dir (printf | tr -d '\n\r') before it is written to GITHUB_OUTPUT and GITHUB_PATH, and both file paths are now quoted. I could only see the start of the earlier action.yml patch because it was cut off. The visible part reflowed the input descriptions, which changes no behaviour, so I did not redo it. I left the post-run command's ${{ steps.setup.outputs.root-path }} expression as upstream wrote it. That value comes from the runner's home directory, a branch and an arch that are now both strictly checked, and mktemp, and newlines are stripped from it.

