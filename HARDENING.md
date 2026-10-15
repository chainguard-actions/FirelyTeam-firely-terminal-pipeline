<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.9.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.9.2** was hardened automatically. 2 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): Multiple run: blocks expand env vars holding workflow-controllable (untrusted) input values without double-quoting, allowing shell metacharacter injection.

• 'Install Firely.Terminal' step: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — $FIRELY_TERMINAL_VERSION (from inputs.FIRELY_TERMINAL_VERSION) is unquoted.

• 'Simplifier login' step: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` — both vars (from inputs.SIMPLIFIER_USERNAME / inputs.SIMPLIFIER_PASSWORD) are unquoted.

• 'Install fsh-sushi' step: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — $SUSHI_VERSION (from inputs.SUSHI_VERSION) is unquoted.

• 'Generate conformance resources with SUSHI' step: `sushi $INPUT_SUSHI_OPTIONS` — $INPUT_SUSHI_OPTIONS (from inputs.SUSHI_OPTIONS) is unquoted.

• 'Run Quality Control checks' step: `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — both vars from inputs are unquoted.

• 'Download Java Validator' step: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — eval expands an attacker-controlled URL input, enabling arbitrary command injection; also `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` is unquoted.

• 'Validate all conformance resources' and 'Validate all example resources' steps: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS` — multiple unquoted vars from inputs.

Locations:

- `action.yml:119`
- `action.yml:131`
- `action.yml:393`
- `action.yml:399`
- `action.yml:463`
- `action.yml:499`
- `action.yml:502`
- `action.yml:540`
- `action.yml:590`

### github-env-injection (severity: high)

The 'Detect FHIR version and determine dependency source' step writes `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` where $fhirVersion is extracted from repository files (package.json or sushi-config.yaml) via jq/yq without any sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A malicious repository could embed newlines in the fhirVersion field to inject arbitrary environment variables into subsequent steps via GITHUB_ENV.

Locations:

- `action.yml:228`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in hardened/action/action.yml:

1. script-injection fixes:
   - Install Firely.Terminal: quoted $FIRELY_TERMINAL_VERSION
   - Simplifier login: quoted email and password arguments
   - Install fsh-sushi: quoted the npm package spec fsh-sushi@$SUSHI_VERSION
   - Generate conformance resources with SUSHI: replaced unquoted $INPUT_SUSHI_OPTIONS with xargs-based tokenization into bash array
   - Run Quality Control checks: quoted $INPUT_EXPECTED_FAILS in echo, used ${VAR:+"$VAR"} for optional path argument
   - Download Java Validator: replaced dangerous eval with safe bash string substitution for $JAVA_VALIDATOR_VERSION in URL, quoted wget URL
   - Validate all conformance/example resources: quoted $INPUT_EXPECTED_FAILS and $IG_DEPENDENCIES in echo, quoted $FHIR_VERSION, tokenized $INPUT_JAVA_VALIDATION_OPTIONS with xargs into java_opts array
   - Re-run SUSHI: same xargs tokenization fix as Generate conformance resources step

2. github-env-injection fix:
   - Detect FHIR version step: sanitized fhirVersion with printf '%s' | tr -d '\n\r' before writing to GITHUB_ENV

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in two steps of action.yml:

1. 'Validate all conformance resources' step: Replaced two unquoted `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` loops with a single xargs-tokenized array `conformance_paths`, guarded by `if [ -n "$INPUT_PATH_TO_CONFORMANCE_RESOURCES" ]`. Updated java commands to use `"$GITHUB_WORKSPACE/$p"*.xml` and `"$GITHUB_WORKSPACE/$p"*.json` (quoting the variable part while leaving the glob * unquoted for proper file expansion).

2. 'Validate all example resources' step: Similarly replaced the unquoted `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` loop with `conformance_paths2` array, and replaced the unquoted `for p in $PATH_TO_EXAMPLES` loop with an `example_paths` array (also xargs-tokenized with guard). Updated java commands to quote `$p` the same way.

The xargs tokenization approach is correct here because both inputs are documented as space-separated lists of paths (plural 'paths' in the description). The `if [ -n "$VAR" ]` guard and `xargs -r` flag prevent empty-token issues.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all 19 script injection locations in action.yml by replacing unquoted boolean input variable expansions used as shell commands in `if` conditions with proper string comparisons. Patterns like `if $INPUT_DOTNET_VALIDATION_ENABLED; then` were replaced with `if [ "$INPUT_DOTNET_VALIDATION_ENABLED" = "true" ]; then`, and compound conditions like `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` were replaced with `if [ "$INPUT_DOTNET_VALIDATION_ENABLED" = "true" ] || [ "$INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED" = "true" ]; then`. This prevents attacker-controlled input values containing shell metacharacters from being executed as shell commands. All boolean inputs affected: DOTNET_VALIDATION_ENABLED, TERMINOLOGY_SERVICE_BFARM_ENABLED, JAVA_VALIDATION_ENABLED, SUSHI_ENABLED, SUSHI_USE_CONFIG_DEPENDENCIES, JAVA_SNAPSHOT_ENABLED, CLOSE_SLICING_FOR_VALIDATION.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in two Java validator steps:

1. 'Validate all conformance resources' step: Replaced string-based $LOCAL_IG_PARAMETERS and $UNESCPAED_IG_DEPENDENCIES with properly quoted bash arrays (local_ig_params and unescaped_ig_deps). The -ig flags and paths are now stored as separate array elements and expanded with "${array[@]}" quoting.

2. 'Validate all example resources' step: Replaced string-based $COMBINED_IG_PARAMETERS and $UNESCPAED_IG_DEPENDENCIES with properly quoted bash arrays (combined_ig_params and unescaped_ig_deps) using the same approach.

In both cases, IG_DEPENDENCIES (from package.json) is tokenized using the xargs-based pattern to properly split space-separated '-ig pkg#ver' entries into individual array elements, preventing shell metacharacter injection while preserving correct argument boundaries.

