<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.24** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): the 'Download Java Validator' step runs an input through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. JAVA_VALIDATOR_DOWNLOAD_LOCATION comes from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION, so any $(...), backticks or ; in the value run as shell commands. The same step also expands it unquoted: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`.

Locations:

- `action.yml:640`

### script-injection (severity: high)

Rule (b): boolean inputs are expanded unquoted and run as commands in almost every step, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. Other examples are `if $INPUT_JAVA_VALIDATION_ENABLED`, `if $INPUT_SUSHI_ENABLED`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES`, `if $CLOSE_SLICING_FOR_VALIDATION`, `if $INPUT_JAVA_SNAPSHOT_ENABLED` and `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED`. Because the shell runs the input's value as a command, a value like `true; malicious` runs code. Compare against a quoted string instead, e.g. [[ "$VAR" == "true" ]].

Locations:

- `action.yml:90`

### script-injection (severity: high)

Rule (b): the 'Install Firely.Terminal' step expands an input unquoted: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null`.

Locations:

- `action.yml:119`

### script-injection (severity: high)

Rule (b): the 'Simplifier login' step expands inputs unquoted: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b): the 'Install fsh-sushi' step expands values unquoted: `sudo chown -R $USER /usr/local/lib/node_modules` (USER is an inherited env var) and `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (SUSHI_VERSION comes from inputs.SUSHI_VERSION).

Locations:

- `action.yml:465`

### script-injection (severity: high)

Rule (b): the 'Generate conformance resources with SUSHI' step expands an input unquoted: `sushi $INPUT_SUSHI_OPTIONS`. The same unquoted line appears again in the 'Re-run SUSHI to restore conformance resources' step.

Locations:

- `action.yml:483`

### script-injection (severity: high)

Rule (b): the 'Run Quality Control checks' step expands inputs unquoted: `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET; then`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:580`

### script-injection (severity: high)

Rule (b): the 'Remove package cache' step expands a variable unquoted: `rm -rf $fhirCacheLocation`.

Locations:

- `action.yml:610`

### script-injection (severity: high)

Rule (b): the 'Validate all conformance resources' step expands workflow-controlled values unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml $GITHUB_WORKSPACE/$p*.json -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`.

Locations:

- `action.yml:680`

### script-injection (severity: high)

Rule (b): the 'Validate all example resources' step expands workflow-controlled values unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $COMBINED_IG_PARAMETERS`.

Locations:

- `action.yml:760`

### github-env-injection (severity: high)

The 'Detect FHIR version and determine dependency source' step runs `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`. fhirVersion comes from repository files (package.json via jq, or sushi-config.yaml via yq, including a base64-decode path that can produce newlines) and is not sanitized before the write. A newline in the value could inject extra variables into GITHUB_ENV. Clean it first with `printf '%s' "$fhirVersion" | tr -d '\n\r'`.

Locations:

- `action.yml:290`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

All changes are in action.yml. Nothing has been run or tested.

- **Boolean inputs:** every `if $INPUT_X` / `elif` / `&&` / `||` check now compares the quoted value, e.g. `[[ "$VAR" == "true" ]]`, so the input's value is never run as a command. Behaviour change: only the exact lowercase string `true` counts as true now. Values such as `True` or an empty string, which upstream's form happened to treat as true, now count as false.
- **Java validator download:** removed `eval echo`. The URL now gets only its documented placeholder filled in, with literal bash replacements of both `$JAVA_VALIDATOR_VERSION` and `${JAVA_VALIDATOR_VERSION}`, and wget gets the URL quoted. Behaviour change: any other `$...` text in a custom download URL is no longer expanded.
- **Quoting single values:** `--version "$FIRELY_TERMINAL_VERSION"`, `"email=..." "password=..."`, `"$USER"`, `"fsh-sushi@$SUSHI_VERSION"`, `rm -rf "$fhirCacheLocation"` and `-version "$FHIR_VERSION"`. The optional QC rules path uses `${VAR:+"$VAR"}` so it drops out entirely when empty.
- **EXPECTED_FAILS checks:** now `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w -q ...`.
- **List inputs** (SUSHI_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES, JAVA_VALIDATION_OPTIONS and the IG dependency list from jq): each is turned into a bash array with an unquoted assignment and expanded as `"${arr[@]}"`. This splits on whitespace and globs exactly as upstream did, but nothing in the value is evaluated.
- **`-ig` parameters:** now built as array elements (`-ig "$path"`) instead of a space-joined string. File globs keep a quoted prefix (`"$GITHUB_WORKSPACE/$p"*.json`).
- **github-env-injection:** fhirVersion has newlines and carriage returns stripped with `printf '%s' | tr -d '\n\r'` before it is written to `$GITHUB_ENV`.

