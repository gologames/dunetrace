# Contributing to Dunetrace

Thanks for helping make agent monitoring better. This guide gets you from a fresh
clone to a merged PR.

## Where to start

- **[Good first issues](https://github.com/dunetrace/dunetrace/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)** —
  most are "add one structural detector," which is a clean, self-contained change.
- **Adding a detector?** Follow the step-by-step
  [Adding a detector guide](docs/contributing/adding-a-detector.md) — it walks the
  full registration path (the one thing newcomers usually miss is regenerating
  the `FailureType` enums, covered there).
- Comment on the issue to get it assigned before you start.

For anything larger (a new integration, an architecture change), open an issue
first so we can agree on the approach.

## Development setup

Requires **[mise](https://mise.jdx.dev/getting-started.html#installing-mise-cli)** with [shell integration](https://mise.jdx.dev/shell-setup.html) configured.
Tool versions are pinned in `mise.toml` and `mise.lock`.

```bash
git clone https://github.com/dunetrace/dunetrace
cd dunetrace

mise install --locked  # first-time setup; afterwards use `task mise:install`

task uv:install
npm --prefix packages/sdk-ts ci
```

## Running services

Requires [Docker](https://docs.docker.com/get-started/get-docker/) + [Docker Compose](https://docs.docker.com/compose/install/).

```bash
cp .env.example .env
task dev:up
```

## Running tests

```bash
task test           # non-DB suites, sequentially; stops on the first failure
task test:detector  # one service (also: test:api / test:explainer / test:alerts / test:ingest)
task test:sdk-py    # Python SDK
task test:sdk-ts    # TypeScript SDK
```

Database checks:

```bash
task db:check-ownership
task db:test-schema         # running Postgres; DATABASE_URL required
task db:test-schema-docker  # temporary Postgres in Docker
```

## Commit messages

Use scoped commit messages:

```text
<scope>: <description>
```

- Scope identifies the affected component, e.g. `ingest`, `api`, `detector`,
  `sdk-py`, `sdk-ts`, `dashboard`, or `docs`.
- Keep the description short, specific and imperative.
- Merge and revert commits may use Git's default format.

Examples:

```text
ingest: reject oversized payloads
sdk-py: retry failed requests
docs: update installation instructions
```

## Before you push

Install the git hooks once — they run ruff, mypy, the docs-consistency check,
and the test suites on the files you touched:

```bash
uv run --locked pre-commit install
uv run --locked pre-commit run --all-files   # to check everything up front
```

CI runs the same checks. The most common first-PR failure is the **generated
`FailureType` enum** check — if you add a detector, add its member to
`packages/schemas-py/dunetrace_schemas/enum_source.py` (the single source of
truth) and run `python scripts/gen_enums.py`, which rewrites both
`packages/sdk-py/dunetrace/_enums.py` and
`packages/schemas-py/dunetrace_schemas/enums.py`. Never edit those two by hand —
`python scripts/gen_enums.py --check` fails CI if they are stale. The detector
guide covers this.

## Opening the PR

1. Fork, branch off `main`.
2. Make the change, add tests, run `task test` + `uv run --locked pre-commit run --all-files`.
3. Push and open a PR describing what and why. Link the issue (`Fixes #NN`).
4. New detectors ship **shadow by default** (they don't alert until validated on
   real traffic) — the guide explains how.

## Code style

- Python: `ruff` handles formatting and linting (config in `pyproject.toml`).
- Match the surrounding code — comment density, naming, and idiom.
- Keep the SDK **zero-runtime-dependency** (`packages/sdk-py` has no required
  deps; anything you import at runtime must be optional and guarded).
