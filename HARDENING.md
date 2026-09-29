<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.15.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.15.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable version tag instead of an immutable SHA256 digest. The image `docker://ghcr.io/somaz94/compress-decompress:v1.15.2` uses the tag `v1.15.2`, which can be silently replaced with different (potentially malicious) content. It should be pinned to a specific SHA256 digest, e.g. `docker://ghcr.io/somaz94/compress-decompress@sha256:<64-hex-char-digest> # v1.15.2`.

Locations:

- `action.yml:96`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from `docker://ghcr.io/somaz94/compress-decompress:v1.15.2` to `docker://ghcr.io/somaz94/compress-decompress:v1.15.2@sha256:279040fffadd8c702c26bd31d07cd4fb34345c03ce2380a4f4ccb1ea1b1445e1`. The `docker://` scheme and tag are preserved; the SHA256 digest makes the reference immutable.

