# Phase 7: Advanced Workarounds (Optional)

**Beyond ATBS**: This phase goes beyond the book into more advanced Python
**Goal**: Explore creative solutions to the Roku metadata limitation
**Status**: Optional — for learners who want to go deeper

---

## Overview

In Phase 5, you discovered that Roku's ECP doesn't provide video titles. This phase explores workarounds:

1. **Packet Inspection** — Watch network traffic for clues
2. **Roku Developer Mode** — Explore unofficial APIs
3. **Web UI** — Build a simple web interface for sharing

These are advanced topics. You don't need to complete them all — pick what interests you!

---

## Option A: Packet Inspection with Scapy

### What You'll Learn

- Network fundamentals (TCP/IP, HTTP)
- Using `scapy` to capture packets
- Analyzing network traffic

### The Idea

When your Roku plays a video, it requests that video from a server. If we can see those requests, we might extract metadata (video ID, title in URL, etc.).

### Setup

```bash
uv add scapy
```

**Warning**: Packet capture requires admin privileges and may require additional setup (WinPcap/Npcap on Windows).

### Basic Packet Capture

```python
from scapy.all import sniff, TCP


def packet_callback(packet):
    """Process each captured packet."""
    if TCP in packet and packet[TCP].dport == 80:
        # HTTP traffic on port 80
        if hasattr(packet, 'load'):
            print(packet.load[:200])  # First 200 bytes


# Capture 100 packets (run as admin!)
sniff(prn=packet_callback, count=100)
```

### What to Look For

- HTTP GET requests to video CDNs
- URLs containing video IDs
- API calls with metadata

### Limitations

- Modern streaming uses HTTPS (encrypted)
- Video IDs often don't map to titles without another API call
- Complex and finicky to set up

---

## Option B: Roku Developer Mode

### What You'll Learn

- Enabling hidden device features
- Exploring undocumented APIs
- Working with device logs

### The Idea

Roku has a developer mode for building Roku channels. It exposes additional interfaces and logging that might reveal more information.

### Enabling Developer Mode

1. On your Roku remote, press: Home 3x, Up 2x, Right, Left, Right, Left, Right
2. Note the IP address and agree to the terms
3. Set a developer password

### What to Explore

- Developer installer: `http://<roku-ip>:8060/`
- Debug console: May show app activity
- Custom channels: Could potentially forward metadata

### Creating a Sideloaded Channel (Very Advanced)

You could theoretically create a simple Roku channel that:
1. Runs alongside the main content
2. Exposes metadata via its own API

This requires learning BrightScript (Roku's programming language) — a significant undertaking.

---

## Option C: Simple Web UI with Flask

### What You'll Learn

- Web development basics
- Flask framework
- HTML forms
- Sharing across devices

### The Idea

Instead of running a CLI on every device, create a web page where:
1. One person enters what they're watching
2. Others can see it and get the YouTube link
3. Works on phones, tablets, computers

### Setup

```bash
uv add flask
```

### Basic Flask App

Create `src/web_app.py`:

```python
from flask import Flask, render_template_string, request, redirect

app = Flask(__name__)

# Store current watch info (in memory — resets on restart)
current_watch = {"title": None, "youtube_url": None}

PAGE_TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>Roku Sync</title>
    <style>
        body { font-family: Arial; max-width: 600px; margin: 50px auto; padding: 20px; }
        .current { background: #e0ffe0; padding: 20px; border-radius: 10px; margin: 20px 0; }
        input[type=text] { width: 100%; padding: 10px; font-size: 16px; }
        button { padding: 10px 20px; font-size: 16px; }
    </style>
</head>
<body>
    <h1>Roku YouTube Sync</h1>

    {% if current.title %}
    <div class="current">
        <h2>Now Watching:</h2>
        <p><strong>{{ current.title }}</strong></p>
        {% if current.youtube_url %}
        <p><a href="{{ current.youtube_url }}" target="_blank">Open on YouTube</a></p>
        {% endif %}
    </div>
    {% else %}
    <p>Nothing set yet.</p>
    {% endif %}

    <h2>Update What You're Watching</h2>
    <form method="post" action="/update">
        <input type="text" name="title" placeholder="Enter video title">
        <button type="submit">Search & Share</button>
    </form>
</body>
</html>
"""


@app.route("/")
def index():
    """Show the main page."""
    return render_template_string(PAGE_TEMPLATE, current=current_watch)


@app.route("/update", methods=["POST"])
def update():
    """Update current watch with YouTube search."""
    title = request.form.get("title", "").strip()
    if not title:
        return redirect("/")

    # Import your existing search code
    from youtube_search import (
        load_api_key,
        search_youtube,
        get_first_video_id,
        create_video_url,
    )

    try:
        api_key = load_api_key()
        response = search_youtube(title, api_key)
        video_id = get_first_video_id(response)

        if video_id:
            url = create_video_url(video_id)
            current_watch["title"] = title
            current_watch["youtube_url"] = url
        else:
            current_watch["title"] = title
            current_watch["youtube_url"] = None
    except Exception as e:
        print(f"Error: {e}")

    return redirect("/")


if __name__ == "__main__":
    print("Starting web server...")
    print("Open http://localhost:5000 in your browser")
    print("Or access from other devices at http://<your-computer-ip>:5000")
    app.run(host="0.0.0.0", port=5000, debug=True)
```

### Running the Web App

```bash
uv run python src/web_app.py
```

Then open `http://localhost:5000` in your browser.

### Making It Accessible to Other Devices

The `host="0.0.0.0"` makes the server accessible from other devices on your network. Find your computer's IP and share `http://<your-ip>:5000` with others.

### Improvements to Consider

1. **Persistent storage**: Save to a file so it survives restarts
2. **Multiple rooms**: Support different "rooms" for different TVs
3. **Auto-refresh**: Use JavaScript to auto-update the page
4. **Better styling**: Make it look nice on mobile

---

## Option D: OCR Approach (Experimental)

### The Idea

If you can capture a screenshot of your TV (via HDMI capture card), you could use OCR (Optical Character Recognition) to read the title from the screen.

### Libraries

- `pytesseract` — Python wrapper for Tesseract OCR
- `opencv-python` — Image processing

This requires hardware (HDMI capture) and is quite complex, but it's the most reliable way to get the actual title.

---

## Choosing Your Path

| Option | Difficulty | Fun Factor | Practical? |
|--------|------------|------------|------------|
| A. Packet Inspection | Hard | High | Maybe |
| B. Roku Dev Mode | Very Hard | Medium | Unlikely |
| C. Flask Web UI | Medium | High | Yes! |
| D. OCR | Hard | High | With hardware |

**Recommendation**: Start with Option C (Flask Web UI). It's the most practical and teaches valuable web development skills.

---

## What You've Learned

By completing this project, you've learned:

- Python fundamentals (variables, functions, loops, conditionals)
- String manipulation and URL encoding
- User input and validation
- Working with APIs (YouTube, Roku)
- Parsing JSON and XML
- File I/O and persistence
- Path handling for cross-platform code
- Type hints and static analysis
- Building CLI tools with argparse
- Error handling
- Code organization and modules
- Linting with Ruff
- Type checking with ty
- (Optional) Web development with Flask

**You've built a real software project from scratch!**

---

## Commit Your Work

If you completed any advanced options:

```bash
git add src/web_app.py  # or whatever you created
git commit -m "Phase 7: Add web UI for sharing"
```
