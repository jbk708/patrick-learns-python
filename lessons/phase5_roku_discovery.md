# Phase 5: Roku Network Discovery

**ATBS Chapter**: 12 (Web Scraping) + external research
**Goal**: Query your Roku device for playback information
**Deliverable**: `src/roku_status.py`

---

## What You'll Learn

- Making HTTP requests to local network devices
- Parsing XML responses (different from JSON!)
- Understanding the Roku External Control Protocol (ECP)
- Handling network errors gracefully
- **Important discovery**: Platform limitations and workarounds

---

## Prerequisites

Before starting this lesson:

1. **Read ATBS Chapter 12** — Review the sections on:
   - Making web requests with `requests`
   - Handling errors and exceptions

2. **Complete Phase 4** — Your project should have:
   - Working search history
   - Robust file paths with `pathlib`
   - Pass `uv run ruff check src/` and `uv run ty check src/`

3. **Have a Roku device on your local network** — This phase requires a physical Roku device (or Roku TV) that's connected to the same network as your computer.

4. **Find your Roku's IP address** — You'll need this to connect:

   **On the Roku:**
   1. Go to Settings → Network → About
   2. Note the IP address (something like `192.168.1.105`)

   **Or use your router's admin page** to find connected devices.

---

## Part 1: Understanding Roku ECP

### What is the Roku ECP?

> 📖 **Reference**: This goes beyond ATBS into real-world device APIs.

Roku devices run a small **web server** that responds to commands. This is called the **External Control Protocol (ECP)**. It's designed for remote control apps to communicate with Roku devices.

The key insight: Your Roku is like a tiny website running on your local network!

**How it works:**

```
Your Computer                     Your Roku
     |                                |
     |  HTTP GET request -----------> |
     |                                | (processes request)
     | <----------- XML response      |
     |                                |
```

### The ECP Endpoints

Your Roku responds to these URLs (replace `ROKU_IP` with your actual IP):

| Endpoint | What It Returns |
|----------|----------------|
| `http://ROKU_IP:8060/query/device-info` | Device name, model, serial number |
| `http://ROKU_IP:8060/query/active-app` | Currently running app |
| `http://ROKU_IP:8060/query/media-player` | Playback status (playing, paused, etc.) |
| `http://ROKU_IP:8060/query/apps` | List of installed apps |

**Notice**: The port is `8060`, not the usual `80` for websites.

### Exercise 5.1: Explore ECP in Your Browser

Before writing code, test the ECP manually.

**What to do:**

1. Make sure your Roku is on and connected to the same network as your computer

