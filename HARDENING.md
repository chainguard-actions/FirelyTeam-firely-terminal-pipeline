<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.19** was hardened automatically. 24 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

(b) Step 'Check if .NET is installed': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. These inputs are expanded unquoted and run as a command, so a caller-supplied value is executed.

Locations:

- `action.yml:90`

### script-injection (severity: high)

(b) Step 'Check .NET SDK Version': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:105`

### script-injection (severity: high)

(b) Step 'Install Firely.Terminal': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED` and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`. Both expand inputs unquoted.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

(b) Step 'Check Firely Terminal Version': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:131`

### script-injection (severity: high)

(b) Step 'Simplifier login': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` and `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`. Both expand inputs unquoted.

Locations:

- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

(b) Step 'Detect FHIR version and determine dependency source': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...`. These inputs are expanded unquoted and run as commands.

Locations:

- `action.yml:168`
- `action.yml:183`
- `action.yml:202`

### script-injection (severity: high)

(b) Step 'Validate FHIR version support': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`. The inputs are expanded unquoted and run as commands.

Locations:

- `action.yml:307`

### script-injection (severity: high)

(b) Step 'Set FHIR specification and restore dependencies': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:386`

### script-injection (severity: high)

(b) Step 'Check if npm is installed': `if $INPUT_SUSHI_ENABLED; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:525`

### script-injection (severity: high)

(b) Step 'Check npm Version': `if $INPUT_SUSHI_ENABLED; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:535`

### script-injection (severity: high)

(b) Step 'Install fsh-sushi': `if $INPUT_SUSHI_ENABLED; then`, `sudo chown -R $USER ...` and `sudo npm install -g fsh-sushi@$SUSHI_VERSION`. These expand inputs and the inherited $USER unquoted.

Locations:

- `action.yml:542`

### script-injection (severity: high)

(b) Step 'Check SUSHI version': `if $INPUT_SUSHI_ENABLED; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:550`

### script-injection (severity: high)

(b) Step 'Generate conformance resources with SUSHI': `if $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`. Both expand inputs unquoted, and neither line uses the exempted shellcheck-disabled array form.

Locations:

- `action.yml:555`

### script-injection (severity: high)

(b) Step 'Pre-processing FHIR profiles (Slicing open-closed)': `if $CLOSE_SLICING_FOR_VALIDATION; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:570`

### script-injection (severity: high)

(b) Step 'Configure Firely Terminal validator engine': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:627`

### script-injection (severity: high)

(b) Step 'Run Quality Control checks': `if $INPUT_DOTNET_VALIDATION_ENABLED`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check [--fail] $INPUT_PATH_TO_QUALITY_CONTROL_RULES`. All three expand inputs unquoted.

Locations:

- `action.yml:652`

### script-injection (severity: high)

(b) Step 'Report Success - .NET Validator': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:669`

### script-injection (severity: high)

(b) Step 'Remove package cache...': `if $INPUT_JAVA_SNAPSHOT_ENABLED; then` and `rm -rf $fhirCacheLocation`. Both expansions are unquoted.

Locations:

- `action.yml:688`

### script-injection (severity: high)

(b) Step 'Check if Java is installed': `if $INPUT_JAVA_VALIDATION_ENABLED; then`. The input is expanded unquoted and run as a command.

Locations:

- `action.yml:697`

### script-injection (severity: high)

(b) Step 'Download Java Validator': `if $INPUT_JAVA_VALIDATION_ENABLED`, `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` and `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. eval runs the caller-supplied inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION through the shell, so any $(...) or ; in the value executes. The other two lines expand inputs unquoted.

Locations:

- `action.yml:707`
- `action.yml:711`

### script-injection (severity: high)

(b) Step 'Validate all conformance resources in scope of the repository': `if $INPUT_JAVA_VALIDATION_ENABLED`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`. All of these expand workflow-controlled values unquoted.

Locations:

- `action.yml:727`

### script-injection (severity: high)

(b) Step 'Validate all example resources in scope of the repository': `if $INPUT_JAVA_VALIDATION_ENABLED`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `for p in $PATH_TO_EXAMPLES`, `echo $INPUT_EXPECTED_FAILS` and `java -jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`. All of these expand workflow-controlled values unquoted.

Locations:

- `action.yml:816`

### script-injection (severity: high)

(b) Step 'Re-run SUSHI to restore conformance resources after slicing modification': `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`. Both expand inputs unquoted.

Locations:

- `action.yml:905`

### github-env-injection (severity: high)

Step 'Detect FHIR version and determine dependency source' runs `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`. fhirVersion is read from repository-controlled package.json (.fhirVersions[0]) or sushi-config.yaml (.fhirVersion, including a base64-decoded fallback). No `printf '%s' ... | tr -d '\n\r'` runs before the write, so a value with a newline in it can inject extra environment variables.

Locations:

- `action.yml:271`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

All changes are in action.yml. Nothing has been run yet.

- **Boolean checks:** each `if $INPUT_X` (and the `||` / `&&` combinations) is now `[ "$X" = "true" ]`. For true/false, and for any other non-empty value, this behaves as before. An empty value used to count as true and now counts as false; every one of these inputs has a default, so this only matters if a caller passes an explicit empty string.
- **Single values now quoted:** the Firely Terminal and SUSHI versions, `$USER`, the Simplifier email and password, the cache path in `rm -rf`, and `$FHIR_VERSION`. `PATH_TO_QUALITY_CONTROL_RULES` is optional, so it uses `${VAR:+"$VAR"}` and drops out when empty. `EXPECTED_FAILS` is now piped to grep with `printf '%s\n' "..."` instead of an unquoted echo.
- **List inputs keep upstream's splitting:** SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES and the IG dependency list from package.json each go into an array assigned unquoted, with `# shellcheck disable=SC2086`. That splits on whitespace and expands globs exactly as before. The `-ig` parameters are built as arrays instead of strings, and the `$p*.xml` / `$p*.json` patterns still expand.
- **eval removed:** the download URL no longer goes through `eval echo`. The step now swaps the literal `$JAVA_VALIDATOR_VERSION` / `${JAVA_VALIDATOR_VERSION}` placeholder (as in the default URL) for the version, and quotes the URL passed to wget. **Behaviour change:** a custom URL that refers to any other shell variable will no longer have it filled in.
- **github-env-injection:** `fhirVersion` has `\n` and `\r` stripped (`printf '%s' | tr -d '\n\r'`) before it is written to `$GITHUB_ENV`.

