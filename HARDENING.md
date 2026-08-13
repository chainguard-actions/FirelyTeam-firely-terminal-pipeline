<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.13** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference `actions/checkout@v4`, which is a mutable tag rather than a pinned 40-character commit SHA. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` key, and none of the individual jobs declare job-level permissions either. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml expand env vars that hold user-controlled inputs without double-quoting, violating rule (b). An attacker-supplied input containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) can break out of the intended command and execute arbitrary code. Affected unquoted expansions include:
- `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` (Simplifier login step)
- `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (Install Firely.Terminal step)
- `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (Install fsh-sushi step)
- `sushi $INPUT_SUSHI_OPTIONS` (Generate conformance resources step)
- `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` (Run Quality Control checks step)
- `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` (Download Java Validator step)
- `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS` (Validate conformance/example resources steps)
- `echo $INPUT_EXPECTED_FAILS | grep -w -q ...` (multiple steps)
- `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` (multiple steps)
All of these should use double-quoted expansions, e.g. `"$INPUT_SUSHI_OPTIONS"`.

Locations:

- `action.yml:148`
- `action.yml:113`
- `action.yml:183`
- `action.yml:199`
- `action.yml:247`
- `action.yml:261`
- `action.yml:310`
- `action.yml:323`
- `action.yml:338`
- `action.yml:360`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262 in all three workflow files (.github/workflows/update_firelyterminal.yml, update_javavalidator.yml, update_sushi.yml).

2. missing-permissions: Added top-level 'permissions: contents: write / pull-requests: write' to all three workflow files (minimum needed for branch push + PR creation).

3. script-injection: Fixed all unquoted variable expansions in action.yml:
   - Quoted fhir login credentials
   - Quoted Firely Terminal version in dotnet install
   - Quoted SUSHI version in npm install
   - Tokenized SUSHI_OPTIONS into bash arrays (two occurrences: Generate and Re-run steps)
   - Used ${VAR:+"$VAR"} for optional PATH_TO_QUALITY_CONTROL_RULES in fhir check
   - Quoted INPUT_EXPECTED_FAILS in echo | grep pipes
   - Quoted JAVA_VALIDATOR_DOWNLOAD_LOCATION in wget
   - Tokenized PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES, JAVA_VALIDATION_OPTIONS, IG_DEPENDENCIES, and LOCAL/COMBINED_IG_PARAMETERS into bash arrays using xargs for proper quote-aware tokenization in both Java validator steps

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two categories of script injection vulnerabilities in action.yml:

1. eval injection in 'Download Java Validator' step: Replaced `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` with safe bash parameter expansion `JAVA_VALIDATOR_DOWNLOAD_LOCATION="${JAVA_VALIDATOR_DOWNLOAD_LOCATION/\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}"`. This performs the same $JAVA_VALIDATOR_VERSION substitution in the URL without executing arbitrary shell code.

2. Unsafe boolean conditions: Replaced all `if $INPUT_*` and `if $CLOSE_SLICING_FOR_VALIDATION` patterns (which execute the variable value as a shell command) with safe string comparisons `if [[ "$VAR" == "true" ]]`. This was applied across 20+ locations throughout the file covering all boolean input variables: INPUT_DOTNET_VALIDATION_ENABLED, INPUT_JAVA_VALIDATION_ENABLED, INPUT_SUSHI_ENABLED, INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED, INPUT_JAVA_SNAPSHOT_ENABLED, INPUT_SUSHI_USE_CONFIG_DEPENDENCIES, and CLOSE_SLICING_FOR_VALIDATION.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed all four github-env-injection findings by adding `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before every write of externally-sourced values to $GITHUB_ENV:

1. action.yml: sanitized `fhirVersion` (extracted from package.json or sushi-config.yaml) before writing FHIR_VERSION to $GITHUB_ENV.

2. .github/workflows/update_firelyterminal.yml: sanitized FIRELY_TERMINAL_VERSION (from yq/action.yml), LATEST_RELEASE (from NuGet API), and FIRELY_TERMINAL_UDPATE_RECOMMENDED before writing to $GITHUB_ENV. Also quoted $GITHUB_ENV references.

3. .github/workflows/update_javavalidator.yml: sanitized JAVA_VALIDATOR_VERSION (from yq/action.yml), LATEST_RELEASE (from GitHub API), and JAVA_UPDATE_RECOMMENDED before writing to $GITHUB_ENV. Also quoted $GITHUB_ENV references.

4. .github/workflows/update_sushi.yml: sanitized SUSHI_VERSION (from yq/action.yml), LATEST_RELEASE (from GitHub API), and SUSHI_UDPATE_RECOMMENDED before writing to $GITHUB_ENV. Also quoted $GITHUB_ENV references.

