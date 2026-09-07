<!-- markdownlint-disable -->

# Hardening Report: mamba-org--setup-micromamba/v3.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **mamba-org--setup-micromamba/v3.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions inside shell commands (rule a), allowing template substitution before the shell parses the command. In test.yml: (1) the output-environment-path-env-file job's run: block uses `test ${{ steps.setup-micromamba.outputs.environment-path }} = ...` and `ls ${{ steps.setup-micromamba.outputs.environment-path }}` without quoting; (2) output-environment-path-env-name-overwrite and output-environment-path-custom-root-prefix use `${{ steps.setup-micromamba.outputs.environment-path }}` in run: blocks; (3) output-no-environment-path uses `${{ steps.setup-micromamba.outputs.environment-path }}`; (4) check-micromamba-on-path uses `echo "${{ runner.temp }}"` and `grep "${{ runner.temp }}/setup-micromamba/micromamba-shell"` directly in a run: block. In test-post-cleanup.yml: the final run: block uses `test -f ${{ runner.temp }}/setup-micromamba/.condarc`. All ${{ ... }} expressions inside run: shell scripts are script-injection risks regardless of context (steps.*, runner.*, matrix.*).

Locations:

- `.github/workflows/test.yml:237`
- `.github/workflows/test.yml:238`
- `.github/workflows/test.yml:249`
- `.github/workflows/test.yml:250`
- `.github/workflows/test.yml:261`
- `.github/workflows/test.yml:262`
- `.github/workflows/test.yml:271`
- `.github/workflows/test.yml:278`
- `.github/workflows/test.yml:282`
- `.github/workflows/test-post-cleanup.yml:68`

### missing-permissions (severity: medium)

Seven workflow files have no top-level permissions: key and no job-level permissions: keys on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write for contents), violating the principle of least privilege. Affected files: build.yml, check-dist.yml, label.yml, test-cache.yml, test-download.yml, test-post-cleanup.yml, test.yml.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/label.yml:1`
- `.github/workflows/test-cache.yml:1`
- `.github/workflows/test-download.yml:1`
- `.github/workflows/test-post-cleanup.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed missing-permissions by adding 'permissions: contents: read' to all 7 workflow files (build.yml, check-dist.yml, label.yml, test-cache.yml, test-download.yml, test-post-cleanup.yml, test.yml). label.yml also got 'pull-requests: read' since it checks PR labels. Fixed script-injection in test.yml for 5 jobs (output-environment-path-env-file, output-environment-path-env-name-overwrite, output-environment-path-custom-root-prefix, output-no-environment-path, check-micromamba-on-path) by moving ${{ steps.setup-micromamba.outputs.environment-path }} and ${{ runner.temp }} into env: blocks as ENVIRONMENT_PATH and RUNNER_TEMP_DIR. Fixed script-injection in test-post-cleanup.yml by moving ${{ runner.temp }} into an env: block as RUNNER_TEMP_DIR in the final run: step.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/test-post-cleanup.yml by moving all ${{ matrix.* }} and ${{ runner.temp }} expressions out of the `with: run: |` shell string and into a step-level `env:` block. The shell script now references MAMBA_INIT_BLOCK_EXISTS, MAMBA_ACTIVATE_EXISTS, ENV_EXISTS, ROOT_EXISTS, BINARY_EXISTS, and RUNNER_TEMP_DIR as plain environment variables, preventing direct expression interpolation inside the shell command string.

