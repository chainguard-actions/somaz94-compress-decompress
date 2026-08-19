<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.10.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files uses a mutable version tag instead of a pinned 40-character SHA commit digest. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Affected references include: actions/checkout@v6, actions/setup-python@v6, actions/upload-artifact@v7, actions/github-script@v9, actions/stale@v10, dependabot/fetch-metadata@v3, orhun/git-cliff-action@v4, softprops/action-gh-release@v3, somaz94/compress-decompress@v1, somaz94/go-changelog-action@v1, somaz94/go-git-commit-action@v1, somaz94/contributors-action@v1, somaz94/major-tag-action@v1, somaz94/multi-git-mirror@v1.

Locations:

- `.github/workflows/ci.yml:21`
- `.github/workflows/ci.yml:25`
- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:83`
- `.github/workflows/changelog-generator.yml:22`
- `.github/workflows/changelog-generator.yml:27`
- `.github/workflows/changelog-generator.yml:32`
- `.github/workflows/contributors.yml:26`
- `.github/workflows/contributors.yml:34`
- `.github/workflows/contributors.yml:42`
- `.github/workflows/dependabot-auto-merge.yml:22`
- `.github/workflows/gitlab-mirror.yml:26`
- `.github/workflows/gitlab-mirror.yml:30`
- `.github/workflows/issue-greeting.yml:16`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:52`
- `.github/workflows/stale-issues.yml:17`
- `.github/workflows/use-action.yml:28`
- `.github/workflows/use-action.yml:42`
- `.github/workflows/use-action.yml:57`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions into shell command strings (sub-rule a), allowing an attacker to inject arbitrary shell commands. Specific violations:

**release.yml** — `CURRENT_TAG="${{ github.ref_name }}"` interpolates a git ref directly into a shell variable assignment (line ~23); `echo "**Full Changelog**: https://github.com/${{ github.repository }}/compare/${{ steps.prev_tag.outputs.tag }}...${{ github.ref_name }}" >> RELEASE.md` interpolates github context and step outputs directly into a shell echo (line ~38).

**ci.yml** — The 'Set Source Path' step interpolates `${{ matrix.dest_config.type }}`, `${{ matrix.dest_config.dest }}`, `${{ matrix.dest_config.destfilename }}`, `${{ matrix.format }}`, `${{ matrix.include_root }}`, `${{ matrix.source }}` directly into shell conditionals and `echo ... >> $GITHUB_OUTPUT` commands. Multiple 'Verify' steps echo `${{ matrix.format }}`, `${{ matrix.include_root }}`, `${{ steps.*.outcome }}`, and `${{ steps.set-path.outputs.source_path }}` directly into shell strings. The 'Check all jobs' step uses `${{ contains(needs.*.result, 'failure') }}` directly in a shell conditional.

**use-action.yml** — The 'Set Archive Path' step uses `${{ matrix.include_root }}` in a shell conditional and writes `${{ matrix.format }}` directly to `$GITHUB_OUTPUT`.

Locations:

- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:38`
- `.github/workflows/ci.yml:66`
- `.github/workflows/ci.yml:96`
- `.github/workflows/ci.yml:107`
- `.github/workflows/ci.yml:113`
- `.github/workflows/ci.yml:1083`
- `.github/workflows/use-action.yml:44`
- `.github/workflows/use-action.yml:46`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from `${{ matrix.* }}` and `${{ steps.*.outputs.* }}` expressions directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). This allows newline injection that can poison subsequent steps' environment.

**ci.yml** — 'Set Source Path' step: `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT` and similar lines write unsanitized matrix values to GITHUB_OUTPUT.

**use-action.yml** — 'Set Archive Path' step: `echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT` and `echo "path=./testdir/testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT` write unsanitized matrix values to GITHUB_OUTPUT.

Locations:

- `.github/workflows/ci.yml:68`
- `.github/workflows/ci.yml:71`
- `.github/workflows/ci.yml:74`
- `.github/workflows/use-action.yml:45`
- `.github/workflows/use-action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three finding types across 8 workflow files. (1) unpinned-uses: Pinned all 14 unique action references to full 40-char SHA digests in ci.yml, release.yml, use-action.yml, changelog-generator.yml, contributors.yml, dependabot-auto-merge.yml, gitlab-mirror.yml, issue-greeting.yml, and stale-issues.yml. (2) script-injection: Moved all ${{ }} expressions out of run: shell strings into step env: blocks in release.yml, ci.yml, and use-action.yml. (3) github-env-injection: Added printf '%s' ... | tr -d newlines sanitization before all writes to $GITHUB_OUTPUT that use matrix/step-output values in ci.yml Set Source Path, use-action.yml Set Archive Path, and release.yml Get previous semver tag.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced hardcoded password literals 'test_password_123' and 'wrong_password' in the test-password-encryption job of .github/workflows/ci.yml. Added a job-level env: block that populates TEST_PASSWORD and WRONG_PASSWORD from GitHub secrets (secrets.TEST_PASSWORD and secrets.WRONG_PASSWORD) with a fallback placeholder value for CI environments where the secrets are not configured. The password: input fields now reference ${{ env.TEST_PASSWORD }} and ${{ env.WRONG_PASSWORD }} instead of hardcoded literal strings.

