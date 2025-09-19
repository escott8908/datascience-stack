Yep—**one venv per project** is the clean, standard way to work in VS Code. It keeps dependencies isolated so Project A’s packages don’t collide with Project B’s.

Below is a simple, repeatable workflow for **3 separate projects** inside your `vscode` (code-server) container.

---

# Folder layout (example)

On the host you’ve mounted `/home/escott/data_analysis` → `/home/coder/project` in the container. Create subfolders:

```
/home/escott/data_analysis/
  project-a/
  project-b/
  project-c/
```

Each project will have **its own** `.venv/` and `.vscode/`.

---

# One-time setup per project

## 1) Create the venv and install deps

Open the **terminal in VS Code** ([http://localhost:8443](http://localhost:8443) → Terminal → New Terminal), then:

```bash
# Go to the project folder you want to set up
cd /home/coder/project/project-a

# Create a venv named .venv (kept inside the project)
python3 -m venv .venv

# Activate it for this terminal
source .venv/bin/activate

# Upgrade pip and install your deps
pip install -U pip
pip install numpy pandas scipy scikit-learn seaborn matplotlib jupyter ipykernel

# (Optionally) install dev tools in the venv too
pip install black isort ruff debugpy
```

> Tip: keep a `requirements.txt` in each project and do `pip install -r requirements.txt`.
> To freeze: `pip freeze > requirements.txt`.

## 2) Make VS Code auto-use the project’s venv

Create **`project-a/.vscode/settings.json`**:

```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.terminal.activateEnvironment": true,

  // nice-to-haves
  "editor.formatOnSave": true,
  "[python]": { "editor.defaultFormatter": "ms-python.black-formatter" },
  "editor.codeActionsOnSave": {
    "source.fixAll.ruff": true,
    "source.organizeImports": true
  }
}
```

> Now, when you open **project-a** as a folder, VS Code will pick **`.venv/bin/python`** automatically and auto-activate it in terminals.

## 3) (If you use notebooks) add a kernel name

This lets the Jupyter extension show a friendly kernel:

```bash
# still inside the activated venv
python -m ipykernel install --user --name project-a --display-name "Python (project-a)"
```

In VS Code notebooks, choose kernel **“Python (project-a)”**.

---

# Do the same for project-b and project-c

Repeat the **same steps** in `project-b/` and `project-c/`. Each will have its own `.venv` and `.vscode/settings.json` pointing to `${workspaceFolder}/.venv/bin/python`.

---

# Switching between projects (and venvs)

You’ve got two easy patterns:

### A) Open one project at a time

* VS Code (code-server): **File → Open Folder…** → choose `project-a/`.
  It auto-selects `project-a/.venv`.
* When you switch: **File → Open Folder…** → `project-b/`.
  It auto-selects `project-b/.venv`.

You’ll also see the active interpreter in the **status bar** (bottom left). Click it any time to **Python: Select Interpreter** manually.

### B) Multi-root workspace (work on all 3 at once)

Create a workspace file that includes all three folders:

1. **File → Add Folder to Workspace…** add `project-a/`, `project-b/`, `project-c/`.
2. **File → Save Workspace As…** e.g., `datascience.code-workspace`.

Each folder keeps its **own** `.vscode/settings.json`, so each uses its own venv. When you switch files between folders, the Python extension uses the right interpreter per folder.

---

# Tips you’ll actually use

* **Don’t commit** `.venv/`: add it to each project’s `.gitignore`.
* Recreate env quickly:

  ```bash
  rm -rf .venv
  python3 -m venv .venv
  source .venv/bin/activate
  pip install -r requirements.txt
  ```
* If you installed Black/Ruff globally in the container earlier and want versions pinned per project, install them **in the venv** too and keep them in `requirements.txt`.
* For MLflow or OpenAI usage in your code: keep **API keys** in env (Compose `.env`) rather than in the venv.

---

## Mini checklist per new project

```bash
cd /home/coder/project/<project-name>
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip -r requirements.txt    # or pip install <packages>
python -m ipykernel install --user --name <project-name> --display-name "Python (<project-name>)"

# .vscode/settings.json
#  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python"
```

That’s it—three projects, three isolated environments, and **instant switching** just by opening the project folder (or via a multi-root workspace).

