# Jupyter Notebooks — quick setup and use

Standalone reference note (not a course chapter recap), requested by Bruno during
the [prompt-evaluation](../prompt-evaluation.md) session since the course starts
using notebooks for the eval pipeline from that chapter onward.

## Install and launch

```bash
python -m pip install jupyterlab
python -m jupyter lab
```

Using `python -m pip` / `python -m jupyter` (rather than bare `pip`/`jupyter`) avoids
PATH issues, especially on Windows and inside virtual environments. If the `jupyter`
command isn't found after install, fall back to `python -m jupyter lab`.

This opens JupyterLab (the modern successor to classic Jupyter Notebook) in the
browser, with a file browser, multi-tab layout, and an integrated terminal.

## Creating and running a notebook

1. In the JupyterLab file browser, use **File → New → Notebook** and pick a Python
   kernel (or the kernel matching your project's venv/conda env).
2. A notebook is a sequence of **cells** — code cells (Python) or Markdown cells (notes).
3. Run a cell with **Shift+Enter** (runs and moves to the next cell) or **Ctrl+Enter**
   (runs and stays on the same cell).
4. Cells share state — a variable defined in one cell is available in cells run
   after it, in whatever order you actually run them (not just file order).

## Useful keyboard shortcuts

- **Esc** — enter command mode (cell-level actions instead of typing into the cell)
- **A** / **B** — insert a new cell above / below (command mode)
- **DD** (press D twice) — delete the selected cell (command mode)
- **M** — convert the selected cell to Markdown (command mode)
- **Y** — convert back to a code cell (command mode)

## Notes for this course's use case

The eval pipeline chapters build up a notebook incrementally: define a client and
env vars in the top cell, then helper functions in later cells, re-running cells as
functions are added or changed. Re-run affected cells top-to-bottom after any edit
higher up, since Jupyter won't auto-propagate changes to already-executed cells.

Sources:
- [How to Start JupyterLab: Install, Launch, and Fix Common Errors](https://docs.kanaries.net/topics/Python/how-to-start-juypter-lab)
- [Installation — JupyterLab documentation](https://jupyterlab.readthedocs.io/en/stable/getting_started/installation.html)
- [Project Jupyter | Installing Jupyter](https://jupyter.org/install)
