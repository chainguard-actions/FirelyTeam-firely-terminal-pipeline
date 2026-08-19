<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.10** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use `actions/checkout@v4` (a mutable tag reference, not a pinned 40-character SHA commit hash). This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Each file should pin to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and no job in any of these files defines its own `permissions:` block. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. Each workflow should declare minimal required permissions (e.g. `permissions: contents: write; pull-requests: write`) at the top level or per job.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple `run:` steps in action.yml expand untrusted input values as unquoted shell variables (rule b), allowing shell metacharacter injection. A calling workflow can supply values containing `;`, `|`, `$(...)`, etc. to execute arbitrary commands.

(1) `Install Firely.Terminal` step: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — `FIRELY_TERMINAL_VERSION` comes from `inputs.FIRELY_TERMINAL_VERSION`, unquoted.

(2) `Simplifier login` step: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` — both variables come from `inputs.SIMPLIFIER_USERNAME` / `inputs.SIMPLIFIER_PASSWORD`, unquoted.

(3) `Install fsh-sushi` step: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — `SUSHI_VERSION` comes from `inputs.SUSHI_VERSION`, unquoted.

(4) `Generate conformance resources with SUSHI` step: `sushi $INPUT_SUSHI_OPTIONS` — `INPUT_SUSHI_OPTIONS` comes from `inputs.SUSHI_OPTIONS`, unquoted. (Also repeated in the post-processing `Re-run SUSHI` step.)

(5) `Run Quality Control checks` step: `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `echo $INPUT_EXPECTED_FAILS | grep` — both from user inputs, unquoted.

(6) `Download Java Validator` step: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — `JAVA_VALIDATOR_DOWNLOAD_LOCATION` comes from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION`; using `eval` with a user-controlled value allows arbitrary command execution. Additionally `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` is unquoted.

(7) `Validate all conformance resources` and `Validate all example resources` steps: `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...` — `INPUT_JAVA_VALIDATION_OPTIONS` comes from `inputs.JAVA_VALIDATION_OPTIONS`, unquoted.

Locations:

- `action.yml:113`
- `action.yml:133`
- `action.yml:441`
- `action.yml:461`
- `action.yml:519`
- `action.yml:527`
- `action.yml:566`
- `action.yml:568`
- `action.yml:608`
- `action.yml:617`
- `action.yml:660`
- `action.yml:663`
- `action.yml:718`
- `action.yml:762`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings:

1. **unpinned-uses**: Pinned `actions/checkout@v4` to full SHA `11d5960a326750d5838078e36cf38b85af677262` in all three workflow files.

2. **permissions**: Added top-level `permissions: contents: write / pull-requests: write` blocks to all three workflow files (needed for branch push and PR creation).

3. **script-injection** (action.yml):
   - Quoted `"$FIRELY_TERMINAL_VERSION"` in `dotnet tool install`
   - Quoted `"$INPUT_SIMPLIFIER_USERNAME"` and `"$INPUT_SIMPLIFIER_PASSWORD"` in `fhir login`
   - Quoted `"fsh-sushi@$SUSHI_VERSION"` in `npm install`
   - Used xargs tokenization for `INPUT_SUSHI_OPTIONS` in both SUSHI run steps (it's a list of CLI options)
   - Quoted `"$INPUT_EXPECTED_FAILS"` in grep calls; used `${VAR:+"$VAR"}` for optional `INPUT_PATH_TO_QUALITY_CONTROL_RULES`
   - Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash parameter substitution; quoted URL in wget
   - Used xargs tokenization for `INPUT_JAVA_VALIDATION_OPTIONS` and `UNESCPAED_IG_DEPENDENCIES` in both Java validation steps

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed all 7 findings across 4 files:

1. action.yml (github-env-injection, line 270): Sanitized fhirVersion with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_ENV.

2. update_firelyterminal.yml (github-env-injection, lines 28+34): Sanitized FIRELY_TERMINAL_VERSION and LATEST_RELEASE with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_ENV. Also quoted $GITHUB_ENV.

3. update_javavalidator.yml (github-env-injection, lines 28+34): Sanitized JAVA_VALIDATOR_VERSION and LATEST_RELEASE with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_ENV. Also quoted $GITHUB_ENV.

4. update_sushi.yml (github-env-injection, lines 28+34): Sanitized SUSHI_VERSION and LATEST_RELEASE with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_ENV. Also quoted $GITHUB_ENV.

5. action.yml (script-injection, lines 596+601): Replaced `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` with xargs-based array tokenization and `for p in "${conformance_paths[@]}"` in the conformance resources validation step. Converted LOCAL_IG_PARAMETERS from a string to a bash array, expanded with `"${LOCAL_IG_PARAMETERS[@]}"`.

6. action.yml (script-injection, line 668+676): Replaced `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` and `for p in $PATH_TO_EXAMPLES` with xargs-based array tokenization in the examples validation step. Converted COMBINED_IG_PARAMETERS from a string to a bash array, expanded with `"${COMBINED_IG_PARAMETERS[@]}"`.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all 20 reported script injection locations in hardened/action/action.yml. Replaced every `if $INPUT_VARIABLE; then` and `if $CLOSE_SLICING_FOR_VALIDATION; then` pattern with safe quoted string comparisons (`if [ "$INPUT_VARIABLE" = "true" ]; then`). Affected variables: INPUT_DOTNET_VALIDATION_ENABLED, INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED, INPUT_JAVA_VALIDATION_ENABLED, INPUT_SUSHI_ENABLED, INPUT_SUSHI_USE_CONFIG_DEPENDENCIES, INPUT_JAVA_SNAPSHOT_ENABLED, and CLOSE_SLICING_FOR_VALIDATION. Also fixed compound conditions using `||` and `&&` operators. No remaining unquoted boolean variable expansions used as shell commands.

