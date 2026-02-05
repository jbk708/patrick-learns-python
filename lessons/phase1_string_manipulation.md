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

1. **Read ATBS Chapters 1–4** — These chapters cover:
   - Chapter 1: Python basics, the interactive shell, and your first program
   - Chapter 2: Flow control (we'll use this more in Phase 2)
   - Chapter 3: Functions, parameters, and return values
   - Chapter 4: Lists (we won't use these much yet, but good to know)

2. **Complete your environment setup** (see README.md):
   - Python installed with "Add to PATH" checked
   - uv installed (`uv --version` works in your terminal)
   - VS Code installed with the Python extension
   - Git installed
   - Project initialized with `uv init`

3. **Test your setup** by opening VS Code, then:
   - Press `Ctrl+`` (backtick) to open the terminal
   - Type `uv run python --version` and press Enter
   - You should see something like `Python 3.12.0`

---

## How to Create Files in VS Code

Since you're new to coding, here's how to create files:

1. **Open VS Code** and open your project folder (`File → Open Folder`)
2. In the left sidebar (called the "Explorer"), you'll see your project files
3. Look for the `src/` folder — if it doesn't exist, right-click in the Explorer and choose "New Folder", then name it `src`
4. Right-click on the `src/` folder and choose "New File"
5. Name the file `search_url.py` (the `.py` extension tells the computer it's a Python file)

Now you have an empty file ready to write code in!

---

## Lesson Overview

By the end of this phase, you'll have a script that does this:

```
Input:  "The Office bloopers season 3"
Output: https://www.youtube.com/results?search_query=The+Office+bloopers+season+3
```

That URL, when pasted into a browser, will search YouTube for that video. We'll build up to this in small steps.

---

## Part 1: Strings and Variables

### Concepts to Understand

> 📖 **Reference**: ATBS Chapter 1, "String Data Type" section

A **string** is how Python stores text. You create a string by putting text inside quotes — either single quotes (`'like this'`) or double quotes (`"like this"`). Both work the same way.

A **variable** is a name that holds a value, like a labeled box that stores something. You create a variable by choosing a name and using the `=` sign to assign a value to it:

```python
title = "The Office"
```

Let's break this down:
- `title` — the variable name (you choose this)
- `=` — the assignment operator (not "equals" like in math — it means "store this value")
- `"The Office"` — the string value being stored

After this line runs, whenever you use `title` in your code, Python will replace it with `"The Office"`.

### The `print()` Function

To see output from your program, you use the `print()` function. It displays whatever you put inside the parentheses:

```python
print("Hello!")        # Displays: Hello!
print(title)           # Displays whatever is stored in the variable 'title'
```

**Important**: Notice the difference:
- `print("title")` — displays the literal word "title"
- `print(title)` — displays the *value* stored in the variable named title

### Exercise 1.1: Your First Script

**What to do:**

1. Open `src/search_url.py` in VS Code (the file you created earlier)
2. Type the following code:

```python
video_title = "The Office bloopers"
print(video_title)
```

3. Save the file (`Ctrl+S`)
4. Open the terminal (`Ctrl+`` `) and run:

```bash
uv run python src/search_url.py
```

**What you should see:**

```
The Office bloopers
```

**Understanding what happened:**
- Line 1 created a variable called `video_title` and stored the string `"The Office bloopers"` in it
- Line 2 used `print()` to display the contents of that variable
- When you ran the script, Python executed these lines from top to bottom

**Check yourself**:
- Does your terminal show the title you chose?
- Try changing `"The Office bloopers"` to something else, save, and run again. Does the output change?

---

## Part 2: String Concatenation

### Concepts to Understand

> 📖 **Reference**: ATBS Chapter 1, "String Concatenation and Replication" section

**Concatenation** means gluing strings together. In Python, you use the `+` operator to join strings:

```python
first = "Hello, "
second = "world!"
combined = first + second
print(combined)  # Displays: Hello, world!
```

You can also concatenate directly in the `print()`:

```python
print("Hello, " + "world!")  # Displays: Hello, world!
```

### Understanding YouTube URLs

Before we write code, let's understand what a YouTube search URL looks like:

```
https://www.youtube.com/results?search_query=funny+cat+videos
```

This URL has parts:
- `https://` — the protocol (how to communicate with the website)
- `www.youtube.com` — the website address
- `/results` — the specific page on YouTube (the search results page)
- `?` — separates the page from the "query parameters"
- `search_query=` — the name of the parameter (tells YouTube what you're searching for)
- `funny+cat+videos` — the actual search term

The part before the search term is always the same:
```
https://www.youtube.com/results?search_query=
```

We call this the **base URL**. We just need to add our search term to the end.

### Exercise 1.2: Build a URL String

**What to do:**

1. Open `src/search_url.py` and replace the previous code with:

```python
video_title = "TheOffice"
base_url = "https://www.youtube.com/results?search_query="
search_url = base_url + video_title
print(search_url)
```

2. Save and run: `uv run python src/search_url.py`

**What you should see:**

```
https://www.youtube.com/results?search_query=TheOffice
```

**Understanding what happened:**
- Line 1: We stored our video title (no spaces for now)
- Line 2: We stored the base YouTube URL that never changes
- Line 3: We concatenated them together with `+` and stored the result
- Line 4: We printed the complete URL

**Check yourself**:
- Does your output look like a YouTube URL?
- Copy the URL from your terminal, paste it into your web browser, and press Enter
- Does YouTube show search results for "TheOffice"?

---

## Part 3: The Problem with Spaces

### Discovering the Problem

**What to do:**

1. Change your `video_title` to include spaces:

```python
video_title = "funny cat videos"
base_url = "https://www.youtube.com/results?search_query="
search_url = base_url + video_title
print(search_url)
```

2. Save and run: `uv run python src/search_url.py`

**What you should see:**

```
https://www.youtube.com/results?search_query=funny cat videos
```

3. Copy that URL and paste it into your browser. What happens?

You'll notice the URL might not work correctly, or the browser changes it. That's because **URLs cannot contain spaces**. This is a rule of how the internet works.

### Concepts to Understand: URL Encoding

> 📖 **Reference**: This goes slightly beyond ATBS, but it's essential for working with web URLs.

When you need to put special characters (like spaces) in a URL, they must be **encoded** — converted to a special format:

- Spaces become `+` or `%20`
- `&` becomes `%26`
- `?` becomes `%3F`

This process is called **URL encoding** or **percent encoding**.

Python has a built-in tool to do this automatically: `quote_plus()` from the `urllib.parse` module.

### Understanding Imports

To use `quote_plus()`, you need to **import** it. An import is like telling Python: "I need to use a tool that's stored somewhere else."

```python
from urllib.parse import quote_plus
```

Let's break this down:
- `from urllib.parse` — the location of the tool (a module in Python's standard library)
- `import quote_plus` — the specific tool we want to use

After this import, you can use `quote_plus()` in your code:

```python
encoded = quote_plus("funny cat videos")
print(encoded)  # Displays: funny+cat+videos
```

### Exercise 1.3: Encode the Title

**What to do:**

1. Update your `src/search_url.py`:

```python
from urllib.parse import quote_plus

video_title = "funny cat videos"
encoded_title = quote_plus(video_title)
base_url = "https://www.youtube.com/results?search_query="
search_url = base_url + encoded_title
print(search_url)
```

2. Save and run: `uv run python src/search_url.py`

**What you should see:**

```
https://www.youtube.com/results?search_query=funny+cat+videos
```

**Understanding what happened:**
- Line 1: We imported the `quote_plus` function
- Line 3: We stored our video title (with spaces)
- Line 4: We used `quote_plus()` to encode the title — spaces became `+`
- Line 5-6: We built the URL as before
- Line 7: We printed the result

**Important**: The import statement must be at the **top** of your file, before any other code.

**Check yourself**:
- Does your URL now have `+` instead of spaces?
- Copy the URL and paste it into your browser — does it search correctly?
- Try a title with other special characters like `&` (example: `"Tom & Jerry"`). What happens?

---

## Part 4: Functions

### Concepts to Understand

> 📖 **Reference**: ATBS Chapter 3, "Defining Your Own Functions" section

Right now, your code runs once from top to bottom. If you want to create URLs for different videos, you'd have to change the code each time.

A **function** is a reusable block of code that you can call whenever you need it. Think of it like a recipe — you write it once, then use it over and over.

Here's a simple function:

```python
def greet(name):
    """Print a greeting to the given name."""
    print("Hello, " + name)
```

Let's break this down:
- `def` — keyword that starts a function definition (short for "define")
- `greet` — the function's name (you choose this)
- `(name)` — the **parameter** — a placeholder for input the function needs
- `:` — required at the end of the `def` line
- The **indented lines** are the function's body — they run when you call the function
- `"""..."""` — a **docstring** — documentation explaining what the function does

**Critical concept: Indentation**

In Python, indentation (the spaces at the beginning of lines) matters! Everything indented under `def greet(name):` is part of that function. When you stop indenting, you're outside the function.

```python
def greet(name):
    print("Hello, " + name)   # This is INSIDE the function (indented)
    print("Nice to meet you") # This is ALSO inside (same indentation)

print("This is OUTSIDE the function")  # No indentation = outside
```

VS Code will usually add the indentation for you when you press Enter after the `:`.

### Calling a Function

To **call** (use) a function, write its name followed by parentheses with the input value:

```python
greet("Patrick")  # Calls greet with name="Patrick", prints: Hello, Patrick
greet("Claude")   # Calls greet with name="Claude", prints: Hello, Claude
```

The value you pass in (`"Patrick"`) is called an **argument**. It gets assigned to the parameter (`name`) when the function runs.

### Return vs Print

> 📖 **Reference**: ATBS Chapter 3, "Return Values and return Statements" section

There's an important difference between functions that **print** and functions that **return**:

- `print()` displays something on the screen, but you can't use that value elsewhere
- `return` sends a value back to wherever the function was called

```python
def add_and_print(a, b):
    """Print the sum of a and b."""
    print(a + b)  # Just displays it

def add_and_return(a, b):
    """Return the sum of a and b."""
    return a + b  # Sends the value back

add_and_print(2, 3)  # Displays: 5, but we can't use that 5

result = add_and_return(2, 3)  # Stores 5 in 'result'
print(result)  # Now we can use it: displays 5
print(result * 2)  # We can do math with it: displays 10
```

For our URL generator, we want to **return** the URL so we can use it later (maybe open it in a browser, save it to a file, etc.).

### Exercise 1.4: Create a Function

**What to do:**

1. Update your `src/search_url.py`:

```python
from urllib.parse import quote_plus


def create_search_url(title):
    """Convert a video title to a YouTube search URL.

    Takes a video title, encodes it for use in a URL, and returns
    the complete YouTube search URL.
    """
    encoded_title = quote_plus(title)
    base_url = "https://www.youtube.com/results?search_query="
    search_url = base_url + encoded_title
    return search_url


# Test the function with different titles
url1 = create_search_url("funny cat videos")
print(url1)

url2 = create_search_url("Python tutorial for beginners")
print(url2)
```

2. Save and run: `uv run python src/search_url.py`

**What you should see:**

```
https://www.youtube.com/results?search_query=funny+cat+videos
https://www.youtube.com/results?search_query=Python+tutorial+for+beginners
```

**Understanding what happened:**
- Lines 4-12: We defined a function that takes a `title` and returns a complete URL
- Line 15: We **called** the function with `"funny cat videos"` and stored the result in `url1`
- Line 16: We printed `url1`
- Lines 18-19: We did it again with a different title

**Notice**: We wrote the URL-building code once, but used it twice with different inputs!

**Check yourself**:
- Can you add a third call with your own video title?
- What happens if you call `create_search_url("The Office & Parks and Rec")`?

---

## Part 5: Putting It Together

### Exercise 1.5: Final Script

Your completed `src/search_url.py` should follow this structure. Here's the complete code with detailed comments:

```python
"""YouTube search URL generator.

This script converts video titles into YouTube search URLs.
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
    search_url = base_url + encoded_title
    return search_url


# This code runs when you execute the script directly
video_title = "The Office bloopers season 3"
url = create_search_url(video_title)
print("Search URL:", url)
```

### Understanding the Docstring Format

> 📖 **Reference**: While ATBS mentions docstrings briefly, Google-style docstrings are a professional convention.

We're using **Google-style docstrings**, which have a specific format:

```python
def create_search_url(title):
    """Convert a video title to a YouTube search URL.

    Args:
        title: The video title to search for.

    Returns:
        A YouTube search URL with the title properly encoded.
    """
```

The parts:
- First line: A brief description of what the function does
- `Args:` section: Lists each parameter and what it represents
- `Returns:` section: Describes what the function returns

This format helps others (and future you!) understand how to use your code.

---

## Testing Your Work

Run your script and verify:

- [ ] It prints a URL starting with `https://www.youtube.com/results?search_query=`
- [ ] Spaces in the title become `+` in the URL
- [ ] The URL works when pasted into a browser (shows YouTube search results)
- [ ] You can change the hardcoded title and get a different URL
- [ ] Your function has a docstring with Args and Returns sections

---

## Stretch Goals (Optional)

If you finish early and want more practice:

1. **f-strings** (Preview of a useful Python feature):

   Instead of `print("Search URL:", url)`, try:
   ```python
   print(f"Search URL: {url}")
   ```
   The `f` before the quote makes it a "format string" — anything in `{curly braces}` gets replaced with its value.

2. **Multiple titles**: Create a list of titles and generate URLs for all of them:
   ```python
   titles = ["funny cats", "cooking tutorial", "Python basics"]
   for title in titles:
       url = create_search_url(title)
       print(url)
   ```
   (This previews loops from Chapter 2!)

3. **Special characters**: Test what happens with titles containing `&`, `?`, or `#`. Does `quote_plus` handle them correctly? What do they become?

---

## Common Mistakes

| Problem | Likely Cause |
|---------|--------------|
| `ModuleNotFoundError: No module named 'urllib'` | Check your import statement — it should be `from urllib.parse import quote_plus` |
| URL has spaces instead of `+` | You're not using `quote_plus()` on the title |
| `NameError: name 'quote_plus' is not defined` | Missing the import at the top of your file |
| Function doesn't output anything | Did you use `return` inside the function AND `print()` when calling it? |
| `IndentationError` | The lines inside your function need to be indented (usually 4 spaces). VS Code should do this automatically. |
| `SyntaxError: expected ':'` | You forgot the colon `:` at the end of the `def` line |

---

## Vocabulary Review

Terms you learned in this phase:

| Term | Definition |
|------|------------|
| **String** | Text data, written inside quotes (`"like this"` or `'like this'`) |
| **Variable** | A name that stores a value |
| **Concatenation** | Joining strings together with `+` |
| **URL encoding** | Converting special characters to be safe for URLs |
| **Function** | A reusable block of code with a name |
| **Parameter** | A variable in a function definition that receives input |
| **Argument** | The actual value you pass to a function when calling it |
| **Return** | Sending a value back from a function |
| **Docstring** | A string that documents what a function does |
| **Import** | Bringing in code from another module |

---

## What's Next

In **Phase 2**, you'll make this script interactive — it will ask the user for a title instead of using a hardcoded one. You'll also learn:
- How to get input from users with `input()`
- How to validate that input (what if they just press Enter?)
- How to use loops to keep asking until they give valid input
- How to use **Ruff** to check your code for style issues

---

## Commit Your Work

Once everything works, save your progress with Git:

1. Open the terminal in VS Code (`Ctrl+`` `)
2. Run these commands:

```bash
git add src/search_url.py
git commit -m "Phase 1: YouTube search URL generator"
```

**What these commands do:**
- `git add src/search_url.py` — tells Git to track changes to this file
- `git commit -m "..."` — saves a snapshot of your code with a message describing what you did

You now have a saved checkpoint you can always come back to!
