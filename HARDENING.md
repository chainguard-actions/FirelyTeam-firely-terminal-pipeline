<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.24** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use `actions/checkout@v4` — a mutable tag reference, not a pinned 40-character commit SHA. An attacker who compromises the actions/checkout repository could push a malicious commit under that tag. Pin to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` block, and no job-level `permissions:` blocks are present either. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Add a minimal `permissions:` block such as `contents: read` at the top level, and grant `contents: write` / `pull-requests: write` only to the job that needs them.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Rule (b): Multiple `run:` steps in action.yml expand env vars that hold caller-controlled `inputs.*` values without double-quoting, allowing shell metacharacter injection.

1. **Install Firely.Terminal** step: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — `$FIRELY_TERMINAL_VERSION` is unquoted (sourced from `inputs.FIRELY_TERMINAL_VERSION`). Fix: use `"$FIRELY_TERMINAL_VERSION"`.

2. **Install fsh-sushi** step: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — `$SUSHI_VERSION` is unquoted (sourced from `inputs.SUSHI_VERSION`). Fix: use `"$SUSHI_VERSION"`.

3. **Generate conformance resources with SUSHI** step: `sushi $INPUT_SUSHI_OPTIONS` — `$INPUT_SUSHI_OPTIONS` is unquoted (sourced from `inputs.SUSHI_OPTIONS`).

4. **Run Quality Control checks** step: `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — both `$INPUT_EXPECTED_FAILS` (from `inputs.EXPECTED_FAILS`) and `$INPUT_PATH_TO_QUALITY_CONTROL_RULES` (from `inputs.PATH_TO_QUALITY_CONTROL_RULES`) are unquoted.

5. **Download Java Validator** step: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` — `$JAVA_VALIDATOR_DOWNLOAD_LOCATION` is unquoted (sourced from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION`). Additionally, `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` is used to expand the URL, which allows shell command injection via the input value.

6. **Validate all conformance resources** step: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `echo $INPUT_EXPECTED_FAILS | grep`, and `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS` — all unquoted (sourced from `inputs.*`).

7. **Validate all example resources** step: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `for p in $PATH_TO_EXAMPLES`, `echo $INPUT_EXPECTED_FAILS | grep`, and `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $COMBINED_IG_PARAMETERS` — all unquoted (sourced from `inputs.*`).

8. **Re-run SUSHI** step: `sushi $INPUT_SUSHI_OPTIONS` — `$INPUT_SUSHI_OPTIONS` is unquoted (sourced from `inputs.SUSHI_OPTIONS`).

Locations:

- `action.yml:107`
- `action.yml:388`
- `action.yml:399`
- `action.yml:461`
- `action.yml:471`
- `action.yml:490`
- `action.yml:540`
- `action.yml:556`
- `action.yml:573`
- `action.yml:620`
- `action.yml:636`
- `action.yml:653`
- `action.yml:700`
- `action.yml:716`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three finding types:

1. unpinned-uses: Pinned actions/checkout to SHA 11d5960a326750d5838078e36cf38b85af677262 (v4) in all three workflow files.

2. missing-permissions: Added top-level `permissions: contents: read` to all three workflow files, and job-level `contents: write` + `pull-requests: write` to the jobs that create PRs.

3. script-injection: Fixed all 8 injection points in action.yml:
   - Quoted $FIRELY_TERMINAL_VERSION in dotnet install command
   - Quoted $SUSHI_VERSION in npm install command (as part of package@version string)
   - Tokenized $INPUT_SUSHI_OPTIONS (list input) via xargs into array in both SUSHI run steps
   - Quoted $INPUT_EXPECTED_FAILS in all echo|grep pipe commands
   - Used ${VAR:+"$VAR"} for optional $INPUT_PATH_TO_QUALITY_CONTROL_RULES argument
   - Replaced dangerous `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash parameter substitution to expand $JAVA_VALIDATOR_VERSION in the URL
   - Quoted $RESOLVED_DOWNLOAD_LOCATION in wget command
   - In both Java validator steps, tokenized list inputs ($INPUT_PATH_TO_CONFORMANCE_RESOURCES, $PATH_TO_EXAMPLES, $INPUT_JAVA_VALIDATION_OPTIONS, $UNESCPAED_IG_DEPENDENCIES) via xargs into arrays and expanded safely with "${array[@]}" syntax

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in all three workflow files:
1. hardened/action/.github/workflows/update_firelyterminal.yml: Sanitized FIRELY_TERMINAL_VERSION (from yq) and LATEST_RELEASE (from NuGet API via wget|jq) using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV. Also quoted $GITHUB_ENV path.
2. hardened/action/.github/workflows/update_javavalidator.yml: Sanitized JAVA_VALIDATOR_VERSION (from yq) and LATEST_RELEASE (from GitHub API via wget|jq) using the same pattern.
3. hardened/action/.github/workflows/update_sushi.yml: Sanitized SUSHI_VERSION (from yq) and LATEST_RELEASE (from GitHub API via wget|jq) using the same pattern.
In each case, the raw value is captured into a local variable, then a sanitized copy is produced via printf+tr, and only the sanitized copy is written to $GITHUB_ENV.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 23+ script injection vulnerabilities in hardened/action/action.yml where boolean inputs (DOTNET_VALIDATION_ENABLED, JAVA_VALIDATION_ENABLED, SUSHI_ENABLED, SUSHI_USE_CONFIG_DEPENDENCIES, TERMINOLOGY_SERVICE_BFARM_ENABLED, JAVA_SNAPSHOT_ENABLED, CLOSE_SLICING_FOR_VALIDATION) were used as bare shell commands via patterns like `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. All occurrences were replaced with safe string comparisons: `[ "$INPUT_..." = "true" ]`. This prevents command injection where a calling workflow could pass arbitrary shell commands instead of the expected `true`/`false` values.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:
1. script-injection (line 155): Added double-quotes around variable expansions in the 'fhir login' command: changed `email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` to `email="$INPUT_SIMPLIFIER_USERNAME" password="$INPUT_SIMPLIFIER_PASSWORD"`. This prevents shell metacharacters in credentials from enabling command injection.
2. github-env-injection (line 260): Added sanitization of the `fhirVersion` value before writing to $GITHUB_ENV. Added `safe_fhirVersion=$(printf '%s' "$fhirVersion" | tr -d '\n\r')` and changed the echo to use `$safe_fhirVersion`. This prevents a malicious repository file (package.json or sushi-config.yaml) from embedding newlines to inject arbitrary environment variables into subsequent workflow steps.

