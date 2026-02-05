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

1. **Read ATBS Chapters 5–6** — These chapters cover:
   - Chapter 5: Dictionaries and structuring data
   - Chapter 6: String manipulation (we'll use `.strip()` and `.replace()`)

2. **Complete Phase 1** — Your `search_url.py` should:
   - Have a working `create_search_url(title)` function
   - Successfully create YouTube search URLs
   - Run without errors: `uv run python src/search_url.py`

3. **Verify your Phase 1 code works**:
   ```bash
   uv run python src/search_url.py
   ```
   You should see a YouTube URL printed.

---

## Tool Setup: Ruff

### What is Ruff?

> 📖 **Reference**: This is a professional tool not covered in ATBS, but essential for real-world Python.

**Ruff** is a code checker (called a "linter"). It reads your code and finds:
- Mistakes that might cause bugs
- Style issues that make code harder to read
- Unused code that can be removed

Professional Python developers use linters constantly. Getting comfortable with Ruff now will make you a better programmer.

### Installing Ruff

In your VS Code terminal:

```bash
uv add --dev ruff
```

**What this command does:**
- `uv add` — tells uv to install a package
- `--dev` — marks this as a "development" tool (something you use while coding, not something your program needs to run)
- `ruff` — the package name

You'll see output showing Ruff being installed. This only needs to be done once per project.

### Using Ruff

To check your code, run:

```bash
uv run ruff check src/
```

**What this command does:**
- `uv run` — runs a command using your project's packages
- `ruff check` — tells Ruff to check code for issues
- `src/` — the folder to check (all `.py` files inside)

**If your code is perfect**, you'll see no output — that means no issues!

**If there are issues**, you'll see something like:

```
src/search_url.py:5:1: F401 [*] `os` imported but unused
Found 1 error.
[*] 1 fixable with the `--fix` option.
```

This tells you:
- **File**: `src/search_url.py`
- **Line 5, column 1**: Where the issue is
- **F401**: The error code (you can search this online for more info)
- **The problem**: You imported `os` but never used it
- **Fixable**: Ruff can automatically fix this

### Auto-fixing Issues

Ruff can fix many issues automatically:

```bash
uv run ruff check src/ --fix
```

Try it! It will remove unused imports, fix spacing, and more.

### Make Ruff Part of Your Workflow

From now on, **always run Ruff before committing your code**:

```bash
uv run ruff check src/
```

If there are errors, fix them. If you're not sure how to fix something, try `--fix` first, then look at what changed.

---

## Part 1: Getting User Input

### Concepts to Understand

> 📖 **Reference**: ATBS Chapter 1, "The input() Function" section

Right now, your script uses a hardcoded title. Let's make it ask the user what to search for.

The `input()` function:
1. Displays a message (called a **prompt**)
2. Waits for the user to type something and press Enter
3. Returns what they typed as a string

```python
name = input("What is your name? ")
print("Hello, " + name)
```

**What happens when you run this:**
1. The program displays `What is your name? ` and waits
2. You type `Patrick` and press Enter
3. `input()` returns the string `"Patrick"`
4. That string is stored in the variable `name`
5. The program prints `Hello, Patrick`

**Important**: `input()` always returns a **string**, even if the user types numbers. If they type `42`, you get `"42"` (the string), not `42` (the number).

### Exercise 2.1: Add User Input

**What to do:**

1. Open `src/search_url.py`
2. Replace your hardcoded title code with this:

```python
from urllib.parse import quote_plus


def create_search_url(title):
    """Convert a video title to a YouTube search URL.

    Args:
        title: The video title to search for.

    Returns:
        A YouTube search URL with the title properly encoded.
    """
    encoded_title = quote_plus(title)
    base_url = "https://www.youtube.com/results?search_query="
    return base_url + encoded_title


# Get input from the user
video_title = input("Enter a video title to search: ")
url = create_search_url(video_title)
print("Search URL:", url)
```

3. Save and run: `uv run python src/search_url.py`

**What you should see:**

```
Enter a video title to search: funny cat videos
Search URL: https://www.youtube.com/results?search_query=funny+cat+videos
```

(The first line waits for you to type. The second line appears after you press Enter.)

**Understanding what happened:**
- Line 17: `input()` displayed the prompt and waited for you to type
- When you typed `funny cat videos` and pressed Enter, that string was stored in `video_title`
- Line 18-19: Your existing function created the URL and printed it

**Check yourself**:
- Does your script wait for you to type something?
- Does it create a URL from what you typed?
- Try different titles — do they all work?

---

## Part 2: Handling Empty Input

### The Problem

**Try this:** Run your script and just press Enter without typing anything.

What happens? You probably get a URL like:
```
https://www.youtube.com/results?search_query=
```

That's a valid URL, but it's useless — there's no search term! We need to **validate** the input (check that it's acceptable before using it).

