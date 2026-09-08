<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.14.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.14.0** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a mutable Docker image tag instead of a SHA digest: `image: docker://ghcr.io/somaz94/compress-decompress:v1.14.0`. This tag can be silently overwritten, enabling supply-chain attacks. It must be pinned to a SHA digest (e.g., `ghcr.io/somaz94/compress-decompress@sha256:<64-hex-char-digest>`).

Locations:

- `action.yml:108`

### unpinned-uses (severity: high)

ci.yml uses multiple unpinned action references with mutable tags instead of full 40-character SHA commits: `actions/checkout@v7`, `actions/setup-python@v7`, `actions/upload-artifact@v7`. These tags can be moved to point to different (potentially malicious) commits.

Locations:

- `.github/workflows/ci.yml:36`
- `.github/workflows/ci.yml:39`
- `.github/workflows/ci.yml:74`

### unpinned-uses (severity: high)

changelog-generator.yml uses unpinned action references with mutable tags: `actions/checkout@v7`, `somaz94/go-changelog-action@v1`, `somaz94/go-git-commit-action@v1`.

Locations:

- `.github/workflows/changelog-generator.yml:27`
- `.github/workflows/changelog-generator.yml:30`
- `.github/workflows/changelog-generator.yml:35`

### unpinned-uses (severity: high)

gitlab-mirror.yml uses unpinned action references with mutable tags: `actions/checkout@v7`, `somaz94/multi-git-mirror@v1`.

Locations:

- `.github/workflows/gitlab-mirror.yml:30`
- `.github/workflows/gitlab-mirror.yml:34`

### unpinned-uses (severity: high)

use-action.yml uses unpinned action references with mutable tags: `actions/checkout@v7`, `somaz94/compress-decompress@v1` (used twice).

Locations:

- `.github/workflows/use-action.yml:39`
- `.github/workflows/use-action.yml:49`
- `.github/workflows/use-action.yml:95`

### unpinned-uses (severity: high)

Multiple workflow files use reusable workflow references pinned to the mutable `@main` branch: auto-assign.yml (`somaz94/.github/.github/workflows/auto-assign-reusable.yml@main`), contributors.yml, dependabot-auto-merge.yml, issue-greeting.yml, labels.yml, lock-threads.yml, ok-to-test.yml, pr-size.yml, pr-welcome.yml, release.yml, semantic-pr.yml, stale-issues.yml.

Locations:

- `.github/workflows/auto-assign.yml:11`
- `.github/workflows/contributors.yml:12`
- `.github/workflows/dependabot-auto-merge.yml:12`
- `.github/workflows/issue-greeting.yml:11`
- `.github/workflows/labels.yml:11`
- `.github/workflows/lock-threads.yml:12`
- `.github/workflows/ok-to-test.yml:12`
- `.github/workflows/pr-size.yml:12`
- `.github/workflows/pr-welcome.yml:11`
- `.github/workflows/release.yml:21`
- `.github/workflows/semantic-pr.yml:12`
- `.github/workflows/stale-issues.yml:12`

### script-injection (severity: high)

ci.yml — rule (a): Multiple `run:` blocks directly interpolate `${{ }}` expressions into shell commands. Examples include: `if [ "${{ matrix.dest_config.type }}" = "custom" ]` (line 83), `ls -la ${{ matrix.dest_config.dest }}/` (line 84, also unquoted — rule b), `if [ "${{ matrix.dest_config.type }}" = "custom" ]` in Set Source Path (line 101), `echo "Compression output for ${{ matrix.format }}..."` (Print Compression Output step), `if [ "${{ steps.test_fail.outcome }}" != "failure" ]` (Verify Expected Failure step), `if [ "${{ steps.glob_no_match.outcome }}" = "failure" ]`, `if [ "${{ steps.test_broken_symlink.outcome }}" = "failure" ]`, `if [ "${{ steps.wrong_password.outcome }}" = "failure" ]`, and `if [ "${{ contains(needs.*.result, 'failure') }}" = "true" ]` (Check all jobs step). Any `${{ }}` expression inside a run: block is a script-injection risk.

