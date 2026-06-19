<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.7-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **abarichello--godot-ci/4.7-stable** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml Docker action references the image 'docker://barichello/godot-ci' with no tag and no SHA digest. This resolves to the mutable 'latest' tag, making the action vulnerable to supply-chain attacks where the upstream image could be replaced with a malicious version. The image reference must be pinned to a specific SHA256 digest (e.g., 'docker://barichello/godot-ci@sha256:<64-hex-char-digest>') to ensure reproducibility and integrity.

Locations:

- `action.yml:8`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable 'docker://barichello/godot-ci' (no tag, resolves to latest) to 'docker://barichello/godot-ci@sha256:d5af2e2b6e0c1e3c4232103578be316856e2215e4273cc17cebb72822eefa29b' with a '# latest' comment. The SHA256 digest was resolved via the Docker Registry API. Note: the 4.7-stable tag was not found in the registry, so the latest digest was used instead.

