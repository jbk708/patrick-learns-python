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
1. Read ATBS Chapter 12 (focus on the `requests` sections)
2. Complete Phase 2 (interactive `search_url.py` working)
3. Get a YouTube API key (instructions in README.md)
4. Create your `config.json` file with your API key

---

## Tool Setup: ty (Type Checking)

Install `ty` — a tool that checks your type hints are correct:

```bash
uv add --dev ty
```

You'll learn what type hints are in this lesson. Once you add them, run:
```bash
uv run ty check src/
```

This catches bugs like passing a number where a string is expected — before you even run your code!

---

## Part 1: Understanding APIs

### Concepts to Understand

An **API** (Application Programming Interface) is a way for programs to talk to each other. The YouTube Data API lets your Python script ask YouTube for information.

Here's how it works:
1. Your script sends an HTTP request to YouTube's servers
2. YouTube processes the request and sends back data (as JSON)
3. Your script parses the JSON to extract what you need

The URL you'll use:
```
https://www.googleapis.com/youtube/v3/search?part=snippet&q=YOUR_SEARCH&type=video&key=YOUR_API_KEY
```

### Exercise 3.1: Test the API in Your Browser

Before writing code, test the API manually:

1. Open your browser
2. Replace `YOUR_SEARCH` with a search term (use `+` for spaces)
3. Replace `YOUR_API_KEY` with your actual key
4. Paste the URL and hit Enter

You should see a wall of JSON text. This is what your Python script will receive!

---

## Part 2: Introduction to Type Hints

### Concepts to Understand

**Type hints** tell Python (and other developers) what type of data a function expects and returns. They don't change how your code runs — they're documentation that tools can check.

```python
def greet(name: str) -> str:
    """Return a greeting for the given name."""
    return "Hello, " + name
```

Breaking it down:
- `name: str` — the `name` parameter should be a string
- `-> str` — this function returns a string

Common types:
- `str` — text
- `int` — whole numbers
- `float` — decimal numbers
- `bool` — True or False
- `list` — a list of items
- `dict` — a dictionary
- `None` — no value / nothing

### Exercise 3.2: Add Type Hints to Phase 1-2 Code

Go back to your `search_url.py` and add type hints:

```python
def create_search_url(title: str) -> str:
    """Convert a video title to a YouTube search URL."""
    # ... your code
```

```python
def main() -> None:
    """Prompt user for a video title and print the search URL."""
    # ... your code
```

`-> None` means the function doesn't return anything (it just prints).

Run the type checker:
```bash
uv run ty check src/
```

Fix any errors it finds!

---

## Part 3: Making HTTP Requests

### Concepts to Understand

The `requests` library makes HTTP requests simple:

```python
import requests

response = requests.get("https://example.com")
print(response.status_code)  # 200 means success
print(response.text)         # The response content as text
```

For JSON APIs, use `.json()` to parse the response:

```python
data = response.json()  # Converts JSON to a Python dictionary
```

### Exercise 3.3: Create the API Request

Create a new file `src/youtube_search.py`:

1. Import `requests`
2. Create a function `search_youtube(query: str, api_key: str) -> dict`:
   - Build the API URL with the query and key
   - Make a GET request
   - Return the parsed JSON response

**Hint**: Use `quote_plus` on the query to handle spaces and special characters.

```python
import requests
from urllib.parse import quote_plus


def search_youtube(query: str, api_key: str) -> dict:
    """Search YouTube and return the API response.

    Args:
        query: The search term.
        api_key: Your YouTube Data API key.

    Returns:
        The parsed JSON response from YouTube.
    """
    # Build the URL
    # Make the request
    # Return the JSON
```

---

## Part 4: Parsing the Response

### Concepts to Understand

The YouTube API returns nested JSON. Here's a simplified version:

```json
{
    "items": [
        {
            "id": {
                "videoId": "dQw4w9WgXcQ"
            },
            "snippet": {
                "title": "Rick Astley - Never Gonna Give You Up"
            }
        }
    ]
}
```

To get the first video ID:
```python
video_id = data["items"][0]["id"]["videoId"]
```

Breaking it down:
- `data["items"]` — get the "items" list
- `[0]` — get the first item
- `["id"]` — get the "id" dictionary
- `["videoId"]` — get the video ID string

### Exercise 3.4: Extract the Video ID

Add a function to extract the first video ID from the response:

```python
def get_first_video_id(response_data: dict) -> str | None:
    """Extract the video ID of the first search result.

    Args:
        response_data: The parsed JSON from YouTube API.

    Returns:
        The video ID string, or None if no results found.
    """
    # Your code here
```

