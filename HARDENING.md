<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.6.1-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **abarichello--godot-ci/4.6.1-stable** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference without a tag or SHA digest: `image: 'docker://barichello/godot-ci'`. This is a mutable reference — if the `latest` (or default) tag on Docker Hub is updated, the action will silently pull a different image on the next run, enabling supply-chain attacks. The image reference must be pinned to a specific SHA digest, e.g. `docker://barichello/godot-ci@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Fixed the unpinned Docker image reference in action.yml. Changed `docker://barichello/godot-ci` (no tag or digest) to `docker://barichello/godot-ci@sha256:d5af2e2b6e0c1e3c4232103578be316856e2215e4273cc17cebb72822eefa29b` with a `# latest` comment outside the YAML quotes. The `4.6.1-stable` and `4.6-stable` tags were not found on Docker Hub, so the `latest` tag digest was used. The comment is placed outside the single quotes per the required format.

