<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.7.1-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abarichello--godot-ci/4.7.1-stable** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands, enabling script injection. Sub-rule (a) violations:
- check-release.yml line 35: `git tag ${{ needs.fetch.outputs.release_tag }}` — a step output (sourced from an external GitHub API call) is interpolated directly into a shell command.
- manual_build.yml line 26: `DOTNET_VERSION=$(./get_dotnet_version.sh ${{ github.event.inputs.version }})` — user-controlled workflow_dispatch input injected directly into a shell command.
- manual_build.yml line 34: `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr ...) >> $GITHUB_ENV` — github context interpolated directly in run:.
- manual_build.yml line 35: `echo IMAGE_TAG=$(echo ${{ github.event.inputs.release_name != 'stable' && ... }}) >> $GITHUB_ENV` — user-controlled input interpolated directly in run:.
- release.yml line 18: `REF_NAME=${{ github.ref_name }}` — github context interpolated directly in run:.
- release.yml line 29: `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr ...) >> $GITHUB_ENV` — github context interpolated directly in run:.
- release.yml line 57: same IMAGE_OWNER pattern repeated in build-mono job.

Locations:

- `.github/workflows/check-release.yml:35`
- `.github/workflows/manual_build.yml:26`
- `.github/workflows/manual_build.yml:34`
- `.github/workflows/manual_build.yml:35`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:57`

### github-env-injection (severity: high)

Untrusted or workflow-controlled values are written to GITHUB_ENV and GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'):
- manual_build.yml line 34: `github.repository_owner` written directly to $GITHUB_ENV via `echo IMAGE_OWNER=... >> $GITHUB_ENV`.
- manual_build.yml line 35: `github.event.inputs.release_name` (user-controlled workflow_dispatch input) written directly to $GITHUB_ENV via `echo IMAGE_TAG=... >> $GITHUB_ENV`.
- release.yml line 18: `github.ref_name` assigned to REF_NAME, then VERSION and release_name derived from it are written to $GITHUB_OUTPUT (lines 20-21) without sanitization.
- release.yml line 29: `github.repository_owner` written directly to $GITHUB_ENV.
- release.yml line 57: `github.repository_owner` written directly to $GITHUB_ENV in build-mono job.

Locations:

- `.github/workflows/manual_build.yml:34`
- `.github/workflows/manual_build.yml:35`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:57`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags/versions rather than immutable 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if those tags are moved or compromised.

action.yml: `image: 'docker://barichello/godot-ci'` — no tag or digest at all (uses the implicit :latest mutable tag).

check-release.yml:
- `uses: actions/checkout@v3` (line 25)
- `uses: actions/checkout@v3` (line 30)
- `uses: softprops/action-gh-release@v0.1.14` (line 38)

godot-ci.yml:
- `uses: actions/checkout@v4` (lines 17, 44, 67, 91)
- `uses: actions/upload-artifact@v4` (lines 32, 55, 79, 104)
- `uses: JamesIves/github-pages-deploy-action@releases/v4` (line 84) — branch ref, not a SHA

manual_build.yml:
- `uses: actions/checkout@v3` (line 24)
- `uses: actions/download-artifact@v4` (line 68)
- `uses: actions/checkout@v3` (line 73)
- `uses: docker/login-action@v1.14.1` (lines 74, 80)
- `uses: docker/login-action@v1` (lines 80, 87)
- `uses: docker/build-push-action@v2.9.0` (line 89)
- Same pattern repeated in build-mono job (lines 96–130)
- `uses: actions/upload-artifact@v4` (lines 57, 62)

release.yml:
- `uses: actions/checkout@v3` (lines 16, 27, 52)
- `uses: docker/login-action@v1.14.1` (lines 30, 55)
- `uses: docker/login-action@v1` (lines 36, 61)
- `uses: docker/build-push-action@v2.9.0` (lines 41, 66)

Locations:

