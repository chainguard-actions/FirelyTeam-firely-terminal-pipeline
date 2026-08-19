<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.6** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple unquoted shell variable expansions of env vars holding untrusted inputs (sub-rule b). In action.yml, the following env vars derived from inputs.* are expanded without double-quoting inside run: blocks, allowing shell metacharacter injection:
- `sushi $INPUT_SUSHI_OPTIONS` (from inputs.SUSHI_OPTIONS) — in 'Generate conformance resources with SUSHI' and 'Re-run SUSHI' steps
- `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` (from inputs.PATH_TO_QUALITY_CONTROL_RULES) — in 'Run Quality Control checks' step
- `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` (from inputs.PATH_TO_CONFORMANCE_RESOURCES) — in multiple Java validator steps
- `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS` (from inputs.JAVA_VALIDATION_OPTIONS) — in Java validation steps
- `echo $INPUT_EXPECTED_FAILS | grep ...` (from inputs.EXPECTED_FAILS) — in multiple steps
- `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` (from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION) — in 'Download Java Validator' step
- `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (from inputs.FIRELY_TERMINAL_VERSION) — in 'Install Firely.Terminal' step
All of these must be double-quoted: `"$VAR"` or `"${VAR}"`

Locations:

- `action.yml:24535`
- `action.yml:28788`
- `action.yml:33281`
- `action.yml:33987`
- `action.yml:28700`
- `action.yml:31286`
- `action.yml:6070`
- `action.yml:39381`

### unpinned-uses (severity: high)

All three workflow files reference `actions/checkout@v4` using a mutable version tag instead of a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved. Use a full SHA pin such as `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/update_firelyterminal.yml:14`
- `.github/workflows/update_javavalidator.yml:14`
- `.github/workflows/update_sushi.yml:14`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and no job-level `permissions:` blocks are present either. Without explicit permissions, workflows inherit the default repository permissions (which may be broad). Each workflow should declare minimal permissions, e.g. `permissions: contents: write` for the PR-creation workflows.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings:

1. script-injection: Quoted all unquoted variable expansions in action.yml. Single optional values (FIRELY_TERMINAL_VERSION, JAVA_VALIDATOR_DOWNLOAD_LOCATION, PATH_TO_QUALITY_CONTROL_RULES, EXPECTED_FAILS) were double-quoted or used ${VAR:+"$VAR"} form. List-type inputs (SUSHI_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, JAVA_VALIDATION_OPTIONS, IG_DEPENDENCIES) were tokenized using xargs into bash arrays to preserve argument boundaries while preventing injection.

2. unpinned-uses: Replaced actions/checkout@v4 with actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4 in all three workflow files (update_firelyterminal.yml, update_javavalidator.yml, update_sushi.yml).

3. missing-permissions: Added top-level permissions blocks (contents: write, pull-requests: write) to all three workflow files, as they create branches and PRs.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 6 security findings across 4 files:

1. action.yml (line 497) - script-injection: Replaced dangerous `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash string substitution `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}` that expands the literal `$JAVA_VALIDATOR_VERSION` placeholder in the URL without executing arbitrary shell code.

2. action.yml (line 385) - script-injection: Quoted the npm install argument to prevent shell metacharacter injection: changed `fsh-sushi@$SUSHI_VERSION` to `"fsh-sushi@$SUSHI_VERSION"`.

3. action.yml (line 218) - github-env-injection: Added newline sanitization before writing fhirVersion to $GITHUB_ENV using `safe_fhirVersion=$(printf '%s' "$fhirVersion" | tr -d '\n\r')`.

4. update_firelyterminal.yml (lines 27, 33) - github-env-injection: Added newline sanitization for both FIRELY_TERMINAL_VERSION and LATEST_RELEASE before writing to $GITHUB_ENV.

5. update_javavalidator.yml (lines 27, 33) - github-env-injection: Added newline sanitization for both JAVA_VALIDATOR_VERSION and LATEST_RELEASE before writing to $GITHUB_ENV.

6. update_sushi.yml (lines 26, 32) - github-env-injection: Added newline sanitization for both SUSHI_VERSION and LATEST_RELEASE before writing to $GITHUB_ENV.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two categories of script injection vulnerabilities in action.yml:

1. Unquoted variable expansions in 'Simplifier login' step (line 121): Added double-quotes around $INPUT_SIMPLIFIER_USERNAME and $INPUT_SIMPLIFIER_PASSWORD in the `fhir login` command to prevent shell metacharacter injection.

2. Unsafe `if $VAR; then` boolean patterns (lines 96, 107, 118, 148, 388, 468, 638 and many more): Replaced all occurrences of `if $INPUT_VAR; then` and `if $CLOSE_SLICING_FOR_VALIDATION; then` patterns with safe `[ "$VAR" = "true" ]` string comparisons. This prevents attacker-controlled input values from being executed as arbitrary shell commands. The fix covers 20+ occurrences across all steps that use boolean inputs (DOTNET_VALIDATION_ENABLED, TERMINOLOGY_SERVICE_BFARM_ENABLED, JAVA_VALIDATION_ENABLED, SUSHI_ENABLED, SUSHI_USE_CONFIG_DEPENDENCIES, JAVA_SNAPSHOT_ENABLED, CLOSE_SLICING_FOR_VALIDATION).

