<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.6-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abarichello--godot-ci/4.6-stable** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml reference actions and Docker images by mutable tags or branch names instead of full 40-character commit SHAs or SHA digests. This exposes the action to supply-chain attacks if the upstream tag is moved.

action.yml: `image: 'docker://barichello/godot-ci'` — no tag and no SHA digest.

check-release.yml: `actions/checkout@v3`, `softprops/action-gh-release@v0.1.14`.

godot-ci.yml: `actions/checkout@v4`, `actions/upload-artifact@v4`, `JamesIves/github-pages-deploy-action@releases/v4` (branch ref).

manual_build.yml: `actions/download-artifact@v4`, `actions/checkout@v3`, `docker/login-action@v1.14.1`, `docker/login-action@v1`, `docker/build-push-action@v2.9.0`, `actions/upload-artifact@v4`.

release.yml: `actions/checkout@v3`, `docker/login-action@v1.14.1`, `docker/login-action@v1`, `docker/build-push-action@v2.9.0`.

Locations:

- `action.yml:8`
- `.github/workflows/check-release.yml:19`
- `.github/workflows/check-release.yml:27`
- `.github/workflows/check-release.yml:36`
- `.github/workflows/godot-ci.yml:18`
- `.github/workflows/godot-ci.yml:47`
- `.github/workflows/godot-ci.yml:68`
- `.github/workflows/godot-ci.yml:90`
- `.github/workflows/godot-ci.yml:96`
- `.github/workflows/manual_build.yml:75`
- `.github/workflows/manual_build.yml:82`
- `.github/workflows/manual_build.yml:88`
- `.github/workflows/manual_build.yml:93`
- `.github/workflows/manual_build.yml:97`
- `.github/workflows/manual_build.yml:120`
- `.github/workflows/manual_build.yml:127`
- `.github/workflows/manual_build.yml:133`
- `.github/workflows/manual_build.yml:138`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:40`
- `.github/workflows/release.yml:45`
- `.github/workflows/release.yml:50`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:73`
- `.github/workflows/release.yml:78`

### permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` key, and no individual job within any of them defines a `permissions:` key either. Without explicit permissions, workflows inherit the repository's default token permissions (often `write-all`), granting unnecessary access.

Locations:

- `.github/workflows/check-release.yml:1`
- `.github/workflows/godot-ci.yml:1`
- `.github/workflows/manual_build.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Several `run:` blocks interpolate GitHub Actions expressions (`${{ ... }}`) directly into shell commands, allowing an attacker to inject arbitrary shell code.

**check-release.yml** — `git tag ${{ needs.fetch.outputs.release_tag }}` interpolates a step output directly into a shell command (sub-rule a).

**manual_build.yml** — `MAJOR_VERSION=$(echo ${{ github.event.inputs.version }} | cut -c -1)` and `MINOR_VERSION=$(echo ${{ github.event.inputs.version }} | cut -c -3)` interpolate user-supplied `workflow_dispatch` input directly into shell (sub-rule a). Also: `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV` and `echo IMAGE_TAG=$(echo ${{ github.event.inputs.release_name != 'stable' && ... }}) >> $GITHUB_ENV` (sub-rule a).

**release.yml** — `REF_NAME=${{ github.ref_name }}` interpolates the git ref name (attacker-controllable via tag names) directly into a shell variable assignment (sub-rule a). Also: `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV` (sub-rule a).

Locations:

- `.github/workflows/check-release.yml:38`
- `.github/workflows/manual_build.yml:22`
- `.github/workflows/manual_build.yml:23`
- `.github/workflows/manual_build.yml:43`
- `.github/workflows/manual_build.yml:44`
- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:34`
- `.github/workflows/release.yml:68`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` or `$GITHUB_ENV` without applying the required sanitization (`printf '%s' ... | tr -d '\n\r'`) before the write. This allows newline injection to poison the environment for subsequent steps.

**check-release.yml** — `echo "tag=$(git tag --list --sort=-creatordate | head --lines 1)" >> $GITHUB_OUTPUT` writes unsanitized git tag output to GITHUB_OUTPUT.

**manual_build.yml** — `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV` and `echo IMAGE_TAG=$(echo ${{ github.event.inputs.release_name ... }}) >> $GITHUB_ENV` write unsanitized values to GITHUB_ENV. Also, `echo "dotnet_version=..." >> $GITHUB_OUTPUT` writes values derived from user-supplied `version` input without sanitization.

**release.yml** — `echo "version=${REF_NAME%-*}" >> $GITHUB_OUTPUT` and `echo "release_name=${REF_NAME#*-}" >> $GITHUB_OUTPUT` write values derived from `${{ github.ref_name }}` (attacker-controllable) to GITHUB_OUTPUT without sanitization. Also `echo IMAGE_OWNER=... >> $GITHUB_ENV` writes unsanitized value.

Locations:

- `.github/workflows/check-release.yml:24`
- `.github/workflows/manual_build.yml:43`
- `.github/workflows/manual_build.yml:44`
- `.github/workflows/manual_build.yml:29`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:34`
- `.github/workflows/release.yml:68`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across action.yml and four workflow files:

1. unpinned-uses: Pinned all action uses references to full 40-char SHAs (actions/checkout v3→a37ce91, v4→11d5960; softprops/action-gh-release v0.1.14→1e07f43; actions/upload-artifact v4→ea165f8; JamesIves/github-pages-deploy-action releases/v4→800bbc8; actions/download-artifact v4→d3f86a1; docker/login-action v1.14.1 and v1→dd4fa06; docker/build-push-action v2.9.0→7f9d37f). Pinned action.yml Docker image to barichello/godot-ci:latest@sha256:622e5ca...

2. permissions: Added top-level permissions blocks to all four workflow files with minimal required permissions (check-release.yml: contents:write; godot-ci.yml: contents:read, pages:write, id-token:write; manual_build.yml: contents:read, packages:write; release.yml: contents:read, packages:write).

3. script-injection: Moved all ${{ }} expressions from run: blocks into step env: blocks and referenced as plain shell variables. Affected: check-release.yml (release_tag), manual_build.yml (version input, repository_owner, release_name), release.yml (ref_name, repository_owner).

4. github-env-injection: Applied printf '%s' ... | tr -d '\n\r' sanitization before all writes to GITHUB_OUTPUT and GITHUB_ENV for values derived from external/attacker-controllable sources.

