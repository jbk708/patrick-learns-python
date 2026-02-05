# Phase 6: CLI Integration + Polish

**ATBS Chapters**: 11 (Debugging), 17 (Keeping Time, Scheduling Tasks, Launching Programs)
**Goal**: Combine all pieces into a polished command-line tool
**Deliverable**: `src/roku_sync.py` — the complete CLI application

---

## What You'll Learn

- Building CLIs with `argparse`
- Organizing code into a clean architecture
- Adding proper error handling throughout
- Creating a smooth user experience
- Debugging techniques when things go wrong

---

## Prerequisites

Before starting this lesson:
1. Read ATBS Chapters 11 and 17
2. Complete Phases 1–5 (all scripts working)
3. All previous code passing ruff and ty checks

---

## Part 1: Introduction to argparse

### Concepts to Understand

The `argparse` module lets you build professional command-line interfaces:

```python
import argparse

parser = argparse.ArgumentParser(description="My cool tool")
parser.add_argument("name", help="Your name")
parser.add_argument("--greeting", default="Hello", help="Greeting to use")

args = parser.parse_args()
print(f"{args.greeting}, {args.name}!")
```

Run it:
```bash
uv run python script.py World
# Output: Hello, World!

uv run python script.py World --greeting Hi
# Output: Hi, World!
```

### Exercise 6.1: Basic Argument Parsing

Create `src/roku_sync.py` with a simple argument parser:

```python
import argparse


def main() -> None:
    """Main entry point for the roku-sync CLI."""
    parser = argparse.ArgumentParser(
        description="Sync Roku playback to YouTube on another device"
    )

    # We'll add subcommands next
    args = parser.parse_args()

    print("Roku Sync Tool")
    print("Use --help to see available commands")


if __name__ == "__main__":
    main()
```

Test it with `uv run python src/roku_sync.py --help`.

---

## Part 2: Adding Subcommands

### Concepts to Understand

Professional CLI tools have subcommands:
- `git status`, `git commit`, `git push`
- `docker run`, `docker build`, `docker ps`

We'll create:
- `roku-sync search` — Search YouTube directly
- `roku-sync roku` — Check Roku status
- `roku-sync history` — View search history
- `roku-sync sync` — Full workflow: Roku → YouTube → browser

### Exercise 6.2: Set Up Subcommands

```python
import argparse


def cmd_search(args: argparse.Namespace) -> None:
    """Handle the 'search' subcommand."""
    print(f"Searching for: {args.query}")


def cmd_roku(args: argparse.Namespace) -> None:
    """Handle the 'roku' subcommand."""
    print("Checking Roku status...")


def cmd_history(args: argparse.Namespace) -> None:
    """Handle the 'history' subcommand."""
    print("Showing search history...")


def cmd_sync(args: argparse.Namespace) -> None:
    """Handle the 'sync' subcommand."""
    print("Starting sync workflow...")


def main() -> None:
    """Main entry point for the roku-sync CLI."""
    parser = argparse.ArgumentParser(
        description="Sync Roku playback to YouTube on another device"
    )

    subparsers = parser.add_subparsers(dest="command", help="Available commands")

    # search command
    search_parser = subparsers.add_parser("search", help="Search YouTube")
    search_parser.add_argument("query", nargs="?", help="Search query")
    search_parser.set_defaults(func=cmd_search)

    # roku command
    roku_parser = subparsers.add_parser("roku", help="Check Roku status")
    roku_parser.set_defaults(func=cmd_roku)

    # history command
    history_parser = subparsers.add_parser("history", help="View search history")
    history_parser.add_argument(
        "-n", "--count", type=int, default=5, help="Number of entries"
    )
    history_parser.set_defaults(func=cmd_history)

    # sync command
    sync_parser = subparsers.add_parser("sync", help="Full Roku to YouTube sync")
    sync_parser.set_defaults(func=cmd_sync)

    args = parser.parse_args()

    if hasattr(args, "func"):
        args.func(args)
    else:
        parser.print_help()


if __name__ == "__main__":
    main()
```

