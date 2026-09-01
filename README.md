# Development Environment

This repository contains my OS shell and terminal configuration managed with [chezmoi](https://www.chezmoi.io/).

The goal is to make a new easy to reproduce without manually remembering every tool and configuration step.

## Overview

### Shell & Terminal

* Zsh
* Powerlevel10k
* WezTerm
* zsh-autosuggestions
* zsh-syntax-highlighting
* Oh My Zsh where needed

### CLI Tools

* Homebrew
* eza
* bat
* fd
* fzf
* fzf-git.sh
* zoxide
* yazi
* thefuck
* direnv
* GitHub CLI (`gh`)
* Neovim

### Python

Python tooling is split between:

* **uv** — Python versions and project environments
* **pipx** — globally installed Python CLI applications

Do **not** use pyenv for this setup.

---

# New  Setup

## 1. Install Homebrew

Install Homebrew if it isn't already installed.

Then make sure it is available:

```bash
brew --version
```

## 2. Install Core Tools

Install the tools needed to bootstrap the environment:

```bash
brew install chezmoi uv pipx
```

Then ensure pipx's binaries are available:

```bash
pipx ensurepath
```

Restart the shell:

```bash
exec zsh
```

---

# 3. Pull Dotfiles

Initialize chezmoi using the dotfiles repository:

```bash
chezmoi init --apply git@github.com:YOUR_USERNAME/YOUR_DOTFILES_REPO.git
```

This pulls the repository and applies the managed configuration files.

Check what chezmoi is managing:

```bash
chezmoi managed
```

---

# 4. Updating an Existing 

When changes have been committed and pushed from another hine:

```bash
chezmoi update
```

This pulls the latest dotfiles and applies them.

Then reload Zsh:

```bash
exec zsh
```

### Preview changes first

If I want to inspect changes before applying them:

```bash
chezmoi git pull
chezmoi diff
chezmoi apply
```

---

# 5. Python Setup

Python versions should be managed with **uv**.

Install the Python versions needed by projects:

```bash
uv python install 3.11
```

For a project, use the version required by that project rather than changing a global Python installation.

Example:

```bash
uv init
uv python pin 3.12
uv sync
```

Another project can use a different version:

```bash
uv python pin 3.11
uv sync
```

This keeps project environments isolated.

---

# 6. Installing Python CLI Tools with pipx

Use `pipx` for Python applications that should be available globally from the shell.

For example:

```bash
pipx install <package>
```

However, **do not blindly use the default Python interpreter for old Python packages**.

Some older packages are incompatible with modern Python versions.

## thefuck

`thefuck` currently requires special handling because older versions depend on Python functionality such as `distutils` that was removed from newer Python releases.

Install Python 3.11 with uv:

```bash
uv python install 3.11
```

Then install `thefuck` using that specific interpreter:

```bash
pipx install --python "$(uv python find 3.11)" thefuck
```

Verify:

```bash
thefuck --version
```

### Why this matters

Avoid:

```bash
pipx install thefuck
```

because pipx may select a newer Python version that is incompatible with `thefuck`.

Instead:

```text
uv
└── Python 3.11
    └── thefuck via pipx
```

This allows the rest of the system to use newer Python versions without breaking `thefuck`.

---

# 7. Shell Configuration

The shell configuration is managed by chezmoi.

Important files include:

```text
~/.zshrc
~/.p10k.zsh
~/.wezterm.lua
```

After changing a managed file:

```bash
chezmoi add ~/.zshrc
```

Then commit and push the changes:

```bash
chezmoi cd
git add .
git commit -m "Update zsh configuration"
git push
```

On another :

```bash
chezmoi update
```

---

# 8. Zsh Plugins

The shell configuration expects:

* zsh-autosuggestions
* zsh-syntax-highlighting

On Apple Silicon s, Homebrew normally installs these under:

```text
/opt/homebrew/
```

The `.zshrc` should therefore use the appropriate Homebrew paths.

After installing/updating plugins:

```bash
exec zsh
```

---

# 9. Homebrew Packages

The dotfiles configure the shell, but **chezmoi does not install every external dependency**.

Keep the hine's installed packages reproducible separately.

Useful commands:

```bash
brew list
```

Export a Brewfile:

```bash
brew bundle dump --force
```

This creates:

```text
Brewfile
```

On a new :

```bash
brew bundle
```

This is the preferred way to reproduce the Homebrew side of the environment.

---

# 10. Environment Structure

The overall setup should look like:

```text
OS
│
├── Homebrew
│   ├── chezmoi
│   ├── uv
│   ├── pipx
│   ├── neovim
│   ├── fzf
│   ├── eza
│   ├── bat
│   ├── yazi
│   ├── zoxide
│   └── other CLI tools
│
├── chezmoi
│   └── Dotfiles repository
│       ├── .zshrc
│       ├── .p10k.zsh
│       ├── .wezterm.lua
│       └── ...
│
├── uv
│   ├── Python 3.11
│   │   └── legacy CLI tools
│   │
│   ├── Python 3.12+
│   │   └── modern projects
│   │
│   └── project-specific environments
│
└── pipx
    └── Global Python CLI applications
```

---

# Troubleshooting

## `thefuck: command not found`

Check whether it is installed:

```bash
pipx list
```

If it isn't:

```bash
pipx install --python "$(uv python find 3.11)" thefuck
```

Check the executable:

```bash
which thefuck
```

If pipx's bin directory isn't on PATH:

```bash
pipx ensurepath
exec zsh
```

---

## `No module named distutils`

Do **not** try to fix this by installing random `distutils` packages into the system Python.

Use Python 3.11 for the legacy package:

```bash
uv python install 3.11
pipx uninstall thefuck
pipx install --python "$(uv python find 3.11)" thefuck
```

---

## Dotfiles aren't updating

Check the current chezmoi state:

```bash
chezmoi status
```

Preview differences:

```bash
chezmoi diff
```

Pull and apply:

```bash
chezmoi update
```

---

# Normal Workflow

### Making a change

```bash
nvim ~/.zshrc
chezmoi add ~/.zshrc

chezmoi cd
git add .
git commit -m "Update zsh configuration"
git push
```

### Getting changes on another 

```bash
chezmoi update
exec zsh
```

### Starting a new Python project

```bash
uv init
uv python pin 3.12
uv sync
```

### Installing a Python CLI

```bash
pipx install <package>
```

If the package is old or has Python compatibility issues, explicitly select a compatible Python:

```bash
pipx install --python "$(uv python find 3.11)" <package>
```

---

# Guiding Principles

1. **chezmoi manages configuration.**
2. **Homebrew manages OS CLI/software dependencies.**
3. **uv manages Python versions and project environments.**
4. **pipx manages globally installed Python CLI applications.**
5. **Don't use pyenv unless there is a specific reason to add it.**
6. **Pin old Python applications to a compatible Python version instead of changing the system Python.**
7. **Use project-specific Python versions with uv rather than relying on one global Python version.**
