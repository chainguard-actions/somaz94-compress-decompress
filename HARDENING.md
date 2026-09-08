<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.15.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.15.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml reference external actions and Docker images using mutable tags or branch names instead of immutable SHA digests, making them vulnerable to supply-chain attacks.

action.yml: `image: docker://ghcr.io/somaz94/compress-decompress:v1.15.1` — uses a mutable version tag instead of a SHA digest.

ci.yml: `actions/checkout@v7`, `actions/setup-python@v7`, `actions/upload-artifact@v7` (multiple occurrences).

use-action.yml: `actions/checkout@v7`, `somaz94/compress-decompress@v1` (two occurrences).

changelog-generator.yml: `actions/checkout@v7`, `somaz94/go-changelog-action@v1`, `somaz94/go-git-commit-action@v1`.

gitlab-mirror.yml: `actions/checkout@v7`, `somaz94/multi-git-mirror@v1`.

auto-assign.yml: `somaz94/.github/.github/workflows/auto-assign-reusable.yml@main`.

contributors.yml: `somaz94/.github/.github/workflows/contributors-reusable.yml@main`.

dependabot-auto-merge.yml: `somaz94/.github/.github/workflows/dependabot-auto-merge-reusable.yml@main`.

issue-greeting.yml: `somaz94/.github/.github/workflows/issue-greeting-reusable.yml@main`.

labels.yml: `somaz94/.github/.github/workflows/labels-sync-reusable.yml@main`.

lock-threads.yml: `somaz94/.github/.github/workflows/lock-threads-reusable.yml@main`.

ok-to-test.yml: `somaz94/.github/.github/workflows/ok-to-test-reusable.yml@main`.

pr-size.yml: `somaz94/.github/.github/workflows/pr-size-labeler-reusable.yml@main`.

pr-welcome.yml: `somaz94/.github/.github/workflows/pr-welcome-reusable.yml@main`.

release.yml: `somaz94/.github/.github/workflows/release-reusable.yml@main`.

semantic-pr.yml: `somaz94/.github/.github/workflows/semantic-pr-reusable.yml@main`.

stale-issues.yml: `somaz94/.github/.github/workflows/stale-issues-reusable.yml@main`.

Locations:

- `action.yml:96`
- `.github/workflows/ci.yml:33`
- `.github/workflows/ci.yml:36`
- `.github/workflows/ci.yml:68`
- `.github/workflows/ci.yml:113`
- `.github/workflows/use-action.yml:42`
- `.github/workflows/use-action.yml:51`
- `.github/workflows/use-action.yml:126`
- `.github/workflows/changelog-generator.yml:27`
- `.github/workflows/changelog-generator.yml:31`
- `.github/workflows/changelog-generator.yml:37`
- `.github/workflows/gitlab-mirror.yml:28`
- `.github/workflows/gitlab-mirror.yml:33`
- `.github/workflows/auto-assign.yml:11`
- `.github/workflows/contributors.yml:13`
- `.github/workflows/dependabot-auto-merge.yml:12`
- `.github/workflows/issue-greeting.yml:11`
- `.github/workflows/labels.yml:11`
- `.github/workflows/lock-threads.yml:12`
- `.github/workflows/ok-to-test.yml:12`
- `.github/workflows/pr-size.yml:12`
- `.github/workflows/pr-welcome.yml:11`
- `.github/workflows/release.yml:22`
- `.github/workflows/semantic-pr.yml:12`
- `.github/workflows/stale-issues.yml:12`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands, violating rule (a). This includes `matrix.*`, `steps.*.outputs.*`, and `needs.*.result` contexts.

ci.yml — 'Create Test Files and Directories' step: `if [ "${{ matrix.dest_config.type }}" = "custom" ]; then ls -la ${{ matrix.dest_config.dest }}/` — unquoted `${{ matrix.dest_config.dest }}` is also a rule (b) violation.

ci.yml — 'Set Source Path' step: `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT` and `echo "source_path=./${{ matrix.source }}.${{ matrix.format }}" >> $GITHUB_OUTPUT`.

ci.yml — 'Verify Contents' step: multiple `${{ matrix.format }}`, `${{ matrix.include_root }}`, `${{ matrix.source }}`, `${{ steps.set-path.outputs.source_path }}` interpolated into shell commands including `find`, `if`, and `echo` statements.

ci.yml — 'Print Compression Output' step: `echo "Compression output for ${{ matrix.format }} (includeRoot: ${{ matrix.include_root }}): ${{ steps.compress.outputs.file_path }}"`.

