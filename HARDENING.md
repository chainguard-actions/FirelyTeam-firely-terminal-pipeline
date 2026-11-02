<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.10.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): inputs routed through env: are expanded without double quotes in many composite run: steps, so shell word-splitting, globbing and in some cases command execution apply to values the caller controls. Examples: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` (the input value is run as a command, and the same pattern repeats for INPUT_JAVA_VALIDATION_ENABLED, INPUT_SUSHI_ENABLED, INPUT_SUSHI_USE_CONFIG_DEPENDENCIES, CLOSE_SLICING_FOR_VALIDATION and INPUT_JAVA_SNAPSHOT_ENABLED in nearly every step); `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`; `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`; `sudo npm install -g fsh-sushi@$SUSHI_VERSION`; `sushi $INPUT_SUSHI_OPTIONS` (twice); `echo $INPUT_EXPECTED_FAILS | grep ...`; `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES`; `rm -rf $fhirCacheLocation`; `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`; `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` / `for p in $PATH_TO_EXAMPLES`; and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`. Each expansion should be double-quoted. For optional values, use the guarded form `${VAR:+"$VAR"}`. Boolean inputs should be compared as strings, e.g. `[[ "$VAR" == "true" ]]`, and never run as commands.

Locations:

- `action.yml:90`
- `action.yml:119`
- `action.yml:143`
- `action.yml:500`
- `action.yml:510`
- `action.yml:600`
- `action.yml:630`
- `action.yml:650`
- `action.yml:700`
- `action.yml:780`
- `action.yml:815`

### script-injection (severity: high)

Rule (b) / eval: the 'Download Java Validator' step runs `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Here the caller-supplied input `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION` is passed to eval, so a value like `$(malicious command)` or `; cmd` is executed. Replace eval with plain string substitution, e.g. `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`.

Locations:

- `action.yml:646`

### github-env-injection (severity: high)

The 'Detect FHIR version and determine dependency source' step runs `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`. The value of fhirVersion is not a literal: it is read from repository files (package.json `.fhirVersions[0]` or sushi-config.yaml `.fhirVersion`, possibly base64-decoded). Anyone who can change those files, for example in a pull request, can embed newlines and inject arbitrary environment variables such as LD_PRELOAD or BASH_ENV. Sanitize before the write: `safe=$(printf '%s' "$fhirVersion" | tr -d '\n\r')`.

Locations:

- `action.yml:273`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

All changes are in action.yml. Nothing was run, so these fixes are untested. My first attempt used parallel edits that clashed and corrupted the file; I then rewrote the whole file from the original in one go.

- **Boolean inputs:** each `if $INPUT_X; then` is now a string comparison, `[[ "$X" == "true" ]]`, so the input value is never run as a command. Side effect: a value written any way other than lowercase `true` (for example `True`) now counts as false.
- **eval removed:** the download URL's `$JAVA_VALIDATOR_VERSION` / `${JAVA_VALIDATOR_VERSION}` placeholder is now filled in by plain text substitution instead of `eval echo`. The wget URL is quoted.
- **Single values:** these are now double-quoted: `FIRELY_TERMINAL_VERSION`, the `fhir login` email and password, `fsh-sushi@$SUSHI_VERSION`, `$USER`, the cache path given to `rm -rf`, `FHIR_VERSION` and `FHIR_RESTORE`. `PATH_TO_QUALITY_CONTROL_RULES` uses `${VAR:+"$VAR"}` so an empty value adds no argument. `EXPECTED_FAILS` is passed to grep with `printf '%s\n' "$VAR"`.
- **Lists:** `SUSHI_OPTIONS`, `JAVA_VALIDATION_OPTIONS`, `PATH_TO_CONFORMANCE_RESOURCES`, `PATH_TO_EXAMPLES` and the IG dependency string from package.json are split on whitespace into bash arrays, each behind an empty-value check. This is the same split as before, except that file-name wildcards inside these values are no longer expanded. The `-ig` parameter lists are built as arrays too.
- **File patterns kept:** the Java validator's `*.xml` / `*.json` matching still works, because only the variable part is quoted: `"$GITHUB_WORKSPACE/$p"*.xml`.
- **GITHUB_ENV write:** `fhirVersion` has newlines and carriage returns stripped (`tr -d '\n\r'`) before it is written as `FHIR_VERSION`.

I also quoted a few things the findings didn't list: the supported-versions loop and the ZTS package download URL.

