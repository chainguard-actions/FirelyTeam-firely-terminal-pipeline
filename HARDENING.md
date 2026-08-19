<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.23** was hardened automatically. 4 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference `actions/checkout@v4` using a mutable version tag instead of a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if the tag is moved. Failing references: `uses: actions/checkout@v4` in all three workflow files.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and none of the individual jobs define job-level `permissions:` blocks. Without explicit permissions, workflows inherit the default (potentially write) repository token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml expand input-derived environment variables without double-quoting, violating rule (b). An attacker-controlled value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) could cause command injection.

- **Install Firely.Terminal** (~line 113): `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — `$FIRELY_TERMINAL_VERSION` is unquoted (sourced from `inputs.FIRELY_TERMINAL_VERSION`).
- **Simplifier login** (~line 131): `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` — both credential variables are unquoted (sourced from `inputs.SIMPLIFIER_USERNAME` / `inputs.SIMPLIFIER_PASSWORD`).
- **Install fsh-sushi** (~line 432): `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — `$SUSHI_VERSION` is unquoted (sourced from `inputs.SUSHI_VERSION`).
- **Generate conformance resources with SUSHI** (~line 449): `sushi $INPUT_SUSHI_OPTIONS` — `$INPUT_SUSHI_OPTIONS` is unquoted (sourced from `inputs.SUSHI_OPTIONS`).
- **Run Quality Control checks** (~line 510): `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — `$INPUT_PATH_TO_QUALITY_CONTROL_RULES` is unquoted (sourced from `inputs.PATH_TO_QUALITY_CONTROL_RULES`).
- **Download Java Validator** (~line 563): `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` — `$JAVA_VALIDATOR_DOWNLOAD_LOCATION` is unquoted (sourced from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION`).
- **Validate conformance/example resources** (~lines 596, 660): `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` and `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — multiple input-derived variables are unquoted.
- **Re-run SUSHI** (~line 700): `sushi $INPUT_SUSHI_OPTIONS` — same as above.

Locations:

- `action.yml:113`
- `action.yml:131`
- `action.yml:432`
- `action.yml:449`
- `action.yml:510`
- `action.yml:563`
- `action.yml:596`
- `action.yml:660`
- `action.yml:700`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from external/untrusted sources to `$GITHUB_ENV` or `$GITHUB_PATH` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

**In workflow files** — `LATEST_RELEASE` is fetched from an external API (NuGet / GitHub API) via `wget | jq` and written directly to `$GITHUB_ENV` without newline sanitization. A malicious API response containing newlines could inject arbitrary environment variables into subsequent steps:
- `echo "LATEST_RELEASE=$LATEST_RELEASE" >> $GITHUB_ENV` in all three update workflows.

**In action.yml** — `fhirVersion` is extracted from repository files (`package.json` or `sushi-config.yaml`) via `jq`/`yq` and written to `$GITHUB_ENV` without sanitization:
- `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` (~line 237). A maliciously crafted `package.json` or `sushi-config.yaml` in the calling repository could inject environment variables.

Locations:

- `.github/workflows/update_firelyterminal.yml:31`
- `.github/workflows/update_javavalidator.yml:30`
- `.github/workflows/update_sushi.yml:30`
- `action.yml:237`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned `actions/checkout@v4` to full SHA `11d5960a326750d5838078e36cf38b85af677262` in all three workflow files (update_firelyterminal.yml, update_javavalidator.yml, update_sushi.yml).

2. **missing-permissions**: Added `permissions: contents: write / pull-requests: write` to all three workflow files (needed for branch push and PR creation).

3. **script-injection**: Fixed all unquoted variable expansions in action.yml: quoted `$FIRELY_TERMINAL_VERSION` in dotnet install; quoted credentials in `fhir login`; quoted `$SUSHI_VERSION` in npm install; used xargs tokenization for `$INPUT_SUSHI_OPTIONS` (args list) in both SUSHI steps; used `${VAR:+"$VAR"}` for optional single-path `$INPUT_PATH_TO_QUALITY_CONTROL_RULES`; quoted `$JAVA_VALIDATOR_DOWNLOAD_LOCATION` in wget; used xargs tokenization for path lists (`$INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `$PATH_TO_EXAMPLES`) and options list (`$INPUT_JAVA_VALIDATION_OPTIONS`) in Java validator steps; quoted `$FHIR_VERSION` in java commands.

4. **github-env-injection**: Sanitized `LATEST_RELEASE` (from external API) in all three workflow files and `fhirVersion` (from repo files) in action.yml using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to `$GITHUB_ENV`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in hardened/action/action.yml:
1. 'Download Java Validator' step (line ~554): Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with `printf '%s' "$JAVA_VALIDATOR_DOWNLOAD_LOCATION" | envsubst '$JAVA_VALIDATOR_VERSION'`. The eval was used to expand $JAVA_VALIDATOR_VERSION within the download URL; envsubst safely expands only the specified variable without allowing arbitrary command execution from user-controlled input.
2. Three steps using unquoted `echo $INPUT_EXPECTED_FAILS | grep` (lines ~510, ~601, ~680): Added quotes around the variable in all three occurrences to produce `echo "$INPUT_EXPECTED_FAILS" | grep`. This prevents shell metacharacters (`;`, `|`, `&`, etc.) embedded in the user-controlled EXPECTED_FAILS input from being interpreted by the shell.

### Iteration 3

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed 3 github-env-injection findings in workflow files (update_firelyterminal.yml, update_javavalidator.yml, update_sushi.yml) by sanitizing yq-derived version strings with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to GITHUB_ENV. Fixed 2 script-injection findings in action.yml by: (1) converting LOCAL_IG_PARAMETERS and COMBINED_IG_PARAMETERS string variables to properly-quoted bash arrays; (2) tokenizing UNESCPAED_IG_DEPENDENCIES into arrays using xargs with a guard; (3) quoting $p in glob patterns as "${GITHUB_WORKSPACE}/${p}"*.xml so the path variable is quoted but the glob wildcard remains outside quotes for intentional file expansion.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed all 25 script injection vulnerabilities in hardened/action/action.yml. Replaced every `if $INPUT_VAR` and `if $CLOSE_SLICING_FOR_VALIDATION` pattern (including compound `||` and `&&` forms) with proper quoted string comparisons using `[[ "$VAR" == "true" ]]`. Affected steps: 'Check if .NET is installed', 'Check .NET SDK Version', 'Install Firely.Terminal', 'Check Firely Terminal Version', 'Simplifier login', 'Detect FHIR version and determine dependency source' (3 occurrences including `INPUT_SUSHI_USE_CONFIG_DEPENDENCIES` and `INPUT_SUSHI_ENABLED`), 'Validate FHIR version support', 'Set FHIR specification and restore dependencies', 'Check if npm is installed', 'Check npm Version', 'Install fsh-sushi', 'Check SUSHI version', 'Generate conformance resources with SUSHI', 'Pre-processing FHIR profiles (Slicing open-closed)', 'Configure Firely Terminal validator engine', 'Run Quality Control checks', 'Report Success - .NET Validator', 'Remove package cache to enable the Java validator to create snapshots', 'Check if Java is installed', 'Download Java Validator', 'Validate all conformance resources', 'Validate all example resources', and 'Re-run SUSHI to restore conformance resources after slicing modification'.

