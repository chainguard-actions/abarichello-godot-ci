<!-- markdownlint-disable -->

# Hardening Report: abarichello--godot-ci/4.6-stable

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **abarichello--godot-ci/4.6-stable** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference `docker://barichello/godot-ci` with no tag and no SHA digest (implicitly `latest`). This is a mutable reference that can change at any time, making the action vulnerable to supply-chain attacks. The image should be pinned to a specific SHA digest, e.g. `docker://barichello/godot-ci@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable `docker://barichello/godot-ci` (implicit latest) to the immutable digest `docker://barichello/godot-ci@sha256:d3b5e88853476a4d00a52e72b3c27b65fc622d6f09137e6f129c24bbd2ebdc7c` (corresponding to the `4.6` tag, the closest available tag for this 4.6-stable action). The `4.6-stable` tag does not exist on Docker Hub; `4.6` was used instead.

