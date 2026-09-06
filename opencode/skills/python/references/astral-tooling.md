# Astral tooling and FastAPI

Prefer uv, Ruff, and ty for new Python projects. Use this reference when the
repository already uses them or a migration is requested. Preserve established
tooling during scoped maintenance. Resolve versions from project metadata,
lockfiles, CI actions, and toolchain metadata before relying on new flags or rules.

## uv

- Use uv to install Python, create and sync environments, add or remove
  dependencies, maintain `uv.lock`, run commands, build distributions, and test
  built artifacts.
- During a requested migration, make uv reproduce the required environments
  and artifacts before removing superseded package-manager configuration or
  lockfiles. Do not migrate as part of an unrelated code change.
- Commit `uv.lock` for applications. For libraries, test both the locked
  development environment and fresh resolutions across the declared dependency
  range.
- Use `uv run` for project commands and `uvx` for isolated tools that do not
  belong in project dependencies. Use `--locked` in CI and deployment when
  manifest drift must fail; it rejects a missing or outdated lockfile.
  `--frozen` uses the existing lockfile without checking freshness. Use it only
  when that lockfile is intentionally authoritative and freshness was checked
  separately, such as a staged container build without all workspace manifests.
- Pin the uv installer, container image, or setup action. Do not use a floating
  `latest` image in a reproducible deployment.

## Ruff

- Configure Ruff in `pyproject.toml`. Use `ruff format` and `ruff check` when
  Ruff is selected. Remove Black, isort, Flake8, or overlapping tools only during
  a requested migration after reviewing formatting and diagnostic differences.
- Select rules deliberately, set `target-version` from the minimum supported
  Python, review fixes before applying them broadly, and distinguish safe from
  unsafe fixes.
- Run `uv run --locked ruff check .` and `uv run --locked ruff format --check .`
  in local and CI verification.

## ty

- Configure ty under `[tool.ty]` in `pyproject.toml` and run
  `uv run --locked ty check`. Keep rule overrides and suppressions narrow and
  reviewable.
- Prefer ty for new projects and preserve the existing checker during maintenance.
  Python 3.7 through 3.9 can be selected, but ty's bundled standard-library stubs
  do not fully cover them;
  verify questionable diagnostics against the target runtime and documentation.
- Align ty's Python version and environment discovery with `requires-python`, uv,
  and the CI matrix. Verify library typing on the minimum supported Python.

## FastAPI

- Load the `fastapi` skill in addition to the Python skill. Keep framework,
  Starlette, Pydantic, ASGI lifecycle, API-contract, and deployment guidance
  there rather than duplicating it here.
- In projects using uv, manage FastAPI and its extras with uv, run development
  and production commands through `uv run`, and lock the complete resolved stack.
- Before finishing FastAPI work, run the established quality tools, test suite,
  and relevant OpenAPI checks. Use Ruff and ty when they are the selected tools.

## Sources

- uv: `https://docs.astral.sh/uv/`
- Locking and syncing: `https://docs.astral.sh/uv/concepts/projects/sync/`
- uv with FastAPI: `https://docs.astral.sh/uv/guides/integration/fastapi/`
- Ruff: `https://docs.astral.sh/ruff/`
- ty: `https://docs.astral.sh/ty/`
- ty Python-version support: `https://docs.astral.sh/ty/python-version/`
