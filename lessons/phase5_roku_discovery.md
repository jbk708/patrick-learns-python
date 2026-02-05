# Phase 5: Roku Network Discovery

**ATBS Chapter**: 12 (Web Scraping) + external research
**Goal**: Query your Roku device for playback information
**Deliverable**: `src/roku_status.py`

---

## What You'll Learn

- Making HTTP requests to local network devices
- Parsing XML responses
- Understanding the Roku External Control Protocol (ECP)
- Handling network errors gracefully
- **Important discovery**: Platform limitations and workarounds

---

## Prerequisites

Before starting this lesson:
1. Complete Phase 4 (history and file paths working)
2. Have a Roku device on your local network
3. Know your Roku's IP address (see README.md for instructions)
4. Test the Roku ECP in your browser first

---

## Part 1: Understanding Roku ECP

### Concepts to Understand

Roku devices run a small web server that responds to commands. This is the **External Control Protocol (ECP)**. It's designed for remote apps to control Roku devices.

Key endpoints:
- `http://<roku-ip>:8060/query/device-info` — Device information
- `http://<roku-ip>:8060/query/active-app` — Currently running app
- `http://<roku-ip>:8060/query/media-player` — Current playback status
- `http://<roku-ip>:8060/query/apps` — List of installed apps

All responses are **XML**, not JSON. We'll learn to parse XML in this phase.

### Exercise 5.1: Explore ECP in Your Browser

