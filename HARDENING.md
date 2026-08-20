<!-- markdownlint-disable -->

# Hardening Report: somaz94--compress-decompress/v1.12.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **somaz94--compress-decompress/v1.12.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml `runs.image:` field references a mutable Docker image tag (`docker://ghcr.io/somaz94/compress-decompress:v1.12.1`) instead of a SHA digest. This means the image can be silently replaced by a malicious version. A SHA digest (e.g. `ghcr.io/somaz94/compress-decompress@sha256:<64-hex-chars>`) is required.

Locations:

- `action.yml:96`

### unpinned-uses (severity: high)

All `uses:` references across workflow files use mutable tags or branch names instead of pinned 40-character SHA commits. Affected references include: `actions/checkout@v7`, `actions/setup-python@v7`, `actions/upload-artifact@v7`, `somaz94/go-changelog-action@v1`, `somaz94/go-git-commit-action@v1`, `somaz94/multi-git-mirror@v1`, `somaz94/compress-decompress@v1`, `somaz94/.github/.github/workflows/*@main`, and others. Any of these can be silently replaced with malicious code.

Locations:

- `.github/workflows/auto-assign.yml:9`
- `.github/workflows/changelog-generator.yml:23`
- `.github/workflows/ci.yml:20`
- `.github/workflows/contributors.yml:12`
- `.github/workflows/dependabot-auto-merge.yml:11`
- `.github/workflows/gitlab-mirror.yml:28`
- `.github/workflows/issue-greeting.yml:9`
- `.github/workflows/labels.yml:10`
- `.github/workflows/lock-threads.yml:10`
- `.github/workflows/ok-to-test.yml:10`
- `.github/workflows/pr-size.yml:10`
- `.github/workflows/pr-welcome.yml:9`
- `.github/workflows/release.yml:22`
- `.github/workflows/semantic-pr.yml:10`
- `.github/workflows/stale-issues.yml:10`
- `.github/workflows/use-action.yml:31`

### script-injection (severity: high)

Multiple `run:` blocks in ci.yml directly interpolate `${{ matrix.* }}` and `${{ steps.*.outputs.* }}` expressions into shell commands (sub-rule a). Examples include: `ls -la ${{ matrix.dest_config.dest }}/` (unquoted, line ~47), `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT` (line ~75), `echo -e "...${YELLOW}${{ matrix.format }}${NC}"` (Verify Contents step), `echo "Compression output for ${{ matrix.format }} (includeRoot: ${{ matrix.include_root }}): ${{ steps.compress.outputs.file_path }}"` (Print Compression Output step), and `if [ "${{ steps.test_fail.outcome }}" != "failure" ]` (Verify Expected Failure step, and similar in multiple other steps). These allow an attacker who controls matrix values or step outputs to inject arbitrary shell commands.

Locations:

- `.github/workflows/ci.yml:47`
- `.github/workflows/ci.yml:75`
- `.github/workflows/ci.yml:100`
- `.github/workflows/ci.yml:186`
- `.github/workflows/ci.yml:232`
- `.github/workflows/ci.yml:258`
- `.github/workflows/ci.yml:284`
- `.github/workflows/ci.yml:310`

### script-injection (severity: high)

Multiple `run:` blocks in use-action.yml directly interpolate `${{ matrix.* }}` and `${{ steps.*.outputs.* }}` expressions into shell commands (sub-rule a). Examples: `if [ "${{ matrix.include_root }}" = "true" ]` in the Set Archive Path step writes to $GITHUB_OUTPUT; `if [ ! -f "${{ steps.archive.outputs.path }}" ]` in Verify Archive Created; `if [ "${{ matrix.include_root }}" = "true" ]` in Verify Extraction. These allow injection of arbitrary shell commands via matrix or step output values.

Locations:

- `.github/workflows/use-action.yml:47`
- `.github/workflows/use-action.yml:55`
- `.github/workflows/use-action.yml:68`
- `.github/workflows/use-action.yml:100`

### github-env-injection (severity: high)

The 'Set Source Path' step in ci.yml writes `${{ matrix.* }}` values directly to `$GITHUB_OUTPUT` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Specifically: `echo "source_path=${{ matrix.dest_config.dest }}/${{ matrix.dest_config.destfilename }}.${{ matrix.format }}" >> $GITHUB_OUTPUT`. A matrix value containing a newline could inject additional key=value pairs into GITHUB_OUTPUT, poisoning downstream steps.

Locations:

- `.github/workflows/ci.yml:75`

### github-env-injection (severity: high)

The 'Set Archive Path' step in use-action.yml writes `${{ matrix.format }}` directly to `$GITHUB_OUTPUT` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Specifically: `echo "path=./testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT` and `echo "path=./testdir/testdir.${{ matrix.format }}" >> $GITHUB_OUTPUT`. A matrix value containing a newline could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/use-action.yml:49`
- `.github/workflows/use-action.yml:51`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all findings across action.yml and 16 workflow files:

1. action.yml: Pinned Docker image docker://ghcr.io/somaz94/compress-decompress:v1.12.1 to SHA256 digest @sha256:46aee9303ab6146919df697ff58dbe04dede55fd2408e3fc63ca0bcd679298e1

2. All workflow files: Pinned all mutable action references to full commit SHAs - actions/checkout@v7, actions/setup-python@v7, actions/upload-artifact@v7, somaz94/go-changelog-action@v1, somaz94/go-git-commit-action@v1, somaz94/multi-git-mirror@v1, somaz94/compress-decompress@v1, and all somaz94/.github reusable workflows @main.

3. ci.yml script injection: Moved all ${{ matrix.* }} and ${{ steps.*.outputs.* }} expressions from run: shell scripts into step env: blocks, referencing them as plain environment variables.

4. use-action.yml script injection: Moved all ${{ matrix.* }} and ${{ steps.*.outputs.* }} expressions from run: shell scripts into step env: blocks.

5. ci.yml github-env-injection: Set Source Path step now sanitizes values with printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

6. use-action.yml github-env-injection: Set Archive Path step now sanitizes matrix.format with printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

