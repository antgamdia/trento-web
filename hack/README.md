# Developer Tools (`hack/`)
<!--
SPDX-FileCopyrightText: SUSE LLC
SPDX-License-Identifier: Apache-2.0
-->

The `hack/` directory contains development helper scripts and tooling for contributors.
These tools are not required to run Trento Web, but they make common development and debugging tasks faster.

## Contents

| Path | Purpose |
|---|---|
| `api_docs_check.sh` | Generate and lint OpenAPI documentation |
| `dump_scenario_from_k8.sh` | Dump test scenarios from a running Kubernetes deployment |
| `get_version_from_git.sh` | Compute the current development version from git tags |
| `flaky_tests_analysis/` | Local tooling for flaky test detection and analysis |

---

## `api_docs_check.sh`

Generates the OpenAPI JSON spec for each versioned API (`All`, `Unversioned`, `V1`, `V2`) and runs three linters against it:

- **redocly** — structure and reference linting
- **spectral** — style and best-practice linting
- **vacuum** — security and quality linting

### Prerequisites

Install the required tools globally:

```bash
npm i -g @redocly/cli@latest
npm i -g @quobix/vacuum@latest
npm i -g @stoplight/spectral-cli
```

You also need a running Elixir environment with project dependencies installed (`mix deps.get`).

### Usage

Run from the repository root:

```bash
./hack/api_docs_check.sh
```

The script generates a temporary `openapi_{version}.json` file for each API version, runs all linters, and removes the temporary files on exit.
Exit code is non-zero if any linter reports errors.

### When to Run

Run this script before opening a PR that adds or modifies API endpoints or their OpenAPI annotations.
CI validates the OpenAPI spec automatically, but running locally first reduces review round-trips.

---

## `dump_scenario_from_k8.sh`

Dumps the current discovery state and recent discarded events from a running `trento-server-web` pod in a Kubernetes cluster.
The output is a scenario directory compatible with [photofinish](https://github.com/trento-project/photofinish), which can be replayed in E2E tests.

### Prerequisites

- `kubectl` configured and authenticated against the target cluster
- The `trento-server-web` deployment must be running

### Usage

```bash
./hack/dump_scenario_from_k8.sh [OPTIONS]
```

**Options:**

| Flag | Default | Description |
|---|---|---|
| `-n`, `--name` | `current` | Name for the scenario directory |
| `-p`, `--path` | current directory | Directory where the scenario is saved |
| `-d`, `--discarded-event-number` | `100` | Number of discarded discovery events to include |

**Examples:**

```bash
# Dump the current state with default settings
./hack/dump_scenario_from_k8.sh

# Dump a named scenario to /tmp, including 5 discarded events
./hack/dump_scenario_from_k8.sh --name failover --path /tmp --discarded-event-number 5
```

The scenario is saved to `{path}/scenarios/{name}/`.

### When to Use

Use this script when you need to reproduce a specific infrastructure state seen on a real cluster in a local or E2E test environment.
It is especially useful for debugging issues that only appear with real SAP discovery data.

After dumping, place the scenario directory under `test/fixtures/scenarios/` and load it with photofinish in your E2E tests.

---

## `get_version_from_git.sh`

Computes the current development version string based on the latest semver git tag.

### Usage

```bash
./hack/get_version_from_git.sh
```

**Output examples:**

- `1.3.0` — when the current commit is exactly at a release tag
- `1.3.0+git.42.1234567890.abc1234` — when there are 42 commits after the tag, with unix timestamp and short SHA

### When to Use

Useful for scripts and release tooling that need to determine the in-development version.
CI uses this via the `release.yaml` workflow and the `VERSION` file during the release process.

---

## `flaky_tests_analysis/`

Local tooling to reproduce the flaky test detection that runs nightly in CI.
It provides Makefile targets to run a test suite many times, collect JUnit XML reports, and then score each test by its "flip rate" (the percentage of runs where the result changed from the previous run).

See the [full flaky tests analysis README](flaky_tests_analysis/README.md) for detailed instructions.

For a high-level overview of how the CI workflows use this analysis, see the
[CI/CD Pipeline Guide](../guides/Development/ci-cd.adoc).
