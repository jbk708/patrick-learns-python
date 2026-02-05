# YouTube URL Generator → Roku Sync

## Project Overview

A progressive Python learning project that starts with a simple YouTube URL generator and builds toward a tool that syncs Roku playback to a YouTube URL on another device. The learner has **no prior coding experience** and is working through *Automate the Boring Stuff with Python* (ATBS) as their primary learning resource.

**Target platform**: Windows

## Architecture Goal

A CLI tool that:
1. Detects what a Roku device is currently playing (via Roku ECP)
2. Searches YouTube for a matching video
3. Returns a direct URL the user can open on another device

## Phase Breakdown

### Phase 1: String Manipulation (ATBS Ch. 1–4)
**Goal**: Script that converts a hardcoded video title into a YouTube search URL.

- String replacement and formatting
- Function definitions
- `urllib.parse.quote()` for URL encoding
- Print the constructed URL

**Deliverable**: `search_url.py` — takes a title string, prints a YouTube search URL.

### Phase 2: User Input + Validation (ATBS Ch. 5–6)
**Goal**: Interactive version with `input()` and edge-case handling.

- Accept user input for video title
- Validate non-empty input
- Handle special characters via `urllib.parse`
- Introduce dictionaries (e.g., mapping common abbreviations)

**Deliverable**: `search_url.py` updated with interactive input loop.

### Phase 3: YouTube API Integration (ATBS Ch. 12)
**Goal**: Query YouTube Data API v3 and return the actual top result URL.

- `requests` library for HTTP calls
- YouTube Data API v3 (free tier, requires API key)
- JSON response parsing to extract video ID
- Construct `https://www.youtube.com/watch?v=<VIDEO_ID>`
- `webbrowser.open()` to launch in browser

**Deliverable**: `youtube_search.py` — given a title, returns and opens the real video URL.

### Phase 4: Persistent History (ATBS Ch. 9–10)
**Goal**: Log searches and results; add config file for API key.

- Read/write JSON files for search history
- `config.json` for API key storage (add to `.gitignore`)
- "Recent searches" recall feature
- Basic file organization

**Deliverable**: `config.json`, `history.json`, updated main script with history commands.

### Phase 5: Roku Network Discovery (ATBS Ch. 12 + external)
**Goal**: Query the Roku External Control Protocol (ECP) for playback metadata.

- HTTP GET to `http://<roku-ip>:8060/query/media-player`
- XML parsing with `xml.etree.ElementTree`
- Extract available metadata (app name, playback state)
- Optional: SSDP device discovery to auto-find Roku IP

**Known limitation**: Roku ECP returns limited metadata — typically the app name and playback state, but NOT the specific video title or ID. The learner should discover this firsthand. Workarounds to explore:
- `query/active-app` for current app info
- `query/apps` for installed apps list
- Manual title entry as fallback

**Deliverable**: `roku_status.py` — discovers Roku on network and prints current playback state.

### Phase 6: CLI Integration + Polish (ATBS Ch. 11, 17)
**Goal**: Combine all pieces into a single CLI tool with `argparse`.

- Subcommands: `search`, `roku`, `history`
- Auto-detect Roku on local network
- Pull Roku metadata → search YouTube → open URL
- Fallback to manual title entry when metadata is insufficient
- Error handling and user-friendly messages
- `webbrowser` module to auto-open results

**Deliverable**: `roku_sync.py` (or package) — unified CLI tool.

### Phase 7 (Optional/Advanced): Workarounds
- Packet inspection with `scapy`
- Roku developer mode investigation
- Simple Flask web UI where one user can "share" what they're watching

## Code Standards

- Google-style docstrings on all functions
- Inline comments only for complex logic
- Type hints encouraged but not required (introduce in Phase 3+)
- Each phase should have working, runnable code before moving on

## Project Structure

```
roku-youtube-sync/
├── README.md
├── CLAUDE.md
├── .gitignore
├── config.json.example
├── requirements.txt
├── lessons/
│   ├── phase1_string_manipulation.md
│   ├── phase2_user_input.md
│   ├── phase3_youtube_api.md
│   ├── phase4_persistence.md
│   ├── phase5_roku_discovery.md
│   ├── phase6_cli_integration.md
│   └── phase7_advanced.md
├── src/
│   ├── search_url.py        # Phase 1-2
│   ├── youtube_search.py    # Phase 3
│   ├── history.py           # Phase 4
│   ├── roku_status.py       # Phase 5
│   └── roku_sync.py         # Phase 6
└── tests/                   # Optional, introduce if learner is interested
```

## Dependencies (accumulated across phases)

**Runtime:**
```
requests
```

