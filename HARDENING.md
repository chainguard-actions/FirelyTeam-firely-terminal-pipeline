<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.9.0** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download Java Validator' step uses `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` where `JAVA_VALIDATOR_DOWNLOAD_LOCATION` is set from `${{ inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION }}`. Using `eval` with a user-controlled value allows arbitrary shell command execution — an attacker can supply a value like `$(malicious_command)` or inject shell metacharacters. Offending line: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`

Locations:

- `action.yml:556`

### script-injection (severity: high)

Sub-rule (b): Multiple `run:` blocks expand user-controlled input env vars without double-quoting, allowing shell word-splitting and glob expansion of attacker-controlled values:

1. 'Install Firely.Terminal' step: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — unquoted, sourced from `${{ inputs.FIRELY_TERMINAL_VERSION }}`.
2. 'Install fsh-sushi' step: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — unquoted, sourced from `${{ inputs.SUSHI_VERSION }}`.
3. 'Generate conformance resources with SUSHI' step: `sushi $INPUT_SUSHI_OPTIONS` — unquoted, sourced from `${{ inputs.SUSHI_OPTIONS }}`.
4. 'Run Quality Control checks' step: `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — both unquoted, sourced from user inputs.
5. 'Validate all conformance resources' and 'Validate all example resources' steps: `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` — unquoted, sourced from `${{ inputs.JAVA_VALIDATION_OPTIONS }}`.
6. 'Download Java Validator' step: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` — unquoted URL sourced from `${{ inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION }}`.

Locations:

- `action.yml:113`
- `action.yml:430`
- `action.yml:437`
- `action.yml:500`
- `action.yml:509`
- `action.yml:557`
- `action.yml:591`
- `action.yml:596`
- `action.yml:648`
- `action.yml:653`

### github-env-injection (severity: high)

The 'Detect FHIR version and determine dependency source' step writes `FHIR_VERSION` to `$GITHUB_ENV` without sanitization: `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`. The value of `fhirVersion` is extracted from repository files (package.json via `jq` or sushi-config.yaml via `yq`). These files are repository-controlled and could contain embedded newlines, enabling an attacker who controls the repository content to inject arbitrary environment variables into subsequent steps via the `GITHUB_ENV` protocol. The required sanitization step (`printf '%s' "$fhirVersion" | tr -d '\n\r'`) is absent before the write.

Locations:

- `action.yml:231`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all three findings in action.yml:

1. script-injection (a) - Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash parameter substitution `"${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}"` to expand the version placeholder without using eval.

2. script-injection (b) - Fixed all unquoted user-controlled variables:
   - Quoted `$FIRELY_TERMINAL_VERSION` in dotnet install command
   - Quoted `fsh-sushi@$SUSHI_VERSION` in npm install command
   - Tokenized `$INPUT_SUSHI_OPTIONS` (list input) with xargs into bash array in both SUSHI steps
   - Quoted `$INPUT_EXPECTED_FAILS` in echo|grep pipes
   - Used `${INPUT_PATH_TO_QUALITY_CONTROL_RULES:+"$INPUT_PATH_TO_QUALITY_CONTROL_RULES"}` for optional path argument
   - Tokenized `$INPUT_JAVA_VALIDATION_OPTIONS` (list input) with xargs into bash array in both Java validator steps
   - Quoted `$JAVA_VALIDATOR_DOWNLOAD_LOCATION` in wget command

3. github-env-injection - Added sanitization of `fhirVersion` before writing to GITHUB_ENV: `safe_fhirVersion=$(printf '%s' "$fhirVersion" | tr -d '\n\r')` then writing `$safe_fhirVersion` to GITHUB_ENV.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all three script-injection findings in action.yml:

1. Simplifier login (line 148): Added double-quotes around $INPUT_SIMPLIFIER_USERNAME and $INPUT_SIMPLIFIER_PASSWORD in the `fhir login email=... password=...` command.

2. Validate conformance resources (lines 497, 501, 511, 514): Replaced the string-accumulator LOCAL_IG_PARAMETERS with a bash array `local_ig_params`, tokenized UNESCPAED_IG_DEPENDENCIES into an array `ig_dep_args` using xargs/printf, quoted $FHIR_VERSION, and used array expansions in the java validator invocation.

3. Validate example resources (lines 567, 571, 581, 584): Same pattern — replaced COMBINED_IG_PARAMETERS with `combined_ig_params` array, tokenized UNESCPAED_IG_DEPENDENCIES into `ig_dep_args` array, quoted $FHIR_VERSION, and used array expansions in the java validator invocation.

Intentional glob patterns ($GITHUB_WORKSPACE/$p*.xml, $GITHUB_WORKSPACE/$p*.json) and intentional word-splitting in for-loops over space-separated path lists were preserved.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yml:

1. 'Validate all conformance resources' step: Tokenized INPUT_PATH_TO_CONFORMANCE_RESOURCES into a bash array (conformance_paths) using xargs for quote-aware splitting. Replaced both unquoted `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;` loops with `for p in "${conformance_paths[@]}";`. Changed unquoted `$GITHUB_WORKSPACE/$p*.xml` to `"$GITHUB_WORKSPACE/${p}"*.xml` (and similarly for .json) to quote the variable portion while preserving glob expansion.

2. 'Validate all example resources' step: Same fix for INPUT_PATH_TO_CONFORMANCE_RESOURCES (into conformance_paths array) and additionally tokenized PATH_TO_EXAMPLES into an example_paths array. Replaced both unquoted for-loops with properly quoted array iterations. Fixed the java command glob patterns the same way.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed all 23+ locations where boolean input variables (DOTNET_VALIDATION_ENABLED, JAVA_VALIDATION_ENABLED, SUSHI_ENABLED, SUSHI_USE_CONFIG_DEPENDENCIES, TERMINOLOGY_SERVICE_BFARM_ENABLED, JAVA_SNAPSHOT_ENABLED, CLOSE_SLICING_FOR_VALIDATION) were used bare as shell commands in `if`/`elif` conditions. Replaced all instances of `if $INPUT_VAR` and `elif $INPUT_VAR` with `[ "$INPUT_VAR" = "true" ]` comparisons, and compound conditions like `if $A || $B` with `[ "$A" = "true" ] || [ "$B" = "true" ]`. This prevents shell metacharacter injection where a caller could supply a value like `true; malicious_command` to execute arbitrary commands.

