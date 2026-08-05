<!-- markdownlint-disable -->

# Hardening Report: mamba-org--setup-micromamba/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **mamba-org--setup-micromamba/v3.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### missing-permissions (severity: medium)

Workflow files build.yml, check-dist.yml, label.yml, test-cache.yml, test-download.yml, test-post-cleanup.yml, and test.yml have no top-level `permissions:` key and no job-level `permissions:` blocks. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Only release.yml has a top-level permissions block.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/label.yml:1`
- `.github/workflows/test-cache.yml:1`
- `.github/workflows/test-download.yml:1`
- `.github/workflows/test-post-cleanup.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions (${{ ... }}) are interpolated directly inside run: shell command strings. These values flow through YAML template substitution before the shell processes them, enabling script injection.

In test-post-cleanup.yml, a run: block (passed as a `with: run:` parameter to lisanna-dettwyler/action-post-run) contains: `${{ matrix.mamba-init-block-exists }}grep -F "mamba initialize" ~/.bash_profile`, `${{ matrix.mamba-activate-exists }}grep -F "mamba activate" ~/.bash_profile`, `${{ matrix.env-exists }}test -d ~/micromamba/envs/env-name`, `${{ matrix.root-exists }}test -d ~/micromamba`, `${{ matrix.binary-exists }}test -f ~/micromamba-bin/micromamba`, and `test -f ${{ runner.temp }}/setup-micromamba/.condarc`. A second `- run: |` block also contains `test -f ${{ runner.temp }}/setup-micromamba/.condarc`.

In test.yml, multiple run: blocks contain direct expression interpolation:
- `test ${{ steps.setup-micromamba.outputs.environment-path }} = "$HOME/micromamba/envs/env-name"` (also unquoted — rule b violation)
- `ls ${{ steps.setup-micromamba.outputs.environment-path }}`
- `test "${{ steps.setup-micromamba.outputs.environment-path }}" = /home/runner/micromamba/envs/test`
- `ls "${{ steps.setup-micromamba.outputs.environment-path }}"`
- `test "${{ steps.setup-micromamba.outputs.environment-path }}" = /home/runner/custom-micromamba-root-prefix/envs/test`
- `test "${{ steps.setup-micromamba.outputs.environment-path }}" = ""`
- `echo "${{ runner.temp }}"`
- `which micromamba-shell | grep "${{ runner.temp }}/setup-micromamba/micromamba-shell"`

Locations:

- `.github/workflows/test-post-cleanup.yml:50`
- `.github/workflows/test-post-cleanup.yml:56`
- `.github/workflows/test-post-cleanup.yml:69`
- `.github/workflows/test.yml:266`
- `.github/workflows/test.yml:267`
- `.github/workflows/test.yml:279`
- `.github/workflows/test.yml:280`
- `.github/workflows/test.yml:292`
- `.github/workflows/test.yml:293`
- `.github/workflows/test.yml:303`
- `.github/workflows/test.yml:317`
- `.github/workflows/test.yml:320`

## Iteration Notes

### Iteration 1

**Fixes applied:** missing-permissions, script-injection

**Notes:**

Fixed missing-permissions by adding top-level `permissions:` blocks to all 7 workflow files that lacked them: build.yml and check-dist.yml and test-cache.yml and test-download.yml and test-post-cleanup.yml and test.yml get `permissions: {}`, while label.yml gets `permissions: { pull-requests: write }` since it labels PRs. Fixed script-injection in test-post-cleanup.yml by moving all matrix variable expressions (${{ matrix.mamba-init-block-exists }}, etc.) and ${{ runner.temp }} into `env:` blocks and referencing them as plain shell variables. Fixed script-injection in test.yml by moving ${{ steps.setup-micromamba.outputs.environment-path }} into `env: ENVIRONMENT_PATH:` for 4 steps, and ${{ runner.temp }} into `env: RUNNER_TEMP:` for the check-micromamba-on-path step.

