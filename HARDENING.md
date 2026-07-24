<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.6.3-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abarichello--godot-ci/4.6.3-stable** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image reference `docker://barichello/godot-ci` with no tag and no SHA digest — this is a fully mutable reference that can be silently replaced. All four workflow files also use tag-based (not SHA-pinned) `uses:` references: actions/checkout@v3, actions/checkout@v4, actions/upload-artifact@v4, actions/download-artifact@v4, softprops/action-gh-release@v0.1.14, JamesIves/github-pages-deploy-action@releases/v4, docker/login-action@v1.14.1, docker/login-action@v1, docker/build-push-action@v2.9.0. None of these are pinned to a full 40-character commit SHA.

Locations:

- `action.yml:8`
- `.github/workflows/check-release.yml:27`
- `.github/workflows/check-release.yml:33`
- `.github/workflows/check-release.yml:40`
- `.github/workflows/godot-ci.yml:19`
- `.github/workflows/godot-ci.yml:34`
- `.github/workflows/godot-ci.yml:43`
- `.github/workflows/godot-ci.yml:57`
- `.github/workflows/godot-ci.yml:68`
- `.github/workflows/godot-ci.yml:82`
- `.github/workflows/godot-ci.yml:93`
- `.github/workflows/godot-ci.yml:102`
- `.github/workflows/manual_build.yml:27`
- `.github/workflows/manual_build.yml:62`
- `.github/workflows/manual_build.yml:63`
- `.github/workflows/manual_build.yml:70`
- `.github/workflows/manual_build.yml:72`
- `.github/workflows/manual_build.yml:76`
- `.github/workflows/manual_build.yml:80`
- `.github/workflows/manual_build.yml:93`
- `.github/workflows/manual_build.yml:101`
- `.github/workflows/manual_build.yml:103`
- `.github/workflows/manual_build.yml:107`
- `.github/workflows/manual_build.yml:111`
- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:35`
- `.github/workflows/release.yml:38`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:59`
- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:66`

### permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` key, and no individual job within any file defines its own `permissions:` block. This means all jobs run with the default (broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/check-release.yml:1`
- `.github/workflows/godot-ci.yml:1`
- `.github/workflows/manual_build.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing an attacker to inject arbitrary shell commands. Affected lines:
- check-release.yml line 38: `git tag ${{ needs.fetch.outputs.release_tag }}` — step output injected directly into a shell command.
- manual_build.yml line 30: `DOTNET_VERSION=$(./get_dotnet_version.sh ${{ github.event.inputs.version }})` — user-controlled workflow_dispatch input injected directly.
- manual_build.yml line 39: `run: echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV` — expression interpolated directly in run.
- manual_build.yml line 40: `run: echo IMAGE_TAG=$(echo ${{ github.event.inputs.release_name != 'stable' && ... }}) >> $GITHUB_ENV` — user-controlled input interpolated directly.
- release.yml line 19: `REF_NAME=${{ github.ref_name }}` — tag/branch name injected directly into shell.
- release.yml line 30: `run: echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV` — expression interpolated directly.
- release.yml line 58: same IMAGE_OWNER pattern in build-mono job.

Locations:

- `.github/workflows/check-release.yml:38`
- `.github/workflows/manual_build.yml:30`
- `.github/workflows/manual_build.yml:39`
- `.github/workflows/manual_build.yml:40`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:58`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted GitHub context expressions directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines to set arbitrary environment variables for subsequent steps.
- manual_build.yml line 39: `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV` — github.repository_owner written to GITHUB_ENV unsanitized.
- manual_build.yml line 40: `echo IMAGE_TAG=$(echo ${{ github.event.inputs.release_name ... }}) >> $GITHUB_ENV` — user-controlled workflow_dispatch input written to GITHUB_ENV unsanitized.
- release.yml line 30: `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV` — same pattern in build job.
- release.yml line 58: same IMAGE_OWNER pattern in build-mono job, also written to GITHUB_ENV unsanitized.

Locations:

- `.github/workflows/manual_build.yml:39`
- `.github/workflows/manual_build.yml:40`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four finding types across action.yml and all four workflow files:

1. unpinned-uses: Pinned all action references to full 40-char commit SHAs (actions/checkout v3/v4, actions/upload-artifact v4, actions/download-artifact v4, softprops/action-gh-release v0.1.14, JamesIves/github-pages-deploy-action releases/v4, docker/login-action v1.14.1/v1, docker/build-push-action v2.9.0). Also pinned container images: barichello/godot-ci (no tag → latest@sha256:...) in action.yml and barichello/godot-ci:4.3@sha256:... in godot-ci.yml.

2. permissions: Added top-level permissions blocks to all four workflow files. check-release.yml gets contents:write (needs to push tags and create releases). godot-ci.yml gets contents:read, pages:write, id-token:write (for GitHub Pages deploy). manual_build.yml and release.yml get contents:read, packages:write (for pushing Docker images to ghcr.io).

3. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks. Fixed: git tag ${{ needs.fetch.outputs.release_tag }} in check-release.yml, ./get_dotnet_version.sh ${{ github.event.inputs.version }} in manual_build.yml, REF_NAME=${{ github.ref_name }} in release.yml.

4. github-env-injection: Replaced direct echo of github context values to GITHUB_ENV with sanitized versions using printf '%s' "$VAR" | tr -d '\n\r'. Fixed IMAGE_OWNER in manual_build.yml and both build/build-mono jobs in release.yml. Also fixed IMAGE_TAG in manual_build.yml using a bash conditional instead of the ternary expression.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all findings across two workflow files:

1. manual_build.yml - script-injection: Moved ${{ env.IMAGE_OWNER }}, ${{ env.IMAGE_NAME }}, and ${{ env.IMAGE_TAG }} expressions from inline shell strings into env: blocks for the 'Set tags', 'Set latest tags', 'Set Mono tags', and 'Set Mono latest tags' steps. They are now referenced as plain shell variables.

2. manual_build.yml - github-env-injection (line 36): Added sanitization for DOTNET_VERSION before writing to $GITHUB_OUTPUT using printf '%s' | tr -d newlines.

3. manual_build.yml - github-env-injection (lines 103, 121): Replaced bare 'cat tags.txt >> GITHUB_ENV' with a loop that strips carriage returns from each line before writing to GITHUB_ENV via heredoc syntax.

4. release.yml - github-env-injection (lines 26-29): Added sanitization for VERSION, release_name, and DOTNET_VERSION before writing each to $GITHUB_OUTPUT using printf '%s' | tr -d newlines.

