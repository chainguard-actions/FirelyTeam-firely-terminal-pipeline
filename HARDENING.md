<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.25

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.25** was hardened automatically. 12 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): the input JAVA_VALIDATOR_DOWNLOAD_LOCATION is passed to eval, which runs it as shell code: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. A value containing $(...) or backticks runs arbitrary commands. The result is then expanded without quotes: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. Fix: replace the eval with a literal substitution of $JAVA_VALIDATOR_VERSION (e.g. `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`) and quote the variable passed to wget. The `if $INPUT_JAVA_VALIDATION_ENABLED` line in this step is also unquoted.

Locations:

- `action.yml:698`
- `action.yml:701`

### script-injection (severity: high)

Rule (b): in step 'Simplifier login', inputs are expanded without quotes: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`. The step also runs the input DOTNET_VALIDATION_ENABLED as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

Rule (b): in step 'Install Firely.Terminal', an input is expanded without quotes: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`. The step also runs the input flags as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

Rule (b): in step 'Install fsh-sushi', an input is expanded without quotes: `sudo npm install -g fsh-sushi@$SUSHI_VERSION`. The step also runs an input as a command: `if $INPUT_SUSHI_ENABLED; then`.

Locations:

- `action.yml:525`
- `action.yml:527`

### script-injection (severity: high)

Rule (b): in step 'Generate conformance resources with SUSHI', an input is expanded without quotes: `sushi $INPUT_SUSHI_OPTIONS`. The step also runs an input as a command: `if $INPUT_SUSHI_ENABLED; then`.

Locations:

- `action.yml:545`
- `action.yml:546`

### script-injection (severity: high)

Rule (b): in step 'Run Quality Control checks', inputs are expanded without quotes: `if echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`. The step also has `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, which runs the input as a command.

Locations:

- `action.yml:638`
- `action.yml:640`
- `action.yml:641`
- `action.yml:643`

### script-injection (severity: high)

Rule (b): in step 'Validate all conformance resources in scope of the repository', inputs are expanded without quotes: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:719`
- `action.yml:742`
- `action.yml:747`
- `action.yml:757`
- `action.yml:758`

### script-injection (severity: high)

Rule (b): in step 'Validate all example resources in scope of the repository', inputs are expanded without quotes: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:793`
- `action.yml:810`
- `action.yml:815`
- `action.yml:825`
- `action.yml:826`

### script-injection (severity: high)

Rule (b): in step 'Re-run SUSHI to restore conformance resources after slicing modification', inputs are expanded without quotes: `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` runs the inputs as commands, and `sushi $INPUT_SUSHI_OPTIONS` is unquoted.

Locations:

- `action.yml:856`
- `action.yml:858`

### script-injection (severity: high)

Rule (b): in step 'Pre-processing FHIR profiles (Slicing open-closed)', an input is expanded without quotes: `if $CLOSE_SLICING_FOR_VALIDATION; then`. The shell runs the input's value as a command.

Locations:

- `action.yml:559`

### script-injection (severity: high)

Rule (b): in step 'Detect FHIR version and determine dependency source', inputs are expanded without quotes and run as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, and `elif $INPUT_SUSHI_ENABLED && ...`.

Locations:

- `action.yml:168`
- `action.yml:183`
- `action.yml:202`

### script-injection (severity: high)

Rule (b): several more steps use an input-derived boolean as a command without quotes, so the shell runs the value: `if $INPUT_DOTNET_VALIDATION_ENABLED ...` in 'Check if .NET is installed' (line 90), 'Check .NET SDK Version' (105), 'Check Firely Terminal Version' (131), 'Validate FHIR version support' (285), 'Set FHIR specification and restore dependencies' (363), 'Configure Firely Terminal validator engine' (614) and 'Report Success - .NET Validator' (654). Also `if $INPUT_SUSHI_ENABLED` in 'Check if npm is installed' (502), 'Check npm Version' (515) and 'Check SUSHI version' (536); `if $INPUT_JAVA_SNAPSHOT_ENABLED` in 'Remove package cache...' (671); and `if $INPUT_JAVA_VALIDATION_ENABLED` in 'Check if Java is installed' (681). Fix: compare quoted values instead, e.g. `if [[ "$VAR" == "true" ]]`.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:131`
- `action.yml:285`
- `action.yml:363`
- `action.yml:502`
- `action.yml:515`
- `action.yml:536`
- `action.yml:614`
- `action.yml:654`
- `action.yml:671`
- `action.yml:681`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

I fixed all twelve script-injection findings in action.yml. Nothing was run, since the action ships no tests.

Changes:
- **Boolean flags:** every `if $INPUT_...` / `if $CLOSE_SLICING_FOR_VALIDATION` check is now a quoted comparison, `[[ "$X" == "true" ]]`. This covers the `||` and `&&` combinations.
- **Download Java Validator:** the `eval echo` is gone. The `$JAVA_VALIDATOR_VERSION` and `${JAVA_VALIDATOR_VERSION}` placeholders in the URL are now replaced as literal text, and the URL passed to `wget` is quoted.
- **Single values are quoted:** the Firely Terminal version, `fsh-sushi@$SUSHI_VERSION`, the Simplifier `email=`/`password=` arguments, `$FHIR_VERSION`, `$USER` and the `rm -rf` cache path.
- **QC rules path:** this is an optional single value, so it is now passed as `${VAR:+"$VAR"}`. When it is empty, no argument is passed, as before.
- **EXPECTED_FAILS check:** now `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w -q ...`.
- **List inputs** (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES): each is loaded into an array with an unquoted assignment (`arr=($VAR)`), so it splits and globs exactly as upstream's unquoted expansion did, then passed on as `"${arr[@]}"`. The IG dependency string is handled the same way.
- **The `-ig` parameter lists** are now built as arrays, so each workspace path stays a single quoted argument.
- **Validator file arguments:** now written as `"$GITHUB_WORKSPACE/$p"*.xml` and `*.json`, so the `*` still matches files.

Behaviour change: a flag now counts as on only when its value is exactly `true`. Upstream ran the value as a command, so `True` or `TRUE` would have errored there and now simply mean off. GitHub boolean inputs arrive as `true`/`false`, so normal callers are unaffected.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In action.yml, I changed all 9 '# shellcheck disable=SC2206' annotations to '# shellcheck disable=SC2086'. Each one sits directly above an unquoted array assignment: sushi_args, conformance_paths, java_opts, example_paths and ig_dep_args. A search confirmed that every annotation is followed straight away by one of these assignments. I kept the unquoted array assignments as they were, so the inputs are still split on whitespace and glob-expanded exactly as upstream did it.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Detect FHIR version' step now passes fhirVersion through printf '%s' | tr -d '\n\r' before writing FHIR_VERSION to $GITHUB_ENV. The other GITHUB_ENV writes in this step only set true/false values defined in the script, so they were left alone.

