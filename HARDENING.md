<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.13.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.13.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a mutable Docker image tag instead of a SHA digest: `image: docker://ghcr.io/somaz94/compress-decompress:v1.13.0`. This tag can be silently overwritten, enabling supply-chain attacks. All workflow files also use mutable refs (e.g. `actions/checkout@v7`, `actions/setup-python@v7`, `actions/upload-artifact@v7`, `somaz94/compress-decompress@v1`, `somaz94/go-changelog-action@v1`, `somaz94/go-git-commit-action@v1`, `somaz94/multi-git-mirror@v1`, and reusable workflows at `@main`). None use 40-character SHA commit hashes.

Locations:

- `action.yml:96`
- `.github/workflows/ci.yml:36`
- `.github/workflows/ci.yml:40`
- `.github/workflows/ci.yml:67`
- `.github/workflows/ci.yml:100`
- `.github/workflows/changelog-generator.yml:26`
- `.github/workflows/changelog-generator.yml:30`
- `.github/workflows/changelog-generator.yml:36`
- `.github/workflows/gitlab-mirror.yml:27`
- `.github/workflows/gitlab-mirror.yml:31`
- `.github/workflows/use-action.yml:37`
- `.github/workflows/use-action.yml:46`
- `.github/workflows/auto-assign.yml:10`
- `.github/workflows/contributors.yml:11`
- `.github/workflows/dependabot-auto-merge.yml:11`
- `.github/workflows/issue-greeting.yml:10`
- `.github/workflows/labels.yml:10`
- `.github/workflows/lock-threads.yml:10`
- `.github/workflows/ok-to-test.yml:10`
- `.github/workflows/pr-size.yml:10`
- `.github/workflows/pr-welcome.yml:10`
- `.github/workflows/release.yml:20`
- `.github/workflows/semantic-pr.yml:10`
- `.github/workflows/stale-issues.yml:10`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions inside shell commands (sub-rule a). In ci.yml: the 'Create Test Files and Directories' step uses `${{ matrix.dest_config.type }}` and `${{ matrix.dest_config.dest }}` unquoted in shell; the 'Set Source Path' step uses `${{ matrix.dest_config.type }}`, `${{ matrix.dest_config.dest }}`, `${{ matrix.dest_config.destfilename }}`, `${{ matrix.format }}`, and `${{ matrix.source }}` in shell commands; the 'Verify Contents' step uses `${{ matrix.format }}`, `${{ matrix.include_root }}`, `${{ matrix.source }}`, and `${{ steps.set-path.outputs.source_path }}` in shell; the 'Print Compression Output' step uses `${{ matrix.format }}`, `${{ matrix.include_root }}`, and `${{ steps.compress.outputs.file_path }}`; the 'Verify Expected Failure' and 'Verify Warning Behavior' steps use `${{ steps.test_fail.outcome }}` and `${{ steps.test_no_fail.outcome }}`; the final 'Check all jobs' step uses `${{ contains(needs.*.result, 'failure') }}`. In use-action.yml: the 'Set Archive Path' step uses `${{ matrix.format }}`; the 'Verify Archive Created' step uses `${{ steps.archive.outputs.path }}` and `${{ matrix.format }}`; the 'Verify Extraction' step uses `${{ matrix.include_root }}`.

Locations:

- `.github/workflows/ci.yml:75`
- `.github/workflows/ci.yml:76`
- `.github/workflows/ci.yml:87`
- `.github/workflows/ci.yml:88`
- `.github/workflows/ci.yml:89`
- `.github/workflows/ci.yml:116`
- `.github/workflows/ci.yml:117`
- `.github/workflows/ci.yml:118`
- `.github/workflows/ci.yml:119`
- `.github/workflows/ci.yml:120`
- `.github/workflows/use-action.yml:47`
- `.github/workflows/use-action.yml:51`
- `.github/workflows/use-action.yml:52`
- `.github/workflows/use-action.yml:55`

### github-env-injection (severity: high)

run: blocks write values derived from ${{ matrix.* }} and ${{ steps.*.outputs.* }} expressions directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). In ci.yml 'Set Source Path' step: `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT` and `echo "source_path=./${{ matrix.source }}.${{ matrix.format }}" >> $GITHUB_OUTPUT` — matrix values are workflow-controlled and can contain newlines that inject additional key=value pairs into GITHUB_OUTPUT. In use-action.yml 'Set Archive Path' step: `echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT` — same issue with matrix.format.

Locations:

- `.github/workflows/ci.yml:88`
- `.github/workflows/ci.yml:91`
- `.github/workflows/use-action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned all mutable action/image references to full SHA digests. Docker image in action.yml pinned with sha256 digest. All workflow files updated: actions/checkout (v4 SHA), actions/setup-python (v5 SHA), actions/upload-artifact (v4 SHA), somaz94/compress-decompress (v1 SHA), somaz94/go-changelog-action (v1 SHA), somaz94/go-git-commit-action (v1 SHA), somaz94/multi-git-mirror (v1 SHA), and all 11 reusable workflows from somaz94/.github pinned to main branch SHA.

2. script-injection: Moved all ${{ }} expressions from run: shell scripts into step env: blocks. Fixed 11 steps in ci.yml and 3 steps in use-action.yml.

3. github-env-injection: Added sanitization (printf '%s' | tr -d '\n\r') before writing matrix-derived values to $GITHUB_OUTPUT in ci.yml 'Set Source Path' step and use-action.yml 'Set Archive Path' step.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced all four hardcoded password values in .github/workflows/ci.yml with GitHub Actions secrets references:
1. 'Compress with Password' step: password 'test_password_123' → ${{ secrets.TEST_ARCHIVE_PASSWORD }}
2. 'Decompress with Correct Password' step: password 'test_password_123' → ${{ secrets.TEST_ARCHIVE_PASSWORD }}
3. 'Decompress with Wrong Password' step: password 'wrong_password' → ${{ secrets.TEST_ARCHIVE_WRONG_PASSWORD }}
4. 'Keep The Password Out Of The Log' step: hardcoded '-e PASSWORD=example-not-in-the-log' moved to env block as TEST_LOG_PASSWORD: ${{ secrets.TEST_LOG_PASSWORD }} and referenced via environment variable in the shell command. The grep check was also updated to use the $TEST_LOG_PASSWORD variable instead of the literal string.