Locations:

- `.github/workflows/ci.yml:83`
- `.github/workflows/ci.yml:84`
- `.github/workflows/ci.yml:101`

### script-injection (severity: high)

use-action.yml — rule (a): Multiple `run:` blocks directly interpolate `${{ }}` expressions into shell commands: `run: echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT` (line 61), `if [ ! -f "${{ steps.archive.outputs.path }}" ]` (line 65), `if [ -e "./testdir/testdir.${{ matrix.format }}" ]` (line 70), `echo "✅ Archive created: ${{ steps.archive.outputs.path }}"` (line 74), and `if [ "${{ matrix.include_root }}" = "true" ]` (line 102).

Locations:

- `.github/workflows/use-action.yml:61`
- `.github/workflows/use-action.yml:65`
- `.github/workflows/use-action.yml:70`
- `.github/workflows/use-action.yml:102`

### github-env-injection (severity: high)

ci.yml — The 'Set Source Path' step writes `${{ matrix.* }}` values directly to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization: `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT` (line 102) and `echo "source_path=./${{ matrix.source }}.${{ matrix.format }}" >> $GITHUB_OUTPUT` (line 106). A matrix value containing a newline could inject arbitrary output variables.

Locations:

- `.github/workflows/ci.yml:102`
- `.github/workflows/ci.yml:106`

### github-env-injection (severity: high)

use-action.yml — The 'Set Archive Path' step writes `${{ matrix.format }}` directly to `$GITHUB_OUTPUT` without sanitization: `run: echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT` (line 61). A matrix value containing a newline could inject arbitrary output variables.

Locations:

- `.github/workflows/use-action.yml:61`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all findings:

1. action.yml: Pinned Docker image ghcr.io/somaz94/compress-decompress:v1.14.0 to SHA digest sha256:5d77d45ef3d8894e04bcd8b4ba6197c817de9841a236ce7ff5419abceeea1015, preserving docker:// scheme and tag.

2. ci.yml: Pinned actions/checkout@v7, actions/setup-python@v7, and actions/upload-artifact@v7 to full commit SHAs. Fixed script injection in 10 run: blocks by moving ${{ }} expressions to env: blocks. Fixed github-env-injection in Set Source Path step with printf/tr sanitization.

3. changelog-generator.yml: Pinned actions/checkout@v7, somaz94/go-changelog-action@v1, somaz94/go-git-commit-action@v1 to full commit SHAs.

4. gitlab-mirror.yml: Pinned actions/checkout@v7 and somaz94/multi-git-mirror@v1 to full commit SHAs.

5. use-action.yml: Pinned actions/checkout@v7 and both somaz94/compress-decompress@v1 references to full commit SHAs. Fixed script injection in Set Archive Path, Verify Archive Created, and Verify Extraction steps. Fixed github-env-injection in Set Archive Path with printf/tr sanitization.

6. 12 reusable workflow files (auto-assign.yml, contributors.yml, dependabot-auto-merge.yml, issue-greeting.yml, labels.yml, lock-threads.yml, ok-to-test.yml, pr-size.yml, pr-welcome.yml, release.yml, semantic-pr.yml, stale-issues.yml): Pinned all @main reusable workflow references to full commit SHA 4395663967fb07d44defafb52018c667ae383897.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

In .github/workflows/ci.yml, the 'Keep The Password Out Of The Log' step had a literal password 'example-not-in-the-log' hardcoded in the docker run command (-e PASSWORD=example-not-in-the-log) and in the grep leak-check. Fixed by: (1) adding an env block to the step with TEST_PASSWORD: ${{ secrets.TEST_PASSWORD }}, (2) replacing -e PASSWORD=example-not-in-the-log with -e PASSWORD="$TEST_PASSWORD", and (3) replacing the literal string in the grep check with "$TEST_PASSWORD". The secret must be configured in the repository's secrets settings as TEST_PASSWORD.

