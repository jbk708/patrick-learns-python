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

---

## Prerequisites

Before starting this lesson:
1. Read ATBS Chapters 9–10
2. Complete Phase 3 (working YouTube API search)
3. Your `youtube_search.py` should pass ruff and ty checks

---

## Part 1: The Path Problem

### Concepts to Understand

In Phase 3, we used `"config.json"` as a file path. This only works if you run the script from the project root directory. If someone runs it from elsewhere, it breaks.

The `pathlib` module gives us reliable, cross-platform paths:

```python
from pathlib import Path

# Get the directory where this script lives
script_dir = Path(__file__).parent

# Build a path relative to the script
config_path = script_dir / "config.json"
```

The `/` operator with `Path` objects builds paths correctly on Windows (`\`) and Mac/Linux (`/`).

### Exercise 4.1: Fix the Config Path

Update `youtube_search.py` to:
1. Import `Path` from `pathlib`
2. Calculate the script's directory
3. Build the config path relative to the script

```python
from pathlib import Path

def get_project_root() -> Path:
    """Get the project root directory.

    Returns:
        Path to the project root (parent of src/).
    """
    return Path(__file__).parent.parent


def load_api_key() -> str:
    """Load the YouTube API key from config.json."""
    config_path = get_project_root() / "config.json"
    with open(config_path, "r") as f:
        config = json.load(f)
    return config["youtube_api_key"]
```

Now the script works from any directory!

---

## Part 2: Creating a History File

### Concepts to Understand

We'll store search history in a JSON file. JSON is perfect for this because:
- It's human-readable
- Python can read/write it easily
- It handles nested data (lists, dictionaries)

Our history file format:
```json
{
    "searches": [
        {
            "query": "funny cat videos",
            "video_id": "abc123",
            "timestamp": "2024-01-15T10:30:00"
        }
    ]
}
```

### Exercise 4.2: Write History

Create a new file `src/history.py`:

```python
import json
from datetime import datetime
from pathlib import Path


def get_history_path() -> Path:
    """Get the path to the history file.

    Returns:
        Path to history.json in the project root.
    """
    return Path(__file__).parent.parent / "history.json"


def load_history() -> dict:
    """Load search history from file.

    Returns:
        The history dictionary, or empty structure if file doesn't exist.
    """
    path = get_history_path()
    if not path.exists():
        return {"searches": []}

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


def add_search(query: str, video_id: str | None) -> None:
    """Add a search to the history.

    Args:
        query: The search query.
        video_id: The video ID found, or None if no results.
    """
    history = load_history()
    history["searches"].append({
        "query": query,
        "video_id": video_id,
        "timestamp": datetime.now().isoformat()
    })
    save_history(history)
```

---

## Part 3: Reading History

### Exercise 4.3: Display Recent Searches

Add a function to show recent searches:

```python
def show_recent(count: int = 5) -> None:
    """Print the most recent searches.

    Args:
        count: Number of recent searches to show.
    """
    history = load_history()
    searches = history["searches"]

    if not searches:
        print("No search history yet.")
        return

    print(f"\nLast {min(count, len(searches))} searches:")
    for search in searches[-count:]:
        query = search["query"]
        video_id = search.get("video_id", "no result")
        print(f"  - {query} ({video_id})")
```

The `[-count:]` slice gets the last `count` items from the list.

---

## Part 4: Integrating History

### Exercise 4.4: Update youtube_search.py

Modify your `youtube_search.py` to:
1. Import from `history.py`
2. Save each search to history after getting results
3. Add an option to show recent searches

```python
from history import add_search, show_recent


def main() -> None:
    """Search YouTube and open the first result."""
    # Add a simple menu
    print("1. New search")
    print("2. Show recent searches")
    choice = input("Choose (1/2): ").strip()

    if choice == "2":
        show_recent()
        return

    # ... rest of your existing search code ...

    # After finding video_id:
    add_search(query, video_id)
```

---

## Part 5: Config Example File

### Concepts to Understand

Your `config.json` contains your secret API key and shouldn't be committed to git. But new users need to know the format. Solution: create an example file.

### Exercise 4.5: Create config.json.example

Create a file `config.json.example` in the project root:

```json
{
    "youtube_api_key": "YOUR_API_KEY_HERE"
}
```

This file IS committed to git. It shows users what `config.json` should look like.

Update your `.gitignore` to ensure `config.json` is ignored but `config.json.example` is not:
```
config.json
```

---

## Part 6: Better Error Messages

### Exercise 4.6: Handle Missing Config

What if someone runs the script without creating `config.json`? Right now, they get a confusing `FileNotFoundError`. Let's improve that:

```python
def load_api_key() -> str:
    """Load the YouTube API key from config.json."""
    config_path = get_project_root() / "config.json"

    if not config_path.exists():
        print("Error: config.json not found!")
        print("Copy config.json.example to config.json and add your API key.")
        raise SystemExit(1)

    with open(config_path, "r") as f:
        config = json.load(f)

    if "youtube_api_key" not in config:
        print("Error: youtube_api_key not found in config.json!")
        raise SystemExit(1)

    return config["youtube_api_key"]
```

`raise SystemExit(1)` stops the program with an error code.

---

## Part 7: Final Structure

After this phase, your project should look like:

```
roku-youtube-sync/
├── config.json           # Your actual config (gitignored)
├── config.json.example   # Template for new users
├── history.json          # Search history (created automatically)
├── src/
│   ├── search_url.py     # Phase 1-2 code
│   ├── youtube_search.py # Phase 3-4 code
│   └── history.py        # NEW: History module
└── ...
```

---

## Testing Your Work

- [ ] Script works when run from any directory
- [ ] `config.json.example` exists with placeholder key
- [ ] Searches are saved to `history.json`
- [ ] "Show recent searches" displays history
- [ ] Missing `config.json` shows a helpful error message
- [ ] `uv run ruff check src/` passes
- [ ] `uv run ty check src/` passes

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| `history.json` in wrong location | Check your `get_history_path()` returns project root, not `src/` |
| Import error for `history` | Make sure the import is `from history import ...` not `from src.history import ...` |
| History not persisting | Check that `save_history()` is being called after modifying the dict |
| Path errors on Windows | Make sure you're using `pathlib.Path`, not string concatenation |

---

## What's Next

In **Phase 5**, you'll connect to your Roku device to see what's currently playing. You'll make HTTP requests to the Roku's ECP interface and parse XML responses.

---

## Commit Your Work

```bash
git add src/history.py src/youtube_search.py config.json.example
git commit -m "Phase 4: Add search history and robust file paths"
```
