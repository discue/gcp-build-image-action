# AGENTS.md

## Overview
**GCP Build Image Action** is a composite GitHub Action that builds and pushes container images to Google Cloud using Cloud Build (`gcloud builds submit`).

### Repository Structure
- `action.yml` — Main composite GitHub Action definition (`runs.using: "composite"`, `shell: bash`).
- `README.md` — User-facing documentation, prerequisites, and input/output reference.
- `.github/workflows/` — CI and automation workflows:
  - `test.yml` — Validates `action.yml` YAML syntax, required metadata fields, and required `README.md` sections.
  - `example.yml` — Manual `workflow_dispatch` dry-run workflow demonstrating action usage against `./example`.
  - `release.yml` — Automated versioning and changelog generation via `googleapis/release-please-action@v4`.
  - `stale.yml` — Automated stale issue and PR management.
- `example/` — Standalone Node.js (Express 5) sample application used to test and demonstrate image builds (see `example/AGENTS.md` for subproject-specific instructions).

## Setup commands
- Root action requires no build step or package installation; it is a pure YAML composite action.
- Dependencies for local validation:
  - `python3` with `PyYAML` (`import yaml`) for YAML syntax checks.
  - `bash` / `grep` for metadata and documentation checks.
  - Node.js `>=18` and `npm` (only needed when working inside `example/`).

## Targeted Workflows & Smallest-Subset Verification
Always run the smallest validation subset matching the files you modified. Never run unrelated checks or attempt live GCP builds in local/CI environments without credentials.

### 1. When modifying `action.yml`, `.github/workflows/**`, or `README.md`
Run only the fast YAML and metadata checks (completes in `<1s`):
- Validate `action.yml` YAML syntax:
  ```bash
  python3 -c "import yaml; yaml.safe_load(open('action.yml'))"
  ```
- Validate required `action.yml` fields (`name:`, `description:`, `runs:`):
  ```bash
  grep -q "name:" action.yml && grep -q "description:" action.yml && grep -q "runs:" action.yml
  ```
- Validate modified workflow YAML files (if `.github/workflows/*.yml` was touched):
  ```bash
  python3 -c "import yaml; yaml.safe_load(open('.github/workflows/test.yml'))"
  ```
- Validate required `README.md` sections (if `README.md` was touched):
  ```bash
  grep -q "## Usage" README.md && grep -q "## Inputs" README.md
  ```
- **Do NOT run**: `npm install` or `npm start` inside `example/` when only root action or workflow files change.
- **Do NOT run**: `gcloud builds submit` locally (requires live Google Cloud authentication, Cloud Build API, and staging buckets).

### 2. When modifying `example/**`
Follow the scoped instructions in `example/AGENTS.md`:
- Run `npm --prefix example test` and verify the Express server endpoints (`/` and `/health`).
- **Do NOT run**: Root workflow validations unless `.github/workflows/example.yml` or `action.yml` was also changed.

### 3. Cross-Boundary / Contract Changes
- `action.yml` is the source of truth for action inputs (`image`, `region`, `impersonate`, `build_service_account`, `quiet`, `source`, `builder`, `staging_bucket_name`, `staging_bucket_dir`, `substitutions`) and outputs (`image-url`).
- If you add, rename, or remove an input or output in `action.yml`, update `README.md` (`## Inputs` / `## Outputs` and usage examples) and `.github/workflows/example.yml` if applicable, then run the root YAML and `README.md` checks.

## Code style & conventions
- **Composite Action Bash Script (`action.yml`)**:
  - Pass GitHub Action inputs via step `env:` (`INPUT_*`) and initialize upper-snake-case `*_INPUT` variables at the top of the `run:` block.
  - Construct individual CLI flag variables (`QUIET_FLAG`, `IMPERSONATE_FLAG`, `BUILD_FLAG`, `GCS_BUCKET_FLAG`, `BUILD_SERVICE_ACCOUNT_FLAG`, `SUBSTITUTIONS_FLAG`) conditionally using `[[ ... ]]` and append them to the `ARGS` array (avoiding `eval`).
  - Write step outputs to `"$GITHUB_OUTPUT"` (e.g., `echo "image-url=${IMAGE_INPUT}" >> "$GITHUB_OUTPUT"`).
- **YAML Formatting**:
  - Use 2-space indentation across `action.yml` and `.github/workflows/*.yml`.
  - Quote strings containing colons, interpolations (`${{ ... }}`), or empty defaults (`""`).

## Commit & PR instructions
- **Strict Conventional Commits**: Every commit message and PR title **must** follow the [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification because `release-please` depends on it to generate releases and update `CHANGELOG.md`:
  ```text
  <type>[optional scope]: <description>
  ```
  - Allowed `<type>` values: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
  - `<description>` must be lowercase, imperative mood, and have **no trailing period** (e.g., `feat: allow passing gcloud build service account email`, `chore: add config flag`).
  - Indicate breaking changes with `!` after type/scope (e.g., `feat!: remove deprecated input`) or a `BREAKING CHANGE:` footer.
- Run the relevant scoped validation commands above before committing.