Test your subcommands:
```bash
uv run python src/roku_sync.py --help
uv run python src/roku_sync.py search "funny cats"
uv run python src/roku_sync.py roku
```

---

## Part 3: Integrating Your Modules

### Exercise 6.3: Import and Use Existing Code

Now wire up the real functionality. Import from your other modules:

```python
import argparse
import webbrowser

# Import from your other modules
from history import add_search, show_recent
from roku_status import get_active_app, get_device_info, get_playback_status
from youtube_search import (
    create_video_url,
    get_first_video_id,
    load_api_key,
    search_youtube,
)
```

### Exercise 6.4: Implement the Search Command

```python
def cmd_search(args: argparse.Namespace) -> None:
    """Handle the 'search' subcommand."""
    query = args.query
    if not query:
        query = input("Enter search query: ").strip()

    if not query:
        print("Error: Query cannot be empty")
        return

    print(f"Searching YouTube for: {query}")

    try:
        api_key = load_api_key()
    except SystemExit:
        return

    response = search_youtube(query, api_key)
    video_id = get_first_video_id(response)

    if video_id:
        url = create_video_url(video_id)
        print(f"Found: {url}")
        add_search(query, video_id)
        webbrowser.open(url)
    else:
        print("No videos found")
        add_search(query, None)
```

### Exercise 6.5: Implement the Roku Command

```python
def get_roku_ip_from_config() -> str | None:
    """Load Roku IP from config, return None if not set."""
    import json
    from pathlib import Path

    config_path = Path(__file__).parent.parent / "config.json"
    if not config_path.exists():
        return None

    with open(config_path, "r") as f:
        config = json.load(f)

    return config.get("roku_ip")


def cmd_roku(args: argparse.Namespace) -> None:
    """Handle the 'roku' subcommand."""
    roku_ip = get_roku_ip_from_config()
    if not roku_ip:
        roku_ip = input("Enter Roku IP address: ").strip()

    print(f"\nConnecting to Roku at {roku_ip}...")

    device = get_device_info(roku_ip)
    if not device:
        print("Could not connect to Roku")
        return

    print(f"\nDevice: {device['name']} ({device['model']})")

    app = get_active_app(roku_ip)
    if app:
        print(f"Active App: {app['name']}")
    else:
        print("No app running")

    playback = get_playback_status(roku_ip)
    if playback:
        print(f"Playback State: {playback['state']}")
```

### Exercise 6.6: Implement the History Command

```python
def cmd_history(args: argparse.Namespace) -> None:
    """Handle the 'history' subcommand."""
    show_recent(args.count)
```

### Exercise 6.7: Implement the Sync Command

This is the main workflow — the whole point of the project!

```python
def cmd_sync(args: argparse.Namespace) -> None:
    """Handle the 'sync' subcommand - full Roku to YouTube workflow."""
    # Step 1: Connect to Roku
    roku_ip = get_roku_ip_from_config()
    if not roku_ip:
        roku_ip = input("Enter Roku IP address: ").strip()

    print(f"\nConnecting to Roku at {roku_ip}...")

    app = get_active_app(roku_ip)
    if not app:
        print("Could not connect to Roku or no app running")
        print("Please enter what you're watching manually.")
        app_name = "Unknown"
    else:
        app_name = app["name"]
        print(f"You're watching: {app_name}")

    # Step 2: Get title from user (Roku can't tell us this)
    print("\nRoku can't tell us the specific video title.")
    query = input("What are you watching? Enter title/description: ").strip()

    if not query:
        print("Error: Query cannot be empty")
        return

    # Optionally include app name in search
    if app_name != "Unknown":
        include_app = input(f"Include '{app_name}' in search? (y/n): ").lower()
        if include_app == "y":
            query = f"{query} {app_name}"

    # Step 3: Search YouTube
    print(f"\nSearching YouTube for: {query}")

    try:
        api_key = load_api_key()
    except SystemExit:
        return

    response = search_youtube(query, api_key)
    video_id = get_first_video_id(response)

    if video_id:
        url = create_video_url(video_id)
        print(f"\nFound: {url}")
        add_search(query, video_id)

        open_browser = input("Open in browser? (y/n): ").lower()
        if open_browser == "y":
            webbrowser.open(url)
    else:
        print("No videos found")
        add_search(query, None)
```

