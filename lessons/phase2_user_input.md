# Phase 2: User Input + Validation

**ATBS Chapters**: 5–6
**Goal**: Make your script interactive with input validation
**Deliverable**: `src/search_url.py` updated with user input and validation

---

## What You'll Learn

- Getting input from the user with `input()`
- Making decisions with `if`, `elif`, and `else`
- Looping until valid input is received
- Using dictionaries to map data
- **NEW TOOL**: Using Ruff to check your code for issues

---

## Prerequisites

Before starting this lesson:
1. Read ATBS Chapters 5–6
2. Complete Phase 1 (your `search_url.py` should work with hardcoded titles)
3. Your code should pass: `uv run python src/search_url.py`

---

## Tool Setup: Ruff

Before we start coding, let's install **Ruff** — a tool that checks your code for mistakes and style issues. Professional Python developers use tools like this constantly.

In your VS Code terminal:
```bash
uv add --dev ruff
```

The `--dev` flag means this is a development tool, not something your program needs to run.

Try it on your existing code:
```bash
uv run ruff check src/
```

If Ruff finds issues, it will tell you what's wrong and which line. Fix them before continuing!

### Make Ruff Part of Your Workflow

From now on, run `uv run ruff check src/` after writing code. It catches mistakes before you even run your program.

---

## Part 1: Getting User Input

### Concepts to Understand

The `input()` function pauses your program and waits for the user to type something:

```python
name = input("What is your name? ")
print("Hello, " + name)
```

The text inside `input()` is the **prompt** — what the user sees before typing.

**Important**: `input()` always returns a **string**, even if the user types a number.

### Exercise 2.1: Add User Input

Modify your `search_url.py` to:
1. Ask the user for a video title using `input()`
2. Pass that title to your `create_search_url()` function
3. Print the resulting URL

Run it with: `uv run python src/search_url.py`

**Check yourself**: Does your script wait for you to type something? Does it create a URL from what you typed?

---

## Part 2: Handling Empty Input

### Concepts to Understand

What happens if the user just presses Enter without typing anything? Try it!

You get an empty URL, which isn't useful. We need to **validate** the input.

An `if` statement lets you run code only when a condition is true:

```python
title = input("Enter title: ")
if title == "":
    print("You didn't enter anything!")
```

The `==` operator checks if two things are equal.

### Exercise 2.2: Reject Empty Input

Update your script to:
1. Check if the user's input is empty
2. If empty, print an error message instead of a URL
3. If not empty, create and print the URL

**Hint**: An empty string is `""`. You can check for it with `if title == "":` or even simpler: `if not title:` (empty strings are "falsy" in Python).

**Check yourself**: What happens when you press Enter without typing? What about when you type a title?

---

## Part 3: Looping Until Valid

### Concepts to Understand

Right now, if the user enters nothing, the program just ends. Better: keep asking until they give valid input.

A `while` loop repeats code as long as a condition is true:

```python
title = ""
while title == "":
    title = input("Enter title: ")
    if title == "":
        print("Title cannot be empty. Try again.")
```

The loop keeps running until `title` is no longer empty.

### Exercise 2.3: Input Loop

Refactor your script to:
1. Keep asking for input until the user provides a non-empty title
2. Print helpful error messages when input is invalid
3. Only create the URL after valid input is received

**Check yourself**: Can you "break" your script by pressing Enter multiple times? It should keep asking.

---

## Part 4: Trimming Whitespace

### Concepts to Understand

What if the user types `"   "` (just spaces)? That's technically not empty, but it's not useful either.

The `.strip()` method removes spaces from the beginning and end of a string:

```python
title = "  hello world  "
clean_title = title.strip()
# clean_title is now "hello world"
```

### Exercise 2.4: Clean the Input

Update your input handling to:
1. Strip whitespace from the user's input
2. Check if the stripped input is empty
3. Handle spaces-only input the same as empty input

**Check yourself**: Does typing only spaces trigger the "empty input" error?

---

## Part 5: Dictionaries for Mapping

### Concepts to Understand

A **dictionary** maps keys to values. Think of it like a real dictionary: you look up a word (key) to find its definition (value).

```python
abbreviations = {
    "ep": "episode",
    "pt": "part",
    "s": "season"
}

# Look up a key
full_word = abbreviations["ep"]  # Returns "episode"
```

This is useful for expanding common abbreviations in search terms.

### Exercise 2.5: Expand Abbreviations (Optional)

Create a dictionary that maps common abbreviations to their full forms. Before creating the URL, replace abbreviations in the title.

Example:
- User types: `"office s3 ep5"`
- Expanded: `"office season 3 episode 5"`

**Hint**: Loop through the dictionary and use `.replace()`:
```python
for abbrev, full in abbreviations.items():
    title = title.replace(abbrev, full)
```

This is optional but good practice with dictionaries.

---

## Part 6: Putting It Together

### Exercise 2.6: Final Interactive Script

Your completed `src/search_url.py` should:

1. Import `quote_plus` from `urllib.parse`
2. Define `create_search_url(title)` function (from Phase 1)
3. Have a `main()` function that:
   - Loops until valid input is received
   - Strips and validates the input
   - Optionally expands abbreviations
   - Creates and prints the URL
4. Call `main()` at the bottom

### Script Structure

```python
from urllib.parse import quote_plus


def create_search_url(title):
    """Convert a video title to a YouTube search URL.

    Args:
        title: The video title to search for.

    Returns:
        A YouTube search URL with the title properly encoded.
    """
    # Your code here


def main():
    """Prompt user for a video title and print the search URL."""
    # Your input loop and validation here


if __name__ == "__main__":
    main()
```

The `if __name__ == "__main__":` block is a Python pattern that runs `main()` only when the script is executed directly (not imported).

---

## Run Ruff Before You Finish

Before committing, check your code:

```bash
uv run ruff check src/
```

Fix any issues Ruff finds. Common issues for beginners:
- Unused imports
- Trailing whitespace
- Missing blank lines between functions

Ruff can auto-fix many issues:
```bash
uv run ruff check src/ --fix
```

---

## Testing Your Work

Run your script and verify:

- [ ] It asks for a video title
- [ ] Pressing Enter (empty input) shows an error and asks again
- [ ] Typing only spaces shows an error and asks again
- [ ] Valid input produces a working YouTube search URL
- [ ] `uv run ruff check src/` passes with no errors

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| Script runs but never asks for input | Did you call `main()` at the bottom? |
| Input loop runs forever | Your condition never becomes false. Check the logic. |
| `.strip()` not working | Make sure you're assigning the result: `title = title.strip()` |
| Ruff complains about `if title == "":` | Ruff prefers `if not title:`. Both work, but the second is more "Pythonic". |

---

## What's Next

In **Phase 3**, you'll connect to the real YouTube API to get actual video URLs instead of just search pages. You'll also learn about type hints and start using `ty` to catch type errors.

---

## Commit Your Work

Once everything works and Ruff passes:

```bash
git add src/search_url.py
git commit -m "Phase 2: Add interactive input with validation"
```
