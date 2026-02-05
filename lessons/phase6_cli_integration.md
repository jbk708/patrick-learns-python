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

1. **Read ATBS Chapters 11 and 17** — These chapters cover:
   - Chapter 11: Debugging techniques and error handling
   - Chapter 17: Launching programs and automation

2. **Complete Phases 1–5** — You should have:
   - `src/search_url.py` — URL generator with input validation
   - `src/youtube_search.py` — YouTube API integration
   - `src/history.py` — Search history management
   - `src/roku_status.py` — Roku device communication

3. **All code passing checks**:
   ```bash
   uv run ruff check src/
   uv run ty check src/
   ```

---

## Part 1: Introduction to argparse

### What is a CLI?

> 📖 **Reference**: This extends concepts from ATBS into professional tool building.

A **CLI** (Command-Line Interface) is a program you interact with by typing commands. You've been using CLIs already:
- `git status`, `git commit`, `git push`
- `uv run python script.py`
- `pip install package`

Professional CLIs have:
- **Subcommands** — Different actions (like `git status` vs `git commit`)
- **Arguments** — Values you pass in (like `git commit -m "message"`)
- **Options/Flags** — Settings you can toggle (like `-m`, `--help`, `-v`)
- **Help text** — Documentation you can access with `--help`

### The argparse Module

Python's `argparse` module helps you build CLIs:

```python
import argparse

# Create a parser
parser = argparse.ArgumentParser(description="My cool tool")

# Add an argument (required)
parser.add_argument("name", help="Your name")

# Add an option (optional, starts with -)
parser.add_argument("--greeting", default="Hello", help="Greeting to use")

# Parse the command line
args = parser.parse_args()

# Use the values
print(f"{args.greeting}, {args.name}!")
```

**Running it:**
```bash
uv run python script.py World
# Output: Hello, World!

uv run python script.py World --greeting Hi
# Output: Hi, World!

uv run python script.py --help
# Shows help text
```

### Subcommands

Real tools have subcommands — think `git status`, `git commit`, `git push`. Each subcommand does something different.

```python
import argparse

parser = argparse.ArgumentParser(description="My tool")
subparsers = parser.add_subparsers(dest="command", help="Available commands")

# Create a "greet" subcommand
greet_parser = subparsers.add_parser("greet", help="Greet someone")
greet_parser.add_argument("name", help="Name to greet")

# Create a "bye" subcommand
bye_parser = subparsers.add_parser("bye", help="Say goodbye")
bye_parser.add_argument("name", help="Name to say bye to")

args = parser.parse_args()

if args.command == "greet":
    print(f"Hello, {args.name}!")
elif args.command == "bye":
    print(f"Goodbye, {args.name}!")
else:
    parser.print_help()
```

**Running it:**
```bash
uv run python script.py greet Patrick
# Output: Hello, Patrick!

uv run python script.py bye Patrick
# Output: Goodbye, Patrick!

uv run python script.py --help
# Shows all available commands
```

### Exercise 6.1: Basic Argument Parsing

**What to do:**

1. Create a new file `src/roku_sync.py`

2. Add this starter code:

```python
"""Roku YouTube Sync CLI.

A command-line tool that syncs Roku playback to YouTube on another device.
"""
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

3. Test it:
   ```bash
   uv run python src/roku_sync.py
   uv run python src/roku_sync.py --help
   ```

---

## Part 2: Adding Subcommands

### Our Tool's Commands

We'll create these subcommands:
- `search` — Search YouTube directly
- `roku` — Check Roku status
- `history` — View search history
- `sync` — Full workflow: Roku → YouTube → browser

### Exercise 6.2: Set Up Subcommands

**What to do:**

Update your `src/roku_sync.py`:

```python
"""Roku YouTube Sync CLI.

A command-line tool that syncs Roku playback to YouTube on another device.
"""
import argparse


def cmd_search(args: argparse.Namespace) -> None:
    """Handle the 'search' subcommand.

    Args:
        args: Parsed command-line arguments.
    """
    print(f"Searching for: {args.query}")


def cmd_roku(args: argparse.Namespace) -> None:
    """Handle the 'roku' subcommand.

    Args:
        args: Parsed command-line arguments.
    """
    print("Checking Roku status...")


def cmd_history(args: argparse.Namespace) -> None:
    """Handle the 'history' subcommand.

    Args:
        args: Parsed command-line arguments.
    """
    print("Showing search history...")


def cmd_sync(args: argparse.Namespace) -> None:
    """Handle the 'sync' subcommand.

    Args:
        args: Parsed command-line arguments.
    """
    print("Starting sync workflow...")


