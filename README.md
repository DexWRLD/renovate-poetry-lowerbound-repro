# Reproduction for Renovate Discussion #35785

This repository demonstrates an issue where Renovate modifies the lower bound version string in `pyproject.toml` when applying a security update for Poetry dependencies, even when using `rangeStrategy: widen` or `bump`.

## Problem Details

When Renovate detects a security vulnerability in `torch` (locked at `2.5.1` via `poetry.lock`, which satisfies the original `^2.3.0,<2.6.0` constraint in `pyproject.toml`), it correctly identifies that version `2.6.0` (or higher) is required to fix the vulnerability.

However, the generated Merge Request changes the constraint in `pyproject.toml` in a way that removes the original lower bound string (`^2.3.0`). We require this lower bound string to be preserved for project consistency and policy reasons.

## Renovate Configuration

The configuration used is in [`renovate.json`](./renovate.json). Key settings include:

- `extends`: `config:recommended`, `:enableVulnerabilityAlerts` (or `security:only-security-updates`)
- `rangeStrategy`: `widen` (or `bump` - specify the one used during testing)
- `osvVulnerabilityAlerts`: `true`

## Files

- **`pyproject.toml`**: Contains the initial dependency constraint `torch = { version = "^2.3.0,<2.6.0" }`.
- **`poetry.lock`**: Locks `torch` to the vulnerable version `2.5.1` (satisfying the original constraint). Generated via temporary pinning.

## Current behavior

Renovate creates a Merge Request that:

1. Updates `poetry.lock` to use `torch` version `2.6.0` (correctly applying the security fix).
2. Changes the `pyproject.toml` constraint for `torch` to `<2.6.1` (or similar, like `^2.6.0`), effectively removing the original `^2.3.0` lower bound string.

**Resulting `pyproject.toml` change:**

```diff
 [tool.poetry.dependencies]
 python = "^3.10"
-torch = { version = "^2.3.0,<2.6.0" }
+torch = "<2.6.1" # Or similar range starting >= 2.6.0

```

## Expected behavior

Renovate should create a Merge Request that:

1. Updates `poetry.lock` to use `torch` version `2.6.0`.
2. Modifies the `pyproject.toml` constraint to allow the fix **while preserving the original lower bound string**.

**Desired `pyproject.toml` change:**

```diff

 [tool.poetry.dependencies]
 python = "^3.10"
 # Original constraint satisfied by vulnerable 2.5.1
 torch = { version = "^2.3.0,<2.6.0" }

```

*(Note: The exact upper bound might vary, but the key is preserving the `^2.3.0` part).*

## Steps to Reproduce

1. Clone this repository.
2. Ensure you have Docker installed.
3. Export your GitHub Personal Access Token (PAT) with `public_repo` scope (or equivalent fine-grained permissions) to `RENOVATE_TOKEN` environment variable locally:

    ```bash
    export RENOVATE_TOKEN="ghp_YOUR_GITHUB_PAT"
    ```

4. Run Renovate locally against this repository using a command like:

    ```bash
    docker run --rm -e LOG_LEVEL="debug" -e RENOVATE_TOKEN -v "$(pwd):/usr/src/app" -w /usr/src/app renovate/renovate:latest --platform=github --token="${RENOVATE_TOKEN}" your-github-username/your-repo-name
    ```

5. Observe the branch/Pull Request created by Renovate and inspect the changes to:

`pyproject.toml`