2. Open your web browser and go to:
   ```
   http://192.168.1.XXX:8060/query/device-info
   ```
   (Replace with your Roku's actual IP address)

3. You should see something like:
   ```xml
   <device-info>
       <model-name>Roku Ultra</model-name>
       <user-device-name>Living Room Roku</user-device-name>
       <serial-number>ABC123DEF456</serial-number>
       ... more fields ...
   </device-info>
   ```

4. Try these other URLs:
   - `http://ROKU_IP:8060/query/active-app`
   - `http://ROKU_IP:8060/query/media-player` (try this while playing a video!)

**What you should notice:**
- The responses are in **XML format**, not JSON
- You can see basic info about the device
- You can see which app is running (Netflix, YouTube, etc.)

**If you get an error:**
- Check that the IP address is correct
- Make sure both devices are on the same network
- Try pinging the Roku: `ping 192.168.1.XXX`
- Check that your firewall isn't blocking port 8060

---

## Part 2: Understanding XML

### What is XML?

> 📖 **Reference**: This introduces a new data format not covered in ATBS.

**XML** (eXtensible Markup Language) is a text format for structured data. It looks similar to HTML:

```xml
<device-info>
    <model-name>Roku Ultra</model-name>
    <serial-number>ABC123</serial-number>
    <user-device-name>Living Room</user-device-name>
</device-info>
```

**Key differences from JSON:**
| XML | JSON |
|-----|------|
| Uses tags: `<name>value</name>` | Uses key-value: `"name": "value"` |
| Attributes: `<app id="123">` | Nested objects: `{"app": {"id": "123"}}` |
| More verbose | More compact |
| Common in older APIs | Common in modern APIs |

### Parsing XML in Python

Python's `xml.etree.ElementTree` module parses XML:

```python
import xml.etree.ElementTree as ET

xml_string = """
<device-info>
    <model-name>Roku Ultra</model-name>
    <user-device-name>Living Room</user-device-name>
</device-info>
"""

# Parse the XML string
root = ET.fromstring(xml_string)

# Find an element and get its text content
model = root.find("model-name")
print(model.text)  # Prints: Roku Ultra

# Using findtext() is simpler (returns text directly)
name = root.findtext("user-device-name")
print(name)  # Prints: Living Room

# With a default if not found
serial = root.findtext("serial-number", "Unknown")
print(serial)  # Prints: Unknown (if not in the XML)
```

### Key XML Methods

| Method | Description |
|--------|-------------|
| `ET.fromstring(text)` | Parse XML text into an Element |
| `element.find("tag")` | Find first child with that tag (returns Element or None) |
| `element.findtext("tag", default)` | Get text of child element (with default if not found) |
| `element.text` | The text content inside the element |
| `element.get("attribute")` | Get an attribute value |

---

## Part 3: Creating the Roku Status Script

### Exercise 5.2: Parse Device Info

**What to do:**

1. Create a new file `src/roku_status.py`

2. Add this code:

```python
"""Roku device status checker.

This script connects to a Roku device and displays its current status.
"""
import xml.etree.ElementTree as ET

import requests


def get_device_info(roku_ip: str) -> dict | None:
    """Get device information from a Roku.

    Args:
        roku_ip: The Roku's IP address (e.g., "192.168.1.105").

    Returns:
        Dictionary with device info, or None if request fails.
    """
    url = f"http://{roku_ip}:8060/query/device-info"

    try:
        # timeout=5 means don't wait more than 5 seconds
        response = requests.get(url, timeout=5)
        # Raise an exception for bad status codes (4xx, 5xx)
        response.raise_for_status()
    except requests.RequestException as e:
        print(f"Error connecting to Roku: {e}")
        return None

    # Parse the XML response
    root = ET.fromstring(response.text)

    # Extract the information we want
    return {
        "model": root.findtext("model-name", "Unknown"),
        "name": root.findtext("user-device-name", "Unknown"),
        "serial": root.findtext("serial-number", "Unknown"),
    }


# Test the function
if __name__ == "__main__":
    roku_ip = input("Enter your Roku's IP address: ").strip()
    device = get_device_info(roku_ip)

    if device:
        print(f"\nDevice: {device['name']}")
        print(f"Model: {device['model']}")
        print(f"Serial: {device['serial']}")
    else:
        print("Could not connect to Roku")
```

3. Test it:
   ```bash
   uv run python src/roku_status.py
   ```

**Understanding the new concepts:**

- `timeout=5` — Don't wait forever if the Roku is unreachable
- `response.raise_for_status()` — Throws an error if the HTTP status is 4xx or 5xx
- `try/except` — Catches errors so the program doesn't crash
- `requests.RequestException` — A general exception for all request-related errors

---

## Part 4: Getting the Active App

### What's Playing?

The `/query/active-app` endpoint tells us which app is currently open on the Roku.

**Example response:**
```xml
<active-app>
    <app id="12">Netflix</app>
</active-app>
```

Or if on the home screen:
```xml
<active-app>
    <app id="">Roku</app>
</active-app>
```

### Exercise 5.3: Query Active App

**What to do:**

Add this function to `src/roku_status.py`:

```python
def get_active_app(roku_ip: str) -> dict | None:
    """Get the currently active app on the Roku.

    Args:
        roku_ip: The Roku's IP address.

    Returns:
        Dictionary with app name and ID, or None if request fails.
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
        "name": app.text,  # The text content: "Netflix"
        "id": app.get("id"),  # The id attribute
    }
```

**Understanding `app.get("id")`:**

In XML, elements can have **attributes**:
```xml
<app id="12">Netflix</app>
```

- `app.text` gives you `"Netflix"` (the content)
- `app.get("id")` gives you `"12"` (the attribute)

---

## Part 5: The Metadata Limitation

This is where things get interesting. Let's check what the Roku tells us about playback.

### Exercise 5.4: Query Media Player

**What to do:**

Add this function:

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
        "state": root.get("state"),  # "play", "pause", "stop", etc.
        "error": root.get("error") == "true",
    }
