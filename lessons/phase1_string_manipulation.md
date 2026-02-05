# Phase 1: String Manipulation

**ATBS Chapters**: 1–4
**Goal**: Write a script that converts a video title into a YouTube search URL
**Deliverable**: `src/search_url.py`

---

## What You'll Learn

- How Python stores and manipulates text (strings)
- Writing reusable code with functions
- Making text safe for URLs (encoding)
- Running Python scripts from the terminal

---

## Prerequisites

Before starting this lesson:
1. Read ATBS Chapters 1–4
2. Complete your environment setup (see README.md):
   - Python installed with "Add to PATH" checked
   - uv installed (`uv --version` works)
   - VS Code installed with Python extension
   - Git installed
   - Project initialized with `uv init`
3. Make sure you can run `uv run python --version` in your VS Code terminal

---

## Lesson Overview

By the end of this phase, you'll have a script that does this:

```
Input:  "The Office bloopers season 3"
Output: https://www.youtube.com/results?search_query=The+Office+bloopers+season+3
```

We'll build up to this in small steps.

---

## Part 1: Strings and Variables

### Concepts to Understand

A **string** is text inside quotes. A **variable** is a name that holds a value.

```python
title = "The Office"
```

Here, `title` is a variable holding the string `"The Office"`.

### Exercise 1.1: Your First Script

Create a new file `src/search_url.py` and write code that:
1. Creates a variable called `video_title` containing any video title you choose
2. Prints the title to the screen

Run it with: `uv run python src/search_url.py`

**Check yourself**: Does your terminal show the title you chose?

---

## Part 2: String Concatenation

### Concepts to Understand

You can glue strings together with the `+` operator:

```python
greeting = "Hello, " + "world!"
```

YouTube search URLs follow this pattern:
```
https://www.youtube.com/results?search_query=YOUR_SEARCH_HERE
```

### Exercise 1.2: Build a URL String

Modify your script to:
1. Store the base YouTube URL in a variable called `base_url`
2. Combine `base_url` with your `video_title` using `+`
3. Print the complete URL

**Check yourself**: Does your output look like a YouTube URL? Try pasting it in a browser — does it work?

---

## Part 3: The Problem with Spaces

### Concepts to Understand

Try your script with a title that has spaces, like `"funny cat videos"`.

Paste that URL into a browser. What happens?

URLs can't have spaces. They need to be **encoded** — spaces become `+` or `%20`, and special characters get converted to codes.

Python has a built-in tool for this: `urllib.parse.quote_plus()`

To use it, add this line at the top of your script:
```python
from urllib.parse import quote_plus
```

Then you can do:
```python
encoded = quote_plus("funny cat videos")
# Result: "funny+cat+videos"
```

### Exercise 1.3: Encode the Title

Update your script to:
1. Import `quote_plus` at the top
2. Encode the video title before adding it to the URL
3. Print the properly encoded URL

**Check yourself**: Does your URL now have `+` instead of spaces? Does it work in a browser?

---

## Part 4: Functions

### Concepts to Understand

Right now, your code runs once from top to bottom. A **function** is a reusable block of code you can call whenever you need it.

```python
def greet(name):
    """Print a greeting to the given name."""
    print("Hello, " + name)

greet("Patrick")  # Prints: Hello, Patrick
greet("Claude")   # Prints: Hello, Claude
```

Key parts:
- `def` starts a function definition
- `greet` is the function name
- `name` is a **parameter** — input the function needs
- The triple-quoted string is a **docstring** — explains what the function does
- `greet("Patrick")` **calls** the function

### Exercise 1.4: Create a Function

Refactor your script to use a function:

1. Define a function called `create_search_url` that:
   - Takes one parameter: `title`
   - Has a docstring explaining what it does
   - Returns the complete, encoded YouTube search URL

2. Call your function with a test title and print the result

**Hint**: Use `return` instead of `print` inside the function:
```python
def create_search_url(title):
    """..."""
    # build the URL here
    return url  # send it back to whoever called the function
```

**Check yourself**: Can you call your function multiple times with different titles and get different URLs?

---

## Part 5: Putting It Together

### Exercise 1.5: Final Script

Your completed `src/search_url.py` should:

1. Import `quote_plus` from `urllib.parse`
2. Define a function `create_search_url(title)` with:
   - A Google-style docstring
   - URL encoding for the title
   - Return the complete URL
3. At the bottom, call the function with a hardcoded title and print the result

### Google-Style Docstring Format

```python
def create_search_url(title):
    """Convert a video title to a YouTube search URL.

    Args:
        title: The video title to search for.

    Returns:
        A YouTube search URL with the title properly encoded.
    """
```

---

## Testing Your Work

Run your script and verify:

- [ ] It prints a URL starting with `https://www.youtube.com/results?search_query=`
- [ ] Spaces in the title become `+` in the URL
- [ ] The URL works when pasted into a browser
- [ ] You can change the hardcoded title and get a different URL

---

## Stretch Goals (Optional)

If you finish early and want more practice:

1. **Multiple titles**: Create a list of titles and generate URLs for all of them using a loop (preview of Chapter 4)
2. **Special characters**: Test what happens with titles containing `&`, `?`, or `#`. Does `quote_plus` handle them?
3. **Print formatting**: Make the output prettier using f-strings: `f"Search URL: {url}"`

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| `ModuleNotFoundError: No module named 'urllib'` | Check your import statement — it should be `from urllib.parse import quote_plus` |
| URL has spaces instead of `+` | You're not using `quote_plus()` on the title |
| `NameError: name 'quote_plus' is not defined` | Missing the import at the top of your file |
| Function doesn't output anything | Did you use `return` and then `print()` the result when calling it? |

---

## What's Next

In **Phase 2**, you'll make this script interactive — it will ask the user for a title instead of using a hardcoded one. You'll also add input validation to handle edge cases.

---

## Commit Your Work

Once everything works:

```bash
git add src/search_url.py
git commit -m "Phase 1: YouTube search URL generator"
```