### Concepts to Understand: If Statements

> 📖 **Reference**: ATBS Chapter 2, "Flow Control" section

An **if statement** runs code only when a condition is true:

```python
age = 18

if age >= 18:
    print("You are an adult")
```

The indented code only runs if `age >= 18` is true. If `age` were 15, nothing would print.

**Comparison operators:**
- `==` — equals (two equals signs, not one!)
- `!=` — not equals
- `>` — greater than
- `<` — less than
- `>=` — greater than or equal to
- `<=` — less than or equal to

**Common mistake**: Using `=` instead of `==`. Remember:
- `=` means "assign this value" (put something in a box)
- `==` means "are these equal?" (compare two things)

### If-Else Statements

You can add an `else` block for when the condition is false:

```python
title = input("Enter title: ")

if title == "":
    print("You didn't enter anything!")
else:
    print("You entered:", title)
```

Either the `if` block runs OR the `else` block runs — never both.

### Exercise 2.2: Reject Empty Input

**What to do:**

1. Update your script to check for empty input:

```python
from urllib.parse import quote_plus


def create_search_url(title):
    """Convert a video title to a YouTube search URL.

    Args:
        title: The video title to search for.

    Returns:
        A YouTube search URL with the title properly encoded.
    """
    encoded_title = quote_plus(title)
    base_url = "https://www.youtube.com/results?search_query="
    return base_url + encoded_title


# Get input from the user
video_title = input("Enter a video title to search: ")

# Check if the input is empty
if video_title == "":
    print("Error: You must enter a title!")
else:
    url = create_search_url(video_title)
    print("Search URL:", url)
```

2. Save and run: `uv run python src/search_url.py`

**Test it:**
- Press Enter without typing → Should show error message
- Type a title and press Enter → Should show URL

**A simpler way to check for empty strings:**

In Python, empty strings are "falsy" — they act like `False` in conditions. So you can write:

```python
if not video_title:
    print("Error: You must enter a title!")
```

This is more "Pythonic" (the style Python programmers prefer). Both ways work!

---

## Part 3: Looping Until Valid

### The Problem

Right now, if the user enters nothing, the program just shows an error and ends. Better: keep asking until they give valid input.

### Concepts to Understand: While Loops

> 📖 **Reference**: ATBS Chapter 2, "while Loop Statements" section

A **while loop** repeats code as long as a condition is true:

```python
count = 0
while count < 3:
    print("Count is:", count)
    count = count + 1
print("Done!")
```

**Output:**
```
Count is: 0
Count is: 1
Count is: 2
Done!
```

The loop checks `count < 3` before each iteration:
- `0 < 3` → True → run the loop → count becomes 1
- `1 < 3` → True → run the loop → count becomes 2
- `2 < 3` → True → run the loop → count becomes 3
- `3 < 3` → False → exit the loop → print "Done!"

**Warning**: If the condition never becomes false, the loop runs forever! This is called an **infinite loop**. If your script seems stuck, press `Ctrl+C` to stop it.

### Using While for Input Validation

We can loop until the user provides valid input:

```python
title = ""  # Start with empty string

while title == "":
    title = input("Enter title: ")
    if title == "":
        print("Title cannot be empty. Try again.")

# Only get here when title is not empty
print("You entered:", title)
```

### Exercise 2.3: Input Loop

**What to do:**

1. Update your script to keep asking until valid input:

