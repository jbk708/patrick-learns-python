# Phase 4: Persistent History

**ATBS Chapters**: 9–10 (Reading and Writing Files, Organizing Files)
**Goal**: Save search history and improve file organization
**Deliverable**: Updated scripts with history feature, `config.json.example`

---

## What You'll Learn

- Reading and writing JSON files
- Using `pathlib` for cross-platform file paths
- Organizing code into reusable modules
- Creating example/template files for configuration
- Working with dates and times

---

## Prerequisites

Before starting this lesson:

1. **Read ATBS Chapters 9–10** — These chapters cover:
   - Chapter 9: Reading and writing files
   - Chapter 10: Organizing files (creating, moving, copying)

2. **Complete Phase 3** — Your `youtube_search.py` should:
   - Successfully search YouTube and open videos
   - Pass `uv run ruff check src/`
   - Pass `uv run ty check src/`

3. **Verify your Phase 3 code works**:
   ```bash
   uv run python src/youtube_search.py
   ```

---

## Part 1: The Path Problem

### What's Wrong with Our Current Code?

In Phase 3, we used `"config.json"` as a file path:

```python
with open("config.json", "r") as f:
    config = json.load(f)
```

**The problem**: This only works if you run the script from the project's root directory. Let's see why:

**If you run from the project root:**
```bash
cd C:\Users\Patrick\roku-youtube-sync
uv run python src/youtube_search.py
```
The script looks for `config.json` in `C:\Users\Patrick\roku-youtube-sync\config.json` ✓

**If you run from the src folder:**
```bash
cd C:\Users\Patrick\roku-youtube-sync\src
uv run python youtube_search.py
```
The script looks for `config.json` in `C:\Users\Patrick\roku-youtube-sync\src\config.json` ✗

The second case fails because `config.json` isn't in the `src` folder!

### The Solution: pathlib

> 📖 **Reference**: ATBS Chapter 9, "The pathlib Module" section

The `pathlib` module lets us build file paths that:
1. Work on any operating system (Windows, Mac, Linux)
2. Are relative to where the script file is located

```python
from pathlib import Path

# __file__ is a special variable that contains this script's path
script_path = Path(__file__)
print(script_path)  # e.g., C:\Users\Patrick\roku-youtube-sync\src\youtube_search.py

# .parent gets the directory containing the file
script_dir = script_path.parent
print(script_dir)  # e.g., C:\Users\Patrick\roku-youtube-sync\src

# .parent again goes up one more level
project_root = script_dir.parent
print(project_root)  # e.g., C:\Users\Patrick\roku-youtube-sync
```

### Understanding `__file__`

`__file__` is a special variable that Python creates automatically. It contains the path to the current script file. This works no matter where you run the script from!

### Building Paths with `/`

With `Path` objects, you can use `/` to join paths (even on Windows!):

```python
from pathlib import Path

root = Path("/Users/Patrick/project")
config = root / "config.json"
print(config)  # /Users/Patrick/project/config.json

# On Windows, it automatically uses backslashes when needed
data_dir = root / "data" / "files"
print(data_dir)  # C:\Users\Patrick\project\data\files (on Windows)
```

This is much cleaner than string concatenation:
```python
# Don't do this:
config = root + "\\config.json"  # Only works on Windows
config = root + "/config.json"   # Only works on Mac/Linux

# Do this instead:
config = root / "config.json"    # Works everywhere!
```

### Exercise 4.1: Fix the Config Path

**What to do:**

1. Open `src/youtube_search.py`

2. Add this function near the top (after the imports):

```python
from pathlib import Path


def get_project_root() -> Path:
    """Get the project root directory.

    The project root is the parent of the src/ folder where this script lives.

    Returns:
        Path to the project root directory.
    """
    # __file__ is the path to this script (youtube_search.py)
    # .parent gets the directory (src/)
    # .parent again gets the parent of that (project root)
    return Path(__file__).parent.parent
```

3. Update `load_api_key()` to use the new function:

