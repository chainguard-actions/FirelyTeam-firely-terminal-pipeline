<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.15** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference `actions/checkout@v4` using a mutable tag instead of a pinned 40-character commit SHA. This means the action could be silently updated to a malicious version without any change to the workflow file. Each file should pin to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### permissions (severity: medium)

None of the three workflow files define a `permissions:` key at the top level or at the job level. Without explicit permissions, workflows inherit the default repository permissions (which may be `write-all`), violating the principle of least privilege. Each workflow should declare minimal required permissions (e.g. `contents: write` and `pull-requests: write` for the PR-creation workflows).

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple script injection vulnerabilities in action.yml:

(a) **eval with user-controlled input** — In the 'Download Java Validator' step, `JAVA_VALIDATOR_DOWNLOAD_LOCATION` is set from `${{ inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION }}` and then used as `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. The `eval` call will execute any shell commands embedded in the input value, giving a caller full remote code execution on the runner.

(b) **Unquoted shell variable expansions** — Several `run:` blocks expand input-derived env vars without double-quoting, allowing shell metacharacter injection:
- `sushi $INPUT_SUSHI_OPTIONS` (SUSHI_OPTIONS from inputs)
- `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` (PATH_TO_QUALITY_CONTROL_RULES from inputs)
- `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` (JAVA_VALIDATOR_DOWNLOAD_LOCATION from inputs)
- `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS` (JAVA_VALIDATION_OPTIONS from inputs)

All of these should be double-quoted: `"$INPUT_SUSHI_OPTIONS"`, `"$INPUT_PATH_TO_QUALITY_CONTROL_RULES"`, `"$JAVA_VALIDATOR_DOWNLOAD_LOCATION"`, `"$INPUT_JAVA_VALIDATION_OPTIONS"`.

Locations:

- `action.yml:481`
- `action.yml:484`
- `action.yml:388`
- `action.yml:444`
- `action.yml:446`
- `action.yml:519`
- `action.yml:523`
- `action.yml:527`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings:

1. **unpinned-uses**: Pinned `actions/checkout@v4` to full SHA `34e114876b0b11c390a56381ad16ebd13914f8d5` in all three workflow files (.github/workflows/update_firelyterminal.yml, update_javavalidator.yml, update_sushi.yml).

2. **permissions**: Added top-level `permissions: contents: write / pull-requests: write` to all three workflow files (minimum needed for branch creation and PR creation).

3. **script-injection** in action.yml:
   - Removed dangerous `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` and replaced with safe bash string substitution `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}`
   - Added double-quotes around `$JAVA_VALIDATOR_DOWNLOAD_LOCATION` in the wget call
   - Fixed `sushi $INPUT_SUSHI_OPTIONS` → `sushi ${INPUT_SUSHI_OPTIONS:+"$INPUT_SUSHI_OPTIONS"}` in both SUSHI steps (Generate and Re-run)
   - Fixed `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` → `fhir check ${INPUT_PATH_TO_QUALITY_CONTROL_RULES:+"$INPUT_PATH_TO_QUALITY_CONTROL_RULES"}` (both with and without --fail)
   - Fixed `$INPUT_JAVA_VALIDATION_OPTIONS` in both Java validator invocations by building a bash array with `read -ra java_opts <<< "$INPUT_JAVA_VALIDATION_OPTIONS"` and expanding as `"${java_opts[@]}"`

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed all 7 findings across 4 files:

1. action.yml (github-env-injection, line 247): Added `safe_fhirVersion=$(printf '%s' "$fhirVersion" | tr -d '\n\r')` before writing FHIR_VERSION to $GITHUB_ENV.

2. action.yml (script-injection, line 530): Quoted `$INPUT_EXPECTED_FAILS` → `"$INPUT_EXPECTED_FAILS"` in the echo command in 'Run Quality Control checks'.

3. action.yml (script-injection, lines 600, 607, 619, 623): In 'Validate all conformance resources', quoted `$INPUT_EXPECTED_FAILS` in echo, `$IG_DEPENDENCIES` in echo (for UNESCPAED_IG_DEPENDENCIES), and `$FHIR_VERSION` in java command.

4. action.yml (script-injection, lines 680, 687, 699, 703): Same fixes applied to 'Validate all example resources' step.

5. update_firelyterminal.yml (github-env-injection, lines 27, 34): Added `printf '%s' | tr -d '\n\r'` sanitization for FIRELY_TERMINAL_VERSION and LATEST_RELEASE before writing to $GITHUB_ENV.

6. update_javavalidator.yml (github-env-injection, lines 27, 33): Same sanitization for JAVA_VALIDATOR_VERSION and LATEST_RELEASE.

7. update_sushi.yml (github-env-injection, lines 27, 33): Same sanitization for SUSHI_VERSION and LATEST_RELEASE.

Note: The for-loop word-splitting over $INPUT_PATH_TO_CONFORMANCE_RESOURCES and $PATH_TO_EXAMPLES is intentional (space-separated path lists) and the $UNESCPAED_IG_DEPENDENCIES/$LOCAL_IG_PARAMETERS/$COMBINED_IG_PARAMETERS expansions are intentionally unquoted (they contain multiple -ig flag tokens). The $GITHUB_WORKSPACE/$p*.xml glob patterns require unquoted expansion for glob matching.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all three script-injection findings in hardened/action/action.yml:

1. Line 116 (Install Firely.Terminal): Quoted `$FIRELY_TERMINAL_VERSION` → `"$FIRELY_TERMINAL_VERSION"` in the `dotnet tool install --global Firely.Terminal --version` command.

2. Line 430 (Install fsh-sushi): Quoted `$SUSHI_VERSION` → `"fsh-sushi@$SUSHI_VERSION"` in the `sudo npm install -g` command.

3. Lines 565, 568, 625, 630 (Java validator steps): Replaced unquoted for-loop expansions of `$INPUT_PATH_TO_CONFORMANCE_RESOURCES` and `$PATH_TO_EXAMPLES` with `read -ra` array splitting followed by iteration over `"${array[@]}"`. This prevents glob expansion and word-splitting on attacker-controlled path values. Also quoted the path concatenations in the java validator commands: `$GITHUB_WORKSPACE/$p*.xml` → `"$GITHUB_WORKSPACE/$p"*.xml` (and similarly for .json). The glob wildcards are intentionally left unquoted so they still expand to match files, but the base path prefix is now properly quoted.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in the 'Simplifier login' step of action.yml (line 130). Changed `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` to `fhir login email="$INPUT_SIMPLIFIER_USERNAME" password="$INPUT_SIMPLIFIER_PASSWORD"`. Both variables are already properly sourced via the step's env block from inputs.SIMPLIFIER_USERNAME and inputs.SIMPLIFIER_PASSWORD, so adding double-quotes prevents shell metacharacter injection without any other structural changes needed.