The `str | None` type hint means "returns a string OR None". This is called a **union type**.

**Important**: Check if there are results before accessing `items[0]`! If the search returns nothing, `items` will be an empty list.

---

## Part 5: Building the Video URL

### Concepts to Understand

YouTube video URLs follow this pattern:
```
https://www.youtube.com/watch?v=VIDEO_ID_HERE
```

### Exercise 3.5: Create the Video URL

Add a function to build the full YouTube URL:

```python
def create_video_url(video_id: str) -> str:
    """Create a YouTube video URL from a video ID.

    Args:
        video_id: The YouTube video ID.

    Returns:
        The full YouTube video URL.
    """
    # Your code here
```

---

## Part 6: Loading the API Key

### Concepts to Understand

Your API key is in `config.json`. You need to read it:

```python
import json

def load_api_key() -> str:
    """Load the YouTube API key from config.json.

    Returns:
        The API key string.

    Raises:
        FileNotFoundError: If config.json doesn't exist.
        KeyError: If youtube_api_key is missing from config.
    """
    with open("config.json", "r") as f:
        config = json.load(f)
    return config["youtube_api_key"]
```

### Exercise 3.6: Add Config Loading

Add the `load_api_key()` function to your script.

**Note on paths**: The path `"config.json"` assumes you run the script from the project root. We'll make this more robust in Phase 4.

---

## Part 7: Opening in Browser

### Concepts to Understand

The `webbrowser` module opens URLs in your default browser:

```python
import webbrowser

webbrowser.open("https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```

This opens a new browser tab automatically!

### Exercise 3.7: Auto-Open the Video

In your `main()` function, after finding the video URL, open it in the browser.

---

## Part 8: Putting It Together

### Exercise 3.8: Complete Script

Your `src/youtube_search.py` should:

1. Import: `requests`, `json`, `webbrowser`, `quote_plus`
2. Define functions (all with type hints and docstrings):
   - `load_api_key() -> str`
   - `search_youtube(query: str, api_key: str) -> dict`
   - `get_first_video_id(response_data: dict) -> str | None`
   - `create_video_url(video_id: str) -> str`
   - `main() -> None`
3. `main()` should:
   - Load the API key
   - Get search query from user (reuse your Phase 2 input validation!)
   - Search YouTube
   - Extract the video ID
   - If found: print the URL and open it in browser
   - If not found: print "No videos found"

### Script Structure

```python
import json
import webbrowser

import requests
from urllib.parse import quote_plus


def load_api_key() -> str:
    """Load the YouTube API key from config.json."""
    # ...


def search_youtube(query: str, api_key: str) -> dict:
    """Search YouTube and return the API response."""
    # ...


def get_first_video_id(response_data: dict) -> str | None:
    """Extract the video ID of the first search result."""
    # ...


def create_video_url(video_id: str) -> str:
    """Create a YouTube video URL from a video ID."""
    # ...


def main() -> None:
    """Search YouTube and open the first result."""
    # ...


if __name__ == "__main__":
    main()
```

---

## Checking Your Code

Run both tools before committing:

```bash
uv run ruff check src/
uv run ty check src/
```

Fix any issues they find!

---

## Testing Your Work

Run your script and verify:

- [ ] It loads the API key from config.json
- [ ] It asks for a search query
- [ ] It prints the actual YouTube video URL (not a search URL)
- [ ] It opens the video in your browser
- [ ] Searching for something obscure shows "No videos found"
- [ ] `uv run ruff check src/` passes
- [ ] `uv run ty check src/` passes

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| `FileNotFoundError: config.json` | Run the script from the project root: `uv run python src/youtube_search.py` |
| `KeyError: 'items'` | API error. Print `response.json()` to see the error message. Might be invalid API key. |
| `IndexError: list index out of range` | No search results. Check `if data["items"]:` before accessing `[0]`. |
| `ty` complains about `dict` | You can be more specific: `dict[str, Any]`. For now, `dict` is fine. |
| Browser doesn't open | Check that `webbrowser.open()` is being called. On Windows, it should work automatically. |

---

## API Quota Notes

YouTube's free tier gives you 10,000 quota units per day. Each search costs about 100 units, so you can do ~100 searches per day. If you hit the limit:
- Wait until tomorrow (quota resets at midnight Pacific Time)
- Create a new project in Google Cloud Console with a new API key

---

## What's Next

In **Phase 4**, you'll add persistent storage — saving your search history to a file so you can recall recent searches. You'll also make the config file path more robust.

---

## Commit Your Work

Once everything works:

```bash
git add src/youtube_search.py
git commit -m "Phase 3: Add YouTube API search with type hints"
```