- `action.yml:8`
- `.github/workflows/check-release.yml:25`
- `.github/workflows/check-release.yml:30`
- `.github/workflows/check-release.yml:38`
- `.github/workflows/godot-ci.yml:17`
- `.github/workflows/godot-ci.yml:32`
- `.github/workflows/godot-ci.yml:44`
- `.github/workflows/godot-ci.yml:55`
- `.github/workflows/godot-ci.yml:67`
- `.github/workflows/godot-ci.yml:79`
- `.github/workflows/godot-ci.yml:84`
- `.github/workflows/godot-ci.yml:91`
- `.github/workflows/godot-ci.yml:104`
- `.github/workflows/manual_build.yml:24`
- `.github/workflows/manual_build.yml:57`
- `.github/workflows/manual_build.yml:62`
- `.github/workflows/manual_build.yml:68`
- `.github/workflows/manual_build.yml:73`
- `.github/workflows/manual_build.yml:74`
- `.github/workflows/manual_build.yml:80`
- `.github/workflows/manual_build.yml:89`
- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:36`
- `.github/workflows/release.yml:41`
- `.github/workflows/release.yml:52`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:61`
- `.github/workflows/release.yml:66`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` key, and no individual job within any of these files defines a `permissions:` key either. This means all jobs run with the default (often write-all) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/check-release.yml:1`
- `.github/workflows/godot-ci.yml:1`
- `.github/workflows/manual_build.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four finding types across 5 files:

**action.yml**: Pinned docker image from bare `docker://barichello/godot-ci` (implicit :latest) to `docker://barichello/godot-ci:latest@sha256:622e5ca81b54cd8038ecf7de5d157b47efc800d7cf635af2eec18a6aee4bab7e`.

**check-release.yml**: Added `permissions: contents: write`; pinned `actions/checkout@v3` → SHA `a37ce91...`, `softprops/action-gh-release@v0.1.14` → SHA `1e07f43...`; fixed script injection by moving `needs.fetch.outputs.release_tag` into env var `RELEASE_TAG` and sanitizing with `tr -d '\n\r'` before `git tag`; sanitized GITHUB_OUTPUT write in fetch job.

**godot-ci.yml**: Added `permissions: contents: read, pages: write, id-token: write`; pinned `actions/checkout@v4` → SHA `11d5960...`, `actions/upload-artifact@v4` → SHA `ea165f8...`, `JamesIves/github-pages-deploy-action@releases/v4` → SHA `800bbc8...`.

**manual_build.yml**: Added `permissions: contents: read, packages: write`; pinned all actions to SHAs; fixed script injection by moving `github.event.inputs.version` → `INPUT_VERSION`, `github.repository_owner` → `REPO_OWNER`, `github.event.inputs.release_name` → `INPUT_RELEASE_NAME` into env blocks; fixed github-env-injection by sanitizing IMAGE_OWNER and IMAGE_TAG with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_ENV.

**release.yml**: Added `permissions: contents: read, packages: write`; pinned all actions to SHAs; fixed script injection by moving `github.ref_name` → `REF_NAME` and `github.repository_owner` → `REPO_OWNER` into env blocks; fixed github-env-injection by sanitizing all values written to GITHUB_OUTPUT and GITHUB_ENV with `printf '%s' | tr -d '\n\r'`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in four run: steps of the get_tags job in .github/workflows/manual_build.yml. Moved ${{ env.IMAGE_OWNER }}, ${{ env.IMAGE_NAME }}, ${{ env.IMAGE_TAG }}, and ${{ secrets.DOCKERHUB_USERNAME }} expressions from inline shell interpolation into each step's env: block (as IMAGE_OWNER_VAR, IMAGE_NAME_VAR, IMAGE_TAG_VAR, DOCKERHUB_USERNAME respectively). The shell scripts now reference these as plain environment variables, preventing shell metacharacter injection via the YAML template engine.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed three github-env-injection vulnerabilities:
1. manual_build.yml 'Set tags' step: Added `safe_version=$(printf '%s' "$INPUT_VERSION" | tr -d '\n\r')` and used `${safe_version}` when writing to tags.txt, preventing newline injection when the file is later cat'd into $GITHUB_ENV via heredoc.
2. manual_build.yml 'Set Mono tags' step: Same sanitization applied for tags_mono.txt.
3. check-release.yml 'current' job parse step: Replaced single-line `echo "tag=$(git tag ...)" >> $GITHUB_OUTPUT` with a multi-line script that captures the tag into a variable, sanitizes it with `printf '%s' ... | tr -d '\n\r'`, then writes the safe value to $GITHUB_OUTPUT — matching the pattern already used in the 'fetch' job.

