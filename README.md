# renovate-poetry-lowerbound-repro #35785

## Current behavior

Renovate creates a Merge Request that:

1. Updates `poetry.lock` to use `torch` version `2.7.0` (correctly applying the security fix).
2. Changes the `pyproject.toml` constraint for `torch` to `<2.7.1` (or similar, like `^2.6.0`), effectively removing the original `^2.3.0` lower bound string.

**Resulting `pyproject.toml` change:**

```diff
 [tool.poetry.dependencies]
 python = "^3.10"
 torch = { version = "<2.7.1" }
```

## Expected behavior

Renovate should create a Merge Request that:

1. Updates `poetry.lock` to use `torch` version `2.7.0`.
2. Modifies the `pyproject.toml` constraint to allow the fix **while preserving the original lower bound string**.

**Desired `pyproject.toml` change:**

```diff
 [tool.poetry.dependencies]
 python = "^3.10"
 torch = { version = "^2.3.0,<2.7.1" }
```

*(Note: The exact upper bound might vary, but the key is preserving the `^2.3.0` part).*

## Link to the Renovate issue or Discussion

[Link to the discussion](https://github.com/renovatebot/renovate/discussions/35785)