**Development (installed with `uv add --dev`):**
```
ruff      # Introduced in Phase 2
ty        # Introduced in Phase 3
```

**Optional (Phase 7):**
```
flask     # Web UI option
scapy     # Packet inspection option
```

Standard library modules used: `urllib.parse`, `json`, `xml.etree.ElementTree`, `webbrowser`, `argparse`, `pathlib`, `datetime`

## Notes for Claude Code

- The learner is a **complete beginner**. Provide clear explanations when generating code.
- Each phase maps to specific ATBS chapters — avoid introducing concepts ahead of schedule.
- Favor simple, readable code over clever or idiomatic Python.
- When the learner hits the Roku metadata limitation in Phase 5, help them understand it's a platform constraint, not a bug in their code.
- Always test that scripts run on Windows (use `pathlib` over hardcoded paths, etc.).

---

## Lesson Progress Tracker

| Phase | Lesson File | Status | Notes |
|-------|-------------|--------|-------|
| 1 | `lessons/phase1_string_manipulation.md` | LESSON COMPLETE | Strings, functions, URL encoding |
| 2 | `lessons/phase2_user_input.md` | LESSON COMPLETE | input(), validation, loops, dictionaries, **introduces Ruff** |
| 3 | `lessons/phase3_youtube_api.md` | LESSON COMPLETE | requests, JSON, APIs, webbrowser, **introduces type hints + ty** |
| 4 | `lessons/phase4_persistence.md` | LESSON COMPLETE | File I/O, pathlib, JSON persistence, config management |
| 5 | `lessons/phase5_roku_discovery.md` | LESSON COMPLETE | HTTP to local devices, XML parsing, discovering limitations |
| 6 | `lessons/phase6_cli_integration.md` | LESSON COMPLETE | argparse, subcommands, error handling, full workflow |
| 7 | `lessons/phase7_advanced.md` | LESSON COMPLETE | Optional: Flask web UI, packet inspection, advanced workarounds |

### Beginner Review Status

Reviewing each lesson to ensure complete beginners can follow along:

| Phase | Reviewed | Enhancements Made |
|-------|----------|-------------------|
| 1 | ✅ DONE | Added: How to create files in VS Code, step-by-step explanations for each exercise, ATBS chapter references, expected output examples, vocabulary review table, detailed git commit explanation |
| 2 | ✅ DONE | Added: Detailed Ruff explanation (what it is, installation, usage, auto-fix), step-by-step for each exercise, comparison operators breakdown, while loop execution trace, `if __name__` explanation, vocabulary table |
| 3 | ✅ DONE | Added: HTTP/API fundamentals, JSON navigation tutorial, type hints introduction with examples, runtime vs dev dependencies, complete code with detailed comments, API quota notes, vocabulary table |
| 4 | ✅ DONE | Added: Path problem explanation with examples, `__file__` and `.parent` breakdown, file I/O modes, datetime/isoformat intro, list slicing tutorial, module imports explanation, vocabulary table |
| 5 | ✅ DONE | Added: ECP endpoint table, XML vs JSON comparison, try/except explanation, network communication diagram, platform limitation discovery walkthrough, vocabulary table |
| 6 | ✅ DONE | Added: CLI concepts intro, argparse tutorial with subcommands, step-by-step command implementations, complete working code, run.bat script for Windows, vocabulary table |
| 7 | ✅ DONE | Added: Flask web basics, decorator explanation, complete web app code, HTML templates with Jinja2, form handling, project summary, "what's next" guidance, vocabulary table |

### Git Learning Progression

Git is taught progressively alongside Python. Each phase introduces new Git concepts:

| Phase | Git Concepts Introduced | Commands Learned |
|-------|------------------------|------------------|
| README | SSH setup, cloning repos, Git workflow overview | `git clone`, `git config` |
| 1 | What is Git, the three areas, basic workflow | `git status`, `git add`, `git commit`, `git log` |
| 2 | Seeing what changed, fixing commits | `git diff`, `git diff --staged`, `git commit --amend` |
| 3 | Working with GitHub, syncing code | `git remote`, `git push`, `git pull` |
| 4 | Undoing changes, reverting commits | `git checkout --`, `git restore`, `git revert`, `git reset` |
| 5 | Branches and merging | `git branch`, `git checkout -b`, `git switch`, `git merge` |

### Learner Progress

- [ ] Phase 1: String Manipulation
- [ ] Phase 2: User Input + Validation
- [ ] Phase 3: YouTube API Integration
- [ ] Phase 4: Persistent History
- [ ] Phase 5: Roku Network Discovery
- [ ] Phase 6: CLI Integration + Polish
- [ ] Phase 7: Advanced Workarounds (Optional)