```python
from urllib.parse import quote_plus


def create_search_url(title):
    """Convert a video title to a YouTube search URL.

    Args:
        title: The video title to search for.

    Returns:
        A YouTube search URL with the title properly encoded.
    """
    encoded_title = quote_plus(title)
    base_url = "https://www.youtube.com/results?search_query="
    return base_url + encoded_title


# Keep asking until we get valid input
video_title = ""

while video_title == "":
    video_title = input("Enter a video title to search: ")
    if video_title == "":
        print("Error: Title cannot be empty. Please try again.")

# We only get here when video_title is not empty
url = create_search_url(video_title)
print("Search URL:", url)
```

2. Save and run: `uv run python src/search_url.py`

**Test it:**
- Press Enter multiple times — it should keep asking
- Type a title — it should show the URL and stop

**Understanding what happened:**
- Line 18: We start with `video_title = ""` so the while condition is true
- Line 20: The loop starts
- Line 21: We ask for input
- Line 22-23: If empty, we print an error (but stay in the loop!)
- The loop repeats until `video_title` is not empty
- Line 26-27: Only runs after valid input

---

## Part 4: Trimming Whitespace

### The Problem

Try this: Run your script and type just spaces, then press Enter.

```
Enter a video title to search:
Search URL: https://www.youtube.com/results?search_query=%20%20%20%20%20
```

The spaces get encoded, but this isn't a useful search. We should treat spaces-only input the same as empty input.

### Concepts to Understand: String Methods

> 📖 **Reference**: ATBS Chapter 6, "strip(), lstrip(), and rstrip()" section

Strings have **methods** — functions that belong to the string. The `.strip()` method removes whitespace (spaces, tabs, newlines) from both ends:

```python
title = "  hello world  "
clean_title = title.strip()
print(clean_title)  # Displays: hello world (no leading/trailing spaces)
```

**Important**: `.strip()` returns a NEW string — it doesn't modify the original:

```python
title = "  hello  "
title.strip()  # This does nothing useful!
print(title)   # Still has spaces: "  hello  "

title = title.strip()  # This works — we reassign the variable
print(title)           # Now it's "hello"
```

### Exercise 2.4: Clean the Input

**What to do:**

1. Update your script to strip whitespace:

```python
from urllib.parse import quote_plus


def create_search_url(title):
    """Convert a video title to a YouTube search URL.

    Args:
        title: The video title to search for.

    Returns:
        A YouTube search URL with the title properly encoded.
    """
    encoded_title = quote_plus(title)
    base_url = "https://www.youtube.com/results?search_query="
    return base_url + encoded_title


# Keep asking until we get valid input
video_title = ""

while not video_title:  # Using the Pythonic way
    user_input = input("Enter a video title to search: ")
    video_title = user_input.strip()  # Remove leading/trailing whitespace

    if not video_title:
        print("Error: Title cannot be empty. Please try again.")

# We only get here when video_title is not empty
url = create_search_url(video_title)
print("Search URL:", url)
```

2. Save and run: `uv run python src/search_url.py`

**Test it:**
- Type only spaces → Should show error and ask again
- Type a valid title with extra spaces → Should work (spaces get stripped)

**Notice**: We now use `not video_title` instead of `video_title == ""`. This is the Pythonic style that Ruff prefers.

---

## Part 5: Dictionaries for Mapping

### Concepts to Understand

> 📖 **Reference**: ATBS Chapter 5, "The Dictionary Data Type" section

A **dictionary** stores key-value pairs. Think of it like a real dictionary where you look up a word (key) to find its definition (value):

```python
# Creating a dictionary
abbreviations = {
    "ep": "episode",
    "pt": "part",
    "s": "season"
}
```

**Dictionary syntax:**
- Curly braces `{}` contain the dictionary
- Each entry is `key: value`
- Entries are separated by commas

**Looking up values:**

```python
full_word = abbreviations["ep"]  # Returns "episode"
print(full_word)
```

If you try to look up a key that doesn't exist, you get a `KeyError`. To avoid this, use `.get()`:

```python
result = abbreviations.get("xyz", "not found")  # Returns "not found"
```

**Looping through a dictionary:**

```python
for abbrev, full in abbreviations.items():
    print(f"{abbrev} = {full}")
```

Output:
```
ep = episode
pt = part
s = season
```