```

### The Discovery

Now let's see what information we actually get. Update the test code at the bottom of your file:

```python
if __name__ == "__main__":
    roku_ip = input("Enter your Roku's IP address: ").strip()

    print("\n--- Device Info ---")
    device = get_device_info(roku_ip)
    if device:
        print(f"Name: {device['name']}")
        print(f"Model: {device['model']}")
    else:
        print("Could not connect")

    print("\n--- Active App ---")
    app = get_active_app(roku_ip)
    if app:
        print(f"App: {app['name']}")
        print(f"App ID: {app['id']}")
    else:
        print("No app running or could not connect")

    print("\n--- Playback Status ---")
    playback = get_playback_status(roku_ip)
    if playback:
        print(f"State: {playback['state']}")
        if playback['error']:
            print("Playback error detected")
    else:
        print("Could not get playback status")
```

**Run this while watching something on your Roku!**

```bash
uv run python src/roku_status.py
```

### What You'll Find

**What the Roku DOES tell us:**
- Device name and model
- Which app is running (Netflix, YouTube, etc.)
- Playback state (playing, paused, stopped)

**What the Roku DOES NOT tell us:**
- The title of the video you're watching
- The episode name or number
- The movie title
- Any content-specific metadata

**This is a platform limitation, not a bug in your code!**

Roku designed ECP for remote control (play, pause, navigate), not for content identification. Each streaming app (Netflix, YouTube, etc.) keeps its content information private.

---

## Part 6: Handling the Limitation

### Our Workaround

Since we can't automatically get the video title, we'll:
1. Show the user what app is running
2. Let them manually enter what they're watching
3. Use that to search YouTube (from our Phase 3 code)

This is still useful — it saves the user from having to remember or type the exact title!

### Exercise 5.5: Complete the Status Display

**What to do:**

Update your main block to provide a helpful message:

```python
def main() -> None:
    """Display Roku status and playback information."""
    roku_ip = input("Enter your Roku's IP address: ").strip()

    if not roku_ip:
        print("Error: IP address cannot be empty")
        return

    print(f"\nConnecting to Roku at {roku_ip}...")

    # Get device info
    device = get_device_info(roku_ip)
    if not device:
        print("\nCould not connect to Roku.")
        print("Check that:")
        print("  - The IP address is correct")
        print("  - Your Roku is on and connected")
        print("  - You're on the same network")
        return

    print(f"\nDevice: {device['name']} ({device['model']})")

    # Get active app
    app = get_active_app(roku_ip)
    if app and app['name']:
        print(f"Currently running: {app['name']}")
    else:
        print("No app running (home screen)")

    # Get playback status
    playback = get_playback_status(roku_ip)
    if playback:
        state = playback['state']
        if state == "play":
            print("Status: Playing something")
        elif state == "pause":
            print("Status: Paused")
        elif state == "stop":
            print("Status: Stopped")
        else:
            print(f"Status: {state}")

    # Explain the limitation
    print("\n" + "=" * 50)
    print("NOTE: Roku cannot tell us WHAT is playing.")
    print("This is a platform limitation — the video title")
    print("is not available through the Roku API.")
    print("=" * 50)
    print("\nTo sync with YouTube, you'll need to manually")
    print("enter what you're watching.")


