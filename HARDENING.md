<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.7.2-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **abarichello--godot-ci/4.7.2-stable** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: blocks (sub-rule a). In manual_build.yml, the user-controlled input `${{ github.event.inputs.version }}` is interpolated directly into a shell command (`DOTNET_VERSION=$(./get_dotnet_version.sh ${{ github.event.inputs.version }})`), and `${{ github.event.inputs.release_name }}` is interpolated into another run: step. In release.yml, `${{ github.ref_name }}` is assigned directly to a shell variable (`REF_NAME=${{ github.ref_name }}`). In check-release.yml, `${{ needs.fetch.outputs.release_tag }}` is passed directly to `git tag`. Additionally, `${{ github.repository_owner }}` is interpolated in run: steps in both manual_build.yml and release.yml. All of these allow an attacker to inject arbitrary shell commands.

Locations:

- `.github/workflows/manual_build.yml:29`
- `.github/workflows/manual_build.yml:37`
- `.github/workflows/manual_build.yml:38`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:52`
- `.github/workflows/check-release.yml:35`

### github-env-injection (severity: high)

Untrusted values derived from ${{ }} expressions are written to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). In release.yml, `REF_NAME=${{ github.ref_name }}` is used to derive values written to $GITHUB_OUTPUT (version, release_name). In manual_build.yml, `${{ github.event.inputs.version }}` is used to derive `dotnet_version` written to $GITHUB_OUTPUT, and `${{ github.repository_owner }}` / `${{ github.event.inputs.release_name }}` are written directly to $GITHUB_ENV via `echo IMAGE_OWNER=... >> $GITHUB_ENV` and `echo IMAGE_TAG=... >> $GITHUB_ENV`. None of these writes are preceded by newline-stripping sanitization.

Locations:

- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:52`
- `.github/workflows/manual_build.yml:29`
- `.github/workflows/manual_build.yml:37`
- `.github/workflows/manual_build.yml:38`

### unpinned-uses (severity: high)

All uses: references across all workflow files use mutable tags or version strings instead of full 40-character commit SHA hashes, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references include: actions/checkout@v3, actions/checkout@v4, actions/upload-artifact@v4, actions/download-artifact@v4, softprops/action-gh-release@v0.1.14, JamesIves/github-pages-deploy-action@releases/v4, docker/login-action@v1.14.1, docker/login-action@v1, docker/build-push-action@v2.9.0. Additionally, action.yml uses `image: docker://barichello/godot-ci` with no tag or SHA digest at all, referencing a mutable latest image.

Locations:

- `action.yml:8`
- `.github/workflows/check-release.yml:21`
- `.github/workflows/check-release.yml:28`
- `.github/workflows/check-release.yml:34`
- `.github/workflows/godot-ci.yml:17`
- `.github/workflows/godot-ci.yml:37`
- `.github/workflows/godot-ci.yml:55`
- `.github/workflows/godot-ci.yml:75`
- `.github/workflows/godot-ci.yml:91`
- `.github/workflows/godot-ci.yml:95`
- `.github/workflows/manual_build.yml:26`
- `.github/workflows/manual_build.yml:62`
- `.github/workflows/manual_build.yml:66`
- `.github/workflows/manual_build.yml:70`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:37`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual job within any workflow defines a `permissions:` key. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege. This affects all four workflow files: check-release.yml, godot-ci.yml, manual_build.yml, and release.yml.

Locations:

- `.github/workflows/check-release.yml:1`
- `.github/workflows/godot-ci.yml:1`
- `.github/workflows/manual_build.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four finding types across action.yml and all four workflow files:

1. **script-injection**: Moved all user-controlled ${{ }} expressions (github.event.inputs.version, github.event.inputs.release_name, github.ref_name, github.repository_owner, needs.fetch.outputs.release_tag) from run: blocks into env: blocks, then referenced them as plain shell variables.

2. **github-env-injection**: All values written to $GITHUB_OUTPUT and $GITHUB_ENV are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing.

3. **unpinned-uses**: Pinned all action references to full 40-char SHAs: actions/checkout@v3→a37ce91, actions/checkout@v4→11d5960, actions/upload-artifact@v4→ea165f8, actions/download-artifact@v4→d3f86a1, softprops/action-gh-release@v0.1.14→1e07f43, JamesIves/github-pages-deploy-action@releases/v4→fa24774, docker/login-action@v1.14.1 and @v1→dd4fa06, docker/build-push-action@v2.9.0→7f9d37f. Pinned barichello/godot-ci in action.yml to :latest@sha256:cdcca31e... preserving the docker:// scheme.

4. **missing-permissions**: Added top-level `permissions: {}` to all four workflow files. Added job-level permissions: contents:read for most jobs, contents:write for the release-creating job, packages:write for Docker push jobs.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed script-injection in get_tags job: moved ${{ env.IMAGE_OWNER }}, ${{ env.IMAGE_NAME }}, and ${{ env.IMAGE_TAG }} expressions from run: shell strings into env: blocks (as ENV_IMAGE_OWNER, ENV_IMAGE_NAME, ENV_IMAGE_TAG) for all four affected steps. Fixed github-env-injection in build and build-mono jobs: replaced bare `cat tags.txt >> "$GITHUB_ENV"` with `tr -d '\r' < tags.txt | grep -v '^EOF$'` to strip carriage returns and prevent heredoc delimiter injection before writing to GITHUB_ENV.

