<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.6.3-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **abarichello--godot-ci/4.6.3-stable** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The Docker action image reference in action.yml uses no tag and no SHA digest (`docker://barichello/godot-ci`), which implicitly resolves to the mutable `latest` tag. This means the action will pull whatever image is currently tagged `latest` at runtime, making it vulnerable to supply-chain attacks if the upstream image is replaced with a malicious one. The image reference must be pinned to an immutable SHA digest, e.g. `docker://barichello/godot-ci@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable implicit `latest` tag (`docker://barichello/godot-ci`) to an immutable SHA256 digest: `docker://barichello/godot-ci@sha256:d5af2e2b6e0c1e3c4232103578be316856e2215e4273cc17cebb72822eefa29b` # latest. The digest was resolved via the Docker Registry HTTP API v2.