```python
def load_api_key() -> str:
    """Load the YouTube API key from config.json.

    Returns:
        The API key string.

    Raises:
        SystemExit: If config.json is missing or doesn't contain the key.
    """
    config_path = get_project_root() / "config.json"

    if not config_path.exists():
        print("Error: config.json not found!")
        print(f"Expected location: {config_path}")
        print("Copy config.json.example to config.json and add your API key.")
        raise SystemExit(1)

    with open(config_path, "r") as f:
        config = json.load(f)

    if "youtube_api_key" not in config:
        print("Error: youtube_api_key not found in config.json!")
        raise SystemExit(1)

    return config["youtube_api_key"]
```

4. Test it by running from different directories:
   ```bash
   # From project root
   uv run python src/youtube_search.py

   # From src folder
   cd src
   uv run python youtube_search.py
   cd ..
   ```

Both should work now!

---

## Part 2: Understanding File I/O

### Reading and Writing Files

> 📖 **Reference**: ATBS Chapter 9, "Reading and Writing Files" section

Python uses `open()` to work with files. The `with` statement ensures the file is properly closed:

```python
# Reading a file
with open("data.txt", "r") as f:  # "r" = read mode
    content = f.read()

# Writing to a file
with open("data.txt", "w") as f:  # "w" = write mode (overwrites!)
    f.write("Hello, world!")

# Appending to a file
with open("data.txt", "a") as f:  # "a" = append mode (adds to end)
    f.write("\nAnother line")
```

**Important modes:**
- `"r"` — read (file must exist)
- `"w"` — write (creates file, overwrites if exists)
- `"a"` — append (creates file, adds to end if exists)

### Working with JSON Files

The `json` module has two main functions:
- `json.load(file)` — read JSON from a file
- `json.dump(data, file)` — write JSON to a file

```python
import json

# Writing JSON
data = {"name": "Patrick", "score": 100}
with open("data.json", "w") as f:
    json.dump(data, f, indent=4)  # indent=4 makes it readable

# Reading JSON
with open("data.json", "r") as f:
    loaded = json.load(f)
print(loaded["name"])  # Patrick
```

The `indent=4` parameter adds nice formatting:

**Without indent:**
```json
{"name": "Patrick", "score": 100}
```

**With indent=4:**
```json
{
    "name": "Patrick",
    "score": 100
}
```

---

## Part 3: Creating a History Module

### Why a Separate Module?

As your code grows, it's helpful to split it into separate files (modules). This:
1. Makes each file smaller and easier to understand
2. Lets you reuse code across different scripts
3. Keeps related functions together

We'll create `history.py` to handle all search history operations.

### Exercise 4.2: Create the History Module

**What to do:**

1. Create a new file `src/history.py`

2. Add the following code:

```python
"""Search history management.

This module handles saving and loading search history to/from a JSON file.
"""
import json
from datetime import datetime
from pathlib import Path


def get_project_root() -> Path:
    """Get the project root directory.

    Returns:
        Path to the project root (parent of src/).
    """
    return Path(__file__).parent.parent


def get_history_path() -> Path:
    """Get the path to the history file.

    Returns:
        Path to history.json in the project root.
    """
    return get_project_root() / "history.json"


def load_history() -> dict:
    """Load search history from file.

    If the history file doesn't exist, returns an empty structure.

    Returns:
        The history dictionary with a "searches" list.
    """
    path = get_history_path()

    # If the file doesn't exist, return empty history
    if not path.exists():
        return {"searches": []}

    # Read and parse the JSON file
    with open(path, "r") as f:
        return json.load(f)


def save_history(history: dict) -> None:
    """Save search history to file.

    Args:
        history: The history dictionary to save.
    """
    path = get_history_path()

    with open(path, "w") as f:
        json.dump(history, f, indent=4)
```

3. Save the file

**Understanding the code:**

- `path.exists()` — returns `True` if the file exists, `False` otherwise
- If the file doesn't exist, we return an empty structure instead of crashing
- `json.dump(history, f, indent=4)` — writes the dictionary to the file with nice formatting

---

## Part 4: Adding Searches to History

### Working with Dates and Times

> 📖 **Reference**: This uses Python's built-in `datetime` module.

The `datetime` module lets us work with dates and times:

```python
from datetime import datetime

# Get the current date and time
now = datetime.now()
print(now)  # 2024-01-15 10:30:45.123456

# Convert to a string format (ISO format is standard)
timestamp = now.isoformat()
print(timestamp)  # 2024-01-15T10:30:45.123456
```

