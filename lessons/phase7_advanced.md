# Phase 7: Advanced Workarounds (Optional)

**Beyond ATBS**: This phase goes beyond the book into more advanced Python
**Goal**: Explore creative solutions to the Roku metadata limitation
**Status**: Optional — for learners who want to go deeper

---

## Overview

In Phase 5, you discovered that Roku's ECP doesn't provide video titles. This phase explores workarounds:

1. **Web UI with Flask** — Build a simple web interface (Recommended!)
2. **Packet Inspection** — Watch network traffic for clues
3. **Roku Developer Mode** — Explore unofficial APIs

These are advanced topics. You don't need to complete them all — pick what interests you!

---

## Choosing Your Path

| Option | Difficulty | What You'll Learn | Practical? |
|--------|------------|-------------------|------------|
| A. Flask Web UI | Medium | Web development basics | Yes! |
| B. Packet Inspection | Hard | Network fundamentals | Maybe |
| C. Roku Dev Mode | Very Hard | Device hacking | Unlikely |

**Recommendation**: Start with **Option A (Flask Web UI)**. It's the most practical and teaches valuable web development skills that apply to many future projects.

---

## Option A: Simple Web UI with Flask

### What is Flask?

> 📖 **Reference**: This is web development, going well beyond ATBS.

**Flask** is a Python web framework. It lets you create websites and web apps with Python. Instead of running a CLI that only works on one computer, you can create a web page that works on any device with a browser — phones, tablets, other computers.

### The Idea

Instead of a CLI, create a web page where:
1. One person enters what they're watching
2. Others on the same network can see it and get the YouTube link
3. Works on phones, tablets, computers — anything with a browser

### Setup

Install Flask:

```bash
uv add flask
```

**What Flask does:**
- Runs a web server on your computer
- Responds to browser requests
- Generates HTML pages dynamically

### Understanding Web Basics

When you visit a website:
1. Your browser sends a **request** to a server
2. The server runs some code (in our case, Python)
3. The server sends back an **HTML response**
4. Your browser displays the HTML

Flask handles steps 1-3. You write the Python code that generates the HTML.

### Exercise 7.1: Your First Flask App

**What to do:**

1. Create a new file `src/web_app.py`

2. Add this starter code:

```python
"""Simple web interface for Roku YouTube Sync.

Run with: uv run python src/web_app.py
Then open http://localhost:5000 in your browser.
"""
from flask import Flask

# Create a Flask app
app = Flask(__name__)


@app.route("/")
def home():
    """Handle requests to the home page."""
    return "<h1>Hello from Flask!</h1><p>Your web app is working.</p>"


if __name__ == "__main__":
    print("Starting web server...")
    print("Open http://localhost:5000 in your browser")
    print("Press Ctrl+C to stop")
    app.run(host="0.0.0.0", port=5000, debug=True)
```

3. Run it:
   ```bash
   uv run python src/web_app.py
   ```

4. Open `http://localhost:5000` in your web browser

**What you should see:**
- "Hello from Flask!" in your browser
- The terminal shows log messages for each request

**Understanding the code:**

- `Flask(__name__)` — Creates a Flask application
- `@app.route("/")` — A **decorator** that says "run this function when someone visits `/`"
- `host="0.0.0.0"` — Makes the server accessible from other devices on your network
- `port=5000` — The port number to use
- `debug=True` — Auto-reloads when you change code

### What is a Decorator?

The `@app.route("/")` syntax is called a **decorator**. It modifies the function that follows it. In Flask's case, it registers the function as a handler for a specific URL.

```python
@app.route("/")        # When someone visits "/"
def home():            # Run this function
    return "Hello!"    # And send back this text
```

You can have multiple routes:

```python
@app.route("/")
def home():
    return "Home page"

@app.route("/about")
def about():
    return "About page"

@app.route("/search")
def search():
    return "Search page"
```

### Exercise 7.2: Add HTML Templates

Instead of returning plain text, let's return proper HTML.

**What to do:**

Update `src/web_app.py`:

```python
"""Simple web interface for Roku YouTube Sync."""
from flask import Flask, render_template_string

app = Flask(__name__)

# Store current watch info (in memory — resets when server restarts)
current_watch = {"title": None, "youtube_url": None}

# HTML template using Jinja2 syntax
PAGE_TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>Roku Sync</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        h1 {
            color: #333;
        }
        .current {
            background: #e0ffe0;
            padding: 20px;
            border-radius: 10px;
            margin: 20px 0;
        }
        .not-set {
            background: #fff3e0;
            padding: 20px;
            border-radius: 10px;
            margin: 20px 0;
        }
        input[type=text] {
            width: 100%;
            padding: 10px;
            font-size: 16px;
            box-sizing: border-box;
            margin-bottom: 10px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            border-radius: 5px;
        }
        button:hover {
            background-color: #45a049;
        }
        a {
            color: #1976D2;
        }
    </style>
</head>
<body>
    <h1>Roku YouTube Sync</h1>

    {% if current.title %}
    <div class="current">
        <h2>Now Watching:</h2>
        <p><strong>{{ current.title }}</strong></p>
        {% if current.youtube_url %}
        <p><a href="{{ current.youtube_url }}" target="_blank">Open on YouTube →</a></p>
        {% else %}
        <p><em>No YouTube match found</em></p>
        {% endif %}
    </div>
    {% else %}
    <div class="not-set">
        <p>No one has shared what they're watching yet.</p>
    </div>
    {% endif %}

    <h2>Update What You're Watching</h2>
    <form method="post" action="/update">
        <input type="text" name="title" placeholder="Enter video title..." required>
        <button type="submit">Search & Share</button>
    </form>

    <hr>
    <p><small>Open this page on any device: <code>http://YOUR_COMPUTER_IP:5000</code></small></p>
</body>
</html>
"""


@app.route("/")
def index():
    """Show the main page."""
    return render_template_string(PAGE_TEMPLATE, current=current_watch)


if __name__ == "__main__":
    print("Starting web server...")
    print("Open http://localhost:5000 in your browser")
    print("Press Ctrl+C to stop")
    app.run(host="0.0.0.0", port=5000, debug=True)
```

**Understanding the template:**

