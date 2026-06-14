# AGENTS.md — vg-number

High-signal context for working in this repo.

## Environment

- Python **3.13** is required (`uv.lock` declares `requires-python = ">=3.13"`). The system `python3` is 3.9.6, so use the repo virtualenv or `uv run`.
- The local `.venv` already contains Python 3.13 and NumPy.
- Package manager: **uv** (`/opt/homebrew/bin/uv`).
- Install editable: `uv pip install -e .` (or `.venv/bin/pip install -e .`).

## Package layout (critical)

- The installable packages are top-level directories: `matrix`, `noise`, `pathfinding`, `utils`.
- They are **NOT** under a `vg_number` namespace, despite what `README.md` shows.
- Correct imports:
  - `from matrix import Matrix2D, MatrixFilters, BlurType`
  - `from pathfinding import astar_grid_2d, Manhattan`
  - `from noise.generators import FastNoise2D`
  - `from utils.math_utils import lerp`
- Build backend is `setuptools.build_meta`; `pyproject.toml` finds packages with `include = ["matrix*", "noise*", "pathfinding*", "utils*"]`.

## Current import/runtime gotchas

- `from matrix import Matrix2D` is currently broken because `matrix/spline.py` imports `scipy.interpolate.CubicSpline`, but **scipy is not declared in `pyproject.toml`** and is not installed.
- `from noise.generators import FastNoise2D` is currently broken because `noise/generators/domain_warp.py` imports from the stale package name `virigir_math_utilities.noise.core`.
- `pathfinding` imports and runs correctly by itself.
- `utils.math_utils` only exports `fade` and `lerp`; `clamp` (shown in README examples) is **not implemented**.

## Running code / tests

- No test runner, linter, formatter, type checker, or CI is configured.
- `pytest` is not installed in `.venv` and is not a project dependency.
- `pathfinding/test_pathfinding.py` imports `virigir_math_utilities.pathfinding`, so it will not run until those imports are fixed.
- `pathfinding/simple_test.py` uses relative imports; run it as a module from the repo root:
  ```bash
  .venv/bin/python -m pathfinding.simple_test
  ```
  Running it directly as `python pathfinding/simple_test.py` fails with `attempted relative import with no known parent package`.

## Dependency drift to watch

- Declared runtime dependency is only `numpy`.
- Actual code also needs `scipy` for `matrix/spline.py`.
- Several source files and docstrings still reference the old names `virigir_math_utilities.*` and `vg_number.*`; treat these as stale.
