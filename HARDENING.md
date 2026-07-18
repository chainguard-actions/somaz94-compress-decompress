<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.9.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable tag-based or version-string refs instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if upstream actions are compromised or tags are moved.

Failing references include:
- ci.yml: actions/checkout@v6, actions/setup-python@v5, actions/upload-artifact@v7
- changelog-generator.yml: actions/checkout@v6, janheinrichmerker/action-github-changelog-generator@v2.4, somaz94/go-git-commit-action@v1
- contributors.yml: actions/checkout@v6, somaz94/go-git-commit-action@v1
- gitlab-mirror.yml: actions/checkout@v6
- issue-greeting.yml: actions/github-script@v8
- linter.yml: actions/checkout@v6, super-linter/super-linter/slim@v8.5.0
- release.yml: actions/checkout@v6, orhun/git-cliff-action@v4, softprops/action-gh-release@v2, somaz94/go-git-commit-action@v1 (×2)
- stale-issues.yml: actions/stale@v10
- use-action.yml: actions/checkout@v6, somaz94/compress-decompress@v1 (×2)

Locations:

- `.github/workflows/ci.yml:18`
- `.github/workflows/changelog-generator.yml:27`
- `.github/workflows/contributors.yml:16`
- `.github/workflows/gitlab-mirror.yml:14`
- `.github/workflows/issue-greeting.yml:12`
- `.github/workflows/linter.yml:18`
- `.github/workflows/release.yml:16`
- `.github/workflows/stale-issues.yml:14`
- `.github/workflows/use-action.yml:30`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions inside shell commands, violating rule (a). This allows expression values to be parsed as shell syntax before the shell ever sees them.

ci.yml — 'Create Test Files and Directories' step: `if [ "${{ matrix.dest_config.type }}" = "custom" ]; then ls -la ${{ matrix.dest_config.dest }}/` — unquoted ${{ matrix.dest_config.dest }} also violates rule (b).

ci.yml — 'Set Source Path' step: `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT` — direct interpolation of matrix values.

ci.yml — 'Verify Contents' step: multiple ${{ matrix.format }}, ${{ matrix.include_root }}, ${{ matrix.source }}, ${{ steps.set-path.outputs.source_path }} interpolated directly in shell.

ci.yml — 'Print Compression Output' step: `echo "Compression output for ${{ matrix.format }} (includeRoot: ${{ matrix.include_root }}): ${{ steps.compress.outputs.file_path }}"`

release.yml — 'Get previous semver tag' step: `CURRENT_TAG="${{ github.ref_name }}"`

release.yml — 'Append full changelog link' step: `echo "**Full Changelog**: https://github.com/${{ github.repository }}/compare/${{ steps.prev_tag.outputs.tag }}...${{ github.ref_name }}" >> RELEASE.md`

release.yml — 'Update major version tag' step: `COMMIT_SHA=$(git rev-parse ${{ github.sha }})` — unquoted ${{ github.sha }} is also a rule (b) violation.

use-action.yml — 'Set Archive Path' step: `if [ "${{ matrix.include_root }}" = "true" ]; then echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT`

use-action.yml — 'Verify Archive Created' step: `if [ ! -f "${{ steps.archive.outputs.path }}" ]`

use-action.yml — 'Verify Extraction' step: `if [ "${{ matrix.include_root }}" = "true" ]`

contributors.yml — 'Get contributors' step: `curl -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" ... "https://api.github.com/repos/${{ github.repository }}/contributors"`

Locations:

- `.github/workflows/ci.yml:55`
- `.github/workflows/ci.yml:75`
- `.github/workflows/ci.yml:107`
- `.github/workflows/ci.yml:148`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:50`
- `.github/workflows/release.yml:72`
- `.github/workflows/use-action.yml:46`
- `.github/workflows/use-action.yml:57`
- `.github/workflows/use-action.yml:68`
- `.github/workflows/contributors.yml:30`

### github-env-injection (severity: high)

Several run: blocks write values derived from untrusted/workflow-controlled inputs directly to $GITHUB_OUTPUT or $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r').

ci.yml — 'Set Source Path' step writes ${{ matrix.dest_config.dest }}, ${{ matrix.dest_config.destfilename }}, ${{ matrix.format }}, ${{ matrix.source }}, and ${{ matrix.include_root }} directly to $GITHUB_OUTPUT:
  `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT`

use-action.yml — 'Set Archive Path' step writes ${{ matrix.format }} directly to $GITHUB_OUTPUT:
  `echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT`

release.yml — 'Update major version tag' step writes MAJOR_VERSION and COMMIT_SHA (derived from ${{ github.ref_name }} and ${{ github.sha }}) to $GITHUB_ENV without sanitization:
  `echo "MAJOR_VERSION=${MAJOR_VERSION}" >> "$GITHUB_ENV"`
  `echo "COMMIT_SHA=${COMMIT_SHA}" >> "$GITHUB_ENV"`

Locations:

- `.github/workflows/ci.yml:75`
- `.github/workflows/use-action.yml:46`
- `.github/workflows/release.yml:72`

### permissions (severity: medium)

The workflow file gitlab-mirror.yml has no top-level `permissions:` key and no job-level `permissions:` key on the `backup` job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/gitlab-mirror.yml:1`

### hardcoded-credentials (severity: high)

ci.yml contains literal hardcoded password values in the `password:` input fields of action steps. While these are test passwords, they match the hardcoded-credentials pattern and establish a bad practice:
- `password: 'test_password_123'` (used in 'Compress with Password' and 'Decompress with Correct Password' steps)
- `password: 'wrong_password'` (used in 'Decompress with Wrong Password' step)
These should use `${{ secrets.* }}` references instead of inline literals.

Locations:

- `.github/workflows/ci.yml:438`
- `.github/workflows/ci.yml:453`
- `.github/workflows/ci.yml:463`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, permissions, hardcoded-credentials

**Notes:**

Fixed all 5 findings across 9 workflow files:

1. unpinned-uses: Pinned all 11 distinct action references to full 40-char SHA digests across ci.yml, changelog-generator.yml, contributors.yml, gitlab-mirror.yml, issue-greeting.yml, linter.yml, release.yml, stale-issues.yml, and use-action.yml.

2. script-injection: Moved all ${{ matrix.* }}, ${{ github.* }}, ${{ steps.*.outcome }}, and ${{ contains(needs.*.result,...) }} expressions from run: shell scripts into env: blocks, referencing them as plain $VAR_NAME shell variables.

3. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to $GITHUB_OUTPUT (ci.yml Set Source Path, use-action.yml Set Archive Path) and $GITHUB_ENV (release.yml Update major version tag).

4. permissions: Added top-level 'permissions: contents: read' to gitlab-mirror.yml.

5. hardcoded-credentials: Replaced literal 'test_password_123' and 'wrong_password' values with ${{ secrets.TEST_PASSWORD }} and ${{ secrets.TEST_WRONG_PASSWORD }} references in ci.yml test-password-encryption job.

