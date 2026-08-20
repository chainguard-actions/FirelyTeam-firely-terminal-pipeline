<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.14** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use `actions/checkout@v4` — a mutable tag reference, not a pinned 40-character commit SHA. An attacker who compromises the `actions/checkout` repository could push a malicious commit under the same tag. All three files must pin to a full SHA (e.g., `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Locations:

- `.github/workflows/update_firelyterminal.yml:17`
- `.github/workflows/update_javavalidator.yml:17`
- `.github/workflows/update_sushi.yml:17`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` block, and no job-level `permissions:` blocks are present either. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Each workflow should declare minimal permissions such as `permissions: contents: write` and `pull-requests: write` only as needed.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml expand env vars sourced from caller-controlled `inputs.*` without double-quoting, violating rule (b). An attacker-controlled input value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) can break out of the intended command and execute arbitrary shell code.

Violations found:
1. `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — `$FIRELY_TERMINAL_VERSION` (from `inputs.FIRELY_TERMINAL_VERSION`) is unquoted. Fix: `"$FIRELY_TERMINAL_VERSION"`.
2. `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` — both credential vars (from `inputs.SIMPLIFIER_USERNAME` / `inputs.SIMPLIFIER_PASSWORD`) are unquoted.
3. `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — `$SUSHI_VERSION` (from `inputs.SUSHI_VERSION`) is unquoted.
4. `sushi $INPUT_SUSHI_OPTIONS` — `$INPUT_SUSHI_OPTIONS` (from `inputs.SUSHI_OPTIONS`) is unquoted (appears in two steps).
5. `echo $INPUT_EXPECTED_FAILS | grep -w -q ...` — `$INPUT_EXPECTED_FAILS` (from `inputs.EXPECTED_FAILS`) is unquoted (appears in three steps).
6. `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` / `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — `$INPUT_PATH_TO_QUALITY_CONTROL_RULES` (from `inputs.PATH_TO_QUALITY_CONTROL_RULES`) is unquoted.
7. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` — `$JAVA_VALIDATOR_DOWNLOAD_LOCATION` (from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION`) is unquoted.
8. `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — `$INPUT_PATH_TO_CONFORMANCE_RESOURCES` (from `inputs.PATH_TO_CONFORMANCE_RESOURCES`) is unquoted in multiple for-loop headers.
9. `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` — `$INPUT_JAVA_VALIDATION_OPTIONS` (from `inputs.JAVA_VALIDATION_OPTIONS`) is unquoted in multiple java invocations.

Locations:

- `action.yml:121`
- `action.yml:143`
- `action.yml:449`
- `action.yml:458`
- `action.yml:530`
- `action.yml:532`
- `action.yml:534`
- `action.yml:581`
- `action.yml:617`
- `action.yml:621`
- `action.yml:635`
- `action.yml:637`
- `action.yml:673`
- `action.yml:680`
- `action.yml:697`
- `action.yml:699`
- `action.yml:730`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three finding types:

1. unpinned-uses: Pinned actions/checkout to full SHA 11d5960a326750d5838078e36cf38b85af677262 (v4) in all three workflow files.

2. missing-permissions: Added top-level `permissions: contents: write` and `pull-requests: write` to all three workflow files (these workflows create branches and PRs).

3. script-injection: Fixed all unquoted variable expansions in action.yml:
   - Quoted $FIRELY_TERMINAL_VERSION in dotnet install command
   - Quoted $INPUT_SIMPLIFIER_USERNAME and $INPUT_SIMPLIFIER_PASSWORD in fhir login
   - Quoted fsh-sushi@$SUSHI_VERSION in npm install
   - Replaced `sushi $INPUT_SUSHI_OPTIONS` with xargs-tokenized array expansion in both SUSHI steps
   - Quoted $INPUT_EXPECTED_FAILS in all three echo|grep patterns
   - Used ${VAR:+"$VAR"} for optional $INPUT_PATH_TO_QUALITY_CONTROL_RULES in fhir check commands
   - Quoted $JAVA_VALIDATOR_DOWNLOAD_LOCATION in wget command
   - Replaced `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` with xargs-tokenized array in both Java validation steps
   - Replaced `$INPUT_JAVA_VALIDATION_OPTIONS` with xargs-tokenized array in both Java validation steps
   - Replaced `for p in $PATH_TO_EXAMPLES` with xargs-tokenized array in examples validation step

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:
1. script-injection (line ~597): Replaced `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` with safe bash string substitution `JAVA_VALIDATOR_DOWNLOAD_LOCATION="${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}"`. This replaces the literal `$JAVA_VALIDATOR_VERSION` placeholder in the URL using bash's built-in pattern substitution, eliminating the eval-based arbitrary command injection vector.
2. github-env-injection (line ~261): Added `safe_fhirVersion=$(printf '%s' "$fhirVersion" | tr -d '\n\r')` before writing to $GITHUB_ENV, and changed the echo to use `$safe_fhirVersion`. This prevents a malicious package.json or sushi-config.yaml from injecting additional environment variables via embedded newlines.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in hardened/action/action.yml:

1. Replaced all `if $INPUT_*` and `if $CLOSE_SLICING_FOR_VALIDATION` boolean-variable-as-command patterns with safe string comparisons (`if [ "$VAR" = "true" ]`). Affected steps: Check if .NET is installed, Check .NET SDK Version, Install Firely.Terminal, Check Firely Terminal Version, Simplifier login, Detect FHIR version (including nested SUSHI_USE_CONFIG_DEPENDENCIES and SUSHI_ENABLED checks), Validate FHIR version support, Set FHIR specification and restore dependencies, Check if npm is installed, Check npm Version, Install fsh-sushi, Check SUSHI version, Generate conformance resources with SUSHI, Pre-processing FHIR profiles, Configure Firely Terminal validator engine, Run Quality Control checks, Report Success - .NET Validator, Remove package cache, Check if Java is installed, Download Java Validator, Validate all conformance resources, Validate all example resources, Re-run SUSHI post-processing.

2. Fixed unquoted list variables in Java validator commands: replaced string-concatenated $LOCAL_IG_PARAMETERS and $COMBINED_IG_PARAMETERS with proper bash arrays (local_ig_params/combined_ig_params); tokenized $UNESCPAED_IG_DEPENDENCIES into ig_dep_args array using xargs/printf pattern; quoted the variable portion of glob paths as "$GITHUB_WORKSPACE/$p"*.xml and "$GITHUB_WORKSPACE/$p"*.json to prevent word splitting while preserving glob expansion.

