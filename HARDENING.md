<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.10.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.10.2** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): in 'Check if .NET is installed', the inputs are expanded without quotes and run as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. Compare against a literal instead, e.g. `[[ "$INPUT_X" == "true" ]]`.

Locations:

- `action.yml:90`

### script-injection (severity: high)

Rule (b): in 'Check .NET SDK Version', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs the input as a command, unquoted.

Locations:

- `action.yml:105`

### script-injection (severity: high)

Rule (b): in 'Install Firely.Terminal', `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED` and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` both expand input-derived variables without quotes.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

Rule (b): in 'Check Firely Terminal Version', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` is unquoted and runs the input.

Locations:

- `action.yml:131`

### script-injection (severity: high)

Rule (b): in 'Simplifier login', `if $INPUT_DOTNET_VALIDATION_ENABLED` and `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` expand inputs without quotes.

Locations:

- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

Rule (b): in 'Detect FHIR version and determine dependency source', `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...` all run inputs as commands, unquoted.

Locations:

- `action.yml:167`
- `action.yml:182`
- `action.yml:201`

### script-injection (severity: high)

Rule (b): in 'Validate FHIR version support', `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then` is unquoted.

Locations:

- `action.yml:284`

### script-injection (severity: high)

Rule (b): in 'Set FHIR specification and restore dependencies', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` is unquoted.

Locations:

- `action.yml:362`

### script-injection (severity: high)

Rule (b): in 'Check if npm is installed', `if $INPUT_SUSHI_ENABLED; then` is unquoted.

Locations:

- `action.yml:502`

### script-injection (severity: high)

Rule (b): in 'Check npm Version', `if $INPUT_SUSHI_ENABLED; then` is unquoted.

Locations:

- `action.yml:515`

### script-injection (severity: high)

Rule (b): in 'Install fsh-sushi', `if $INPUT_SUSHI_ENABLED`, `sudo chown -R $USER ...` and `sudo npm install -g fsh-sushi@$SUSHI_VERSION` expand variables without quotes.

Locations:

- `action.yml:525`
- `action.yml:527`

### script-injection (severity: high)

Rule (b): in 'Check SUSHI version', `if $INPUT_SUSHI_ENABLED; then` is unquoted.

Locations:

- `action.yml:536`

### script-injection (severity: high)

Rule (b): in 'Generate conformance resources with SUSHI', `if $INPUT_SUSHI_ENABLED` and `sushi $INPUT_SUSHI_OPTIONS` expand inputs without quotes.

Locations:

- `action.yml:545`
- `action.yml:546`

### script-injection (severity: high)

Rule (b): in 'Pre-processing FHIR profiles (Slicing open-closed)', `if $CLOSE_SLICING_FOR_VALIDATION; then` runs the input value unquoted.

Locations:

- `action.yml:559`

### script-injection (severity: high)

Rule (b): in 'Configure Firely Terminal validator engine', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` is unquoted.

Locations:

- `action.yml:614`

### script-injection (severity: high)

Rule (b): in 'Run Quality Control checks', `if $INPUT_DOTNET_VALIDATION_ENABLED`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` / `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` expand inputs without quotes.

Locations:

- `action.yml:638`
- `action.yml:640`
- `action.yml:641`
- `action.yml:643`

### script-injection (severity: high)

Rule (b): in 'Report Success - .NET Validator', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` is unquoted.

Locations:

- `action.yml:654`

### script-injection (severity: high)

Rule (b): in 'Remove package cache to enable the Java validator to create snapshots', `if $INPUT_JAVA_SNAPSHOT_ENABLED; then` is unquoted.

Locations:

- `action.yml:671`

### script-injection (severity: high)

Rule (b): in 'Check if Java is installed', `if $INPUT_JAVA_VALIDATION_ENABLED; then` is unquoted.

Locations:

- `action.yml:681`

### script-injection (severity: high)

Rule (b): in 'Download Java Validator', `if $INPUT_JAVA_VALIDATION_ENABLED` is unquoted. `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` passes the user-supplied input through eval, so any shell syntax in it runs. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` is also unquoted. Replace the eval with a plain string substitution of $JAVA_VALIDATOR_VERSION.

Locations:

- `action.yml:694`
- `action.yml:698`
- `action.yml:701`

### script-injection (severity: high)

Rule (b): in 'Validate all conformance resources in scope of the repository', `if $INPUT_JAVA_VALIDATION_ENABLED`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` expand input/env-derived variables without quotes.

Locations:

- `action.yml:719`
- `action.yml:743`
- `action.yml:758`
- `action.yml:759`

### script-injection (severity: high)

Rule (b): in 'Validate all example resources in scope of the repository', `if $INPUT_JAVA_VALIDATION_ENABLED`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `for p in $PATH_TO_EXAMPLES`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` are unquoted.

Locations:

- `action.yml:794`
- `action.yml:812`
- `action.yml:817`
- `action.yml:827`
- `action.yml:828`

### script-injection (severity: high)

Rule (b): in 'Re-run SUSHI to restore conformance resources after slicing modification', `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED` and `sushi $INPUT_SUSHI_OPTIONS` expand inputs without quotes.

Locations:

- `action.yml:858`
- `action.yml:860`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

I rewrote action.yml in full. My first batch of parallel edits overwrote each other and broke the file, so I rebuilt it from the original content.

What changed:
- **Boolean checks:** Every boolean input that was run as a command (`if $X`, `$X && $Y`) is now a comparison: `[[ "$X" == "true" ]]`. One behaviour change: an input set explicitly to an empty string used to count as true, and now counts as false. The defaults are unaffected.
- **Single values are now quoted:** `--version "$FIRELY_TERMINAL_VERSION"`, `"fsh-sushi@$SUSHI_VERSION"`, `"$USER"`, `fhir login "email=..." "password=..."`, `rm -rf "$fhirCacheLocation"`.
- **Optional rules path:** `PATH_TO_QUALITY_CONTROL_RULES` now uses `${VAR:+"$VAR"}`, so the argument is left out entirely when the input is empty.
- **EXPECTED_FAILS:** checked with `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w -q ...` instead of an unquoted `echo`.
- **Download URL:** the `eval echo` on `JAVA_VALIDATOR_DOWNLOAD_LOCATION` is gone. The literal `$JAVA_VALIDATOR_VERSION` (or `${JAVA_VALIDATOR_VERSION}`) placeholder is now swapped for the version by plain string replacement, and `wget` gets the URL quoted.
- **List inputs:** `SUSHI_OPTIONS`, `JAVA_VALIDATION_OPTIONS`, `PATH_TO_CONFORMANCE_RESOURCES` and `PATH_TO_EXAMPLES` used to be expanded unquoted. Each is now split on whitespace into an array with `read -r -d '' -a ... || true`, guarded so an empty input adds no argument. This keeps upstream's splitting without treating quotes in the value specially. The IG dependency string and the local/combined `-ig` parameters are built as arrays too.
- **Java validator call:** `-version "$FHIR_VERSION"` is quoted, and the file globs stay outside the quotes (`"$GITHUB_WORKSPACE/$p"*.xml`) so they still expand as before.

I haven't run the action, so none of this is tested.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Detect FHIR version' step now removes CR/LF characters from fhirVersion before writing FHIR_VERSION to $GITHUB_ENV, so a crafted version string can't add other environment variables.

