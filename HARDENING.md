<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.7-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abarichello--godot-ci/4.7-stable** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a mutable Docker image reference 'docker://barichello/godot-ci' with no tag (defaults to 'latest'), which is not pinned to a SHA digest. All four workflow files also use mutable tag/version refs instead of pinned 40-character SHA commits:
- check-release.yml: actions/checkout@v3, softprops/action-gh-release@v0.1.14
- godot-ci.yml: actions/checkout@v4, actions/upload-artifact@v4, JamesIves/github-pages-deploy-action@releases/v4
- manual_build.yml: actions/checkout@v3, actions/upload-artifact@v4, actions/download-artifact@v4, docker/login-action@v1.14.1, docker/login-action@v1, docker/build-push-action@v2.9.0
- release.yml: actions/checkout@v3, docker/login-action@v1.14.1, docker/login-action@v1, docker/build-push-action@v2.9.0

Locations:

- `action.yml:8`
- `.github/workflows/check-release.yml:27`
- `.github/workflows/check-release.yml:35`
- `.github/workflows/check-release.yml:43`
- `.github/workflows/godot-ci.yml:18`
- `.github/workflows/godot-ci.yml:32`
- `.github/workflows/godot-ci.yml:51`
- `.github/workflows/godot-ci.yml:65`
- `.github/workflows/godot-ci.yml:79`
- `.github/workflows/manual_build.yml:24`
- `.github/workflows/manual_build.yml:57`
- `.github/workflows/manual_build.yml:60`
- `.github/workflows/manual_build.yml:63`
- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:34`
- `.github/workflows/release.yml:38`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:61`
- `.github/workflows/release.yml:65`

### permissions (severity: medium)

None of the workflow files define a top-level 'permissions:' key, and no individual job defines its own 'permissions:' block. This means workflows run with the default (broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/check-release.yml:1`
- `.github/workflows/godot-ci.yml:1`
- `.github/workflows/manual_build.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands (sub-rule a), allowing an attacker to inject arbitrary shell commands:

1. release.yml: `REF_NAME=${{ github.ref_name }}` — a tag name (attacker-influenced via release events) is interpolated directly into the shell.
2. release.yml (build job): `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr ...) >> $GITHUB_ENV` — expression interpolated directly in run.
3. release.yml (build-mono job): same IMAGE_OWNER pattern repeated.
4. check-release.yml: `git tag ${{ needs.fetch.outputs.release_tag }}` — step output (derived from external Godot release tag) interpolated directly into shell.
5. manual_build.yml: `DOTNET_VERSION=$(./get_dotnet_version.sh ${{ github.event.inputs.version }})` — user-supplied workflow_dispatch input interpolated directly into shell command.
6. manual_build.yml: `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr ...) >> $GITHUB_ENV` — expression interpolated directly in run.
7. manual_build.yml: `echo IMAGE_TAG=$(echo ${{ github.event.inputs.release_name ... }}) >> $GITHUB_ENV` — user-supplied input interpolated directly in run.

Locations:

- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:57`
- `.github/workflows/check-release.yml:38`
- `.github/workflows/manual_build.yml:26`
- `.github/workflows/manual_build.yml:31`
- `.github/workflows/manual_build.yml:32`

### github-env-injection (severity: high)

Multiple run: steps write values derived from untrusted/workflow-controlled expressions directly to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'):

1. release.yml line 19: `REF_NAME=${{ github.ref_name }}` followed by `echo "version=$VERSION" >> $GITHUB_OUTPUT` and `echo "release_name=${REF_NAME#*-}" >> $GITHUB_OUTPUT` — github.ref_name flows unsanitized into GITHUB_OUTPUT.
2. release.yml line 29: `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV` — expression written directly to GITHUB_ENV without newline sanitization.
3. release.yml line 57: same IMAGE_OWNER pattern in build-mono job.
4. manual_build.yml line 26: `DOTNET_VERSION=$(./get_dotnet_version.sh ${{ github.event.inputs.version }})` then `echo "dotnet_version=$DOTNET_VERSION" >> $GITHUB_OUTPUT` — user input flows unsanitized into GITHUB_OUTPUT.
5. manual_build.yml line 31: `echo IMAGE_OWNER=$(echo ${{ github.repository_owner }} | tr ...) >> $GITHUB_ENV` — written directly to GITHUB_ENV.
6. manual_build.yml line 32: `echo IMAGE_TAG=$(echo ${{ github.event.inputs.release_name ... }}) >> $GITHUB_ENV` — user-supplied input written directly to GITHUB_ENV.

Locations:

- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:57`
- `.github/workflows/manual_build.yml:26`
- `.github/workflows/manual_build.yml:27`
- `.github/workflows/manual_build.yml:31`
- `.github/workflows/manual_build.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four finding categories:

1. **unpinned-uses**: Pinned all action references to full commit SHAs with tag comments:
   - actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - softprops/action-gh-release@v0.1.14 → @1e07f4398721186383de40550babbdf2b84acfc5
   - actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
   - actions/download-artifact@v4 → @d3f86a106a0bac45b974a628896c90dbdf5c8093
   - JamesIves/github-pages-deploy-action@releases/v4 → @800bbc83122db9d4c38fcd284782d5afe62e52df
   - docker/login-action@v1.14.1 and @v1 → @dd4fa0671be5250ee6f50aedf4cb05514abda2c7
   - docker/build-push-action@v2.9.0 → @7f9d37fa544684fb73bfe4835ed7214c255ce02b
   - action.yml Docker image pinned: docker://barichello/godot-ci:latest@sha256:622e5ca81b54cd8038ecf7de5d157b47efc800d7cf635af2eec18a6aee4bab7e

2. **permissions**: Added top-level `permissions: {}` to all four workflow files. Added job-level permissions: `contents: read` for most jobs, `contents: write` for the create job in check-release.yml, and `packages: write` for Docker push jobs in release.yml and manual_build.yml.

3. **script-injection**: Moved all `${{ github.ref_name }}`, `${{ github.repository_owner }}`, `${{ github.event.inputs.version }}`, `${{ github.event.inputs.release_name }}`, and `${{ needs.fetch.outputs.release_tag }}` expressions out of `run:` blocks into `env:` blocks, referencing them as plain shell variables.

4. **github-env-injection**: All values written to $GITHUB_OUTPUT and $GITHUB_ENV are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing. The IMAGE_OWNER computation now uses a safe intermediate variable. The IMAGE_TAG computation in manual_build.yml was rewritten as a proper shell conditional to avoid direct expression interpolation.