def main() -> None:
    """Main entry point for the roku-sync CLI."""
    parser = argparse.ArgumentParser(
        description="Sync Roku playback to YouTube on another device"
    )

    # Create subcommands
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
        "-n", "--count", type=int, default=5, help="Number of entries to show"
    )
    history_parser.set_defaults(func=cmd_history)

    # sync command
    sync_parser = subparsers.add_parser("sync", help="Full Roku to YouTube sync")
    sync_parser.set_defaults(func=cmd_sync)

    # Parse and execute
    args = parser.parse_args()

    if hasattr(args, "func"):
        args.func(args)
    else:
        parser.print_help()


if __name__ == "__main__":
    main()
```

**Test your subcommands:**
```bash
uv run python src/roku_sync.py --help
uv run python src/roku_sync.py search "funny cats"
uv run python src/roku_sync.py roku
uv run python src/roku_sync.py history -n 3
```

**Understanding the code:**

- `subparsers.add_parser("name")` — Creates a new subcommand
- `parser.add_argument("query", nargs="?")` — `nargs="?"` makes it optional
- `-n`, `--count` — Short and long forms of the same option
- `type=int` — Converts the argument to an integer
- `set_defaults(func=...)` — Associates a function with the subcommand
- `hasattr(args, "func")` — Checks if a subcommand was chosen

---

## Part 3: Integrating Your Modules

Now let's wire up the real functionality by importing from your other modules.

### Exercise 6.3: Import Your Existing Code

**What to do:**

Add these imports at the top of `roku_sync.py`:

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

**What to do:**

Replace the `cmd_search` function:

```python
def cmd_search(args: argparse.Namespace) -> None:
    """Handle the 'search' subcommand.

    Args:
        args: Parsed command-line arguments.
    """
    # Get query from argument or prompt for it
    query = args.query
    if not query:
        query = input("Enter search query: ").strip()

    if not query:
        print("Error: Query cannot be empty")
        return

    print(f"Searching YouTube for: {query}")

    # Load API key and search
    try:
        api_key = load_api_key()
    except SystemExit:
        return  # Error was already printed

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

**What to do:**

First, add a helper function to get the Roku IP:

```python
import json
from pathlib import Path


def get_project_root() -> Path:
    """Get the project root directory."""
    return Path(__file__).parent.parent


def get_roku_ip_from_config() -> str | None:
    """Load Roku IP from config, return None if not set.

    Returns:
        The Roku IP address, or None if not configured.
    """
    config_path = get_project_root() / "config.json"
    if not config_path.exists():
        return None

    with open(config_path, "r") as f:
        config = json.load(f)

    ip = config.get("roku_ip")
    if ip and ip != "YOUR_ROKU_IP_HERE":
        return ip
    return None
```

Then replace `cmd_roku`:

```python
def cmd_roku(args: argparse.Namespace) -> None:
    """Handle the 'roku' subcommand.

    Args:
        args: Parsed command-line arguments.
    """
    # Get Roku IP from config or user
    roku_ip = get_roku_ip_from_config()
    if not roku_ip:
        roku_ip = input("Enter Roku IP address: ").strip()

    if not roku_ip:
        print("Error: IP address cannot be empty")
        return

    print(f"\nConnecting to Roku at {roku_ip}...")

    # Get device info
    device = get_device_info(roku_ip)
    if not device:
        print("Could not connect to Roku")
        return

    print(f"\nDevice: {device['name']} ({device['model']})")

    # Get active app
    app = get_active_app(roku_ip)
    if app:
        print(f"Active App: {app['name']}")
    else:
        print("No app running (home screen)")

    # Get playback status
    playback = get_playback_status(roku_ip)
    if playback:
        print(f"Playback State: {playback['state']}")
```

### Exercise 6.6: Implement the History Command

**What to do:**

Replace `cmd_history`:

```python
def cmd_history(args: argparse.Namespace) -> None:
    """Handle the 'history' subcommand.

    Args:
        args: Parsed command-line arguments.
    """
    show_recent(args.count)
```

This one's simple — we just call our existing function!

### Exercise 6.7: Implement the Sync Command

This is the main workflow — the whole point of the project!

**What to do:**

Replace `cmd_sync`:

