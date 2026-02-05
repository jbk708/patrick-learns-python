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

### 5. Configure Git (First-Time Setup)

Git needs to know who you are so it can label your commits. Run these commands once (replace with your info):

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

This stores your info in a global config file. You only need to do this once per computer.

---

## Setting Up GitHub and SSH

### What is GitHub?

**GitHub** is a website that stores Git repositories online. It lets you:
- Back up your code in the cloud
- Share code with others
- Collaborate on projects
- Access your code from any computer

### Creating a GitHub Account

1. Go to [github.com](https://github.com)
2. Click "Sign up" and follow the prompts
3. Choose the free plan (it's plenty for learning)
4. Verify your email address

### Setting Up SSH Keys

**SSH keys** let you connect to GitHub without typing your password every time. It's like a secure digital ID card.

**Step 1: Check for existing SSH keys**

Open Command Prompt (or Git Bash, which came with Git) and run:
```bash
ls ~/.ssh
```

If you see files like `id_rsa` and `id_rsa.pub`, you already have keys. Skip to Step 3.

If you get "No such file or directory", continue to Step 2.

**Step 2: Generate a new SSH key**

Run this command (use the email you registered on GitHub):
```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
```

When it asks "Enter file in which to save the key", just press Enter to accept the default.

When it asks for a passphrase, you can either:
- Press Enter for no passphrase (easier but less secure)
- Type a passphrase you'll remember (more secure)

**Step 3: Add your SSH key to the SSH agent**

Start the SSH agent:
```bash
eval "$(ssh-agent -s)"
```

Add your key:
```bash
ssh-add ~/.ssh/id_ed25519
```

**Step 4: Add the SSH key to GitHub**

Copy your public key to the clipboard:
```bash
cat ~/.ssh/id_ed25519.pub
```

Select and copy the entire output (starts with `ssh-ed25519` and ends with your email).

Then:
1. Go to [GitHub → Settings → SSH and GPG keys](https://github.com/settings/keys)
2. Click "New SSH key"
3. Give it a title like "My Windows Laptop"
4. Paste the key into the "Key" field
5. Click "Add SSH key"

**Step 5: Test the connection**

```bash
ssh -T git@github.com
```

If it asks "Are you sure you want to continue connecting?", type `yes`.

You should see: "Hi username! You've successfully authenticated..."

---

## Cloning This Repository

If someone has shared this project with you on GitHub, here's how to get it on your computer:

**Step 1: Get the repository URL**

On the GitHub repository page:
1. Click the green "Code" button
2. Make sure "SSH" is selected (not HTTPS)
3. Copy the URL (looks like `git@github.com:username/repo-name.git`)

**Step 2: Clone the repository**

Open Command Prompt and navigate to where you want the project:
```bash
cd Documents
```

Clone the repository:
```bash
git clone git@github.com:username/patrick-learns-python.git
```

This creates a new folder with all the project files.

**Step 3: Open in VS Code**

```bash
cd patrick-learns-python
code .
```

**Step 4: Install dependencies**

```bash
uv sync
```

This reads `pyproject.toml` and installs all the packages the project needs.

You're ready to start working!

---

## Project Setup (Starting Fresh)

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

### Understanding Virtual Environments

When you installed `requests` above, `uv` created a **virtual environment**. This is an important concept worth understanding.

**The Problem Virtual Environments Solve**

Imagine you have two Python projects:
- Project A needs `requests` version 2.25
- Project B needs `requests` version 2.31

If you install packages "globally" (for your whole computer), you can only have ONE version of `requests`. That breaks one of your projects!

**The Solution: Isolation**

A **virtual environment** is an isolated Python installation for a single project. Each project gets its own:
- Set of installed packages
- Package versions
- No conflicts with other projects

```
Your Computer
├── Project A/
│   └── .venv/           ← Project A's virtual environment
│       └── requests 2.25
├── Project B/
│   └── .venv/           ← Project B's virtual environment
│       └── requests 2.31
└── System Python        ← Stays clean, no packages installed here
```

**How uv Manages This**

When you run `uv add requests`:
1. uv creates a `.venv/` folder in your project (if it doesn't exist)
2. uv installs `requests` inside that `.venv/` folder
3. uv records the dependency in `pyproject.toml`

When you run `uv run python your_script.py`:
1. uv automatically uses the Python inside `.venv/`
2. Your script can access packages installed in `.venv/`
3. You don't need to "activate" anything manually

**Why `uv run` Instead of Just `python`?**

| Command | What It Uses |
|---------|--------------|
| `python script.py` | System Python (no project packages!) |
| `uv run python script.py` | Project's virtual environment (has your packages) |

If you forget `uv run`, you'll see errors like `ModuleNotFoundError: No module named 'requests'` even though you installed it.

**The .venv Folder**

- Located at `.venv/` in your project root
- Contains a copy of Python and all installed packages
- Should NOT be committed to Git (it's in `.gitignore`)
- Can be recreated anytime with `uv sync`

**If Things Go Wrong**

Virtual environment broken? Delete it and recreate:
```bash
rm -rf .venv
uv sync
```

This reinstalls everything from `pyproject.toml`.

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

---

## Git Basics Overview

Git is a **version control system** — it tracks changes to your files over time. Think of it like "save points" in a video game.

### The Three Areas of Git

```
Working Directory → Staging Area → Repository
     (your files)      (ready to save)   (saved history)
```

1. **Working Directory**: The files you see and edit
2. **Staging Area**: Files you've marked to be included in the next save
3. **Repository**: The complete history of all your saves (commits)

### Essential Commands

| Command | What It Does |
|---------|--------------|
| `git status` | Shows what's changed since your last commit |
| `git add <file>` | Stages a file for the next commit |
| `git add .` | Stages all changed files |
| `git commit -m "message"` | Saves staged changes with a description |
| `git log` | Shows the history of commits |
| `git push` | Uploads your commits to GitHub |
| `git pull` | Downloads changes from GitHub |

### A Typical Workflow

1. **Make changes** to your code
2. **Check what changed**: `git status`
3. **Stage the changes**: `git add .`
4. **Commit with a message**: `git commit -m "Add feature X"`
5. **Push to GitHub**: `git push`

You'll learn these commands gradually through the phases. Each phase introduces new Git concepts:

| Phase | Git Concepts |
|-------|--------------|
| 1 | `git status`, `git add`, `git commit`, `git log` |
| 2 | `git diff`, amending commits |
| 3 | `git push`, `git pull`, remotes |
| 4 | Reverting commits, undoing changes |
| 5 | Branches, merging |

### When Things Go Wrong

Don't panic! Git is designed to help you recover:

- **Made a typo in your last commit message?** `git commit --amend`
- **Want to undo uncommitted changes?** `git checkout -- <file>`
- **Need to undo a commit?** `git revert <commit>`

Each phase teaches you more recovery techniques. The beauty of Git is that almost nothing is permanent — you can almost always get back to a good state.