### Exercise 2.5: Expand Abbreviations (Optional)

This exercise is optional but teaches important dictionary concepts.

**Goal**: Automatically expand abbreviations like "s3" to "season 3" and "ep5" to "episode 5".

**What to do:**

1. Create a function that expands abbreviations:

```python
def expand_abbreviations(title):
    """Expand common abbreviations in a title.

    Args:
        title: The title that may contain abbreviations.

    Returns:
        The title with abbreviations expanded.
    """
    abbreviations = {
        " ep": " episode",
        " pt": " part",
        " s": " season"
    }

    result = title.lower()  # Convert to lowercase for matching

    for abbrev, full in abbreviations.items():
        result = result.replace(abbrev, full)

    return result
```

2. Use it before creating the URL:

```python
video_title = expand_abbreviations(video_title)
url = create_search_url(video_title)
```

**Try it:**
- Input: `office s3 ep5`
- Output should search for: `office season3 episode5`

**Note**: This is a simple implementation. It won't handle every case perfectly, but it demonstrates how dictionaries work!

---

## Part 6: Organizing with a Main Function

### Concepts to Understand

> 📖 **Reference**: ATBS Chapter 3 discusses functions, but main() is a convention you'll see everywhere.

As your code grows, it's good to organize it into functions. A common pattern is to put your program's main logic in a function called `main()`:

```python
def main():
    """Main program logic goes here."""
    print("Hello from main!")

# This calls main() when you run the script
main()
```

### The `if __name__ == "__main__":` Pattern

You'll often see this at the bottom of Python files:

```python
if __name__ == "__main__":
    main()
```

**What this means:**
- When you run a Python file directly (`python script.py`), Python sets `__name__` to `"__main__"`
- When you import a file into another file, `__name__` is set to the module name instead
- This `if` block only runs the code when the file is executed directly, not when imported

**Why use it?**
- Later, you might want to import `create_search_url()` from another script
- Without this pattern, running `import search_url` would also run your main code
- With this pattern, importing only loads the functions

For now, just know it's good practice to include it.

### Exercise 2.6: Final Interactive Script

**What to do:**

1. Organize your script with a `main()` function:

```python
"""YouTube search URL generator with user input.

This script asks the user for a video title and creates a YouTube search URL.
"""
from urllib.parse import quote_plus


def create_search_url(title):
    """Convert a video title to a YouTube search URL.

    Args:
        title: The video title to search for.

    Returns:
        A YouTube search URL with the title properly encoded.
    """
    encoded_title = quote_plus(title)
    base_url = "https://www.youtube.com/results?search_query="
    return base_url + encoded_title


def get_video_title():
    """Prompt the user for a video title until valid input is received.

    Returns:
        A non-empty, stripped video title string.
    """
    video_title = ""

    while not video_title:
        user_input = input("Enter a video title to search: ")
        video_title = user_input.strip()

        if not video_title:
            print("Error: Title cannot be empty. Please try again.")

    return video_title


def main():
    """Main program: get title from user and display search URL."""
    video_title = get_video_title()
    url = create_search_url(video_title)
    print("Search URL:", url)


if __name__ == "__main__":
    main()
```

2. Save and run: `uv run python src/search_url.py`

**Understanding the structure:**
- `create_search_url()` — converts a title to a URL (from Phase 1)
- `get_video_title()` — handles all the input validation
- `main()` — coordinates everything
- `if __name__ == "__main__":` — ensures `main()` runs when we execute the script

---

## Run Ruff Before You Finish

Before committing, check your code:

```bash
uv run ruff check src/
```

**Common issues Ruff finds:**
- Unused imports (you imported something but never used it)
- Trailing whitespace (invisible spaces at the end of lines)
- Missing blank lines between functions (PEP 8 style says 2 blank lines)
- Using `== ""` instead of `not variable`

If Ruff finds issues, try:

```bash
uv run ruff check src/ --fix
```

This auto-fixes most problems. Then run `ruff check` again to verify.

---

## Testing Your Work

Run your script and verify:

- [ ] It asks for a video title
- [ ] Pressing Enter (empty input) shows an error and asks again
- [ ] Typing only spaces shows an error and asks again
- [ ] Valid input produces a working YouTube search URL
- [ ] The URL works when pasted into a browser
- [ ] `uv run ruff check src/` passes with no errors

