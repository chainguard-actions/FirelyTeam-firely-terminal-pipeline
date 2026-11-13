<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.1** was hardened automatically. 4 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): many run steps expand workflow-controlled inputs (routed through env: from inputs.*) without double quotes. The worst pattern is `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` and similar (`if $INPUT_SUSHI_ENABLED;`, `if $INPUT_JAVA_VALIDATION_ENABLED;`, `if $CLOSE_SLICING_FOR_VALIDATION;`, `if $INPUT_JAVA_SNAPSHOT_ENABLED;`, `elif $INPUT_REPORT_RENDERER_ENABLED;`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES;`). These run the input's value as a shell command. Other unquoted expansions: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`, `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`, `sudo npm install -g fsh-sushi@$SUSHI_VERSION`, `sushi $INPUT_SUSHI_OPTIONS` (in two steps), `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS`, `echo $INPUT_EXPECTED_FAILS | grep ...`, `rm -rf $fhirCacheLocation`, `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` / `for p in $PATH_TO_EXAMPLES`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`. Each of these steps needs its expansions double-quoted, or the boolean inputs compared as strings (e.g. [[ "$X" == "true" ]]).

Locations:

- `action.yml:108`
- `action.yml:119`
- `action.yml:132`
- `action.yml:152`
- `action.yml:175`
- `action.yml:575`
- `action.yml:590`
- `action.yml:600`
- `action.yml:645`
- `action.yml:705`
- `action.yml:725`
- `action.yml:780`
- `action.yml:850`
- `action.yml:905`

### script-injection (severity: high)

Rule (b): the 'Download Java Validator' step runs a workflow-controlled input through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. JAVA_VALIDATOR_DOWNLOAD_LOCATION comes from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION, so a value containing $(...) or backticks runs arbitrary commands. The next line, `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION`, is also unquoted.

Locations:

- `action.yml:722`

### github-env-injection (severity: high)

The 'Determine FHIR package cache key' step writes values derived from inputs to $GITHUB_OUTPUT without the printf '%s' ... | tr -d '\n\r' step: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED come from inputs.*, so a newline in either value can inject extra outputs.

Locations:

- `action.yml:366`

### unpinned-uses (severity: high)

action.yml uses a version tag instead of a full commit SHA: `uses: actions/upload-artifact@v7` in the 'Upload Quality Control report' step. The other uses: references (actions/cache/restore, actions/cache/save, patrick-werner/validation-outcome-markdown-renderer) are pinned to 40-character SHAs.

Locations:

- `action.yml:660`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Changes in action.yml:
- script-injection (unquoted inputs): every `if $X;` / `elif $X;` / `$X && $Y` test now reads `[ "$X" = "true" ]`. That covers DOTNET/JAVA/SUSHI/TERMINOLOGY_SERVICE_BFARM/JAVA_SNAPSHOT/REPORT_RENDERER/CLOSE_SLICING/SUSHI_USE_CONFIG_DEPENDENCIES. The Firely Terminal version, the Simplifier login ("email=..." "password=..."), fsh-sushi@version, $USER and rm -rf "$fhirCacheLocation" are now quoted. EXPECTED_FAILS is passed to grep with printf '%s\n' "...". fhir check now gets an optional quoted rules path plus an array of check args (CHECK_ARGS/NO_FAIL_ARGS). SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES and the IG dependency list are each assigned unquoted to an array, which splits and globs exactly as upstream did, and then expanded as "${arr[@]}". The -ig parameter lists are built as arrays. In the java -jar calls the path prefix and FHIR_VERSION are quoted, and the *.xml/*.json globs still expand.
- script-injection (eval): `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` is replaced by a literal bash substitution of `$JAVA_VALIDATOR_VERSION` and `${JAVA_VALIDATOR_VERSION}` in the URL, so the default URL still resolves. wget now gets the quoted URL.
- github-env-injection: the cache key is assembled into a variable (the sha256 is computed separately) and passed through printf '%s' | tr -d '\n\r' before it is written to GITHUB_OUTPUT.
- unpinned-uses: actions/upload-artifact@v7 is pinned to cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Run Quality Control checks' step (qc), the REPORT path (built from RUNNER_TEMP) now goes through printf '%s' | tr -d '\n\r' before it is written as report= to $GITHUB_OUTPUT. The REPORT variable itself is unchanged, so later uses in the step behave the same.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml step 'Detect FHIR version and determine dependency source', fhirVersion is now sanitized via printf '%s' | tr -d '\n\r' into safeFhirVersion before writing FHIR_VERSION to $GITHUB_ENV. Other writes are literal true/false and were left unchanged.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Added '|| true' to the packageJsonHash=$(sha256sum package.json | cut -c1-16) assignment so that, as in upstream (where the command ran inside echo), a failure no longer aborts the step under errexit.

