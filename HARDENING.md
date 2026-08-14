<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.22** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use `actions/checkout@v4` — a mutable tag reference, not a pinned 40-character commit SHA. If the tag is moved (e.g. by a supply-chain compromise), the action will silently execute different code. Each file should pin to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/update_firelyterminal.yml:17`
- `.github/workflows/update_javavalidator.yml:17`
- `.github/workflows/update_sushi.yml:17`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` block, and no job within them declares job-level permissions either. Without explicit permissions the GITHUB_TOKEN inherits the repository's default (often broad write) permissions. A minimal `permissions: contents: write` (or read-only where write is not needed) should be declared.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple `run:` steps in action.yml expand user-controlled input variables without double-quoting, violating rule (b). An attacker who controls these inputs can inject shell metacharacters (`;`, `|`, `$(...)`, etc.) to execute arbitrary commands.

(1) **Generate conformance resources with SUSHI** — `sushi $INPUT_SUSHI_OPTIONS` is unquoted; `SUSHI_OPTIONS` is a free-form user input. Should be `sushi "$INPUT_SUSHI_OPTIONS"`.

(2) **Download Java Validator** — `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` passes a user-controlled string through `eval`, allowing arbitrary shell code execution. Additionally `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` is unquoted. The `eval` should be removed and the variable double-quoted.

(3) **Run Quality Control checks** — `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` / `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` are all unquoted expansions of user inputs.

(4) **Validate all conformance resources / Validate all example resources** — `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS` contains multiple unquoted user-controlled variables.

(5) **Install Firely.Terminal** — `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` is unquoted.

(6) **Install fsh-sushi** — `sudo npm install -g fsh-sushi@$SUSHI_VERSION` is unquoted.

Locations:

- `action.yml:393`
- `action.yml:494`
- `action.yml:452`
- `action.yml:133`
- `action.yml:530`
- `action.yml:107`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three finding types:

1. unpinned-uses: Pinned actions/checkout to full SHA 11d5960a326750d5838078e36cf38b85af677262 # v4 in all three workflow files.

2. missing-permissions: Added top-level permissions block (contents: write, pull-requests: write) to all three workflow files. Both permissions are needed since the workflows create branches, commit, push, and open PRs via gh CLI.

3. script-injection (6 locations in action.yml):
   - Install Firely.Terminal: Quoted "$FIRELY_TERMINAL_VERSION"
   - Install fsh-sushi: Quoted "fsh-sushi@$SUSHI_VERSION"
   - Generate conformance resources with SUSHI (both occurrences): Replaced bare `sushi $INPUT_SUSHI_OPTIONS` with xargs tokenization into array `sushi "${sushi_opts[@]}"`
   - Download Java Validator: Removed dangerous `eval echo` and replaced with safe bash parameter substitution; quoted wget URL
   - Run Quality Control checks: Quoted $INPUT_EXPECTED_FAILS in echo pipe; used ${VAR:+"$VAR"} for optional PATH_TO_QUALITY_CONTROL_RULES argument
   - Validate all conformance/example resources: Quoted $INPUT_EXPECTED_FAILS and $IG_DEPENDENCIES in echo pipes; quoted $FHIR_VERSION; tokenized INPUT_JAVA_VALIDATION_OPTIONS into array via xargs

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings in action.yml:

1. Simplifier login (line 116): Quoted the email and password arguments to `fhir login` as `"email=$INPUT_SIMPLIFIER_USERNAME"` and `"password=$INPUT_SIMPLIFIER_PASSWORD"` to prevent shell word-splitting on attacker-controlled values.

2. Conformance resources validation (line 533): Tokenized `$INPUT_PATH_TO_CONFORMANCE_RESOURCES` into a `conformance_paths` bash array using xargs (quote-aware). Changed `LOCAL_IG_PARAMETERS` from a string to a bash array. Tokenized `UNESCPAED_IG_DEPENDENCIES` into an `ig_dep_args` array using xargs. All java invocations now use `"${array[@]}"` expansions. Glob patterns use `"$GITHUB_WORKSPACE/${p}"*.xml` form.

3. Examples validation (line 607): Tokenized both `$INPUT_PATH_TO_CONFORMANCE_RESOURCES` and `$PATH_TO_EXAMPLES` into bash arrays using xargs. Changed `COMBINED_IG_PARAMETERS` from a string to a bash array. Tokenized `UNESCPAED_IG_DEPENDENCIES` into `ig_dep_args` array. All java invocations now use `"${array[@]}"` expansions.

