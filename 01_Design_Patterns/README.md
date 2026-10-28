# Design Patterns

The Design Patterns guide is maintained in
[mxagar/design_patterns_notes](https://github.com/mxagar/design_patterns_notes)
and included here as the [`design_patterns_notes`](./design_patterns_notes/)
Git submodule.

See the [root README](../README.md#cloning-with-submodules) for instructions on
cloning this guide with its submodules or initializing them after cloning.

## Installation

### Python

Install [uv](https://docs.astral.sh/uv/getting-started/installation/), then run
these commands from the root of this repository:

```bash
git submodule update --init --recursive
cd 01_Design_Patterns/design_patterns_notes
uv sync --locked
```

The uv project files (`pyproject.toml`, `.python-version`, and `uv.lock`) live
inside the submodule. Run all following commands from that directory.
`uv sync --locked` creates `.venv` there and installs the locked dependencies,
including the default `dev` group. Python 3.13 is selected by `.python-version`;
uv can download it if needed. To create the environment separately, run
`uv venv`, then `uv sync --locked` to install dependencies.

The 74 Python scripts and four notebooks use only standard-library imports.
JupyterLab and ipykernel are included as development dependencies for working
with notebooks. For scripts alone, use `uv sync --locked --no-dev`.

Run examples and notebooks without activating the environment:

```bash
uv run python 01_SOLID/ocp.py
uv run jupyter lab .
```

Alternatively, activate it on Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Or on macOS/Linux with bash or zsh:

```bash
source .venv/bin/activate
```

Then use `python` or `jupyter` directly; run `deactivate` to leave the environment.
In your editor, select the Python interpreter inside
`01_Design_Patterns/design_patterns_notes/.venv` as the notebook kernel.

Use `uv add <package>` for runtime dependencies or `uv add --dev <package>` for
development tools. Commit the updated `pyproject.toml` and `uv.lock` inside the
submodule, then update the submodule commit recorded in this repository.
Keep `.venv` out of Git. After pulling dependency changes, run `uv sync --locked`
again. See the [uv project guide](https://docs.astral.sh/uv/guides/projects/).
