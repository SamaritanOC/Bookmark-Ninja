# Bookmark Ninja

**Turn your browser bookmarks into a knowledge base your AI agent can actually search.**

Converts any browser HTML bookmark export into a clean, structured JSON index with full folder hierarchy preserved, deduplication, merge support, and optional URL liveness verification.

Built as a core utility inside [The Samaritan Project](https://github.com/SamaritanOC/SamaritanV01) — a self-hosted, multi-agent OSINT investigation platform running on OpenClaw. Bookmark Ninja powers Samaritan's 1,900+ entry intelligence source library across 246 categories.

---

## How It Works

Browser export (HTML) -> bookmark-parser.py -> bookmarks-index.json -> agent queries it

One command. No API keys. No external services. No configuration.

---

## Quick Start

**1. Export your bookmarks**

| Browser | Steps |
|---|---|
| Chrome | Menu -> Bookmarks -> Bookmark Manager -> More -> Export bookmarks |
| Firefox | Menu -> Bookmarks -> Manage Bookmarks -> Import and Backup -> Export Bookmarks to HTML |
| Edge / Brave | Same as Chrome |

Saves as bookmarks_[date].html

**2. Run the parser**

    python3 bookmark-ninja/bookmark-parser.py bookmarks.html

Output: bookmarks-index.json — structured, searchable, agent-ready.

**3. Query from your agent**

    import json

    with open("bookmarks-index.json") as f:
        bookmarks = json.load(f)

    # Search by keyword
    results = [b for b in bookmarks if "osint" in b["title"].lower()]

    # Filter by category
    email_tools = [b for b in bookmarks if "Email Search" in b["category"]]

    # Find dead links
    dead = [b for b in bookmarks if b.get("alive") == False]

---

## Installation

**Requirements:** Python 3.7+. No API keys. No external services.

**Optional:** requests library for URL liveness checking:

    pip install requests

### Standalone (no OpenClaw)

    git clone https://github.com/SamaritanOC/Bookmark-Ninja.git
    cd Bookmark-Ninja
    python3 bookmark-ninja/bookmark-parser.py your-bookmarks.html

### OpenClaw Skill Installation

    git clone https://github.com/SamaritanOC/Bookmark-Ninja.git
    cp -r Bookmark-Ninja/bookmark-ninja ~/.openclaw/skills/
    systemctl --user restart openclaw-gateway.service
    openclaw skills list

Bookmark Ninja will appear in your agent skill list and activate automatically on bookmark-related tasks.

---

## Usage

### Parse bookmarks

    python3 bookmark-ninja/bookmark-parser.py bookmarks.html

### Custom output location

    python3 bookmark-ninja/bookmark-parser.py bookmarks.html -o ~/research/sources.json

### Merge new bookmarks into existing index

    # First import
    python3 bookmark-ninja/bookmark-parser.py bookmarks-jan.html -o index.json

    # Add new bookmarks later — conflicts prompted interactively
    python3 bookmark-ninja/bookmark-parser.py bookmarks-apr.html -o index.json --merge

    # Auto-resolve conflicts
    python3 bookmark-ninja/bookmark-parser.py bookmarks-apr.html -o index.json --merge --keep-new

### Verify URL liveness

    python3 bookmark-ninja/bookmark-parser.py bookmarks.html --check-alive

Adds alive: true/false to each entry. Note: 5s timeout per URL — slow on large collections.

### Statistics without saving

    python3 bookmark-ninja/bookmark-parser.py bookmarks.html --stats

### Export as CSV

    python3 bookmark-ninja/bookmark-parser.py bookmarks.html --format csv

---

## Output Format

    [
      {
        "url": "https://dehashed.com/",
        "title": "DeHashed — #FreeThePassword",
        "category": "OSINT > Breach Data/Dumps",
        "description": "Searches breach, paste, or leak data for emails, usernames, and exposed records.",
        "date_added": "2026-03-14T09:20:00",
        "icon": "",
        "alive": true
      }
    ]

Fields: url, title, category (full folder path as breadcrumb), description, date_added (ISO 8601), icon, alive (present only if liveness check was run).

---

## Command Reference

    usage: bookmark-parser.py [-h] [-o OUTPUT] [--format {json,csv,both}]
                              [--merge] [--keep-old] [--keep-new]
                              [--check-alive] [--stats]
                              input

    positional arguments:
      input           HTML bookmark file to parse

    options:
      -o OUTPUT       Output path (default: bookmarks-index.json)
      --format        Output format: json, csv, or both (default: json)
      --merge         Merge with existing index file
      --keep-old      On conflict, keep old entry (default: prompt)
      --keep-new      On conflict, keep new entry (default: prompt)
      --check-alive   Verify URL liveness via HEAD request
      --stats         Show statistics only, do not save

---

## Part of The Samaritan Project

Bookmark Ninja was extracted from [The Samaritan Project](https://github.com/SamaritanOC/SamaritanV01), a self-hosted multi-agent OSINT and intelligence research platform built on OpenClaw. In production it manages a 1,900+ entry intelligence source library used across 11 specialized investigation agents.

Other open source tools from the same platform: [Search Ninja](https://github.com/SamaritanOC/Search-Ninja)

---

## License

MIT — free to use, modify, and distribute.

---

*Built by [Harold Mansfield](https://github.com/SamaritanOC) — AI Systems Engineer*
