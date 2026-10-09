# DeepLCodingKatas

## Requirements

This repository uses [uv](https://docs.astral.sh/uv/getting-started/installation/) and [ruff](https://docs.astral.sh/ruff/installation/).

## Python environments and notebook kernels

The repository contains two independent uv projects, each with its own
`pyproject.toml`, `uv.lock`, and `.venv`:

| Project directory | Python | Notebook kernel | Purpose |
| --- | --- | --- | --- |
| Repository root | 3.13 (pinned in `.python-version`) | DeepL Coding Katas | General notebooks, including PVA1 |
| `PVA2/teachable-machine` | 3.11 | Teachable Machine (Python 3.11) | Loading the Teachable Machine export with TensorFlow 2.15.1 |

A kernel is the Python process that executes notebook cells. Registering a kernel
lets Jupyter and Zed find a project's virtual environment. The notebook's selected
kernel determines its Python version and available packages, independently of
where the notebook file is stored or which environment runs JupyterLab.

These projects are intentionally separate: a uv workspace would share a lockfile
and dependency resolution, which is unsuitable for their different requirements.

### Create the environments and register kernels

Run all commands below from the repository root. No environment activation is
needed; `uv run --project` selects the specified project explicitly.

```bash
uv python install 3.13 3.11

# General environment (includes the dev dependencies and ipykernel).
uv sync --project .
uv run --project . python -m ipykernel install --user --name deeplcodingkatas --display-name "DeepL Coding Katas"

# Separate environment for the legacy Teachable Machine model.
uv sync --project PVA2/teachable-machine --python 3.11
uv run --project PVA2/teachable-machine python -m ipykernel install --user --name teachable-machine --display-name "Teachable Machine (Python 3.11)"
```

`--user` registers the kernel for your local user account; it does not install the
project's packages globally. Kernel registrations contain absolute paths to their
environment's Python executable. Repeat the registration command after moving the
repository or recreating an environment at a different location.

### Use the kernels

To launch JupyterLab from the repository root:

```bash
uv run --project . jupyter lab
```

Open a notebook and use **Kernel → Change Kernel** to choose the appropriate
kernel from the table above. The same JupyterLab server can run notebooks using
both environments.

In **Zed**, select the appropriate registered kernel in the notebook's kernel
picker. If it is missing, run **`repl: refresh kernelspecs`** from the command
palette or restart Zed. Zed's Python toolchain selection and execution kernel are
separate settings. Kernel registration enables execution; it does not by itself
enable Zed's experimental `.ipynb` editor.

Verify the selected environment by running this notebook cell:

```python
import sys

print(sys.version)
print(sys.executable)
```

For Teachable Machine, the executable should be inside
`PVA2/teachable-machine/.venv` and the Python version should be 3.11.

### Add dependencies to the correct project

Use `uv add` from a terminal so dependencies are recorded in the project's
`pyproject.toml` and lockfile. For example, to add the CV notebook dependencies to
the root project:

```bash
uv add --project . opencv-python rasterio pillow
```

To add scikit-learn to the Teachable Machine environment:

```bash
uv add --project PVA2/teachable-machine scikit-learn
```

Restart the affected notebook kernel after changing installed packages. Prefer
these commands over `%pip install` in notebook cells, which does not update uv's
project dependency records. After pulling dependency changes, run `uv sync` for
each affected project.

Commit each project's `pyproject.toml` and `uv.lock`. Keep `.venv` directories and
user-specific kernel registrations out of version control.

### Inspect or remove kernel registrations

```bash
uv run --project . jupyter kernelspec list

# Remove a registration that is no longer needed.
uv run --project . jupyter kernelspec uninstall teachable-machine
```

Removing a registration does not delete its virtual environment. If a kernel
fails to start because its Python executable no longer exists, sync its project
and register the kernel again.

References: [uv and Jupyter](https://docs.astral.sh/uv/guides/integration/jupyter/),
[IPython kernel installation](https://ipython.readthedocs.io/en/stable/install/kernel_install.html),
[Zed kernels](https://zed.dev/docs/repl).
