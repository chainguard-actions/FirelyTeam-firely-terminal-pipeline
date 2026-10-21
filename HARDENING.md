<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.9.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.9.3** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): the boolean input flags are run directly as shell commands without quotes, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`. Bash runs whatever the input contains, so a value like `true; <cmd>` runs extra commands. This pattern appears in almost every step (lines listed). Compare with literals instead, e.g. `[[ "$INPUT_X" == "true" ]]`.

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

Rule (b), step 'Install Firely.Terminal': unquoted input expansion `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null`.

Locations:

- `action.yml:119`

### script-injection (severity: high)

Rule (b), step 'Simplifier login': unquoted input expansion `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b), step 'Install fsh-sushi': unquoted expansions `sudo chown -R $USER /usr/local/lib/node_modules` and `sudo npm install -g fsh-sushi@$SUSHI_VERSION` ($SUSHI_VERSION comes from inputs.SUSHI_VERSION).

Locations:

- `action.yml:525`
- `action.yml:526`

### script-injection (severity: high)

Rule (b), step 'Generate conformance resources with SUSHI': unquoted input expansion `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:545`

### script-injection (severity: high)

Rule (b), step 'Run Quality Control checks': unquoted expansions `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:639`
- `action.yml:640`
- `action.yml:642`

### script-injection (severity: high)

Rule (b), step 'Remove package cache...': unquoted variable used by a destructive command, `rm -rf $fhirCacheLocation`. The step is enabled by an input flag that is itself run unquoted (`if $INPUT_JAVA_SNAPSHOT_ENABLED`).

Locations:

- `action.yml:672`

### script-injection (severity: high)

Rule (b), step 'Download Java Validator': the input inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION is passed to eval, `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Any $(...) or ; in the value runs as a command. The result is then used unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. Replace eval with a literal substitution, e.g. `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`, and quote the variable.

Locations:

- `action.yml:697`
- `action.yml:700`

### script-injection (severity: high)

Rule (b), step 'Validate all conformance resources in scope of the repository': unquoted input expansions `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml $GITHUB_WORKSPACE/$p*.json -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`.

Locations:

- `action.yml:741`
- `action.yml:746`
- `action.yml:756`
- `action.yml:757`
- `action.yml:759`

### script-injection (severity: high)

Rule (b), step 'Validate all example resources in scope of the repository': unquoted input expansions `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $COMBINED_IG_PARAMETERS`.

Locations:

- `action.yml:809`
- `action.yml:814`
- `action.yml:824`
- `action.yml:825`
- `action.yml:827`

### script-injection (severity: high)

Rule (b), step 'Re-run SUSHI to restore conformance resources after slicing modification': unquoted expansions `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:857`

### github-env-injection (severity: high)

Step 'Detect FHIR version and determine dependency source': `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` writes a value read from repository files (`jq -r '.fhirVersions[0]' package.json` / `yq eval '.fhirVersion' sushi-config.yaml`, which a pull request can change). It is not a literal set in the script, and newlines are not stripped with `printf '%s' ... | tr -d '\n\r'` first, so a crafted value could add extra environment variables.

Locations:

- `action.yml:271`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

- Boolean inputs are no longer run as commands. Every 'if $INPUT_X' is now a literal check, [[ "$X" == "true" ]]. This includes SUSHI_USE_CONFIG_DEPENDENCIES and the 'elif $INPUT_SUSHI_ENABLED' branch, which the findings did not list.
- Single values are now quoted: FIRELY_TERMINAL_VERSION, the Simplifier login values ('email=...' and 'password=...'), $USER, and fsh-sushi@$SUSHI_VERSION. PATH_TO_QUALITY_CONTROL_RULES is optional, so it uses ${VAR:+"$VAR"} and drops out when empty.
- The EXPECTED_FAILS checks now use printf '%s\n' "$VAR" | grep.
- The cache removal is now 'rm -rf -- "$fhirCacheLocation"' and only runs if the path is not empty.
- Removed the eval on JAVA_VALIDATOR_DOWNLOAD_LOCATION. The script now does a literal substitution of $JAVA_VALIDATOR_VERSION and ${JAVA_VALIDATOR_VERSION}, and the wget URL is quoted.
- List inputs are split into arrays on whitespace with 'read -r -d "" -a' (guarded, '|| true'), the same way upstream's unquoted $VAR split them. That covers SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES and the IG dependency list (double quotes removed as before). The -ig parameters are now built as arrays. The validator file globs keep working: the path prefix is quoted and *.xml / *.json stay unquoted.
- github-env-injection: FHIR_VERSION has newlines stripped with tr -d '\n\r' before it is written to GITHUB_ENV.
- Also quoted a few other expansions the findings did not list: the supportedVersions loop, 'echo "$FHIR_RESTORE"', and the ZTS download URL.
- A batch of edits made at the same time garbled the file, so I rewrote it in full from the original with all fixes applied.
- Nothing was run to test the changes.

