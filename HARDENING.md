<!-- markdownlint-disable -->

# Hardening Report: mamba-org--setup-micromamba/v2.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **mamba-org--setup-micromamba/v2.0.7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tag refs (e.g. @v4, @v5, @v1) instead of immutable 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the tag is moved. Affected references include: build.yml — actions/checkout@v5, pnpm/action-setup@v4, actions/setup-node@v4; check-dist.yml — actions/checkout@v5, pnpm/action-setup@v4, actions/setup-node@v4, actions/upload-artifact@v4; label.yml — mheap/github-action-required-labels@v5; release.yml — actions/checkout@v5, Quantco/ui-actions/version-metadata@v1; test-cache.yml — actions/checkout@v5; test-download.yml — actions/checkout@v5; test.yml — actions/checkout@v5.

Locations:

- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:16`
- `.github/workflows/build.yml:21`
- `.github/workflows/check-dist.yml:13`
- `.github/workflows/check-dist.yml:16`
- `.github/workflows/check-dist.yml:21`
- `.github/workflows/check-dist.yml:37`
- `.github/workflows/label.yml:10`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:12`
- `.github/workflows/test-cache.yml:18`
- `.github/workflows/test-download.yml:13`
- `.github/workflows/test.yml:13`

### missing-permissions (severity: medium)

The following workflow files have no top-level 'permissions:' key and no job-level 'permissions:' keys on any job. Without explicit permissions, workflows inherit the default repository token permissions (which may be write-all depending on repository settings), violating the principle of least privilege: build.yml, check-dist.yml, label.yml, test-cache.yml, test-download.yml, test-post-cleanup.yml, test.yml.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/label.yml:1`
- `.github/workflows/test-cache.yml:1`
- `.github/workflows/test-download.yml:1`
- `.github/workflows/test-post-cleanup.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions (${{ ... }}) are interpolated directly inside run: shell command strings. In test.yml, the 'output-environment-path-env-file' job uses `test ${{ steps.setup-micromamba.outputs.environment-path }} = ...` and `ls ${{ steps.setup-micromamba.outputs.environment-path }}` unquoted in a run: block. The 'output-environment-path-env-name-overwrite' and 'output-environment-path-custom-root-prefix' jobs similarly interpolate `${{ steps.setup-micromamba.outputs.environment-path }}` in run: blocks. The 'output-no-environment-path' job uses `test "${{ steps.setup-micromamba.outputs.environment-path }}" = ""`. The 'check-micromamba-on-path' job uses `echo "${{ runner.temp }}"` and `which micromamba-shell | grep "${{ runner.temp }}/setup-micromamba/micromamba-shell"` in a run: block. In test-post-cleanup.yml, the lisanna-dettwyler/action-post-run step's run: block interpolates ${{ matrix.mamba-init-block-exists }}, ${{ matrix.mamba-activate-exists }}, ${{ matrix.env-exists }}, ${{ matrix.root-exists }}, ${{ matrix.binary-exists }}, and ${{ runner.temp }} directly into shell commands. A later run: block also uses `test -f ${{ runner.temp }}/setup-micromamba/.condarc`. All of these allow expression values to be interpreted by the shell before quoting can occur.

Locations:

- `.github/workflows/test.yml:196`
- `.github/workflows/test.yml:197`
- `.github/workflows/test.yml:205`
- `.github/workflows/test.yml:206`
- `.github/workflows/test.yml:215`
- `.github/workflows/test.yml:216`
- `.github/workflows/test.yml:222`
- `.github/workflows/test.yml:234`
- `.github/workflows/test.yml:237`
- `.github/workflows/test-post-cleanup.yml:44`
- `.github/workflows/test-post-cleanup.yml:45`
- `.github/workflows/test-post-cleanup.yml:46`
- `.github/workflows/test-post-cleanup.yml:47`
- `.github/workflows/test-post-cleanup.yml:48`
- `.github/workflows/test-post-cleanup.yml:49`
- `.github/workflows/test-post-cleanup.yml:50`
- `.github/workflows/test-post-cleanup.yml:66`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 8 workflow files:

1. unpinned-uses: Pinned all mutable tag references to full SHA commits in build.yml, check-dist.yml, label.yml, release.yml, test-cache.yml, test-download.yml, test-post-cleanup.yml, and test.yml. Actions pinned: actions/checkout@v5→fbc6f39, pnpm/action-setup@v4→b906aff, actions/setup-node@v4→49933ea, actions/upload-artifact@v4→ea165f8, mheap/github-action-required-labels@v5→23e10fd, Quantco/ui-actions/version-metadata@v1→5bfb8ce.

2. missing-permissions: Added top-level 'permissions: contents: read' to build.yml, check-dist.yml, test-cache.yml, test-download.yml, test-post-cleanup.yml, and test.yml. Added 'permissions: pull-requests: read' to label.yml (appropriate for checking PR labels). release.yml already had 'permissions: contents: write'.

3. script-injection: In test.yml, moved ${{ steps.setup-micromamba.outputs.environment-path }} and ${{ runner.temp }} expressions from run: shell strings into env: blocks for the output-environment-path-* jobs, output-no-environment-path, and check-micromamba-on-path jobs. In test-post-cleanup.yml, moved ${{ matrix.mamba-init-block-exists }}, ${{ matrix.mamba-activate-exists }}, ${{ matrix.env-exists }}, ${{ matrix.root-exists }}, ${{ matrix.binary-exists }}, and ${{ runner.temp }} into env: blocks and referenced them as plain environment variables in the shell scripts.

