<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.6.1-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abarichello--godot-ci/4.6.1-stable** was hardened automatically. 1 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker action with a mutable image reference `docker://barichello/godot-ci` (no explicit tag and no SHA digest). This means the action will pull whatever image is currently tagged as `latest` on Docker Hub, making it vulnerable to supply-chain attacks if the upstream image is replaced with a malicious one. The image reference must be pinned to a specific SHA digest, e.g. `docker://barichello/godot-ci@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from `docker://barichello/godot-ci` (no tag, no digest) to `docker://barichello/godot-ci:latest@sha256:622e5ca81b54cd8038ecf7de5d157b47efc800d7cf635af2eec18a6aee4bab7e`. The docker:// scheme is preserved (required for GitHub Actions Docker container actions), the :latest tag is kept inline, and the SHA256 digest pins the reference to an immutable image to prevent supply-chain attacks.

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four workflow files:

**check-release.yml**: Added `permissions: {}` top-level and job-level permissions. Pinned actions/checkout@v3 and softprops/action-gh-release@v0.1.14 to full SHAs. Moved `needs.fetch.outputs.release_tag` from direct shell interpolation into an `env:` block. Sanitized TAG values written to $GITHUB_OUTPUT with `printf '%s' | tr -d '\n\r'`. Fixed the `current` job to use a proper multi-line run block with sanitization.

**release.yml**: Added `permissions: {}` top-level and job-level permissions. Pinned all actions to full SHAs. Moved `github.ref_name` into an `env:` block (REF_NAME) and sanitized all values written to $GITHUB_OUTPUT. Moved `github.repository_owner` into an `env:` block and sanitized before writing to $GITHUB_ENV.

**manual_build.yml**: Added `permissions: {}` top-level and job-level permissions. Pinned all actions to full SHAs. Moved `github.event.inputs.version` and `github.repository_owner` and `github.event.inputs.release_name` into `env:` blocks. Replaced the ternary expression for IMAGE_TAG with a proper bash if/else. Sanitized all values written to $GITHUB_OUTPUT and $GITHUB_ENV. Replaced direct `${{ }}` interpolation in tag-writing run steps with env vars.

**godot-ci.yml**: Added `permissions: {}` top-level and job-level permissions (contents: read for most jobs, plus pages: write and id-token: write for the web export job that deploys to GitHub Pages). Pinned actions/checkout@v4, actions/upload-artifact@v4, and JamesIves/github-pages-deploy-action@releases/v4 to full SHAs.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection occurrences in .github/workflows/manual_build.yml:
1. build job (was line 103): Replaced `cat tags.txt >> "$GITHUB_ENV"` inside a heredoc with a while-loop that reads each line, sanitizes it with `printf '%s' "$line" | tr -d '\n\r'`, and writes the sanitized value using `printf 'TAGS<<EOF\n%s\nEOF\n' "$TAGS_VALUE" >> "$GITHUB_ENV"`.
2. build-mono job (was line 133): Same fix applied for `tags_mono.txt`.
This prevents newline injection from user-controlled `version` and `release_name` workflow_dispatch inputs from being used to inject arbitrary environment variables into $GITHUB_ENV.