---

## Part 4: Error Handling

### Concepts to Understand

Things go wrong. Networks fail, APIs timeout, files get deleted. Good error handling:
1. Catches errors before they crash the program
2. Tells the user what went wrong
3. Suggests how to fix it

### Exercise 6.8: Add Error Handling

Wrap network calls in try/except:

```python
def cmd_search(args: argparse.Namespace) -> None:
    """Handle the 'search' subcommand."""
    # ... query setup code ...

    try:
        api_key = load_api_key()
        response = search_youtube(query, api_key)
    except Exception as e:
        print(f"Error: {e}")
        print("Check your internet connection and API key.")
        return

    # ... rest of function ...
```

---

## Part 5: Polish and User Experience

### Exercise 6.9: Add a Welcome Message

```python
def main() -> None:
    """Main entry point for the roku-sync CLI."""
    parser = argparse.ArgumentParser(
        description="Sync Roku playback to YouTube on another device",
        epilog="Example: roku-sync search 'funny cats'",
    )

    # If no arguments, show a friendly message
    import sys
    if len(sys.argv) == 1:
        print("Roku YouTube Sync")
        print("=" * 40)
        print("\nCommands:")
        print("  search  - Search YouTube directly")
        print("  roku    - Check Roku device status")
        print("  history - View your search history")
        print("  sync    - Full workflow: Roku → YouTube")
        print("\nRun 'roku-sync <command> --help' for more info")
        return

    # ... rest of main() ...
```

### Exercise 6.10: Add Version Flag

```python
parser.add_argument(
    "--version", action="version", version="roku-sync 1.0.0"
)
```

---

## Part 6: Making It a Real Command

### Concepts to Understand

Right now you run: `uv run python src/roku_sync.py`

Wouldn't it be nicer to just run: `roku-sync`?

You can configure this in `pyproject.toml`:

```toml
[project.scripts]
roku-sync = "roku_sync:main"
```

But this requires restructuring as a proper Python package, which is more advanced. For now, you can create a simple wrapper script or alias.

### Exercise 6.11: Create a Run Script (Windows)

Create `run.bat` in the project root:

```batch
@echo off
uv run python src/roku_sync.py %*
```

Now you can run: `run search "funny cats"`

---

## Testing Your Work

Test each command:

```bash
uv run python src/roku_sync.py --help
uv run python src/roku_sync.py search "test query"
uv run python src/roku_sync.py roku
uv run python src/roku_sync.py history -n 3
uv run python src/roku_sync.py sync
```

Checklist:
- [ ] `--help` shows all commands
- [ ] `search` works with and without query argument
- [ ] `roku` connects and shows device info
- [ ] `history` shows recent searches
- [ ] `sync` runs the full workflow
- [ ] Errors are handled gracefully (try wrong Roku IP, no API key)
- [ ] `uv run ruff check src/` passes
- [ ] `uv run ty check src/` passes

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| ImportError for your modules | Run from project root, or check your import paths |
| "command not recognized" | Make sure you're using subcommands: `roku-sync search` not just `roku-sync` |
| argparse errors | Check argument names match in both parser setup and function access |

---

## Congratulations!

You've built a complete CLI application that:
1. Connects to a local network device (Roku)
2. Queries a web API (YouTube)
3. Manages persistent data (history)
4. Provides a professional command-line interface

This is real software engineering!

---

## What's Next

**Phase 7** (optional) explores advanced workarounds for the Roku metadata limitation — things like packet inspection, Roku developer mode, or building a web UI.

---

## Commit Your Work

```bash
git add src/roku_sync.py
git commit -m "Phase 6: Complete CLI with argparse and full workflow"
```
