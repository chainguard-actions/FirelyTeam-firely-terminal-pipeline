<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.25

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.25** was hardened automatically. 3 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): Multiple unquoted shell variable expansions of env vars holding workflow-controllable inputs in action.yml run: blocks.

1. 'Generate conformance resources with SUSHI' step: `sushi $INPUT_SUSHI_OPTIONS` — INPUT_SUSHI_OPTIONS holds inputs.SUSHI_OPTIONS and is expanded unquoted, allowing shell metacharacter injection.

2. 'Run Quality Control checks' step: `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` / `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — both INPUT_EXPECTED_FAILS (inputs.EXPECTED_FAILS) and INPUT_PATH_TO_QUALITY_CONTROL_RULES (inputs.PATH_TO_QUALITY_CONTROL_RULES) are unquoted.

3. 'Download Java Validator' step: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — eval is called with an env var holding inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION, enabling arbitrary command execution. Additionally `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` is unquoted.

4. 'Re-run SUSHI to restore conformance resources after slicing modification' step: `sushi $INPUT_SUSHI_OPTIONS` — same unquoted expansion as finding #1.

Locations:

- `action.yml:430`
- `action.yml:500`
- `action.yml:502`
- `action.yml:503`
- `action.yml:540`
- `action.yml:543`
- `action.yml:690`

### unpinned-uses (severity: high)

All three workflow files reference actions/checkout@v4 — a mutable tag rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling supply-chain attacks.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and none of the jobs within them define job-level `permissions:` blocks. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write for contents), violating the principle of least privilege.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings:

1. script-injection (action.yml):
   - 'Generate conformance resources with SUSHI': replaced unquoted `sushi $INPUT_SUSHI_OPTIONS` with xargs-based array tokenization to safely handle the options list
   - 'Run Quality Control checks': quoted `echo "$INPUT_EXPECTED_FAILS"` and used `${VAR:+"$VAR"}` for the optional PATH_TO_QUALITY_CONTROL_RULES path argument
   - 'Download Java Validator': replaced dangerous `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash string substitution `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}` (handles the default URL template), and quoted the wget URL argument
   - 'Re-run SUSHI to restore conformance resources': same xargs-based array fix as the first SUSHI step

2. unpinned-uses: Pinned all three `actions/checkout@v4` references to full commit SHA `11d5960a326750d5838078e36cf38b85af677262 # v4` in update_firelyterminal.yml, update_javavalidator.yml, and update_sushi.yml

3. missing-permissions: Added top-level `permissions: contents: write` and `pull-requests: write` blocks to all three workflow files (these workflows push branches and create PRs, requiring these permissions)

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted variable expansions in action.yml:
1. Quoted $FIRELY_TERMINAL_VERSION in 'dotnet tool install --global Firely.Terminal --version "$FIRELY_TERMINAL_VERSION"'
2. Quoted $SUSHI_VERSION in 'sudo npm install -g "fsh-sushi@$SUSHI_VERSION"'
3. Replaced 'for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES' with xargs-based tokenization into a bash array (conformance_paths) in both Java validator steps
4. Replaced 'for p in $PATH_TO_EXAMPLES' with xargs-based tokenization into a bash array (example_paths)
5. Quoted $INPUT_EXPECTED_FAILS in 'echo "$INPUT_EXPECTED_FAILS" | grep -w -q ...' in both validator steps
6. Replaced unquoted $INPUT_JAVA_VALIDATION_OPTIONS with xargs tokenization into java_opts array
7. Replaced unquoted $UNESCPAED_IG_DEPENDENCIES with xargs tokenization into ig_dep_args array
8. Changed LOCAL_IG_PARAMETERS and COMBINED_IG_PARAMETERS from unquoted strings to bash arrays with proper quoting
9. Quoted $FHIR_VERSION as "$FHIR_VERSION" in java validator -version argument
10. Properly quoted $GITHUB_WORKSPACE/$p glob patterns as "$GITHUB_WORKSPACE/$p"*.xml and "$GITHUB_WORKSPACE/$p"*.json

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Simplifier login' step at line 152 of action.yml. The `fhir login` command used `$INPUT_SIMPLIFIER_USERNAME` and `$INPUT_SIMPLIFIER_PASSWORD` unquoted, which could allow shell metacharacter injection. Added double quotes around both variable expansions: `fhir login email="$INPUT_SIMPLIFIER_USERNAME" password="$INPUT_SIMPLIFIER_PASSWORD"`. The variables were already correctly placed in the `env:` block, so only the quoting fix was needed.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed 25 instances of script injection in hardened/action/action.yml where boolean input values (DOTNET_VALIDATION_ENABLED, JAVA_VALIDATION_ENABLED, SUSHI_ENABLED, SUSHI_USE_CONFIG_DEPENDENCIES, TERMINOLOGY_SERVICE_BFARM_ENABLED, JAVA_SNAPSHOT_ENABLED, CLOSE_SLICING_FOR_VALIDATION) were used bare as shell commands in the form `if $INPUT_VARNAME; then`. All occurrences have been replaced with safe string comparisons using `if [[ "$INPUT_VARNAME" == 'true' ]]; then`. Compound conditions with `||` and `&&` were also properly converted. This prevents an attacker from passing values like `true; malicious_command` to achieve arbitrary command execution.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in all three workflow files:
1. update_firelyterminal.yml (line 33): Added `safe=$(printf '%s' "$LATEST_RELEASE" | tr -d '\n\r')` and changed the GITHUB_ENV write to use `$safe` instead of `$LATEST_RELEASE`.
2. update_javavalidator.yml (line 32): Same fix applied for the Java validator LATEST_RELEASE.
3. update_sushi.yml (line 31): Same fix applied for the SUSHI LATEST_RELEASE.
In all cases, the external API response (NuGet registry or GitHub API) is now sanitized by stripping newline and carriage return characters before writing to $GITHUB_ENV, preventing environment variable injection attacks.