ISO format (`2024-01-15T10:30:45`) is a standard way to represent dates that:
- Sorts correctly when stored as text
- Is understood worldwide (no ambiguity about month/day order)
- Can be parsed back into a datetime object

### Exercise 4.3: Add the Search Recording Function

**What to do:**

Add this function to `src/history.py`:

```python
def add_search(query: str, video_id: str | None) -> None:
    """Add a search to the history.

    Args:
        query: The search query the user entered.
        video_id: The video ID found, or None if no results.
    """
    # Load existing history
    history = load_history()

    # Create a new search record
    search_record = {
        "query": query,
        "video_id": video_id,
        "timestamp": datetime.now().isoformat()
    }

    # Add it to the list
    history["searches"].append(search_record)

    # Save back to file
    save_history(history)
```

**Understanding the code:**

- We load the existing history (or empty structure if no file)
- Create a new dictionary with the search info
- `append()` adds the record to the end of the list
- Save the updated history back to file

**After a few searches, history.json might look like:**

```json
{
    "searches": [
        {
            "query": "funny cat videos",
            "video_id": "abc123",
            "timestamp": "2024-01-15T10:30:45.123456"
        },
        {
            "query": "python tutorials",
            "video_id": "xyz789",
            "timestamp": "2024-01-15T10:31:20.654321"
        }
    ]
}
```

---

## Part 5: Displaying Recent Searches

### Understanding List Slicing

> 📖 **Reference**: ATBS Chapter 4 covers list slicing.

You can get portions of a list using **slicing**:

```python
items = ["a", "b", "c", "d", "e"]

items[0:3]   # First 3 items: ["a", "b", "c"]
items[:3]    # Same as above (0 is optional)
items[2:]    # From index 2 to end: ["c", "d", "e"]
items[-2:]   # Last 2 items: ["d", "e"]
items[-3:]   # Last 3 items: ["c", "d", "e"]
```

The `-count:` syntax is perfect for getting the "most recent" items.

### Exercise 4.4: Add the Display Function

**What to do:**

Add this function to `src/history.py`:

```python
def show_recent(count: int = 5) -> None:
    """Print the most recent searches.

    Args:
        count: Maximum number of recent searches to show. Defaults to 5.
    """
    history = load_history()
    searches = history["searches"]

    if not searches:
        print("No search history yet.")
        return

    # Get the last 'count' searches
    recent = searches[-count:]

    # Display them
    print(f"\nLast {len(recent)} searches:")
    for search in recent:
        query = search["query"]
        video_id = search.get("video_id", "no result")
        timestamp = search.get("timestamp", "unknown time")
        # Just show the date part for cleaner output
        date_part = timestamp.split("T")[0] if "T" in timestamp else timestamp
        print(f"  [{date_part}] {query} → {video_id}")
```

**Understanding the code:**

- `searches[-count:]` — gets the last `count` items (or all if fewer exist)
- `len(recent)` — actual number of items (might be less than `count`)
- `search.get("video_id", "no result")` — safely get value with a default
- `timestamp.split("T")[0]` — splits `"2024-01-15T10:30:45"` at the `T` and takes the date part

**Example output:**

```
Last 3 searches:
  [2024-01-15] funny cat videos → abc123
  [2024-01-15] python tutorials → xyz789
  [2024-01-15] javascript basics → None
```

---

## Part 6: Integrating History into the Main Script

### Importing from Your Own Modules

When you have a file `src/history.py`, you can import from it like this:

```python
from history import add_search, show_recent
```

This works because both files are in the same `src/` directory. Python looks for modules in the same folder as the running script.

### Exercise 4.5: Update youtube_search.py

**What to do:**

1. Open `src/youtube_search.py`

2. Add the import at the top (with the other imports):

```python
from history import add_search, show_recent
```

3. Update the `main()` function to include a menu and history:

