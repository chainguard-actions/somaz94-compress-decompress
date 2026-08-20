<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.12.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.12.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The docker image reference in action.yml uses a mutable version tag (`v1.12.0`) instead of a SHA256 digest, making it vulnerable to supply-chain attacks if the tag is moved. Additionally, all `uses:` references across every workflow file use version tags or branch names (e.g., `@v7`, `@v1`, `@main`) instead of pinned 40-character commit SHAs. Affected references include: `actions/checkout@v7`, `actions/setup-python@v7`, `actions/upload-artifact@v7`, `somaz94/compress-decompress@v1`, `somaz94/go-changelog-action@v1`, `somaz94/go-git-commit-action@v1`, `somaz94/multi-git-mirror@v1`, `somaz94/.github/.github/workflows/*@main`, and others.

Locations:

- `action.yml:96`
- `.github/workflows/ci.yml:23`
- `.github/workflows/ci.yml:26`
- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:104`
- `.github/workflows/use-action.yml:32`
- `.github/workflows/use-action.yml:40`
- `.github/workflows/use-action.yml:73`
- `.github/workflows/changelog-generator.yml:20`
- `.github/workflows/changelog-generator.yml:24`
- `.github/workflows/changelog-generator.yml:28`
- `.github/workflows/contributors.yml:12`
- `.github/workflows/dependabot-auto-merge.yml:11`
- `.github/workflows/gitlab-mirror.yml:28`
- `.github/workflows/gitlab-mirror.yml:32`
- `.github/workflows/issue-greeting.yml:10`
- `.github/workflows/labels.yml:10`
- `.github/workflows/lock-threads.yml:11`
- `.github/workflows/ok-to-test.yml:10`
- `.github/workflows/pr-size.yml:10`
- `.github/workflows/pr-welcome.yml:10`
- `.github/workflows/release.yml:22`
- `.github/workflows/semantic-pr.yml:11`
- `.github/workflows/stale-issues.yml:11`
- `.github/workflows/auto-assign.yml:10`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions into shell commands, violating rule (a). This allows an attacker to inject arbitrary shell commands if any of these values are attacker-controlled. Specific violations:

**ci.yml — 'Create Test Files and Directories' step (sub-rule a+b):** `ls -la ${{ matrix.dest_config.dest }}/` — the expression is interpolated directly AND the result is unquoted.

**ci.yml — 'Set Source Path' step (sub-rule a):** `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT` and similar lines.

**ci.yml — 'Verify Contents' step (sub-rule a):** Multiple `${{ matrix.format }}`, `${{ matrix.include_root }}`, `${{ matrix.source }}`, `${{ steps.set-path.outputs.source_path }}` interpolated directly into shell strings.

**ci.yml — 'Verify Expected Failure', 'Verify Warning Behavior', 'Verify Warning Case' steps (sub-rule a):** `if [ "${{ steps.test_fail.outcome }}" != "failure" ]` and similar patterns.

**ci.yml — 'Verify No Match Behavior' step (sub-rule a):** `if [ "${{ steps.glob_no_match.outcome }}" = "failure" ]`.

**ci.yml — 'Verify Broken Symlink Error Handling' step (sub-rule a):** `${{ steps.test_broken_symlink.outcome }}` interpolated into shell.

**ci.yml — 'Verify Wrong Password Handling' step (sub-rule a):** `if [ "${{ steps.wrong_password.outcome }}" = "failure" ]`.

**ci.yml — 'Check all jobs' step (sub-rule a):** `if [ "${{ contains(needs.*.result, 'failure') }}" = "true" ]`.

**use-action.yml — 'Set Archive Path' step (sub-rule a):** `echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT`.

**use-action.yml — 'Verify Archive Created' step (sub-rule a):** `if [ ! -f "${{ steps.archive.outputs.path }}" ]`.

**use-action.yml — 'Verify Outputs' step (sub-rule a):** Multiple `${{ steps.compress.outputs.* }}` interpolated into shell strings.

**use-action.yml — 'Verify Extraction' step (sub-rule a):** `if [ "${{ matrix.include_root }}" = "true" ]`.

Locations:

- `.github/workflows/ci.yml:62`
- `.github/workflows/ci.yml:63`
- `.github/workflows/ci.yml:80`
- `.github/workflows/ci.yml:130`
- `.github/workflows/ci.yml:175`
- `.github/workflows/ci.yml:200`
- `.github/workflows/ci.yml:230`
- `.github/workflows/ci.yml:1500`
- `.github/workflows/ci.yml:2370`
- `.github/workflows/ci.yml:2510`
- `.github/workflows/ci.yml:2740`
- `.github/workflows/use-action.yml:46`
- `.github/workflows/use-action.yml:54`
- `.github/workflows/use-action.yml:62`
- `.github/workflows/use-action.yml:87`

### github-env-injection (severity: high)

Two `run:` steps write values derived from `${{ }}` expressions directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who can influence `matrix.*` or `steps.*.outputs.*` values could inject newlines to poison the output file and set arbitrary environment variables or outputs for subsequent steps.

**ci.yml — 'Set Source Path' step:** Writes `${{ matrix.dest_config.dest }}`, `${{ matrix.dest_config.destfilename }}`, `${{ matrix.format }}`, `${{ matrix.source }}`, and `${{ matrix.include_root }}` directly to `$GITHUB_OUTPUT` via `echo "source_path=..." >> $GITHUB_OUTPUT`.

**use-action.yml — 'Set Archive Path' step:** Writes `${{ matrix.format }}` directly to `$GITHUB_OUTPUT` via `echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT`.

Locations:

- `.github/workflows/ci.yml:80`
- `.github/workflows/use-action.yml:46`

### hardcoded-credentials (severity: high)

The ci.yml workflow contains two literal plaintext password values assigned to password-named fields, matching the hardcoded-credentials pattern:
1. `password: 'test_password_123'` in the 'Compress with Password' step of the `test-password-encryption` job (a literal password value passed to the action's `password:` input).
2. `PASSWORD=example-not-in-the-log` in the 'Keep The Password Out Of The Log' step of the `test-image-from-source` job (a literal value assigned to the PASSWORD environment variable in a docker run command).
Neither value uses a `${{ secrets.* }}` expression.

Locations:

- `.github/workflows/ci.yml:2430`
- `.github/workflows/ci.yml:2710`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned all `uses:` references to full 40-char commit SHAs across all 16 workflow files. Pinned the docker image in action.yml with its SHA256 digest (`docker://ghcr.io/somaz94/compress-decompress:v1.12.0@sha256:5047affa...`). All reusable workflow `@main` references pinned to `4395663967fb07d44defafb52018c667ae383897`.

2. **script-injection**: Moved all `${{ }}` expressions out of `run:` shell strings into `env:` blocks in ci.yml (11 steps fixed) and use-action.yml (4 steps fixed). All shell commands now reference plain environment variables.

3. **github-env-injection**: The 'Set Source Path' step in ci.yml and 'Set Archive Path' step in use-action.yml now sanitize values with `printf '%s' ... | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.

4. **hardcoded-credentials**: Replaced `password: 'test_password_123'` (compress and decompress steps) and `PASSWORD=example-not-in-the-log` (docker run env var) with `${{ secrets.TEST_ZIP_PASSWORD }}` references. The 'Keep The Password Out Of The Log' test was updated to use the secret value in both the docker run command and the grep check.

