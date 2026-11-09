<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.23** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): the input JAVA_VALIDATOR_DOWNLOAD_LOCATION is run through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. eval re-parses the value, so a value containing $(...) or backticks runs as a command. Two steps follow: the result is expanded unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`, and JAVA_VALIDATOR_VERSION is substituted into the URL. A safer approach is to replace the literal '$JAVA_VALIDATOR_VERSION' token with a fixed string substitution such as ${VAR//pattern/repl} instead of using eval, then quote the expansion.

Locations:

- `action.yml:650`
- `action.yml:653`

### script-injection (severity: high)

Rule (b): boolean inputs are expanded unquoted and run as commands in almost every step, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`. The shell runs whatever the input contains, so a value like `rm -rf / ;true` executes arbitrary commands. Compare quoted strings instead, e.g. [[ "$VAR" == "true" ]].

Locations:

- `action.yml:100`

### script-injection (severity: high)

Rule (b): step 'Install Firely.Terminal' expands an input unquoted: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`.

Locations:

- `action.yml:122`

### script-injection (severity: high)

Rule (b): step 'Simplifier login' expands credentials from inputs unquoted: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:140`

### script-injection (severity: high)

Rule (b): step 'Install fsh-sushi' expands an input unquoted: `sudo npm install -g fsh-sushi@$SUSHI_VERSION`.

Locations:

- `action.yml:505`

### script-injection (severity: high)

Rule (b): steps 'Generate conformance resources with SUSHI' and 'Re-run SUSHI...' expand an input unquoted: `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:517`
- `action.yml:825`

### script-injection (severity: high)

Rule (b): step 'Run Quality Control checks' expands inputs unquoted: `echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:600`

### script-injection (severity: high)

Rule (b): step 'Remove package cache...' runs `rm -rf $fhirCacheLocation` with the variable unquoted.

Locations:

- `action.yml:630`

### script-injection (severity: high)

Rule (b): step 'Validate all conformance resources' expands workflow-controlled values unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`. FHIR_VERSION comes from env and INPUT_JAVA_VALIDATION_OPTIONS from inputs. Neither has a '# shellcheck disable=SC2086' array form.

Locations:

- `action.yml:700`

### script-injection (severity: high)

Rule (b): step 'Validate all example resources' expands workflow-controlled values unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:770`

### github-env-injection (severity: high)

Step 'Detect FHIR version and determine dependency source' writes `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`. fhirVersion is not a literal: it is read from the repository's package.json or sushi-config.yaml with jq/yq (and in one branch decoded with base64 -d), and both files can be changed in a PR. The write has no `printf '%s' ... | tr -d '\n\r'` step first, so a value containing a newline could inject extra environment variables.

Locations:

- `action.yml:280`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

All changes are in action.yml. Nothing was run or tested, but a search found no unquoted boolean checks, eval calls or unquoted `echo $INPUT` left.

- **Boolean inputs:** every `if $X` / `elif $X` / `$A || $B` / `$A && $B` check is now a string comparison, `[[ "$X" == "true" ]]`. This gives the same result as before for `true` and `false`, and no longer runs whatever text the input contains.
- **Java validator download:** removed `eval`. The URL is now built by replacing the literal text `$JAVA_VALIDATOR_VERSION` or `${JAVA_VALIDATOR_VERSION}` with the version, and the URL passed to wget is quoted.
- **Single values are now quoted:** the Firely Terminal version, the Simplifier login arguments, the SUSHI version, `$USER`, the package cache path given to `rm -rf`, and `-version "$FHIR_VERSION"`. The optional quality-control rules path uses `${VAR:+"$VAR"}`, so an empty value adds no argument.
- **EXPECTED_FAILS:** checked with `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w`.
- **List inputs:** SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES are now loaded into arrays with an unquoted assignment (marked with `# shellcheck disable=SC2086`). That splits words and expands globs exactly as before, then each array is passed quoted. The IG dependency list is handled the same way. The local and combined `-ig` arguments are now arrays, so each path stays one argument. The `$p*.xml` / `$p*.json` patterns keep their glob, with only the folder part quoted.
- **FHIR_VERSION:** newlines are stripped with `printf '%s' | tr -d '\n\r'` before it is written to `$GITHUB_ENV`.
- **Extra change not in the findings:** `echo $FHIR_RESTORE` is now quoted.