```python
def main() -> None:
    """Search YouTube and open the first result in the browser."""
    # Show menu
    print("\nYouTube Search")
    print("=" * 30)
    print("1. New search")
    print("2. Show recent searches")
    print("=" * 30)

    choice = input("Choose an option (1 or 2): ").strip()

    if choice == "2":
        show_recent()
        return

    if choice != "1":
        print("Invalid choice. Please enter 1 or 2.")
        return

    # Load the API key from config
    api_key = load_api_key()

    # Get search query from user
    query = get_search_query()
    print(f"Searching YouTube for: {query}")

    # Search YouTube
    response = search_youtube(query, api_key)

    # Extract the first video ID
    video_id = get_first_video_id(response)

    # Save to history (whether found or not)
    add_search(query, video_id)

    # Handle results
    if video_id is None:
        print("No videos found for that search.")
    else:
        url = create_video_url(video_id)
        print(f"Found video: {url}")
        print("Opening in browser...")
        webbrowser.open(url)
```

4. Save and test:
   ```bash
   uv run python src/youtube_search.py
   ```

**Test these scenarios:**
- Choose option 1, search for something — should save to history
- Choose option 2 — should show your recent searches
- Check that `history.json` was created in your project root

---

## Part 7: Creating the Config Example File

### Why an Example File?

Your `config.json` contains your secret API key. You should **never** commit secrets to Git because:
- Anyone who can see your repository can see your key
- Even after deleting, it stays in Git history forever
- Bots scan GitHub for leaked API keys!