---

## Debugging with VS Code

When `print()` isn't enough, VS Code has a powerful **debugger** that lets you pause your code and inspect what's happening step by step.

### What is Debugging?

**Debugging** is the process of finding and fixing bugs in your code. The VS Code debugger lets you:
- **Pause** your code at any line
- **Inspect** the values of variables at that moment
- **Step through** code one line at a time
- **Watch** how variables change

### Setting a Breakpoint

A **breakpoint** is a marker that tells the debugger "pause here."

**To set a breakpoint:**
1. Open your Python file in VS Code
2. Click in the **left margin** next to a line number (the gray area to the left of the line numbers)
3. A red dot appears — that's your breakpoint!

Try it: Set a breakpoint on this line in your code:
```python
url = create_search_url(video_title)
```

### Running in Debug Mode

Instead of running with `uv run python`, we'll use VS Code's debugger:

1. Open your Python file
2. Press **F5** (or go to Run → Start Debugging)
3. If asked, select "Python Debugger" then "Python File"

The code runs until it hits your breakpoint, then **pauses**.

### The Debug Interface

When paused at a breakpoint, you'll see:

```
┌─────────────────────────────────────────────────────────────┐
│  VARIABLES          │                                       │
│  ─────────          │     Your code                         │
│  video_title: "cat" │     (current line highlighted)        │
│  url: undefined     │                                       │
│                     │                                       │
│  WATCH              │                                       │
│  ─────              │                                       │
│  (add expressions)  │                                       │
│                     │                                       │
│  CALL STACK         │                                       │
│  ──────────         │                                       │
│  main()             │                                       │
│  <module>           │                                       │
└─────────────────────────────────────────────────────────────┘
```

**Key panels:**
- **VARIABLES**: Shows all variables and their current values
- **WATCH**: Add expressions to monitor (right-click → "Add Expression")
- **CALL STACK**: Shows which functions called which (useful for complex code)

### Debug Controls

At the top of VS Code, you'll see these controls:

| Button | Keyboard | What It Does |
|--------|----------|--------------|
| ▶️ Continue | F5 | Run until the next breakpoint |
| ⏭️ Step Over | F10 | Run the current line, move to the next |
| ⬇️ Step Into | F11 | If the line calls a function, go inside it |
| ⬆️ Step Out | Shift+F11 | Finish the current function, return to caller |
| 🔄 Restart | Ctrl+Shift+F5 | Start over from the beginning |
| ⏹️ Stop | Shift+F5 | Stop debugging |

**Most useful for beginners**: **Step Over (F10)** — it runs one line at a time.

### Exercise: Debug Your Script

1. Set a breakpoint on the first line inside `get_user_input()`
2. Press F5 to start debugging
3. When it pauses, look at the VARIABLES panel
4. Press F10 (Step Over) to move through the code
5. Watch how `video_title` gets its value after `input()` runs
6. Continue pressing F10 to see the flow

### Quick Debugging with print()

Sometimes the debugger is overkill. For quick checks, `print()` works great:

```python
def get_user_input():
    """Get a video title from the user."""
    while True:
        video_title = input("Enter a video title: ").strip()
        print(f"DEBUG: got input '{video_title}'")  # Temporary debug line
        print(f"DEBUG: length is {len(video_title)}")  # Temporary debug line

        if video_title:
            return video_title
        print("Please enter a title (not just spaces).")
```

**Tips for print debugging:**
- Prefix with `DEBUG:` so you remember to remove them later
- Print the variable AND its type: `print(f"DEBUG: x = {x}, type = {type(x)}")`
- Remove debug prints before committing!

### When to Use Each

