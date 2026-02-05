# Phase 3: YouTube API Integration

**ATBS Chapter**: 12 (Web Scraping)
**Goal**: Query the real YouTube API and return actual video URLs
**Deliverable**: `src/youtube_search.py`

---

## What You'll Learn

- Making HTTP requests with the `requests` library
- Working with JSON data from web APIs
- Handling API keys securely
- Opening URLs in a web browser automatically
- **NEW**: Type hints to document what your functions expect and return
- **NEW TOOL**: Using `ty` to catch type errors

---

## Prerequisites

Before starting this lesson:

1. **Read ATBS Chapter 12** — Focus on:
   - "Downloading Files from the Web with the requests Module" section
   - "Saving Downloaded Files to the Hard Drive" section

2. **Complete Phase 2** — Your `search_url.py` should:
   - Have a working input validation loop
   - Pass `uv run ruff check src/`

3. **Get a YouTube API key** — Follow these steps:
   - Go to [Google Cloud Console](https://console.cloud.google.com/)
   - Create a new project (or use an existing one)
   - Enable the "YouTube Data API v3" for your project
   - Create an API key in "Credentials"
   - See README.md for detailed instructions

4. **Create your config file**:
   - Create a file called `config.json` in your project root (same level as `src/`)
   - Add your API key:
   ```json
   {
       "youtube_api_key": "YOUR_ACTUAL_API_KEY_HERE"
   }
   ```
   - Make sure `config.json` is in your `.gitignore` so you don't accidentally share your key!

---

## Dependency Setup: requests

Before we can make HTTP requests, we need to install the `requests` library:

```bash
uv add requests
```

**What this does:**
- `uv add` — tells uv to install a package
- `requests` — a library for making HTTP requests (talking to websites)

Unlike `--dev` packages (like Ruff), this is a **runtime** dependency — your program needs it to run.

You can verify it installed by checking your `pyproject.toml` file — you should see `requests` listed under dependencies.

---

## Tool Setup: ty (Type Checking)

### What is Type Checking?

In Phase 2, we used Ruff to check for style issues and common mistakes. Now we'll add another tool: `ty` (a type checker).

> 📖 **Reference**: Type hints are mentioned briefly in ATBS, but are a more advanced topic. We're introducing them early because they help catch bugs.

**Type checking** finds bugs by analyzing what types of data your functions expect. For example, if a function expects a string but you pass it a number, the type checker will warn you — before you even run the code!

### Installing ty

```bash
uv add --dev ty
```

You'll learn how to use it after we learn about type hints.

---

## Part 1: Understanding APIs

### What is an API?

> 📖 **Reference**: This extends the web concepts from ATBS Chapter 12.

An **API** (Application Programming Interface) is a way for programs to talk to each other. When you browse YouTube in your web browser, you're seeing a human-friendly interface. But behind the scenes, there's an API that programs can use to access the same data.

**How it works:**

1. **Your script** sends an **HTTP request** to YouTube's servers
2. **YouTube's servers** process the request
3. **YouTube sends back** data in a structured format (JSON)
4. **Your script** parses the JSON to extract what you need

This is like sending a letter:
- You write a letter (the request) asking for information
- You send it to an address (YouTube's API URL)
- You get a letter back (the response) with the answer

### What is HTTP?

**HTTP** (HyperText Transfer Protocol) is how computers communicate on the web. There are different types of requests:

- **GET** — "Give me this data" (what we'll use)
- **POST** — "Here's some data to store"
- **PUT** — "Update this data"
- **DELETE** — "Remove this data"

When you type a URL in your browser, it makes a GET request.

### The YouTube API URL

The YouTube Data API uses this URL pattern:

```
https://www.googleapis.com/youtube/v3/search?part=snippet&q=YOUR_SEARCH&type=video&key=YOUR_API_KEY
```

Let's break this down:

| Part | Meaning |
|------|---------|
| `https://www.googleapis.com` | Google's API server |
| `/youtube/v3/search` | The YouTube search endpoint |
| `?` | Start of query parameters |
| `part=snippet` | What data to include in response |
| `q=YOUR_SEARCH` | The search query |
| `type=video` | Only return videos (not playlists or channels) |
| `key=YOUR_API_KEY` | Your authentication key |

### Exercise 3.1: Test the API in Your Browser

Before writing code, let's see what the API returns.

**What to do:**

1. Copy this URL:
   ```
   https://www.googleapis.com/youtube/v3/search?part=snippet&q=funny+cats&type=video&key=YOUR_KEY
   ```

2. Replace `YOUR_KEY` with your actual API key

3. Paste it into your browser's address bar and press Enter

**What you should see:**

A wall of text that looks something like this (simplified):

```json
{
  "items": [
    {
      "id": {
        "videoId": "hY7m5jjJ9mM"
      },
      "snippet": {
        "title": "Funny Cats Compilation"
      }
    }
  ]
}
```

This is **JSON** (JavaScript Object Notation) — a format for structured data that both humans and computers can read. Your Python script will receive exactly this data!

**If you get an error:**
- `quotaExceeded` — You've used your daily limit (wait until tomorrow)
- `keyInvalid` — Check that you copied your API key correctly
- `accessNotConfigured` — You need to enable the YouTube Data API in Google Cloud Console

---

## Part 2: Understanding JSON

### What is JSON?

> 📖 **Reference**: ATBS Chapter 12 touches on JSON, and Chapter 9 covers data structures.

**JSON** is a text format for storing data. It looks a lot like Python dictionaries and lists:

```json
{
    "name": "Patrick",
    "age": 25,
    "hobbies": ["coding", "gaming", "reading"]
}
```

**JSON types map to Python types:**

| JSON | Python |
|------|--------|
| `{ }` (object) | `dict` (dictionary) |
| `[ ]` (array) | `list` |
| `"text"` (string) | `str` |
| `123` (number) | `int` or `float` |
| `true`/`false` | `True`/`False` (`bool`) |
| `null` | `None` |

### Navigating Nested JSON

The YouTube API returns nested data. Here's how to access it in Python:

```python
# Example response (as a Python dictionary)
response = {
    "items": [
        {
            "id": {
                "videoId": "abc123"
            },
            "snippet": {
                "title": "My Video"
            }
        }
    ]
}

# Getting the video ID:
items = response["items"]        # Get the list of items
first_item = items[0]            # Get the first item (index 0)
id_object = first_item["id"]     # Get the "id" dictionary
video_id = id_object["videoId"]  # Get the video ID string

# Or all at once:
video_id = response["items"][0]["id"]["videoId"]
```

**Reading it step by step:**
1. `response["items"]` → the list of search results
2. `[0]` → the first result
3. `["id"]` → the ID information for that result
4. `["videoId"]` → the actual video ID string

---

## Part 3: Introduction to Type Hints

### What Are Type Hints?

> 📖 **Reference**: Type hints are a more advanced Python feature not covered in ATBS.

**Type hints** are annotations that tell Python what type of data a function expects and returns. They don't change how your code runs — they're like comments that tools can check automatically.

**Without type hints:**
```python
def greet(name):
    return "Hello, " + name
```

**With type hints:**
```python
def greet(name: str) -> str:
    return "Hello, " + name
```

Breaking it down:
- `name: str` — the parameter `name` should be a string
- `-> str` — this function returns a string

### Why Use Type Hints?

1. **Documentation** — Makes it clear what your function expects
2. **Bug catching** — Tools like `ty` can find mistakes before you run the code
3. **Editor help** — VS Code can give better autocomplete suggestions

### Common Type Hints

| Type | Description | Example |
|------|-------------|---------|
| `str` | Text | `"hello"` |
| `int` | Whole number | `42` |
| `float` | Decimal number | `3.14` |
| `bool` | True or False | `True` |
| `list` | A list | `[1, 2, 3]` |
| `dict` | A dictionary | `{"key": "value"}` |
| `None` | No value | `None` |

### Functions That Return Nothing

If a function doesn't return anything (it just prints or does something), use `-> None`:

```python
def say_hello(name: str) -> None:
    print("Hello, " + name)
    # No return statement — returns None implicitly
```

### Union Types: "This OR That"

Sometimes a function might return different types. For example, it might return a string if successful, or `None` if it fails:

```python
def find_video(query: str) -> str | None:
    # Returns a video ID string, or None if not found
    ...
```

The `|` means "or" — this function returns `str | None` (string OR None).

### Exercise 3.2: Add Type Hints to Your Phase 2 Code

**What to do:**

1. Open `src/search_url.py` from Phase 2

2. Add type hints to all your functions:

```python
def create_search_url(title: str) -> str:
    """Convert a video title to a YouTube search URL.

    Args:
        title: The video title to search for.

    Returns:
        A YouTube search URL with the title properly encoded.
    """
    # ... your existing code ...


def get_video_title() -> str:
    """Prompt the user for a video title until valid input is received.

    Returns:
        A non-empty, stripped video title string.
    """
    # ... your existing code ...


def main() -> None:
    """Main program: get title from user and display search URL."""
    # ... your existing code ...
```

3. Save the file

4. Run the type checker:
   ```bash
   uv run ty check src/
   ```

**What you should see:**

If everything is correct, you'll see no output (no errors). If there are type errors, `ty` will tell you what's wrong and where.

---

## Part 4: Making HTTP Requests

### Concepts to Understand

The `requests` library makes HTTP requests easy. Here's a basic example:

```python
import requests

# Make a GET request
response = requests.get("https://example.com")

# Check if it worked
print(response.status_code)  # 200 means success

# Get the response content
print(response.text)  # The raw text content
```

**Response status codes:**
- `200` — Success
- `400` — Bad request (something wrong with your request)
- `401` — Unauthorized (API key problem)
- `403` — Forbidden (access denied)
- `404` — Not found
- `500` — Server error (YouTube's problem)

### Parsing JSON Responses

APIs return JSON data. The `requests` library can parse it automatically:

```python
response = requests.get("https://api.example.com/data")
data = response.json()  # Converts JSON to a Python dictionary

# Now you can access it like a dictionary
print(data["items"])
```

### Exercise 3.3: Create the YouTube Search Function

**What to do:**

1. Create a new file `src/youtube_search.py`

2. Add this code:

```python
"""YouTube video search using the YouTube Data API.

This script searches YouTube and returns actual video URLs.
"""
import requests
from urllib.parse import quote_plus


def search_youtube(query: str, api_key: str) -> dict:
    """Search YouTube and return the API response.

    Args:
        query: The search term.
        api_key: Your YouTube Data API key.

    Returns:
        The parsed JSON response from YouTube as a dictionary.
    """
    # Encode the query for use in a URL
    encoded_query = quote_plus(query)

    # Build the API URL
    base_url = "https://www.googleapis.com/youtube/v3/search"
    params = f"?part=snippet&q={encoded_query}&type=video&key={api_key}"
    full_url = base_url + params

    # Make the request
    response = requests.get(full_url)

    # Convert JSON response to Python dictionary and return it
    return response.json()


# Test the function (we'll remove this later)
if __name__ == "__main__":
    # For testing only - don't hardcode keys in real code!
    test_key = "YOUR_API_KEY_HERE"  # Replace with your actual key temporarily
    result = search_youtube("funny cats", test_key)
    print(result)
```

3. Replace `YOUR_API_KEY_HERE` with your actual API key (just for testing)

4. Run it:
   ```bash
   uv run python src/youtube_search.py
   ```

**What you should see:**

A Python dictionary with the search results (lots of text). Look for the `items` list — each item is a video result.

**Important**: Remove the hardcoded API key after testing! We'll load it from a config file next.

---

## Part 5: Extracting the Video ID

### Understanding the Response Structure

When you search YouTube, you get back a list of results. Each result has:
- `id.videoId` — the unique video identifier
- `snippet.title` — the video title
- `snippet.description` — the video description
- And more...

To build a YouTube URL, we need the `videoId`.

### Exercise 3.4: Create the Video ID Extractor

**What to do:**

Add this function to `src/youtube_search.py`:

```python
def get_first_video_id(response_data: dict) -> str | None:
    """Extract the video ID of the first search result.

    Args:
        response_data: The parsed JSON response from YouTube API.

    Returns:
        The video ID string, or None if no results were found.
    """
    # Get the items list from the response
    items = response_data.get("items", [])

    # Check if there are any results
    if not items:
        return None

    # Get the first result
    first_result = items[0]

    # Navigate to the video ID
    video_id = first_result["id"]["videoId"]

    return video_id
```

**Understanding the code:**

- `response_data.get("items", [])` — safely get "items", return empty list if it doesn't exist
- `if not items:` — check if the list is empty
- `items[0]` — get the first item
- `["id"]["videoId"]` — navigate into nested dictionaries

**Why return `None`?**

If the search finds no results, we return `None` to signal "nothing found". The calling code can check for this:

```python
video_id = get_first_video_id(response)
if video_id is None:
    print("No videos found")
else:
    print(f"Found video: {video_id}")
```

---

## Part 6: Building the Video URL

YouTube video URLs follow this pattern:
```
https://www.youtube.com/watch?v=VIDEO_ID_HERE
```

For example: `https://www.youtube.com/watch?v=dQw4w9WgXcQ`

### Exercise 3.5: Create the URL Builder

Add this function:

```python
def create_video_url(video_id: str) -> str:
    """Create a YouTube video URL from a video ID.

    Args:
        video_id: The YouTube video ID (e.g., "dQw4w9WgXcQ").

    Returns:
        The full YouTube video URL.
    """
    return f"https://www.youtube.com/watch?v={video_id}"
```

This uses an **f-string** — the `{video_id}` gets replaced with the actual video ID value.

---

## Part 7: Loading the API Key Securely

### Why Not Hardcode the API Key?

Hardcoding your API key in the code is dangerous:
- If you share your code (e.g., on GitHub), others can see and use your key
- They could use up your quota or even get your account banned
- It's hard to change the key if needed

Instead, we store the key in a separate file (`config.json`) that's not shared.

### Exercise 3.6: Create the Config Loader

Add this function (put it near the top, after the imports):

```python
import json


def load_api_key() -> str:
    """Load the YouTube API key from config.json.

    Returns:
        The API key string.

    Raises:
        SystemExit: If config.json is missing or doesn't contain the key.
    """
    try:
        with open("config.json", "r") as f:
            config = json.load(f)
    except FileNotFoundError:
        print("Error: config.json not found!")
        print("Create a config.json file with your YouTube API key.")
        print("See config.json.example for the format.")
        raise SystemExit(1)

    if "youtube_api_key" not in config:
        print("Error: youtube_api_key not found in config.json!")
        raise SystemExit(1)

    return config["youtube_api_key"]
```

**Understanding the code:**

- `try/except` — catches errors (we'll learn more about this in later phases)
- `with open(...) as f:` — opens a file safely (automatically closes it)
- `json.load(f)` — reads JSON from a file and converts to Python dictionary
- `raise SystemExit(1)` — stops the program with an error code

**Note about paths:**

This code assumes you run the script from the project root directory. We'll make this more robust in Phase 4.

---

## Part 8: Opening in Browser

### The webbrowser Module

Python can open URLs in your default web browser:

```python
import webbrowser

webbrowser.open("https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```

This opens a new browser tab automatically — no user interaction needed!

### Exercise 3.7: Add Browser Opening

We'll add this in the main function (next section).

---

## Part 9: Putting It All Together

### Exercise 3.8: Complete the Main Function

Now let's combine everything into a working script.

**What to do:**

Update your `src/youtube_search.py` with the complete code:

```python
"""YouTube video search using the YouTube Data API.

This script searches YouTube and opens the first result in your browser.
"""
import json
import webbrowser

import requests
from urllib.parse import quote_plus


def load_api_key() -> str:
    """Load the YouTube API key from config.json.

    Returns:
        The API key string.

    Raises:
        SystemExit: If config.json is missing or doesn't contain the key.
    """
    try:
        with open("config.json", "r") as f:
            config = json.load(f)
    except FileNotFoundError:
        print("Error: config.json not found!")
        print("Create a config.json file with your YouTube API key.")
        print("See config.json.example for the format.")
        raise SystemExit(1)

    if "youtube_api_key" not in config:
        print("Error: youtube_api_key not found in config.json!")
        raise SystemExit(1)

    return config["youtube_api_key"]


def search_youtube(query: str, api_key: str) -> dict:
    """Search YouTube and return the API response.

    Args:
        query: The search term.
        api_key: Your YouTube Data API key.

    Returns:
        The parsed JSON response from YouTube as a dictionary.
    """
    encoded_query = quote_plus(query)
    base_url = "https://www.googleapis.com/youtube/v3/search"
    params = f"?part=snippet&q={encoded_query}&type=video&key={api_key}"
    full_url = base_url + params

    response = requests.get(full_url)
    return response.json()


def get_first_video_id(response_data: dict) -> str | None:
    """Extract the video ID of the first search result.

    Args:
        response_data: The parsed JSON response from YouTube API.

    Returns:
        The video ID string, or None if no results were found.
    """
    items = response_data.get("items", [])

    if not items:
        return None

    first_result = items[0]
    video_id = first_result["id"]["videoId"]
    return video_id


def create_video_url(video_id: str) -> str:
    """Create a YouTube video URL from a video ID.

    Args:
        video_id: The YouTube video ID.

    Returns:
        The full YouTube video URL.
    """
    return f"https://www.youtube.com/watch?v={video_id}"


def get_search_query() -> str:
    """Prompt the user for a search query until valid input is received.

    Returns:
        A non-empty, stripped search query string.
    """
    query = ""

    while not query:
        user_input = input("Enter a video to search for: ")
        query = user_input.strip()

        if not query:
            print("Error: Search query cannot be empty. Please try again.")

    return query


def main() -> None:
    """Search YouTube and open the first result in the browser."""
    # Load the API key from config
    api_key = load_api_key()

    # Get search query from user
    query = get_search_query()
    print(f"Searching YouTube for: {query}")

    # Search YouTube
    response = search_youtube(query, api_key)

    # Extract the first video ID
    video_id = get_first_video_id(response)

    # Handle results
    if video_id is None:
        print("No videos found for that search.")
    else:
        url = create_video_url(video_id)
        print(f"Found video: {url}")
        print("Opening in browser...")
        webbrowser.open(url)


if __name__ == "__main__":
    main()
```

Save and run:
```bash
uv run python src/youtube_search.py
```

**What should happen:**
1. It asks you for a search query
2. It searches YouTube
3. It prints the video URL
4. It opens the video in your browser!

---

## Checking Your Code

Before committing, run both checkers:

```bash
uv run ruff check src/
uv run ty check src/
```

Fix any issues they find!

**Common ty warnings:**
- `ty` might warn about `dict` being too vague. This is fine for now — in more advanced code, you'd specify the exact structure.

---

## Testing Your Work

Run your script and verify:

- [ ] It loads the API key from `config.json`
- [ ] It asks for a search query
- [ ] It shows the actual YouTube video URL (starts with `https://www.youtube.com/watch?v=`)
- [ ] It opens the video in your browser
- [ ] Searching for something very obscure shows "No videos found"
- [ ] `uv run ruff check src/` passes
- [ ] `uv run ty check src/` passes

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| `FileNotFoundError: config.json` | Run from project root: `uv run python src/youtube_search.py` |
| `KeyError: 'items'` | API error. Print the response to see the error message. Check your API key. |
| `IndexError: list index out of range` | No results. Make sure you check `if not items:` before accessing `items[0]`. |
| Browser doesn't open | Check that `webbrowser.open()` is being called. On Windows, it should work automatically. |
| `quotaExceeded` error | You've used your 100 daily searches. Wait until tomorrow. |
| `ty` complains about types | Make sure all functions have type hints. Add `-> None` for functions that don't return anything. |

---

## API Quota Notes

YouTube's free tier gives you **10,000 quota units per day**. Each search costs about 100 units, so you can do approximately **100 searches per day**.

If you hit the limit:
- Wait until tomorrow (quota resets at midnight Pacific Time)
- Create a new project in Google Cloud Console with a new API key

The quota is generous for learning — just don't run your script in a loop!

---

## Vocabulary Review

| Term | Definition |
|------|------------|
| **API** | Application Programming Interface — a way for programs to communicate |
| **HTTP** | Protocol for web communication (GET, POST, etc.) |
| **GET request** | An HTTP request that retrieves data |
| **JSON** | JavaScript Object Notation — a text format for structured data |
| **requests** | A Python library for making HTTP requests |
| **Type hint** | An annotation showing what type a variable or return value should be |
| **Union type** | A type that can be one of several types (e.g., `str \| None`) |
| **ty** | A type checker tool for Python |
| **API key** | A secret code that authenticates your requests to an API |
| **Status code** | A number indicating if an HTTP request succeeded (200) or failed |
| **f-string** | A string with `f` prefix that allows `{variable}` substitution |

---

## What's Next

In **Phase 4**, you'll add:
- **Persistent history** — save your searches to a file
- **Robust file paths** — make config loading work from any directory
- **Recent searches** — recall what you searched for before

---

## Git: Working with GitHub (Remotes, Push, Pull)

You've been saving commits locally. Now let's learn how to sync with GitHub so your code is backed up online and accessible from anywhere.

### What is a Remote?

A **remote** is a copy of your repository stored somewhere else — usually on GitHub. It lets you:
- **Back up** your code in the cloud
- **Share** your code with others
- **Collaborate** on projects
- **Work from multiple computers**

### Checking Your Remotes

See what remotes are configured:
```bash
git remote -v
```

If you cloned the repository from GitHub, you'll see:
```
origin  git@github.com:username/repo-name.git (fetch)
origin  git@github.com:username/repo-name.git (push)
```

`origin` is the conventional name for your main remote.

### Adding a Remote (If You Started Fresh)

If you created the project locally with `git init`, you need to add a remote manually:

**Step 1: Create a repository on GitHub**
1. Go to [github.com/new](https://github.com/new)
2. Name it `patrick-learns-python` (or similar)
3. Don't initialize with README (you already have files)
4. Click "Create repository"

**Step 2: Connect your local repo to GitHub**
```bash
git remote add origin git@github.com:YOUR_USERNAME/patrick-learns-python.git
```

Replace `YOUR_USERNAME` with your GitHub username.

### Uploading Your Commits: `git push`

The `git push` command sends your commits to GitHub:

```bash
git push origin main
```

This pushes your `main` branch to the `origin` remote.

**First push?** You might need to set the upstream branch:
```bash
git push -u origin main
```

The `-u` flag sets `origin/main` as the default — after this, you can just type `git push`.

### Downloading Changes: `git pull`

The `git pull` command downloads commits from GitHub:

```bash
git pull
```

Use this when:
- You're working on multiple computers
- Someone else pushed changes to the repository
- GitHub shows your repo is ahead of your local copy

**What happens during a pull?**
1. Git **fetches** the new commits from GitHub
2. Git **merges** them into your local branch

If you've made local changes that conflict with the remote changes, Git will ask you to resolve the conflict. We'll cover this more in Phase 5.

### The Push/Pull Workflow

Here's a typical workflow when working with GitHub:

```
1. Make changes to your code
2. git add <files>
3. git commit -m "message"
4. git pull (get any new changes from GitHub first)
5. git push (upload your changes)
```

**Why pull before push?** If someone else (or you on another computer) pushed changes, you need to get them first. Otherwise, Git will reject your push.

### Exercise: Commit and Push Your Work

**Step 1: Commit locally**
```bash
git add src/youtube_search.py
git commit -m "Phase 3: Add YouTube API search with type hints"
```

**Step 2: Pull any remote changes**
```bash
git pull
```

(If this is your first time and there's no remote set up, skip this step.)

**Step 3: Push to GitHub**
```bash
git push
```

**Step 4: Verify on GitHub**

Go to your repository on GitHub and refresh the page. You should see your new commit!

### Git Commands Learned So Far

| Command | What It Does |
|---------|--------------|
| `git status` | Show what's changed |
| `git add <file>` | Stage changes for commit |
| `git commit -m "msg"` | Save staged changes |
| `git log` | Show commit history |
| `git diff` | Show changes line-by-line |
| `git commit --amend` | Fix your last commit |
| `git remote -v` | List remotes |
| `git push` | Upload commits to GitHub |
| `git pull` | Download commits from GitHub |

You've built a program that talks to a real web API — and it's backed up on GitHub!