The solution:
1. Add `config.json` to `.gitignore` (so it's never committed)
2. Create `config.json.example` (a template showing the format)
3. Tell users to copy the example and add their own key

### Exercise 4.6: Create the Example File

**What to do:**

1. Create a file called `config.json.example` in your project root (same level as `src/`):

```json
{
    "youtube_api_key": "YOUR_API_KEY_HERE"
}
```

2. Make sure your `.gitignore` includes `config.json`:

```
config.json
history.json
__pycache__/
*.pyc
```

3. Update the error message in `load_api_key()` to reference the example file:

```python
if not config_path.exists():
    print("Error: config.json not found!")
    print(f"Expected location: {config_path}")
    print()
    print("To fix this:")
    print("1. Copy config.json.example to config.json")
    print("2. Replace YOUR_API_KEY_HERE with your actual API key")
    raise SystemExit(1)
```

---

## Part 8: Final Project Structure

After completing this phase, your project should look like this:

```
roku-youtube-sync/
├── .gitignore                # Ignores config.json and history.json
├── config.json               # Your actual config (NOT in git)
├── config.json.example       # Template for new users (IN git)
├── history.json              # Search history (NOT in git, auto-created)
├── pyproject.toml            # Project dependencies
├── src/
│   ├── search_url.py         # Phase 1-2 code
│   ├── youtube_search.py     # Phase 3-4 code
│   └── history.py            # NEW: History module
└── lessons/
    └── ...
```

---

## Testing Your Work

Run through this checklist:

- [ ] Script works when run from project root: `uv run python src/youtube_search.py`
- [ ] Script works when run from src folder: `cd src && uv run python youtube_search.py`
- [ ] `config.json.example` exists with placeholder key
- [ ] After a search, `history.json` is created/updated
- [ ] Option 2 shows recent searches
- [ ] Missing `config.json` shows a helpful error message
- [ ] `uv run ruff check src/` passes
- [ ] `uv run ty check src/` passes

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| `history.json` in wrong location | Check that `get_history_path()` uses `.parent.parent` to get project root |
| `ModuleNotFoundError: No module named 'history'` | Run from project root OR make sure both files are in `src/` |
| History not saving | Make sure `add_search()` is called after getting results, and `save_history()` is called in `add_search()` |
| Path errors on Windows | Use `pathlib.Path` and `/` operator, never string concatenation with `\\` |
| `TypeError: Object of type WindowsPath is not JSON serializable` | Convert Path to string: `str(path)` when printing or saving to JSON |

---

## Vocabulary Review

| Term | Definition |
|------|------------|
| **pathlib** | Python module for working with file system paths |
| **Path** | A class representing a file system path |
| **__file__** | Special variable containing the current script's file path |
| **.parent** | Gets the parent directory of a Path |
| **exists()** | Method to check if a file or directory exists |
| **with statement** | Ensures files are properly closed after use |
| **json.load()** | Read JSON from a file into a Python object |
| **json.dump()** | Write a Python object to a file as JSON |
| **datetime** | Python module for working with dates and times |
| **isoformat()** | Convert a datetime to a standard string format |
| **List slicing** | Getting a portion of a list using `[start:end]` syntax |
| **Module** | A Python file that can be imported into other files |
| **.gitignore** | File that tells Git which files to not track |

---

## What's Next

In **Phase 5**, you'll connect to your Roku device to see what's currently playing. You'll learn:
- How to make HTTP requests to local network devices
- How to parse XML responses (Roku uses XML, not JSON)
- Discovering platform limitations firsthand

---

## Git: Undoing Changes and Reverting Commits

Everyone makes mistakes. Git gives you powerful tools to undo changes — whether you haven't committed yet, or you need to undo a commit you already made.

### Scenario 1: Undo Uncommitted Changes in a File

You edited a file, but you want to discard those changes and go back to the last commit:

```bash
git checkout -- src/history.py
```

**Warning**: This permanently discards your uncommitted changes! There's no undo for this.

**Newer syntax (Git 2.23+):**
```bash
git restore src/history.py
```

Both commands do the same thing.

### Scenario 2: Unstage a File

You ran `git add` but changed your mind:

```bash
git reset HEAD src/history.py
```

This removes the file from the staging area but keeps your changes in the working directory.

**Newer syntax:**
```bash
git restore --staged src/history.py
```

### Scenario 3: Undo a Commit (Safe Way): `git revert`

You made a commit, pushed it, and now you want to undo it. The safe way is `git revert`:

```bash
git revert HEAD
```

This creates a **new commit** that undoes the changes from the previous commit. The history shows:
1. Original commit (the mistake)
2. Revert commit (the undo)

**Why is this "safe"?** It doesn't rewrite history. If you've already pushed, this won't cause problems for others.

**Reverting an older commit:**
```bash
git log --oneline  # Find the commit ID
git revert abc1234  # Revert that specific commit
```

### Scenario 4: Undo a Commit (Rewrite History): `git reset`

If you haven't pushed yet and want to undo your last commit:

**Keep the changes (just undo the commit):**
```bash
git reset --soft HEAD~1
```

Your changes are still staged, ready to commit again.

**Unstage the changes too:**
```bash
git reset HEAD~1
```

Your changes are still in your files, just unstaged.

**Discard everything (dangerous!):**
```bash
git reset --hard HEAD~1
```

**Warning**: `--hard` permanently discards your changes! Use with caution.

**What's `HEAD~1`?** It means "one commit before HEAD (the current commit)."
- `HEAD~2` = two commits back
- `HEAD~3` = three commits back

### When to Use Each

| Situation | Command |
|-----------|---------|
| Discard uncommitted changes in a file | `git checkout -- <file>` or `git restore <file>` |
| Unstage a file | `git reset HEAD <file>` or `git restore --staged <file>` |
| Undo a commit you already pushed | `git revert HEAD` |
| Undo a commit you haven't pushed | `git reset HEAD~1` |
| Nuclear option: reset everything | `git reset --hard HEAD~1` (careful!) |

### Exercise: Practice Undoing (Optional)

Try this in a safe way:

1. Make a small change to `src/history.py` (add a comment)
2. Check the change: `git diff`
3. Undo it: `git checkout -- src/history.py`
4. Verify it's gone: `git diff` (should show nothing)

### Commit Your Work

Once everything works and passes the checks:

**Step 1: Review changes**
```bash
git status
git diff
```

**Step 2: Stage and commit**
```bash
git add src/history.py src/youtube_search.py config.json.example .gitignore
git commit -m "Phase 4: Add search history and robust file paths"
```

**Step 3: Push to GitHub**
```bash
git push
```

### Git Commands Learned So Far

| Command | What It Does |
|---------|--------------|
| `git status` | Show what's changed |
| `git add <file>` | Stage changes for commit |
| `git commit -m "msg"` | Save staged changes |
| `git log` | Show commit history |
| `git diff` | Show changes line-by-line |
| `git commit --amend` | Fix your last commit |
| `git push` | Upload commits to GitHub |
| `git pull` | Download commits from GitHub |
| `git checkout -- <file>` | Discard uncommitted changes |
| `git restore <file>` | Discard uncommitted changes (newer syntax) |
| `git revert <commit>` | Create a commit that undoes another commit |
| `git reset HEAD~1` | Undo the last commit (keep changes) |

You can now track your search history and recover from mistakes!