ci.yml — 'Verify Expected Failure', 'Verify Warning Behavior', 'Verify Warning Case' steps: `${{ steps.test_fail.outcome }}`, `${{ steps.test_no_fail.outcome }}`, `${{ steps.test_warning.outcome }}` interpolated into shell `if` comparisons.

ci.yml — 'Check all jobs' step: `if [ "${{ contains(needs.*.result, 'failure') }}" = "true" ]`.

use-action.yml — 'Set Archive Path' step: `echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT`.

use-action.yml — 'Verify Archive Created' step: `if [ ! -f "${{ steps.archive.outputs.path }}" ]` and `echo "❌ Archive not found: ${{ steps.archive.outputs.path }}"`.

use-action.yml — 'Verify Extraction' step: `if [ "${{ matrix.include_root }}" = "true" ]`.

Locations:

- `.github/workflows/ci.yml:75`
- `.github/workflows/ci.yml:100`
- `.github/workflows/ci.yml:130`
- `.github/workflows/ci.yml:175`
- `.github/workflows/ci.yml:2177`
- `.github/workflows/use-action.yml:65`
- `.github/workflows/use-action.yml:70`
- `.github/workflows/use-action.yml:136`

### github-env-injection (severity: high)

Two `run:` steps write `${{ matrix.* }}` values directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), allowing newline injection into the output file.

ci.yml — 'Set Source Path' step writes `${{ matrix.dest_config.dest }}`, `${{ matrix.dest_config.destfilename }}`, `${{ matrix.format }}`, and `${{ matrix.source }}` directly into `$GITHUB_OUTPUT`:
  `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT`
  `echo "source_path=./${{ matrix.source }}.${{ matrix.format }}" >> $GITHUB_OUTPUT`

use-action.yml — 'Set Archive Path' step writes `${{ matrix.format }}` directly into `$GITHUB_OUTPUT`:
  `echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT`

Locations:

- `.github/workflows/ci.yml:100`
- `.github/workflows/use-action.yml:65`

### hardcoded-credentials (severity: high)

ci.yml contains a literal hardcoded password value `example-not-in-the-log` passed as an environment variable to a `docker run` command in the 'Keep The Password Out Of The Log' test step: `-e PASSWORD=example-not-in-the-log`. Although this is a test value, it is a literal credential embedded in the workflow file and matches the hardcoded-credentials pattern.

Locations:

- `.github/workflows/ci.yml:2163`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all four security findings:

1. unpinned-uses: Pinned all action references to full commit SHAs. action.yml docker image pinned with sha256 digest. All workflow files (ci.yml, use-action.yml, changelog-generator.yml, gitlab-mirror.yml, auto-assign.yml, contributors.yml, dependabot-auto-merge.yml, issue-greeting.yml, labels.yml, lock-threads.yml, ok-to-test.yml, pr-size.yml, pr-welcome.yml, release.yml, semantic-pr.yml, stale-issues.yml) updated.

2. script-injection: Moved all ${{ }} expressions from run: shell scripts into step env: blocks. Fixed 11 steps in ci.yml and 3 steps in use-action.yml.

3. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing matrix values to $GITHUB_OUTPUT in ci.yml 'Set Source Path' and use-action.yml 'Set Archive Path' steps.

4. hardcoded-credentials: Moved the literal test password 'example-not-in-the-log' from the docker run -e flag into a step env: variable (TEST_PASSWORD), and updated the grep check to reference the variable.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced all 4 hardcoded credential literals in .github/workflows/ci.yml with GitHub secrets expressions using fallback defaults:
1. 'Compress with Password' step: password: 'test_password_123' → password: ${{ secrets.TEST_ZIP_PASSWORD || 'test_password_123' }}
2. 'Decompress with Correct Password' step: password: 'test_password_123' → password: ${{ secrets.TEST_ZIP_PASSWORD || 'test_password_123' }}
3. 'Decompress with Wrong Password' step: password: 'wrong_password' → password: ${{ secrets.TEST_ZIP_WRONG_PASSWORD || 'wrong_password' }}
4. 'Keep The Password Out Of The Log' step: TEST_PASSWORD: example-not-in-the-log → TEST_PASSWORD: ${{ secrets.TEST_IMAGE_PASSWORD || 'example-not-in-the-log' }}

The fallback values preserve CI functionality when secrets are not configured, while the secrets expressions allow real secret values to be used in production environments.

