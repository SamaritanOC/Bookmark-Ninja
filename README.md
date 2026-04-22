# Bookmark Ninja
<img src="https://labb.run/wp-content/uploads/2025/11/bkm-ninja-logo.png">
**Turn your browser bookmarks into a knowledge base your AI agent can actually search.**

Converts any browser HTML bookmark export into a clean, structured JSON index with full folder hierarchy preserved, deduplication, merge support, and optional URL liveness verification.

Built as an internal tool for [The Samaritan Project](https://buymeacoffee.com/thesamaritanproject) — a self-hosted, multi-agent OSINT and intelligence research platform built on OpenClaw running on Parrot OS bare metal. In production it manages a 1,900+ entry intelligence source library across 246 categories, used by 11 specialized investigation agents.

---

## How It Works

Browser export (HTML) -> bookmark-parser.py -> bookmarks-index.json -> agent queries it

One command. No API keys. No external services. No configuration.

---

## Installation

**Requirements:** Python 3.7+. No API keys. No external services.

**Optional:** requests library for URL liveness checking:

    pip install requests

### Method 1: OpenClaw Dashboard (Recommended)

1. Click the green **Code** button on this page and select **Download ZIP**
2. Extract the ZIP and locate the `bookmark-ninja` folder inside
3. Open your OpenClaw dashboard
4. Navigate to the **Skills** tab
5. Click **Add Skill** or **Upload Skill**
6. Select the extracted `bookmark-ninja` folder
7. Enable the skill

Bookmark Ninja will appear in your agent skill list and activate automatically on bookmark-related tasks.

### Method 2: Command Line

    git clone https://github.com/SamaritanOC/Bookmark-Ninja.git
    cp -r Bookmark-Ninja/bookmark-ninja ~/.openclaw/skills/
    systemctl --user restart openclaw-gateway.service
    openclaw skills list

### Method 3: Standalone (no OpenClaw)

    git clone https://github.com/SamaritanOC/Bookmark-Ninja.git
    cd Bookmark-Ninja
    python3 bookmark-ninja/bookmark-parser.py your-bookmarks.html

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
        "title": "DeHashed -- #FreeThePassword",
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

This tool was built for and extracted from [The Samaritan Project](https://buymeacoffee.com/thesamaritanproject) — a build-in-public, self-hosted, multi-agent OSINT and intelligence research platform built on OpenClaw running on Parrot OS bare metal.

Samaritan is an OSINT automation platform that runs 11 specialized investigation agents. Bookmark Ninja is the pipeline that feeds its 1,900+ entry intelligence source library across 246 categories. This is one of several internal tools extracted and open sourced from a production AI investigation platform.

Support The Samaritan Project at [buymeacoffee.com/thesamaritanproject](https://buymeacoffee.com/thesamaritanproject)

---

## License

MIT — free to use, modify, and distribute.

---
## About the developer

[**Harold Mansfield**](https://linkedin.com/in/haroldmansfield/) - AI Integration Consultant | AI Coaching | Agentic OSINT Automation | Sec+ CySA+


