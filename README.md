# Roku YouTube Sync

A Python learning project: build a tool that finds YouTube URLs for what your Roku is playing, so you can watch along on another device.

This project is designed to be built progressively as you work through [Automate the Boring Stuff with Python](https://automatetheboringstuff.com/). Each phase introduces new concepts and produces working code.

---

## Prerequisites

You need four things installed: Python, uv, VS Code, and Git.

### 1. Install Python

1. Go to [python.org/downloads](https://www.python.org/downloads/) and download the latest Python 3.x installer for Windows.
2. **IMPORTANT**: On the first screen of the installer, check the box that says **"Add python.exe to PATH"**. This is easy to miss and will cause problems later if skipped.
3. Click "Install Now" and let it finish.
4. Verify it worked — open **Command Prompt** (search "cmd" in the Start menu) and type:
   ```
   python --version
   ```
   You should see something like `Python 3.12.x`. If you get an error, uninstall and reinstall with the PATH checkbox checked.

### 2. Install uv

`uv` is a modern Python package manager that's much faster than `pip`. It also handles virtual environments for you.

In Command Prompt, run:
```
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Close and reopen Command Prompt, then verify:
```
uv --version
```

You should see something like `uv 0.5.x`.

### 3. Install VS Code

1. Go to [code.visualstudio.com](https://code.visualstudio.com/) and download the Windows installer.
2. Run the installer. On the "Select Additional Tasks" screen, check:
   - **Add "Open with Code" action to Windows Explorer file context menu** (both options)
   - **Add to PATH** (should be checked by default)
3. Open VS Code after installation.
4. Install the **Python extension**:
   - Click the Extensions icon in the left sidebar (looks like four squares)
   - Search for "Python" and install the one by Microsoft
   - This gives you syntax highlighting, error detection, and a built-in terminal

### 4. Install Git

1. Go to [git-scm.com/download/win](https://git-scm.com/download/win) and download the installer.
2. Run the installer — the default options are fine for everything. Just click "Next" through each screen.
3. Verify in Command Prompt:
   ```
   git --version
   ```

---

## Project Setup

Open Command Prompt and run these commands one at a time:

```bash
# Create a folder for the project
mkdir roku-youtube-sync
cd roku-youtube-sync

# Initialize a git repository
git init

# Create the project structure
mkdir src
mkdir tests
```

Now open this folder in VS Code:
```bash
code .
```

### Initialize with uv

In the VS Code terminal (`` Ctrl+` `` to open it), initialize the project:

```bash
uv init
```

This creates a `pyproject.toml` file that tracks your project configuration and dependencies.

### Install dependencies

With uv, installing packages is simple:
```bash
uv add requests
```

This automatically creates a virtual environment (in `.venv/`) and installs the package. You don't need to manually activate anything — `uv run` handles it.

### Create a .gitignore

Create a file called `.gitignore` in the project root with this content:

```
.venv/
__pycache__/
*.pyc
config.json
.vscode/
```

This tells Git to ignore your virtual environment, cached files, and your config (which will eventually hold your API key).

---

## Running Your Code

In the VS Code terminal:

```bash
uv run python src/search_url.py
```

The `uv run` prefix ensures your code runs with the correct virtual environment and dependencies.

Or press `F5` in VS Code to run the currently open file with the debugger (after configuring VS Code to use the `.venv` Python interpreter).

---

## Project Phases

See `CLAUDE.md` for the full project plan. Here's the summary:

| Phase | What You Build | ATBS Chapters |
|-------|---------------|---------------|
| 1 | Title → YouTube search URL | 1–4 |
| 2 | Interactive input with validation | 5–6 |
| 3 | YouTube API integration (real results) | 12 |
| 4 | Search history + config files | 9–10 |
| 5 | Roku network discovery | 12 + external |
| 6 | Combined CLI tool | 11, 17 |

Start with Phase 1. Read the corresponding ATBS chapters, then build the code. Each phase should work on its own before you move on.

---

## Developer Tools

As you progress through the phases, you'll learn to use professional Python tools:

### Phase 2+: Ruff (Linting)

Ruff checks your code for common mistakes and style issues. Install it:
```bash
uv add --dev ruff
```

Run it on your code:
```bash
uv run ruff check src/
```

Fix auto-fixable issues:
```bash
uv run ruff check src/ --fix
```

### Phase 3+: ty (Type Checking)

Once you start using type hints (Phase 3), `ty` helps catch type errors before you run your code:
```bash
uv add --dev ty
```

Check your code:
```bash
uv run ty check src/
```

---

## Getting a YouTube API Key (Phase 3)

You'll need this when you reach Phase 3:

1. Go to [console.cloud.google.com](https://console.cloud.google.com/)
2. Create a new project (any name is fine)
3. Go to **APIs & Services → Library**
4. Search for "YouTube Data API v3" and enable it
5. Go to **APIs & Services → Credentials**
6. Click **Create Credentials → API Key**
7. Copy the key and save it in a `config.json` file:
   ```json
   {
       "youtube_api_key": "YOUR_KEY_HERE"
   }
   ```
   This file is in `.gitignore` so it won't be committed to Git.

The free tier gives you 10,000 quota units/day, which is roughly 100 search queries — more than enough for this project.

---

## Finding Your Roku's IP Address (Phase 5)

You'll need this when you reach Phase 5:

1. On your Roku, go to **Settings → Network → About**
2. Note the IP address (something like `192.168.1.xxx`)
3. Test it by opening a browser and going to: `http://<your-roku-ip>:8060/query/active-app`
4. You should see an XML response with your Roku's currently active app

---

## Tips for Beginners

- **Read the error messages.** Python error messages tell you exactly what went wrong and on which line. Start reading from the bottom.
- **Run your code often.** Don't write 50 lines and then run — write 5 lines, run, verify, repeat.
- **Use `print()` to debug.** When something isn't working, add `print()` statements to see what your variables actually contain.
- **Google the error.** Copy-paste error messages into Google. Someone has hit the same problem before.
- **Commit after each working change.** In the terminal:
  ```bash
  git add .
  git commit -m "describe what you changed"
  ```
  This gives you a save point to go back to if you break something.