if __name__ == "__main__":
    main()
```

---

## Part 7: Saving the Roku IP

It's tedious to type the IP address every time. Let's save it to the config file.

### Exercise 5.6: Add Roku IP to Config

**What to do:**

1. Update your `config.json` to include the Roku IP:
   ```json
   {
       "youtube_api_key": "YOUR_KEY_HERE",
       "roku_ip": "192.168.1.XXX"
   }
   ```

2. Update `config.json.example` to match:
   ```json
   {
       "youtube_api_key": "YOUR_API_KEY_HERE",
       "roku_ip": "YOUR_ROKU_IP_HERE"
   }
   ```

3. Add a config loading function to `roku_status.py`:

```python
import json
from pathlib import Path


def get_project_root() -> Path:
    """Get the project root directory."""
    return Path(__file__).parent.parent


def load_config() -> dict:
    """Load configuration from config.json.

    Returns:
        The config dictionary, or empty dict if file doesn't exist.
    """
    config_path = get_project_root() / "config.json"
    if not config_path.exists():
        return {}
    with open(config_path, "r") as f:
        return json.load(f)


def get_roku_ip() -> str:
    """Get Roku IP from config or user input.

    Returns:
        The Roku IP address.
    """
    config = load_config()
    saved_ip = config.get("roku_ip")

    if saved_ip and saved_ip != "YOUR_ROKU_IP_HERE":
        use_saved = input(f"Use saved Roku IP ({saved_ip})? (y/n): ").lower()
        if use_saved == "y":
            return saved_ip

    return input("Enter your Roku's IP address: ").strip()
```

4. Update `main()` to use this:

```python
def main() -> None:
    """Display Roku status and playback information."""
    roku_ip = get_roku_ip()

    if not roku_ip:
        print("Error: IP address cannot be empty")
        return

    # ... rest of the function stays the same ...
```

---

## Testing Your Work

Run through this checklist:

- [ ] Script connects to your Roku and shows device info
- [ ] Active app displays correctly (try different apps!)
- [ ] Playback status shows when something is playing
- [ ] Script handles Roku being off or unreachable gracefully
- [ ] Roku IP can be saved in and loaded from config.json
- [ ] You understand the metadata limitation
- [ ] `uv run ruff check src/` passes
- [ ] `uv run ty check src/` passes

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| Connection timeout | Wrong IP address, Roku is off, or firewall blocking port 8060 |
| `XML parsing error` | The response might be empty or invalid. Print `response.text` to see what you got. |
| "No app running" when app IS running | Some apps don't report to active-app endpoint properly. Try a different app. |
| Can't see video title | This is the platform limitation! Roku ECP doesn't provide content metadata. |
| `requests` not found | Run `uv add requests` if you haven't already |

---

## Vocabulary Review

| Term | Definition |
|------|------------|
| **ECP** | External Control Protocol — Roku's API for device control |
| **XML** | eXtensible Markup Language — a text format for structured data |
| **Element** | A piece of XML data with a tag, content, and optional attributes |
| **Attribute** | A name-value pair inside an XML tag (`<app id="123">`) |
| **timeout** | Maximum time to wait for a response before giving up |
| **raise_for_status()** | Method that throws an error for bad HTTP status codes |
| **try/except** | Python's way of catching and handling errors |
| **Local network** | Devices connected to the same router/WiFi |
| **Port** | A number identifying a specific service (8060 for Roku ECP) |
| **Platform limitation** | A restriction of what the hardware/software allows |

---

## What's Next

In **Phase 6**, you'll combine everything into a unified CLI tool. The tool will:
1. Connect to your Roku
2. Show what app is running
3. Ask what you're watching (since Roku can't tell us)
4. Search YouTube for that content
5. Open the result in your browser

---

## Commit Your Work

Once everything works:

```bash
git add src/roku_status.py config.json.example
git commit -m "Phase 5: Add Roku ECP status queries"
```

You've learned how to communicate with a local network device!
