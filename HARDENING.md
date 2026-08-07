<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.11.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image with a mutable tag instead of a SHA digest: `image: docker://ghcr.io/somaz94/compress-decompress:v1.11.0`. This is vulnerable to supply-chain attacks if the tag is moved to a different image.

Locations:

- `action.yml:76`

### unpinned-uses (severity: high)

Multiple workflow files use `uses:` references pinned to mutable tags or branch names instead of full 40-character SHA commits. Failing references include: `actions/checkout@v7`, `actions/setup-python@v7`, `actions/upload-artifact@v7`, `somaz94/go-changelog-action@v1`, `somaz94/go-git-commit-action@v1`, `somaz94/multi-git-mirror@v1`, `somaz94/compress-decompress@v1`, and multiple reusable workflow calls at `@main`.

Locations:

- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:23`
- `.github/workflows/ci.yml:88`
- `.github/workflows/changelog-generator.yml:22`
- `.github/workflows/changelog-generator.yml:26`
- `.github/workflows/changelog-generator.yml:30`
- `.github/workflows/gitlab-mirror.yml:29`
- `.github/workflows/gitlab-mirror.yml:33`
- `.github/workflows/use-action.yml:28`
- `.github/workflows/use-action.yml:44`
- `.github/workflows/auto-assign.yml:11`
- `.github/workflows/contributors.yml:13`
- `.github/workflows/dependabot-auto-merge.yml:12`
- `.github/workflows/issue-greeting.yml:11`
- `.github/workflows/labels.yml:11`
- `.github/workflows/lock-threads.yml:12`
- `.github/workflows/ok-to-test.yml:12`
- `.github/workflows/pr-size.yml:11`
- `.github/workflows/pr-welcome.yml:11`
- `.github/workflows/release.yml:24`
- `.github/workflows/semantic-pr.yml:11`
- `.github/workflows/stale-issues.yml:11`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in ci.yml directly interpolate `${{ matrix.* }}`, `${{ steps.*.outputs.* }}`, and `${{ contains(needs.*.result, ...) }}` expressions inside shell commands. Examples include: `ls -la ${{ matrix.dest_config.dest }}/` (unquoted), `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT`, `if [ "${{ matrix.dest_config.type }}" = "custom" ]`, `echo "Compression output for ${{ matrix.format }} (includeRoot: ${{ matrix.include_root }}): ${{ steps.compress.outputs.file_path }}"`, and `if [ "${{ contains(needs.*.result, 'failure') }}" = "true" ]`. Any of these expressions could contain shell metacharacters injected via matrix values or step outputs.

Locations:

- `.github/workflows/ci.yml:47`
- `.github/workflows/ci.yml:55`
- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:59`
- `.github/workflows/ci.yml:61`
- `.github/workflows/ci.yml:97`
- `.github/workflows/ci.yml:99`
- `.github/workflows/ci.yml:101`
- `.github/workflows/ci.yml:103`
- `.github/workflows/ci.yml:105`
- `.github/workflows/ci.yml:155`
- `.github/workflows/ci.yml:1706`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in use-action.yml directly interpolate `${{ matrix.include_root }}`, `${{ matrix.format }}`, and `${{ steps.archive.outputs.path }}` expressions inside shell commands. Examples: `if [ "${{ matrix.include_root }}" = "true" ]`, `echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT`, `if [ ! -f "${{ steps.archive.outputs.path }}" ]`, `echo "❌ Archive not found: ${{ steps.archive.outputs.path }}"`. These expressions are interpolated before the shell parses the command, allowing injection of shell metacharacters.

Locations:

- `.github/workflows/use-action.yml:42`
- `.github/workflows/use-action.yml:43`
- `.github/workflows/use-action.yml:44`
- `.github/workflows/use-action.yml:50`
- `.github/workflows/use-action.yml:52`
- `.github/workflows/use-action.yml:57`
- `.github/workflows/use-action.yml:62`
- `.github/workflows/use-action.yml:70`

### github-env-injection (severity: high)

The 'Set Source Path' step in ci.yml writes `${{ matrix.dest_config.dest }}`, `${{ matrix.dest_config.destfilename }}`, `${{ matrix.format }}`, `${{ matrix.source }}`, and `${{ matrix.include_root }}` values directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A matrix value containing a newline could inject arbitrary key=value pairs into the output file.

Locations:

- `.github/workflows/ci.yml:55`
- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:59`
- `.github/workflows/ci.yml:61`

### github-env-injection (severity: high)

The 'Set Archive Path' step in use-action.yml writes `${{ matrix.format }}` directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A matrix value containing a newline could inject arbitrary key=value pairs into the output file.

Locations:

- `.github/workflows/use-action.yml:43`
- `.github/workflows/use-action.yml:45`

### hardcoded-credentials (severity: high)

A literal hardcoded password `test_password_123` is assigned to the `password:` input in the 'Compress with Password' and 'Decompress with Correct Password' steps. Even though this is a test workflow, hardcoded credential values in workflow files are a security risk and should be replaced with secrets references (e.g., `${{ secrets.TEST_PASSWORD }}`).

Locations:

- `.github/workflows/ci.yml:1641`
- `.github/workflows/ci.yml:1655`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all 7 findings across action.yml and 16 workflow files:

1. action.yml: Pinned docker://ghcr.io/somaz94/compress-decompress:v1.11.0 to SHA256 digest.

2. Workflow files: Pinned all mutable references - actions/checkout@v7, actions/setup-python@v7, actions/upload-artifact@v7, somaz94/go-changelog-action@v1, somaz94/go-git-commit-action@v1, somaz94/multi-git-mirror@v1, somaz94/compress-decompress@v1, and all 12 somaz94/.github reusable workflow @main references to their full commit SHAs.

3. ci.yml script-injection: Moved ${{ matrix.* }}, ${{ steps.*.outputs.* }}, ${{ steps.*.outcome }}, and ${{ contains(needs.*.result,...) }} expressions from run: blocks to env: blocks in 11 steps.

4. use-action.yml script-injection: Moved ${{ matrix.include_root }}, ${{ matrix.format }}, and ${{ steps.archive.outputs.path }} from run: blocks to env: blocks in 3 steps.

5. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to $GITHUB_OUTPUT in Set Source Path (ci.yml) and Set Archive Path (use-action.yml).

6. hardcoded-credentials: Replaced literal 'test_password_123' with ${{ secrets.TEST_PASSWORD }} in both password steps in ci.yml.