Before coding, test these URLs in your browser (replace with your Roku's IP):

1. `http://192.168.1.XXX:8060/query/device-info`
2. `http://192.168.1.XXX:8060/query/active-app`
3. `http://192.168.1.XXX:8060/query/media-player`

Note what information each endpoint returns. Write down what you observe!

---

## Part 2: Parsing XML

### Concepts to Understand

XML looks like HTML — it has tags with opening `<tag>` and closing `</tag>`:

```xml
<device-info>
    <model-name>Roku Ultra</model-name>
    <serial-number>ABC123</serial-number>
</device-info>
```

Python's `xml.etree.ElementTree` module parses XML:

```python
import xml.etree.ElementTree as ET

xml_string = "<root><item>Hello</item></root>"
root = ET.fromstring(xml_string)

# Find an element
item = root.find("item")
print(item.text)  # Prints: Hello
```

### Exercise 5.2: Parse Device Info

Create `src/roku_status.py`:

```python
import xml.etree.ElementTree as ET

import requests


def get_device_info(roku_ip: str) -> dict | None:
    """Get device information from a Roku.

    Args:
        roku_ip: The Roku's IP address.

    Returns:
        Dictionary with device info, or None if request fails.
    """
    url = f"http://{roku_ip}:8060/query/device-info"

    try:
        response = requests.get(url, timeout=5)
        response.raise_for_status()
    except requests.RequestException as e:
        print(f"Error connecting to Roku: {e}")
        return None

    root = ET.fromstring(response.text)

    return {
        "model": root.findtext("model-name", "Unknown"),
        "name": root.findtext("user-device-name", "Unknown"),
        "serial": root.findtext("serial-number", "Unknown"),
    }
```

Key points:
- `timeout=5` — Don't wait forever if Roku is unreachable
- `raise_for_status()` — Raise an exception if HTTP error (404, 500, etc.)
- `findtext("tag", "default")` — Get text content, with a default if not found

---

## Part 3: Getting the Active App

### Exercise 5.3: Query Active App

Add a function to get what app is currently running:

```python
def get_active_app(roku_ip: str) -> dict | None:
    """Get the currently active app on the Roku.

    Args:
        roku_ip: The Roku's IP address.

    Returns:
        Dictionary with app info, or None if request fails.
    """
    url = f"http://{roku_ip}:8060/query/active-app"

    try:
        response = requests.get(url, timeout=5)
        response.raise_for_status()
    except requests.RequestException as e:
        print(f"Error connecting to Roku: {e}")
        return None

    root = ET.fromstring(response.text)
    app = root.find("app")

    if app is None:
        return None

    return {
        "name": app.text,
        "id": app.get("id"),
    }
```

The `app.get("id")` gets an **attribute** from the tag: `<app id="12345">Netflix</app>`.

---

## Part 4: The Metadata Limitation

### Exercise 5.4: Query Media Player

This is where you'll discover something important. Add this function:

```python
def get_playback_status(roku_ip: str) -> dict | None:
    """Get current playback status from the Roku.

    Args:
        roku_ip: The Roku's IP address.

    Returns:
        Dictionary with playback info, or None if request fails.
    """
    url = f"http://{roku_ip}:8060/query/media-player"

    try:
        response = requests.get(url, timeout=5)
        response.raise_for_status()
    except requests.RequestException as e:
        print(f"Error connecting to Roku: {e}")
        return None

    root = ET.fromstring(response.text)

    return {
        "state": root.get("state"),
        "error": root.get("error") == "true",
        # Add any other fields you find!
    }
```

Now test it while playing a video on your Roku.

### The Discovery

**What you'll find**: The Roku ECP tells you very little about what's actually playing. You can see:
- Which app is running (Netflix, YouTube, etc.)
- Whether something is playing/paused
- Maybe some playback position

**What you WON'T find**: The title of the video, the show name, the episode number, or any content-specific metadata.

This is a **platform limitation**, not a bug in your code! Roku designed ECP for remote control, not for content identification.

---

## Part 5: Handling the Limitation

### Concepts to Understand

When you hit a limitation like this, you have options:

1. **Accept it**: Show what information IS available
2. **Workaround**: Fall back to manual entry
3. **Research**: Look for alternative approaches (Phase 7)

For now, we'll implement option 2: show the app name and let the user manually enter the title.

### Exercise 5.5: Build the Status Display

Add a `main()` function that shows available info:

```python
def main() -> None:
    """Display Roku status and playback information."""
    roku_ip = input("Enter your Roku IP address: ").strip()

    print("\n--- Device Info ---")
    device = get_device_info(roku_ip)
    if device:
        print(f"Model: {device['model']}")
        print(f"Name: {device['name']}")

    print("\n--- Active App ---")
    app = get_active_app(roku_ip)
    if app:
        print(f"App: {app['name']}")
        print(f"App ID: {app['id']}")
    else:
        print("No app running or Roku is on home screen")

    print("\n--- Playback Status ---")
    playback = get_playback_status(roku_ip)
    if playback:
        print(f"State: {playback['state']}")
        if playback['error']:
            print("Playback error detected")
    else:
        print("Could not get playback status")

    # Note the limitation
    print("\n--- Note ---")
    print("Roku ECP does not provide video title information.")
    print("You'll need to manually enter what you're watching.")


if __name__ == "__main__":
    main()
```

---

## Part 6: Saving the Roku IP

It's annoying to type the IP every time. Let's save it to config.

### Exercise 5.6: Add Roku IP to Config

Update your `config.json`:
```json
{
    "youtube_api_key": "YOUR_KEY_HERE",
    "roku_ip": "192.168.1.XXX"
}
```

Update `config.json.example` to match.

Then modify your script to load from config (with fallback to prompting):

```python
import json
from pathlib import Path


def get_project_root() -> Path:
    """Get the project root directory."""
    return Path(__file__).parent.parent


def load_config() -> dict:
    """Load configuration from config.json."""
    config_path = get_project_root() / "config.json"
    if not config_path.exists():
        return {}
    with open(config_path, "r") as f:
        return json.load(f)


def get_roku_ip() -> str:
    """Get Roku IP from config or user input."""
    config = load_config()
    if "roku_ip" in config:
        return config["roku_ip"]
    return input("Enter your Roku IP address: ").strip()
```

---

## Testing Your Work

- [ ] Script connects to your Roku
- [ ] Device info displays correctly
- [ ] Active app shows when an app is running
- [ ] Playback status shows when something is playing
- [ ] Script handles Roku being off/unreachable gracefully
- [ ] Roku IP can be saved in config
- [ ] `uv run ruff check src/` passes
- [ ] `uv run ty check src/` passes

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| Connection timeout | Wrong IP, or Roku is off, or firewall blocking port 8060 |
| XML parsing error | The response might be empty. Check `response.text` before parsing. |
| "No app running" when app IS running | Some apps don't report to active-app. Try a different app. |
| Can't see video title | This is the limitation! Roku ECP doesn't provide this. |

---

## What's Next

In **Phase 6**, you'll combine everything into a unified CLI tool with subcommands. The tool will:
1. Check what's playing on Roku
2. Ask for the title (since we can't get it automatically)
3. Search YouTube for that title
4. Open the result in your browser

---

## Commit Your Work

```bash
git add src/roku_status.py
git commit -m "Phase 5: Add Roku ECP status queries"
```