| Situation | Use |
|-----------|-----|
| "What value does this variable have?" | Quick `print()` |
| "Why does this loop run forever?" | Debugger with breakpoint |
| "What's the flow through my code?" | Debugger, step through |
| "Is my function being called?" | Quick `print("function called!")` |
| "Complex logic I don't understand" | Debugger |

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| Script runs but never asks for input | Did you call `main()` at the bottom? |
| Input loop runs forever | Your condition never becomes false. Check that you're updating `video_title` inside the loop. |
| `.strip()` not working | Make sure you assign the result: `title = title.strip()` |
| `if title = "":` gives error | You used `=` (assignment) instead of `==` (comparison) |
| Ruff complains about `if title == "":` | Ruff prefers `if not title:`. Both work, but the second is more Pythonic. |
| `IndentationError` in if/while | All code inside the if/while must be indented the same amount |

---

## Vocabulary Review

Terms you learned in this phase:

| Term | Definition |
|------|------------|
| **input()** | Function that prompts the user and returns what they type |
| **Prompt** | The message shown to the user before they type |
| **Validation** | Checking that input is acceptable before using it |
| **If statement** | Runs code only when a condition is true |
| **Else** | Runs code when the if condition is false |
| **While loop** | Repeats code as long as a condition is true |
| **Infinite loop** | A loop that never ends (usually a bug) |
| **Dictionary** | A data structure that maps keys to values |
| **Key** | The lookup term in a dictionary |
| **Value** | What you get back when you look up a key |
| **.strip()** | String method that removes whitespace from both ends |
| **Linter** | A tool that checks code for mistakes and style issues |
| **Ruff** | The Python linter we're using |

---

## What's Next

In **Phase 3**, you'll connect to the real YouTube API to get actual video URLs instead of just search pages. You'll learn:
- How to make HTTP requests with the `requests` library
- How to work with JSON data from APIs
- How to open URLs in the browser automatically
- **Type hints** — annotations that help catch bugs
- **ty** — a type checker tool (like Ruff, but for types)

---

## Git: Seeing Changes with `git diff`

In Phase 1, you learned `git status`, `git add`, and `git commit`. Now let's learn how to see exactly what changed.

### Viewing Uncommitted Changes: `git diff`

The `git diff` command shows you the exact lines that changed:

```bash
git diff
```

Output looks like this:
```diff
diff --git a/src/search_url.py b/src/search_url.py
--- a/src/search_url.py
+++ b/src/search_url.py
@@ -15,6 +15,14 @@ def create_search_url(title):
     return search_url


+def get_user_input():
+    """Prompt user for a video title with validation."""
+    while True:
+        title = input("Enter video title: ").strip()
+        if title:
+            return title
+        print("Please enter a title!")
+
```

Reading the diff:
- Lines starting with `+` (green) were **added**
- Lines starting with `-` (red) were **removed**
- Other lines are context (unchanged, just for reference)

**Try it now:**
```bash
git diff src/search_url.py
```

### Viewing Staged Changes

Once you've run `git add`, `git diff` shows nothing (the changes are staged). To see staged changes:

```bash
git diff --staged
```

### Fixing Commit Mistakes: `git commit --amend`

Made a typo in your commit message? Forgot to include a file? The `--amend` flag lets you fix your last commit:

**Fix the commit message:**
```bash
git commit --amend -m "Phase 2: Add interactive input with validation and abbreviations"
```

**Add a forgotten file to the last commit:**
```bash
git add forgotten_file.py
git commit --amend --no-edit
```

The `--no-edit` flag keeps the original message.

**Warning**: Only amend commits you haven't pushed yet! Amending rewrites history, which can cause problems if others have already seen the original commit.

### Exercise: Commit Your Work

Once everything works and Ruff passes:

**Step 1: See what changed**
```bash
git diff src/search_url.py
```

Review the changes — do they look right?

**Step 2: Check status**
```bash
git status
```

**Step 3: Stage and commit**
```bash
git add src/search_url.py
git commit -m "Phase 2: Add interactive input with validation"
```

**Step 4: Verify**
```bash
git log --oneline -3
```

This shows your last 3 commits.

### Git Commands Learned So Far

| Command | What It Does |
|---------|--------------|
| `git status` | Show what's changed |
| `git add <file>` | Stage changes for commit |
| `git commit -m "msg"` | Save staged changes |
| `git log` | Show commit history |
| `git diff` | Show unstaged changes line-by-line |
| `git diff --staged` | Show staged changes line-by-line |
| `git commit --amend` | Fix your last commit |

You've made your first interactive Python program!
