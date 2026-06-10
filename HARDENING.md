<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.12** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml use unquoted shell variable expansions of env vars that hold user-controlled inputs (rule b), allowing shell metacharacter injection. Additionally, one step uses `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` where `JAVA_VALIDATOR_DOWNLOAD_LOCATION` is set from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION` (rule a — eval with user-controlled data).

Violating lines include:
- `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` (unquoted inputs.SIMPLIFIER_USERNAME and inputs.SIMPLIFIER_PASSWORD)
- `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (unquoted inputs.FIRELY_TERMINAL_VERSION)
- `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (unquoted inputs.SUSHI_VERSION)
- `sushi $INPUT_SUSHI_OPTIONS` (unquoted inputs.SUSHI_OPTIONS, appears twice)
- `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` (unquoted inputs.PATH_TO_QUALITY_CONTROL_RULES)
- `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` (unquoted inputs.PATH_TO_CONFORMANCE_RESOURCES, multiple occurrences)
- `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` (unquoted inputs.JAVA_VALIDATION_OPTIONS)
- `echo $INPUT_EXPECTED_FAILS | grep ...` (unquoted inputs.EXPECTED_FAILS)
- `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` (unquoted inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION)
- `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — eval with user-controlled input allows arbitrary command execution

Locations:

- `action.yml:128`
- `action.yml:148`
- `action.yml:440`
- `action.yml:515`
- `action.yml:560`
- `action.yml:595`
- `action.yml:610`
- `action.yml:660`
- `action.yml:700`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from user-controlled or externally-sourced data to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` — `fhirVersion` is extracted from repository files (package.json or sushi-config.yaml) via jq/yq. These files can contain newlines or special characters that allow injection of additional environment variables.

2. `echo "NEED_SUSHI_CONVERSION=$needSushiConversion" >> "$GITHUB_ENV"`, `echo "USE_SUSHI_CONFIG=$useSushiConfig" >> "$GITHUB_ENV"`, `echo "USE_PACKAGE_JSON=$usePackageJson" >> "$GITHUB_ENV"` — these boolean variables are derived from `inputs.SUSHI_USE_CONFIG_DEPENDENCIES` and `inputs.SUSHI_ENABLED` (via env vars) without sanitization before writing to GITHUB_ENV.

None of these writes are preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization pipeline.

Locations:

- `action.yml:210`
- `action.yml:211`
- `action.yml:212`
- `action.yml:250`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in action.yml:

**script-injection fixes:**
- Quoted fhir login arguments: `"email=$INPUT_SIMPLIFIER_USERNAME"` and `"password=$INPUT_SIMPLIFIER_PASSWORD"`
- Quoted FIRELY_TERMINAL_VERSION in dotnet tool install: `"$FIRELY_TERMINAL_VERSION"`
- Quoted SUSHI_VERSION in npm install: `"fsh-sushi@$SUSHI_VERSION"`
- Used `${INPUT_SUSHI_OPTIONS:+"$INPUT_SUSHI_OPTIONS"}` for optional SUSHI_OPTIONS in both sushi invocations
- Used `${INPUT_PATH_TO_QUALITY_CONTROL_RULES:+"$INPUT_PATH_TO_QUALITY_CONTROL_RULES"}` for optional QC rules path in fhir check
- Quoted `"$INPUT_EXPECTED_FAILS"` in echo/grep pipelines (3 occurrences)
- Quoted `"$IG_DEPENDENCIES"` in echo/tr pipelines (2 occurrences)
- Quoted `"$FHIR_VERSION"` in -version arguments (4 occurrences)
- Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash parameter expansion `"${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}"` to eliminate arbitrary command execution risk
- Quoted `"$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` in wget call

**github-env-injection fixes:**
- Added `printf '%s' "$needSushiConversion" | tr -d '\n\r'` sanitization before writing NEED_SUSHI_CONVERSION to GITHUB_ENV
- Added `printf '%s' "$useSushiConfig" | tr -d '\n\r'` sanitization before writing USE_SUSHI_CONFIG to GITHUB_ENV
- Added `printf '%s' "$usePackageJson" | tr -d '\n\r'` sanitization before writing USE_PACKAGE_JSON to GITHUB_ENV
- Added `printf '%s' "$fhirVersion" | tr -d '\n\r'` sanitization before writing FHIR_VERSION to GITHUB_ENV

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted shell variable expansions in both Java validation steps:

1. 'Validate all conformance resources' step: Replaced `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` (two loops) with `IFS=' ' read -ra CONFORMANCE_PATHS <<< "$INPUT_PATH_TO_CONFORMANCE_RESOURCES"` followed by `for p in "${CONFORMANCE_PATHS[@]}"`. Replaced unquoted `$INPUT_JAVA_VALIDATION_OPTIONS` in java commands with `IFS=' ' read -ra JAVA_OPTS <<< "$INPUT_JAVA_VALIDATION_OPTIONS"` and `"${JAVA_OPTS[@]}"`.

2. 'Validate all example resources' step: Same fix for `$INPUT_PATH_TO_CONFORMANCE_RESOURCES` and `$INPUT_JAVA_VALIDATION_OPTIONS`, plus replaced `for p in $PATH_TO_EXAMPLES` with `IFS=' ' read -ra EXAMPLE_PATHS <<< "$PATH_TO_EXAMPLES"` followed by `for p in "${EXAMPLE_PATHS[@]}"`.

The array approach (`read -ra`) safely handles word-splitting for space-separated path lists while preventing glob expansion and shell metacharacter injection from attacker-controlled inputs.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in action.yml:

1. **'Validate all conformance resources' step**: Converted `LOCAL_IG_PARAMETERS` from an unquoted string to a bash array (`LOCAL_IG_PARAMETERS=()`), using `LOCAL_IG_PARAMETERS+=("-ig" "$GITHUB_WORKSPACE/$p")` to properly quote each element. Converted `UNESCPAED_IG_DEPENDENCIES` from an unquoted string to a bash array using `IFS=' ' read -ra`. Quoted `$GITHUB_WORKSPACE/$p` in the glob patterns as `"$GITHUB_WORKSPACE/$p"*.xml` and `"$GITHUB_WORKSPACE/$p"*.json`. Expanded all arrays with `"${LOCAL_IG_PARAMETERS[@]}"` and `"${UNESCPAED_IG_DEPENDENCIES[@]}"`.

2. **'Validate all example resources' step**: Applied the same fixes for `COMBINED_IG_PARAMETERS` and `UNESCPAED_IG_DEPENDENCIES` — converted both to bash arrays with proper quoting, and quoted the glob path prefix in the java validator invocation.

These changes prevent an attacker-controlled path value (from `inputs.PATH_TO_CONFORMANCE_RESOURCES` or `inputs.PATH_TO_EXAMPLES`) containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) from achieving command injection.