```python
def cmd_sync(args: argparse.Namespace) -> None:
    """Handle the 'sync' subcommand - full Roku to YouTube workflow.

    Args:
        args: Parsed command-line arguments.
    """
    # Step 1: Connect to Roku
    roku_ip = get_roku_ip_from_config()
    if not roku_ip:
        roku_ip = input("Enter Roku IP address: ").strip()

    if not roku_ip:
        print("Error: IP address cannot be empty")
        return

    print(f"\nConnecting to Roku at {roku_ip}...")

    # Get active app
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
    if app_name != "Unknown" and app_name != "Roku":
        include_app = input(f"Include '{app_name}' in search? (y/n): ").lower().strip()
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

        open_browser = input("Open in browser? (y/n): ").lower().strip()
        if open_browser == "y":
            webbrowser.open(url)
            print("Opened in browser!")
    else:
        print("No videos found")
        add_search(query, None)
```

---

## Part 4: Error Handling

### Concepts to Understand

> 📖 **Reference**: ATBS Chapter 11, "Handling Errors with try/except" section

Things go wrong. Networks fail, APIs timeout, files get deleted. Good error handling:
1. Catches errors before they crash the program
2. Tells the user what went wrong
3. Suggests how to fix it

### Exercise 6.8: Add Error Handling

The `try/except` pattern catches exceptions:

```python
try:
    # Code that might fail
    result = risky_operation()
except SomeError as e:
    # What to do if it fails
    print(f"Error: {e}")
```

We've already added some error handling. Let's make the search command more robust:

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
        response = search_youtube(query, api_key)
    except Exception as e:
        print(f"Error searching YouTube: {e}")
        print("Check your internet connection and API key.")
        return

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

---

## Part 5: Polish and User Experience

### Exercise 6.9: Add a Welcome Message

When the user runs the tool without a command, show a friendly message:

**What to do:**

Update the bottom of `main()`:

```python
def main() -> None:
    """Main entry point for the roku-sync CLI."""
    import sys

    parser = argparse.ArgumentParser(
        description="Sync Roku playback to YouTube on another device",
        epilog="Example: roku-sync search 'funny cats'",
    )

    # ... subparser setup (keep existing code) ...

    # If no arguments, show a friendly message
    if len(sys.argv) == 1:
        print("Roku YouTube Sync")
        print("=" * 40)
        print()
        print("Commands:")
        print("  search  - Search YouTube directly")
        print("  roku    - Check Roku device status")
        print("  history - View your search history")
        print("  sync    - Full workflow: Roku → YouTube")
        print()
        print("Run 'uv run python src/roku_sync.py <command> --help' for more info")
        return

    args = parser.parse_args()

    if hasattr(args, "func"):
        args.func(args)
    else:
        parser.print_help()
```

### Exercise 6.10: Add Version Flag

**What to do:**

Add this line after creating the parser:

```python
parser = argparse.ArgumentParser(
    description="Sync Roku playback to YouTube on another device"
)
parser.add_argument(
    "--version", action="version", version="roku-sync 1.0.0"
)
```

Now users can run `uv run python src/roku_sync.py --version` to see the version.

---

## Part 6: Making It Easier to Run

### Creating a Run Script (Windows)

Right now you run: `uv run python src/roku_sync.py search "cats"`

Wouldn't it be nicer to just run: `run search "cats"`?

**What to do:**

Create a file `run.bat` in your project root:

```batch
@echo off
uv run python src/roku_sync.py %*
```

**What this does:**
- `@echo off` — Don't show the command itself
- `%*` — Pass all arguments through to Python

Now you can run:
```bash
run search "funny cats"
run roku
run history -n 3
```

---

## Part 7: Complete Script

Here's the complete `src/roku_sync.py` for reference:

```python
"""Roku YouTube Sync CLI.

A command-line tool that syncs Roku playback to YouTube on another device.
"""
import argparse
import json
import sys
import webbrowser
from pathlib import Path

from history import add_search, show_recent
from roku_status import get_active_app, get_device_info, get_playback_status
from youtube_search import (
    create_video_url,
    get_first_video_id,
    load_api_key,
    search_youtube,
)


def get_project_root() -> Path:
    """Get the project root directory."""
    return Path(__file__).parent.parent


def get_roku_ip_from_config() -> str | None:
    """Load Roku IP from config, return None if not set."""
    config_path = get_project_root() / "config.json"
    if not config_path.exists():
        return None

    with open(config_path, "r") as f:
        config = json.load(f)

    ip = config.get("roku_ip")
    if ip and ip != "YOUR_ROKU_IP_HERE":
        return ip
    return None


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
        response = search_youtube(query, api_key)
    except Exception as e:
        print(f"Error: {e}")
        print("Check your internet connection and API key.")
        return

    video_id = get_first_video_id(response)

    if video_id:
        url = create_video_url(video_id)
        print(f"Found: {url}")
        add_search(query, video_id)
        webbrowser.open(url)
    else:
        print("No videos found")
        add_search(query, None)


def cmd_roku(args: argparse.Namespace) -> None:
    """Handle the 'roku' subcommand."""
    roku_ip = get_roku_ip_from_config()
    if not roku_ip:
        roku_ip = input("Enter Roku IP address: ").strip()

    if not roku_ip:
        print("Error: IP address cannot be empty")
        return

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


def cmd_history(args: argparse.Namespace) -> None:
    """Handle the 'history' subcommand."""
    show_recent(args.count)


def cmd_sync(args: argparse.Namespace) -> None:
    """Handle the 'sync' subcommand - full Roku to YouTube workflow."""
    roku_ip = get_roku_ip_from_config()
    if not roku_ip:
        roku_ip = input("Enter Roku IP address: ").strip()

    if not roku_ip:
        print("Error: IP address cannot be empty")
        return

    print(f"\nConnecting to Roku at {roku_ip}...")

    app = get_active_app(roku_ip)
    if not app:
        print("Could not connect to Roku or no app running")
        app_name = "Unknown"
    else:
        app_name = app["name"]
        print(f"You're watching: {app_name}")

    print("\nRoku can't tell us the specific video title.")
    query = input("What are you watching? Enter title: ").strip()

    if not query:
        print("Error: Query cannot be empty")
        return

    if app_name not in ("Unknown", "Roku"):
        include_app = input(f"Include '{app_name}' in search? (y/n): ").lower().strip()
        if include_app == "y":
            query = f"{query} {app_name}"

    print(f"\nSearching YouTube for: {query}")

    try:
        api_key = load_api_key()
        response = search_youtube(query, api_key)
    except Exception as e:
        print(f"Error: {e}")
        return

    video_id = get_first_video_id(response)

    if video_id:
        url = create_video_url(video_id)
        print(f"\nFound: {url}")
        add_search(query, video_id)

        open_browser = input("Open in browser? (y/n): ").lower().strip()
        if open_browser == "y":
            webbrowser.open(url)
    else:
        print("No videos found")
        add_search(query, None)


def main() -> None:
    """Main entry point for the roku-sync CLI."""
    parser = argparse.ArgumentParser(
        description="Sync Roku playback to YouTube on another device",
        epilog="Example: roku-sync search 'funny cats'",
    )
    parser.add_argument("--version", action="version", version="roku-sync 1.0.0")

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

    # Handle no arguments
    if len(sys.argv) == 1:
        print("Roku YouTube Sync")
        print("=" * 40)
        print()
        print("Commands:")
        print("  search  - Search YouTube directly")
        print("  roku    - Check Roku device status")
        print("  history - View your search history")
        print("  sync    - Full workflow: Roku → YouTube")
        print()
        print("Run with --help for more info")
        return

    args = parser.parse_args()

    if hasattr(args, "func"):
        args.func(args)
    else:
        parser.print_help()


if __name__ == "__main__":
    main()
```

---

## Testing Your Work

Test each command:

```bash
uv run python src/roku_sync.py --help
uv run python src/roku_sync.py search "funny cats"
uv run python src/roku_sync.py roku
uv run python src/roku_sync.py history -n 3
uv run python src/roku_sync.py sync
```

Checklist:
- [ ] `--help` shows all commands
- [ ] `--version` shows version number
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
| `ModuleNotFoundError` for your modules | Run from project root, or check import paths |
| "Command not recognized" | Make sure you're using subcommands: `roku_sync search`, not just `roku_sync` |
| argparse errors | Check argument names match between parser setup and function usage |
| Functions not called | Did you add `set_defaults(func=...)` for each subparser? |

---

## Vocabulary Review

| Term | Definition |
|------|------------|
| **CLI** | Command-Line Interface — a program you interact with by typing |
| **argparse** | Python module for building command-line tools |
| **Subcommand** | A secondary command (like `git commit` where `commit` is the subcommand) |
| **Argument** | A value passed to a command |
| **Option/Flag** | An optional setting, usually starts with `-` or `--` |
| **parser** | An object that reads and interprets command-line arguments |
| **Namespace** | An object holding the parsed argument values |
| **nargs** | How many values an argument accepts (`"?"` = 0 or 1) |
| **set_defaults** | Associates a function with a subcommand |

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

**Phase 7** (optional) explores advanced workarounds for the Roku metadata limitation:
- Building a web UI with Flask
- Packet inspection with scapy
- Roku developer mode exploration

---

## Commit Your Work

Once everything works and passes checks:

```bash
git add src/roku_sync.py run.bat
git commit -m "Phase 6: Complete CLI with argparse and full workflow"
```

You've completed the main project!
