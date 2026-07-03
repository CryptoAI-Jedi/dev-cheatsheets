# Python Environments & Packaging Cheatsheet

> venv, pip, pyproject.toml, pipx, and uv — isolated environments, reproducible installs, and deployments that survive contact with a VPS. One venv per project, never `sudo pip`, always `python -m pip`.

---

## Table of Contents
- [The Mental Model](#the-mental-model)
- [venv](#venv)
- [pip Essentials](#pip-essentials)
- [Requirements & Locking](#requirements--locking)
- [pyproject.toml](#pyprojecttoml)
- [Ubuntu / Debian Quirks](#ubuntu--debian-quirks)
- [pipx (CLI tools)](#pipx-cli-tools)
- [uv (the fast consolidation)](#uv-the-fast-consolidation)
- [Common Workflows](#common-workflows)
- [Tips & Gotchas](#tips--gotchas)

---

## THE MENTAL MODEL

```text
Interpreter   → /usr/bin/python3 (system) — leave its packages alone
venv          → Project-local copy of the interpreter + its own site-packages
pip           → Installs INTO whichever environment its python belongs to
pyproject.toml → Declares what your project needs (the intent)
requirements.txt → Pins exactly what got installed (the lockfile)
pipx          → venvs-as-a-service for CLI tools (ruff, httpie, ...)
```

---

## VENV

```bash
sudo apt install python3-venv            # Ubuntu ships venv separately
python3 -m venv .venv                    # Create (convention: .venv in project root)
python3 -m venv --upgrade-deps .venv     # Create with pip/setuptools upgraded
source .venv/bin/activate                # Activate (prompt shows (.venv))
deactivate                               # Leave
```

### No activation needed — bin paths work directly

```bash
.venv/bin/python script.py               # Runs with the venv's packages
.venv/bin/pip install httpx              # Installs into the venv
# This is how systemd units and cron jobs should call Python:
# ExecStart=/opt/agent/.venv/bin/python main.py   (see systemd sheet)
```

### Verify which environment you're actually in

```bash
which python                             # Should show .venv/bin/python when active
python -c "import sys; print(sys.prefix)"
pip --version                            # Shows which site-packages it targets
```

### Reset a broken venv (they're cheap — never debug one for long)

```bash
rm -rf .venv
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
```

```bash
echo ".venv/" >> .gitignore              # Never commit a venv
```

---

## PIP ESSENTIALS

```bash
python -m pip install httpx              # `python -m pip` — always targets THIS python
pip install "httpx==0.27.0"              # Exact version
pip install "httpx>=0.27,<1.0"           # Range
pip install "pydantic~=2.7.0"            # Compatible release (>=2.7.0, <2.8)
pip install --upgrade httpx              # Upgrade one package
pip uninstall httpx
```

```bash
pip install -e .                         # Editable install of current project (needs pyproject.toml)
pip install -e ".[dev]"                  # Editable + optional dependency group
pip install "git+https://github.com/user/repo.git@main"   # Straight from git
```

### Inspection

```bash
pip list                                 # Installed packages
pip list --outdated                      # What has updates
pip show httpx                           # Version, deps, install location
pip check                                # Detect broken/conflicting dependencies
pip cache purge                          # Reclaim disk from the wheel cache
```

---

## REQUIREMENTS & LOCKING

```bash
pip install -r requirements.txt          # Install pinned set
pip freeze > requirements.txt            # Snapshot EVERYTHING (see caveat below)
```

```text
⚠️ pip freeze pins your deps AND their deps AND pip's plumbing, with no
record of what you actually asked for. Fine for quick snapshots; for
maintained projects, declare intent in pyproject.toml and compile a lock:
```

### pip-tools (compile intent → lockfile)

```bash
pip install pip-tools
pip-compile pyproject.toml -o requirements.txt    # Resolve + pin from [project.dependencies]
pip-compile --upgrade -o requirements.txt pyproject.toml          # Refresh all pins
pip-compile --upgrade-package httpx -o requirements.txt pyproject.toml   # Bump ONE dep
pip-sync requirements.txt                # Make env match the file EXACTLY (removes strays)
```

### Constraints (cap versions without requiring the package)

```bash
pip install -r requirements.txt -c constraints.txt
```

---

## PYPROJECT.TOML

One file for project metadata, dependencies, and tool config — replaces setup.py, setup.cfg, requirements.in, and scattered dotfiles.

```toml
[build-system]
requires = ["setuptools>=68"]
build-backend = "setuptools.build_meta"

[project]
name = "myagent"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
    "httpx>=0.27",
    "pydantic>=2.0,<3",
]

[project.optional-dependencies]
dev = ["pytest", "ruff"]

[project.scripts]
myagent = "myagent.main:run"        # `myagent` CLI command after install

[tool.ruff]
line-length = 100

[tool.pytest.ini_options]
testpaths = ["tests"]
```

```bash
pip install -e ".[dev]"              # Install project + dev extras, editable
```

```text
Division of labor:
pyproject.toml   → committed, human-edited, loose ranges (intent)
requirements.txt → committed, machine-generated, exact pins (deploy artifact)
```

---

## UBUNTU / DEBIAN QUIRKS

```bash
sudo apt install python3-venv python3-pip python3-dev build-essential
# python3-dev + build-essential: needed when pip compiles C extensions
# ("error: Python.h: No such file" → this is the fix)
```

```text
⚠️ NEVER `sudo pip install` — it writes into apt's territory and breaks
system tooling. The system interpreter belongs to apt; your code gets a venv.
```

### PEP 668: "externally-managed-environment" error

```text
Ubuntu 23.04+/Debian 12+ block bare `pip install` outside a venv by design.
Correct fixes: use a venv (projects) or pipx (CLI tools).
Escape hatch (understand what you're doing): pip install --break-system-packages
Ubuntu 22.04 doesn't enforce this yet — but behave as if it does.
```

### Newer Python on Ubuntu 22.04 (deadsnakes)

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.12 python3.12-venv
python3.12 -m venv .venv                 # venv pinned to that interpreter
```

---

## PIPX (CLI TOOLS)

Each tool gets its own hidden venv; the command lands on your PATH. For tools you *run*, not libraries you *import*.

```bash
sudo apt install pipx
pipx ensurepath                          # Put ~/.local/bin on PATH (new shell after)
pipx install ruff                        # Install a CLI tool
pipx install --python python3.12 tool    # Pin the interpreter
pipx list
pipx upgrade ruff
pipx upgrade-all
pipx uninstall ruff
pipx run cowsay -t "no install"          # Ephemeral: run without installing
pipx inject ipython rich                 # Add a lib INTO a pipx tool's venv
```

---

## UV (THE FAST CONSOLIDATION)

One Rust binary replacing pip + venv + pip-tools + pipx + pyenv, 10–100x faster resolves. Same concepts, fewer moving parts.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Drop-in replacements

```bash
uv venv                                  # Create .venv (fast)
uv pip install -r requirements.txt       # pip, but fast
uv pip compile pyproject.toml -o requirements.txt   # pip-compile, but fast
uv pip sync requirements.txt             # pip-sync equivalent
uv tool install ruff                     # pipx equivalent
uv python install 3.12                   # Managed interpreter (no PPA needed)
```

### Project mode (uv-native)

```bash
uv init myproject                        # Scaffold pyproject.toml
uv add httpx                             # Add dep + update lockfile
uv add --dev pytest                      # Dev dependency
uv lock                                  # Write uv.lock
uv sync                                  # Make .venv match the lockfile
uv run python main.py                    # Run inside the env, no activation
uv run pytest
```

```text
💡 Adoption path: start with the drop-in commands against your existing
requirements files — zero workflow change, immediate speed. Move to
project mode (uv.lock) per-repo when ready.
```

---

## COMMON WORKFLOWS

### New project, pyproject-first

```bash
mkdir myagent && cd myagent
python3 -m venv .venv && source .venv/bin/activate
# write pyproject.toml ([project] block above)
pip install -e ".[dev]"
pip-compile pyproject.toml -o requirements.txt
git init && echo ".venv/" >> .gitignore
```

### Deploy to a VPS (pairs with systemd sheet)

```bash
git clone git@github:CryptoAI-Jedi/myagent.git /opt/agent
cd /opt/agent
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt     # Exact pins — same env as tested
# systemd unit: ExecStart=/opt/agent/.venv/bin/python main.py
sudo systemctl enable --now myagent
```

### Upgrade one dependency safely

```bash
pip-compile --upgrade-package httpx -o requirements.txt pyproject.toml
pip-sync requirements.txt
pytest                                    # Verify before it ships
git diff requirements.txt                 # Review exactly what moved
```

### Docker tie-in (layer caching — see Docker sheet)

```text
COPY requirements.txt first → RUN pip install → COPY . .
The lockfile IS the cache key: code edits stop reinstalling the world.
```

---

## TIPS & GOTCHAS

- **"pip installed it but import fails"** — pip and python pointed at different environments. Cure: `python -m pip install ...` always; diagnose with `which python; pip --version`.
- **venvs don't survive being moved or renamed** — activation scripts and shebangs hardcode absolute paths. Renamed the project dir? `rm -rf .venv` and recreate; it takes seconds.
- **systemd/cron don't "activate"** — activation just edits PATH for interactive shells. Non-interactive contexts use the absolute path: `/opt/agent/.venv/bin/python`.
- **Build failures on a slim VPS** (`Python.h missing`, `gcc not found`) — `sudo apt install python3-dev build-essential`; the wheel for your platform didn't exist so pip fell back to compiling.
- **`pip freeze` after months of experiments** — your "lockfile" now includes every abandoned experiment. `pip-sync`/`uv pip sync` exist precisely to remove what the file doesn't list.
- **`~=` beats `>=` for apps you deploy** — `pydantic~=2.7.0` accepts patch fixes, refuses surprise minor jumps. Loose ranges belong in libraries, not deploy targets.
- **Venv Python version is frozen at creation** — apt upgrading python3 can break existing venvs (missing modules, segfaults). Recreate venvs after interpreter upgrades.
- **`pipx ensurepath` needs a new shell** — the tool "isn't found" until PATH reloads; `exec $SHELL` or open a new terminal.
- **Quote extras in zsh** — `pip install -e ".[dev]"`; unquoted square brackets are glob syntax and fail confusingly.

---
*Last Updated: 2026-07*