- `{% if ... %}` and `{% endif %}` — Jinja2 conditionals (like Python's `if`)
- `{{ current.title }}` — Jinja2 variable substitution (insert the value here)
- `render_template_string(TEMPLATE, current=...)` — Renders the template with variables

Save and refresh your browser to see the styled page!

### Exercise 7.3: Handle Form Submissions

When someone enters a title and clicks the button, we need to:
1. Receive the form data
2. Search YouTube
3. Update the current watch info
4. Redirect back to the main page

**What to do:**

Update `src/web_app.py` to handle the form:

```python
"""Simple web interface for Roku YouTube Sync."""
from flask import Flask, redirect, render_template_string, request

app = Flask(__name__)

# Store current watch info (in memory — resets when server restarts)
current_watch = {"title": None, "youtube_url": None}

# Import your YouTube search code
# We need to handle potential import errors gracefully
try:
    from youtube_search import (
        load_api_key,
        search_youtube,
        get_first_video_id,
        create_video_url,
    )
    YOUTUBE_AVAILABLE = True
except ImportError:
    YOUTUBE_AVAILABLE = False
    print("Warning: YouTube search not available. Make sure youtube_search.py is in src/")

# ... PAGE_TEMPLATE stays the same (copy from above) ...


@app.route("/")
def index():
    """Show the main page."""
    return render_template_string(PAGE_TEMPLATE, current=current_watch)


@app.route("/update", methods=["POST"])
def update():
    """Handle the form submission to update current watch."""
    title = request.form.get("title", "").strip()

    if not title:
        return redirect("/")

    # Try to find on YouTube
    youtube_url = None

    if YOUTUBE_AVAILABLE:
        try:
            api_key = load_api_key()
            response = search_youtube(title, api_key)
            video_id = get_first_video_id(response)

            if video_id:
                youtube_url = create_video_url(video_id)
        except Exception as e:
            print(f"YouTube search error: {e}")

    # Update the shared state
    current_watch["title"] = title
    current_watch["youtube_url"] = youtube_url

    return redirect("/")


if __name__ == "__main__":
    print("Starting web server...")
    print("Open http://localhost:5000 in your browser")
    print("Press Ctrl+C to stop")
    app.run(host="0.0.0.0", port=5000, debug=True)
```

**Understanding the new parts:**

- `methods=["POST"]` — This route only handles POST requests (form submissions)
- `request.form.get("title")` — Gets the value from the form field named "title"
- `redirect("/")` — Sends the browser back to the home page

### Making It Accessible to Other Devices

The server runs on your computer. To access it from other devices (your phone, another computer):

1. **Find your computer's IP address:**
   - Open Command Prompt
   - Type `ipconfig`
   - Look for "IPv4 Address" (something like `192.168.1.100`)

2. **Access from other devices:**
   - Open a browser on your phone/tablet
   - Go to `http://192.168.1.100:5000` (use your actual IP)

Now anyone on your network can share and see what's playing!

### Improvements to Consider

If you want to go further with the web UI:

1. **Persistent storage**: Save to a file so data survives restarts
   ```python
   import json
   from pathlib import Path

   def save_current():
       with open("current_watch.json", "w") as f:
           json.dump(current_watch, f)

   def load_current():
       path = Path("current_watch.json")
       if path.exists():
           with open(path, "r") as f:
               return json.load(f)
       return {"title": None, "youtube_url": None}
   ```

2. **Multiple rooms**: Support different "rooms" for different TVs
3. **Auto-refresh**: Use JavaScript to auto-update the page every few seconds
4. **Better styling**: Make it look nice with more CSS

---

## Option B: Packet Inspection with Scapy

### What You'll Learn

- Network fundamentals (TCP/IP, HTTP)
- Using `scapy` to capture network packets
- Analyzing traffic to find metadata

### The Idea

When your Roku plays a video, it requests that video from a server. If we can see those requests, we might find metadata (video ID, title in URL, etc.).

### Setup

```bash
uv add scapy
```

**Warning**: Packet capture requires:
- Administrator/root privileges
- WinPcap or Npcap installed (on Windows)
- Complex network configuration

### Basic Packet Capture

```python
"""Capture network packets to analyze Roku traffic."""
from scapy.all import sniff, TCP


def packet_callback(packet):
    """Process each captured packet."""
    if TCP in packet and packet[TCP].dport == 80:
        # HTTP traffic on port 80
        if hasattr(packet, 'load'):
            payload = packet.load[:200]  # First 200 bytes
            print(payload)


# Capture 100 packets (run as admin!)
print("Capturing packets... (run as Administrator)")
sniff(prn=packet_callback, count=100)
```

### Limitations

This approach is difficult because:
- Modern streaming uses HTTPS (encrypted) — you can't see the content
- Video IDs often don't map to titles without another API call
- Requires complex setup and admin privileges
- Very finicky to get working correctly

**Verdict**: Interesting for learning about networking, but not practical for our use case.

---

## Option C: Roku Developer Mode

### What You'll Learn

- Enabling hidden device features
- Exploring undocumented APIs
- Working with device logs

### The Idea

Roku has a developer mode for building Roku channels. It exposes additional interfaces and logging that might reveal more information.

### Enabling Developer Mode

1. On your Roku remote, press: Home 3x, Up 2x, Right, Left, Right, Left, Right
2. Note the IP address shown
3. Agree to the developer terms
4. Set a developer password

### What to Explore

Once in developer mode:
- **Developer installer**: `http://<roku-ip>:8060/`
- **Debug console**: May show app activity
- **Custom channels**: Could potentially forward metadata

### Creating a Sideloaded Channel (Very Advanced)

You could theoretically create a simple Roku channel that:
1. Runs alongside the main content
2. Exposes metadata via its own API

This requires learning **BrightScript** (Roku's programming language) — a significant undertaking beyond the scope of this project.

**Verdict**: Interesting exploration, but the effort far exceeds the benefit for our use case.

---

## What You've Learned

By completing this project, you've learned:

### Python Fundamentals
- Variables, strings, functions
- Loops and conditionals
- Type hints

### Working with Data
- JSON and XML parsing
- File I/O with `pathlib`
- Dictionaries and lists

### Web and APIs
- HTTP requests with `requests`
- API integration (YouTube)
- URL encoding

### Tools and Workflow
- `uv` for package management
- `ruff` for linting
- `ty` for type checking
- Git for version control

### Advanced Topics
- CLI development with `argparse`
- Network device communication
- (Optional) Web development with Flask

### Real-World Skills
- Handling platform limitations
- Error handling and user experience
- Code organization and modules
- Documentation with docstrings

**You've built a real software project from scratch!**

---

## Vocabulary Review

| Term | Definition |
|------|------------|
| **Flask** | A Python web framework for building websites |
| **Route** | A URL path that maps to a Python function |
| **Decorator** | Syntax (`@something`) that modifies a function |
| **Template** | HTML with placeholders for dynamic content |
| **Jinja2** | The template language Flask uses |
| **POST** | HTTP method for sending data (like form submissions) |
| **GET** | HTTP method for requesting data (like loading a page) |
| **Redirect** | Sending the browser to a different URL |
| **Packet** | A unit of data sent over a network |

---

## Commit Your Work

If you completed the Flask web UI:

```bash
git add src/web_app.py
git commit -m "Phase 7: Add web UI for sharing what you're watching"
```

---

## Congratulations!

You've completed the entire Roku YouTube Sync project! Here's a summary of what you built:

| Phase | What You Built |
|-------|---------------|
| 1 | YouTube search URL generator |
| 2 | Interactive input with validation |
| 3 | YouTube API integration with type hints |
| 4 | Persistent history and robust paths |
| 5 | Roku device communication |
| 6 | Professional CLI with subcommands |
| 7 | (Optional) Web interface with Flask |

You went from zero Python experience to building a complete, multi-component application. That's a huge accomplishment!

---

## What's Next for Your Python Journey?

Now that you've completed this project, here are some directions you could go:

1. **Finish ATBS** — Complete any remaining chapters
2. **Build your own project** — Apply what you've learned to something you care about
3. **Learn web development** — Explore Django or more Flask
4. **Data science** — Try pandas, NumPy, and Jupyter notebooks
5. **Automation** — Automate tasks on your computer with Python
6. **Contribute to open source** — Find a beginner-friendly project on GitHub

The skills you've built here — reading docs, debugging, using tools, building complete programs — will serve you in any direction you choose.

Happy coding!
