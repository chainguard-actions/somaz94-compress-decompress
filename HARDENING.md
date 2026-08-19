<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.9.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files is pinned to a mutable version tag rather than an immutable 40-character SHA commit hash, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

Affected references include (non-exhaustive):
- actions/checkout@v6
- actions/setup-python@v5
- actions/upload-artifact@v7
- actions/github-script@v8
- actions/stale@v10
- janheinrichmerker/action-github-changelog-generator@v2.4
- somaz94/go-git-commit-action@v1
- orhun/git-cliff-action@v4
- softprops/action-gh-release@v2
- super-linter/super-linter/slim@v8.5.0
- somaz94/compress-decompress@v1

Locations:

- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:26`
- `.github/workflows/ci.yml:75`
- `.github/workflows/changelog-generator.yml:26`
- `.github/workflows/changelog-generator.yml:42`
- `.github/workflows/changelog-generator.yml:48`
- `.github/workflows/contributors.yml:17`
- `.github/workflows/contributors.yml:40`
- `.github/workflows/gitlab-mirror.yml:14`
- `.github/workflows/issue-greeting.yml:11`
- `.github/workflows/linter.yml:19`
- `.github/workflows/linter.yml:24`
- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:52`
- `.github/workflows/stale-issues.yml:13`
- `.github/workflows/use-action.yml:30`
- `.github/workflows/use-action.yml:40`
- `.github/workflows/use-action.yml:57`

### missing-permissions (severity: medium)

`.github/workflows/gitlab-mirror.yml` has no top-level `permissions:` key and its only job (`backup`) also has no job-level `permissions:` key. Without an explicit permissions block the workflow inherits the default repository permissions, which may include write access to contents and other scopes.

Locations:

- `.github/workflows/gitlab-mirror.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) inside shell command strings (rule a). Before the shell executes the command, GitHub Actions substitutes the expression value verbatim into the shell string, allowing an attacker who controls the value to inject arbitrary shell commands.

Specific violations:

**ci.yml – "Create Test Files and Directories" step (rule a + b):**
```
if [ "${{ matrix.dest_config.type }}" = "custom" ]; then
  ls -la ${{ matrix.dest_config.dest }}/   # unquoted expression in shell
```

**ci.yml – "Set Source Path" step (rule a):**
```
echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT
echo "source_path=./${{ matrix.source }}.${{ matrix.format }}" >> $GITHUB_OUTPUT
```

**ci.yml – "Print Compression Output" step (rule a):**
```
echo "Compression output for ${{ matrix.format }} ...: ${{ steps.compress.outputs.file_path }}"
```

**ci.yml – "Verify Expected Failure" step (rule a):**
```
if [ "${{ steps.test_fail.outcome }}" != "failure" ]; then
```

**ci.yml – "ci-result / Check all jobs" step (rule a):**
```
if [ "${{ contains(needs.*.result, 'failure') }}" = "true" ] || ...
```

**use-action.yml – "Set Archive Path" step (rule a):**
```
if [ "${{ matrix.include_root }}" = "true" ]; then
  echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT
```

**use-action.yml – "Verify Archive Created" step (rule a):**
```
if [ ! -f "${{ steps.archive.outputs.path }}" ]; then
  echo "❌ Archive not found: ${{ steps.archive.outputs.path }}"
```

**use-action.yml – "Verify Extraction" step (rule a):**
```
if [ "${{ matrix.include_root }}" = "true" ]; then
```

**release.yml – "Update major version tag" step (rule a):**
```
COMMIT_SHA=$(git rev-parse ${{ github.sha }})
```

Locations:

- `.github/workflows/ci.yml:62`
- `.github/workflows/ci.yml:63`
- `.github/workflows/ci.yml:76`
- `.github/workflows/ci.yml:78`
- `.github/workflows/ci.yml:80`
- `.github/workflows/ci.yml:100`
- `.github/workflows/ci.yml:113`
- `.github/workflows/ci.yml:65`
- `.github/workflows/use-action.yml:47`
- `.github/workflows/use-action.yml:48`
- `.github/workflows/use-action.yml:50`
- `.github/workflows/use-action.yml:55`
- `.github/workflows/use-action.yml:56`
- `.github/workflows/use-action.yml:65`
- `.github/workflows/release.yml:42`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from workflow expressions directly to `$GITHUB_OUTPUT` or `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who can influence these values could inject additional environment variable assignments or output entries.

**ci.yml – "Set Source Path" step:** Writes `${{ matrix.dest_config.dest }}`, `${{ matrix.dest_config.destfilename }}`, `${{ matrix.format }}`, `${{ matrix.source }}`, and `${{ matrix.include_root }}` directly into `$GITHUB_OUTPUT` without sanitization:
```bash
echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT
echo "source_path=./${{ matrix.source }}.${{ matrix.format }}" >> $GITHUB_OUTPUT
```

**use-action.yml – "Set Archive Path" step:** Writes `${{ matrix.format }}` directly into `$GITHUB_OUTPUT` without sanitization:
```bash
echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT
echo "path=./testdir/testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT
```

**release.yml – "Update major version tag" step:** Writes env vars `MAJOR_VERSION` and `COMMIT_SHA` (derived from `${{ github.ref_name }}` via `CURRENT_TAG` env var) to `$GITHUB_ENV` without sanitization:
```bash
echo "MAJOR_VERSION=${MAJOR_VERSION}" >> "$GITHUB_ENV"
echo "COMMIT_SHA=${COMMIT_SHA}" >> "$GITHUB_ENV"
```

Locations:

- `.github/workflows/ci.yml:76`
- `.github/workflows/ci.yml:78`
- `.github/workflows/ci.yml:80`
- `.github/workflows/use-action.yml:48`
- `.github/workflows/use-action.yml:50`
- `.github/workflows/release.yml:46`
- `.github/workflows/release.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four security findings across 8 workflow files:

1. **unpinned-uses**: Pinned all 29 `uses:` references to immutable 40-character SHA hashes across ci.yml, changelog-generator.yml, contributors.yml, gitlab-mirror.yml, issue-greeting.yml, linter.yml, release.yml, stale-issues.yml, and use-action.yml. Original tags preserved as comments.

2. **missing-permissions**: Added `permissions: contents: read` top-level block to gitlab-mirror.yml.

3. **script-injection**: Moved all `${{ }}` expressions out of `run:` shell strings into `env:` blocks in ci.yml (multiple steps including Create Test Files, Set Source Path, Print Compression Output, Verify Expected Failure, Verify Warning Behavior, Verify Warning Case, Verify No Match Behavior, Verify Broken Symlink Error Handling, Check all jobs), use-action.yml (Set Archive Path, Verify Archive Created, Verify Extraction), and release.yml (Update major version tag). Also fixed contributors.yml to use env vars for GITHUB_TOKEN and GITHUB_REPOSITORY.

4. **github-env-injection**: Sanitized all values written to $GITHUB_OUTPUT and $GITHUB_ENV using `printf '%s' ... | tr -d '\n\r'` before writing. Fixed in ci.yml (Set Source Path), use-action.yml (Set Archive Path), and release.yml (Update major version tag - both MAJOR_VERSION and COMMIT_SHA).

