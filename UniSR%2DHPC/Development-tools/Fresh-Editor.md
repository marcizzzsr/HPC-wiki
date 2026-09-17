# Fresh Editor

[Fresh](https://github.com/sinelaw/fresh) is a modern, terminal-based text editor with zero configuration, familiar VSCode/Sublime-like keybindings, mouse support, and IDE-level features (command palette, file explorer, LSP integration, git support, and more).

Unlike the other tools in this section, Fresh is **not** an Apptainer/SLURM launcher — it's a lightweight standalone binary you run directly in your shell, on the login node or inside an interactive job. This makes it a great option for quickly browsing and editing files, or as a full terminal IDE for Python development.

## Installation

Install the self-updating universal Linux build with:

```bash
curl -fsSL https://raw.githubusercontent.com/sinelaw/fresh/refs/heads/master/scripts/install.sh | sh
```

This unpacks the binary to `~/.local/share/fresh-editor` and symlinks it at `~/.local/bin/fresh`. Make sure `~/.local/bin` is in your `PATH` — if `which fresh` doesn't find it, add the following to your `~/.bashrc` and reload it (`source ~/.bashrc`):

```bash
export PATH="$HOME/.local/bin:$PATH"
```

**Verify the installation:**

```bash
fresh --version
```

## Python Development Setup (optional)

If you only plan to use Fresh for file exploring or quick edits, you can skip this section. To develop Python code with a full language server and formatter, first install [uv](https://docs.astral.sh/uv/), a fast Python package and environment manager:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
```

**Verify the installation:**

```bash
uv --version
```

Then install a language server and a formatter/linter as isolated `uv` tools:

```bash
uv tool install basedpyright  # language server (LSP)
uv tool install ruff          # formatter and linter
```

## Typical Workflow

1. Create a virtual environment inside your project folder and install the packages you need, e.g.:

   ```bash
   uv venv
   uv add <package>
   ```

2. `cd` into the project directory.
3. Start editing with:

   ```bash
   fresh .
   ```

   If the LSP doesn't get picked up automatically (e.g. it can't find the venv's interpreter), run Fresh from inside the environment instead:

   ```bash
   uv run fresh
   ```

## Clipboard problems?
If you are experiencing any issu with fresh copy-paste functionalities while using it inside `tmux` or `byobu`, try enabling the `set-clipboard` option in your config file (create it if not yet present):
- If using **tmux**: add `set -s set-clipboard on` to your `~/.tmux.conf` (apply changes with `tmux source-file ~/.tmux.conf`)
- If using **byobu**: add `set -s set-clipboard on` to your `~/.byobu/.tmux.conf` (apply changes with `byobu source-file ~/.byobu/.tmux.conf`)


## Learn More

This is only a brief introduction. Fresh has many more features (multi-cursor editing, split panes, integrated terminal, git integration, plugins, and more) — check the [official documentation](https://getfresh.dev/docs/) and the [GitHub repository](https://github.com/sinelaw/fresh) to learn how to use them.

---

[[_TOSP_]]
