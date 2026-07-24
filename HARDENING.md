<!-- markdownlint-disable -->

# Hardening Report: mamba-org--setup-micromamba/v3.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **mamba-org--setup-micromamba/v3.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any job. Without explicit permissions, workflows inherit the default repository token permissions, which may be overly broad. Each file should declare minimal required permissions.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/label.yml:1`
- `.github/workflows/test-cache.yml:1`
- `.github/workflows/test-download.yml:1`
- `.github/workflows/test-post-cleanup.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Sub-rule (a): `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings, allowing template substitution before the shell parses the value. In test.yml: `test ${{ steps.setup-micromamba.outputs.environment-path }} = ...` and `ls ${{ steps.setup-micromamba.outputs.environment-path }}` (output-environment-path-env-file job), `test "${{ steps.setup-micromamba.outputs.environment-path }}" = ...` (multiple jobs), `echo "${{ runner.temp }}"` and `which micromamba-shell | grep "${{ runner.temp }}/..."` (check-micromamba-on-path job). In test-post-cleanup.yml: `${{ matrix.mamba-init-block-exists }}grep ...`, `${{ matrix.mamba-activate-exists }}grep ...`, `${{ matrix.env-exists }}test ...`, `${{ matrix.root-exists }}test ...`, `${{ matrix.binary-exists }}test ...` are prepended directly to shell commands (the matrix values control whether `! ` is prepended, but the interpolation still occurs before shell parsing), and `test -f ${{ runner.temp }}/setup-micromamba/.condarc`. These should be moved to `env:` blocks and referenced as shell variables.

Locations:

- `.github/workflows/test.yml:196`
- `.github/workflows/test.yml:197`
- `.github/workflows/test.yml:204`
- `.github/workflows/test.yml:205`
- `.github/workflows/test.yml:213`
- `.github/workflows/test.yml:222`
- `.github/workflows/test.yml:250`
- `.github/workflows/test.yml:252`
- `.github/workflows/test-post-cleanup.yml:43`
- `.github/workflows/test-post-cleanup.yml:44`
- `.github/workflows/test-post-cleanup.yml:45`
- `.github/workflows/test-post-cleanup.yml:46`
- `.github/workflows/test-post-cleanup.yml:47`
- `.github/workflows/test-post-cleanup.yml:48`
- `.github/workflows/test-post-cleanup.yml:49`
- `.github/workflows/test-post-cleanup.yml:65`

## Iteration Notes

### Iteration 1

**Fixes applied:** missing-permissions, script-injection

**Notes:**

Fixed missing-permissions by adding `permissions: {}` to build.yml, check-dist.yml, test-cache.yml, test-download.yml, test.yml, and test-post-cleanup.yml; added `permissions: pull-requests: read` to label.yml. Fixed script-injection in test.yml by moving ${{ steps.setup-micromamba.outputs.environment-path }} and ${{ runner.temp }} expressions into `env:` blocks for 5 jobs. Fixed script-injection in test-post-cleanup.yml by changing matrix values from shell command prefixes ('', '! ') to boolean strings ('true'/'false') and using if/else logic in the shell script, and moving ${{ runner.temp }} to `env:` blocks.

