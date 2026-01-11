# Python Version Management Tools

## Overview

There are several tools that can automatically detect and switch to the correct Python version based on `pyproject.toml` or other configuration files, similar to `nvm` for Node.js or `sdkman` for Java.

## Recommended Solutions

### 1. pyenv + pyenv-virtualenv (Most Popular)

**pyenv** is the Python equivalent of `nvm`. It can automatically switch Python versions based on a `.python-version` file.

**Installation (macOS):**
```bash
brew install pyenv
brew install pyenv-virtualenv

# Add to your ~/.zshrc or ~/.bash_profile:
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

**Usage:**
```bash
# Install Python 3.12
pyenv install 3.12.0

# Set local version for this project (creates .python-version file)
pyenv local 3.12.0

# Poetry will automatically use this version
poetry install
```

**Auto-switching:** pyenv will automatically switch to the version specified in `.python-version` when you `cd` into the directory.

### 2. mise (formerly rtx) - Modern & Fast ⭐ Recommended

**mise** (formerly rtx) is a modern, fast version manager that supports Python and many other tools. It can automatically read `pyproject.toml` and switch versions.

**Note:** The project was renamed from "rtx" to "mise" to avoid confusion with NVIDIA's RTX graphics cards.

**Installation (macOS):**
```bash
brew install mise
```

**Setup:**
```bash
# Add to ~/.zshrc:
eval "$(mise activate zsh)"
```

**Usage:**
```bash
# mise automatically reads pyproject.toml and installs/uses the correct Python version
cd /path/to/project
mise install python@3.12  # Install if needed
# mise will automatically use the version from pyproject.toml
```

**Auto-switching:** mise automatically detects `pyproject.toml` and switches Python versions when you enter the directory.

### 3. asdf - Multi-language Version Manager

**asdf** is similar to rtx but older and more established. It supports Python and many other languages.

**Installation (macOS):**
```bash
brew install asdf

# Add to ~/.zshrc:
. $(brew --prefix asdf)/libexec/asdf.sh
```

**Setup for Python:**
```bash
asdf plugin add python
asdf install python 3.12.0
```

**Usage:**
```bash
# Create .tool-versions file (or rtx/asdf can read pyproject.toml with plugins)
echo "python 3.12.0" > .tool-versions
asdf install
```

### 4. Poetry with pyenv (Hybrid Approach)

Poetry can work with pyenv. If you have pyenv installed and a `.python-version` file, Poetry will use that version.

**Setup:**
```bash
# Install pyenv (see option 1)
# Set Python version for project
pyenv local 3.12.0

# Poetry will detect and use it
poetry env use $(pyenv which python)
poetry install
```

### 5. direnv + pyenv (Advanced)

**direnv** can automatically load environment variables and run commands when you enter a directory. Combined with pyenv, it can auto-switch Python versions.

**Installation:**
```bash
brew install direnv

# Add to ~/.zshrc:
eval "$(direnv hook zsh)"
```

**Create `.envrc` in project root:**
```bash
# .envrc
use pyenv 3.12.0
```

## Quick Setup Guide for This Project

### Option A: Using mise (Easiest) ⭐

```bash
# 1. Install mise
brew install mise

# 2. Add to ~/.zshrc
echo 'eval "$(mise activate zsh)"' >> ~/.zshrc
source ~/.zshrc

# 3. In your project directory
cd /Users/ruslanbes/projects/metaculus/metac-bot-template
mise install python@3.12  # Install Python 3.12
mise use python@3.12       # Use it for this project

# 4. Poetry will use the mise-managed Python
poetry env use $(mise which python)
poetry install
```

### Option B: Using pyenv (Most Common)

```bash
# 1. Install pyenv
brew install pyenv pyenv-virtualenv

# 2. Add to ~/.zshrc
cat >> ~/.zshrc << 'EOF'
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
EOF
source ~/.zshrc

# 3. Install Python 3.12
pyenv install 3.12.0

# 4. Set for this project
cd /Users/ruslanbes/projects/metaculus/metac-bot-template
pyenv local 3.12.0

# 5. Poetry will use it automatically
poetry install
```

## Comparison

| Tool | Auto-detects pyproject.toml | Speed | Ease of Use | Multi-language |
|------|------------------------------|-------|-------------|---------------|
| **mise** | ✅ Yes | ⚡ Very Fast | ⭐⭐⭐⭐⭐ | ✅ Yes |
| **asdf** | ⚠️ With plugin | 🐢 Slower | ⭐⭐⭐ | ✅ Yes |
| **pyenv** | ❌ No (needs .python-version) | ⚡ Fast | ⭐⭐⭐⭐ | ❌ Python only |
| **direnv** | ❌ No | ⚡ Fast | ⭐⭐⭐ | ✅ Yes |

## Recommendation

For this project, I recommend **mise** because:
1. ✅ Automatically reads `pyproject.toml`
2. ✅ Very fast
3. ✅ Simple setup
4. ✅ Works with Poetry seamlessly
5. ✅ Can manage other tools too (Node, Java, etc.)

## Creating .python-version for pyenv

If you prefer pyenv, you can create a `.python-version` file in the project root:

```bash
echo "3.12.0" > .python-version
```

This file should be committed to git so all team members use the same Python version.

## Poetry-Specific Note

Poetry doesn't automatically switch Python versions. You need to either:
1. Use a version manager (pyenv/mise) that sets the system Python
2. Explicitly tell Poetry: `poetry env use python3.12`

The version manager approach is cleaner because it works system-wide and Poetry will automatically detect it.
