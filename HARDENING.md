<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.12** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use `actions/checkout@v4` — a mutable tag reference, not a pinned 40-character commit SHA. An attacker who compromises the upstream action repository could push malicious code to that tag. Pin to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` block, and no job-level `permissions:` blocks are present either. Without explicit permissions the workflow runs with the default (potentially write-all) token permissions. Add a minimal `permissions:` block (e.g. `contents: read`) at the top level of each workflow.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple `run:` steps in action.yml expand user-controlled input values as unquoted shell variables (rule b), allowing shell metacharacter injection. A calling workflow can supply values containing `;`, `|`, `$(...)`, etc. to execute arbitrary commands. Affected patterns:

1. `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — `FIRELY_TERMINAL_VERSION` comes from `inputs.FIRELY_TERMINAL_VERSION` (unquoted).
2. `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` — both variables come from `inputs.SIMPLIFIER_USERNAME` / `inputs.SIMPLIFIER_PASSWORD` (unquoted).
3. `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — `SUSHI_VERSION` comes from `inputs.SUSHI_VERSION` (unquoted).
4. `sushi $INPUT_SUSHI_OPTIONS` — `INPUT_SUSHI_OPTIONS` comes from `inputs.SUSHI_OPTIONS` (unquoted).
5. `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — `INPUT_PATH_TO_CONFORMANCE_RESOURCES` comes from `inputs.PATH_TO_CONFORMANCE_RESOURCES` (unquoted word-split loop).
6. `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` / `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — from `inputs.PATH_TO_QUALITY_CONTROL_RULES` (unquoted).
7. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` — from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION` (unquoted).
8. `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` — from `inputs.JAVA_VALIDATION_OPTIONS` (unquoted, appears in two steps).
9. `echo $INPUT_EXPECTED_FAILS | grep -w -q ...` — from `inputs.EXPECTED_FAILS` (unquoted, appears in three steps).
10. `for p in $PATH_TO_EXAMPLES` — from `inputs.PATH_TO_EXAMPLES` (unquoted word-split loop).

All of these must be double-quoted: `"$VAR"` or `"${VAR}"`. For optional multi-word options that must be word-split (e.g. SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS), consider using a bash array or `eval` with strict input validation.

Locations:

- `action.yml:108`
- `action.yml:131`
- `action.yml:132`
- `action.yml:388`
- `action.yml:392`
- `action.yml:418`
- `action.yml:432`
- `action.yml:476`
- `action.yml:495`
- `action.yml:499`
- `action.yml:543`
- `action.yml:547`
- `action.yml:596`
- `action.yml:600`
- `action.yml:638`
- `action.yml:642`
- `action.yml:680`
- `action.yml:684`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:

1. **unpinned-uses**: Pinned `actions/checkout@v4` to full SHA `11d5960a326750d5838078e36cf38b85af677262` in all three workflow files, preserving `# v4` comment.

2. **missing-permissions**: Added `permissions: contents: write / pull-requests: write` to all three workflow files. These workflows create branches and open PRs, so both permissions are needed.

3. **script-injection**: Fixed all 10 categories of unquoted variable expansions in action.yml:
   - `$FIRELY_TERMINAL_VERSION` → `"$FIRELY_TERMINAL_VERSION"` in dotnet tool install
   - `$INPUT_SIMPLIFIER_USERNAME`/`$INPUT_SIMPLIFIER_PASSWORD` → quoted as `"email=$VAR"` / `"password=$VAR"` in fhir login
   - `$SUSHI_VERSION` → `"fsh-sushi@$SUSHI_VERSION"` in npm install
   - `$INPUT_SUSHI_OPTIONS` (list) → xargs tokenization into array in both SUSHI run steps
   - `$INPUT_EXPECTED_FAILS` → `"$INPUT_EXPECTED_FAILS"` in all three grep checks
   - `$INPUT_PATH_TO_QUALITY_CONTROL_RULES` (optional single path) → `${VAR:+"$VAR"}` in fhir check commands
   - `$JAVA_VALIDATOR_DOWNLOAD_LOCATION` → `"$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` in wget
   - `$INPUT_JAVA_VALIDATION_OPTIONS` (list) → xargs tokenization into array in both Java validation steps
   - `$INPUT_PATH_TO_CONFORMANCE_RESOURCES` (list) → xargs tokenization into array in both Java validation steps
   - `$PATH_TO_EXAMPLES` (list) → xargs tokenization into array in examples validation step

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` with `JAVA_VALIDATOR_DOWNLOAD_LOCATION="${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}"` in the 'Download Java Validator' step. The bash parameter substitution safely replaces the literal string `$JAVA_VALIDATOR_VERSION` in the URL with the actual version value without using `eval`, eliminating the command injection risk from user-controlled input.

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 18+ occurrences of unsafe `if $INPUT_*` boolean shell execution patterns in action.yml by replacing them with safe quoted comparisons `[[ "$VAR" == 'true' ]]`. Fixed github-env-injection in all three update workflow files (update_firelyterminal.yml, update_javavalidator.yml, update_sushi.yml) by sanitizing the LATEST_RELEASE variable with `printf '%s' "$LATEST_RELEASE" | tr -d '\n\r'` before writing to $GITHUB_ENV.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in three workflow files:
1. `.github/workflows/update_firelyterminal.yml` (line 31): Added `safe=$(printf '%s' "$FIRELY_TERMINAL_VERSION" | tr -d '\n\r')` and changed the echo to write `$safe` to `$GITHUB_ENV`.
2. `.github/workflows/update_javavalidator.yml` (line 31): Added `safe=$(printf '%s' "$JAVA_VALIDATOR_VERSION" | tr -d '\n\r')` and changed the echo to write `$safe` to `$GITHUB_ENV`.
3. `.github/workflows/update_sushi.yml` (line 31): Added `safe=$(printf '%s' "$SUSHI_VERSION" | tr -d '\n\r')` and changed the echo to write `$safe` to `$GITHUB_ENV`.

In all cases, the pattern matches the already-correct sanitization applied to the LATEST_RELEASE variable in the same files. Also quoted `$GITHUB_ENV` for consistency.

