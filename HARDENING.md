<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.15.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.15.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The Docker action's runs.image points to a mutable tag, not a SHA256 digest: `image: docker://ghcr.io/somaz94/compress-decompress:v1.15.3`. Pin it to a digest, e.g. `docker://ghcr.io/somaz94/compress-decompress@sha256:<digest> # v1.15.3`.

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned runs.image to docker://ghcr.io/somaz94/compress-decompress:v1.15.3@sha256:55749fc61a32d4d7048706760164e8bb93e0ae12a443d6914b889c27e97657fd

