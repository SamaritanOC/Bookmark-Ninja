<img src="https://smbconsultants.ai/wp-content/uploads/2026/04/Bookmark-Ninja.png">

**Turn your browser bookmarks into a knowledge base your AI agent can actually search.**

Bookmark Ninja converts any browser HTML bookmark export into a clean, structured JSON or CSV index with full folder hierarchy preserved, deduplication, merge support, and optional URL liveness verification. One command, no API keys, no external services, no configuration.

**Version:** 1.1.0

---

## Compatibility

Bookmark Ninja works on any platform that supports the Agent Skill standard:

- Claude.ai
- Cowork
- Claude Code
- Codex
- OpenClaw
- Any platform supporting the Agent Skill standard

**Runtime requirements:** Python 3.7+. Optional: `requests` library for URL liveness checking (`pip install requests`).

---

## Installation

### Claude.ai / Cowork
1. Download this repository as a ZIP (green **Code** button → **Download ZIP**)
2. Extract the ZIP
3. Go to **Settings → Capabilities → Skills**
4. Upload the `bookmark-ninja` folder
5. Enable the skill

### Claude Code
```bash
cp -r bookmark-ninja ~/.claude/skills/
```

### OpenClaw Dashboard
1. Click the green **Code** button → **Download ZIP**
2. Extract the ZIP and locate the `bookmark-ninja` folder
3. Open your OpenClaw dashboard → **Skills** tab
4. Click **Add Skill** and select the extracted `bookmark-ninja` folder
5. Enable the skill

### OpenClaw terminal
```bash
cp -r bookmark-ninja ~/.openclaw/skills/
systemctl --user restart openclaw-gateway.service
```

### Standalone (no agent platform)
```bash
git clone https://github.com/SamaritanOC/Bookmark-Ninja.git
cd Bookmark-Ninja
python3 scripts/bookmark-parser.py your-bookmarks.html
```

---

## Quick Start

**1. Export your bookmarks**

| Browser | Steps |
|---|---|
| Chrome | Menu → Bookmarks → Bookmark Manager → ⋮ → Export bookmarks |
| Firefox | Menu → Bookmarks → Manage Bookmarks → Import and Backup → Export Bookmarks to HTML |
| Edge | Menu → Favorites → ⋮ → Export favorites |
| Brave | Same as Chrome |

Saves as `bookmarks_[date].html`.

**2. Run the parser**

```bash
python3 scripts/bookmark-parser.py bookmarks.html
```

Output: `bookmarks-index.json` — structured, searchable, agent-ready.

**3. Query from your agent**

```python
import json

with open("bookmarks-index.json") as f:
    bookmarks = json.load(f)

# Search by keyword
results = [b for b in bookmarks if "osint" in b["title"].lower()]

# Filter by category
email_tools = [b for b in bookmarks if "Email Search" in b["category"]]

# Find dead links
dead = [b for b in bookmarks if b.get("alive") == False]
```

---

## Usage

### Parse bookmarks

```bash
python3 scripts/bookmark-parser.py bookmarks.html
```

### Custom output location

```bash
python3 scripts/bookmark-parser.py bookmarks.html -o ~/research/sources.json
```

### Merge new bookmarks into existing index

```bash
# First import
python3 scripts/bookmark-parser.py bookmarks-jan.html -o index.json

# Add new bookmarks later — conflicts prompted interactively
python3 scripts/bookmark-parser.py bookmarks-apr.html -o index.json --merge

# Auto-resolve conflicts
python3 scripts/bookmark-parser.py bookmarks-apr.html -o index.json --merge --keep-new
```

### Verify URL liveness

```bash
python3 scripts/bookmark-parser.py bookmarks.html --check-alive
```

Adds `alive: true/false` to each entry. Note: 5s timeout per URL — slow on large collections.

### Statistics without saving

```bash
python3 scripts/bookmark-parser.py bookmarks.html --stats
```

### Export as CSV

```bash
python3 scripts/bookmark-parser.py bookmarks.html --format csv
```

### Both formats at once

```bash
python3 scripts/bookmark-parser.py bookmarks.html --format both
```

---

## Output Format

```json
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
```

Fields: `url`, `title`, `category` (full folder path as breadcrumb), `description`, `date_added` (ISO 8601), `icon`, `alive` (present only if liveness check was run).

---

## Command Reference

```
usage: bookmark-parser.py [-h] [-o OUTPUT] [--format {json,csv,both}]
                          [--merge] [--keep-old] [--keep-new]
                          [--check-alive] [--stats]
                          input

Convert browser bookmarks to machine-readable index

positional arguments:
  input                 HTML bookmark file to parse

options:
  -h, --help            show this help message and exit
  -o OUTPUT, --output OUTPUT
                        Output file path (default: bookmarks-index.json)
  --format {json,csv,both}
                        Output format (default: json)
  --merge               Merge with existing index file
  --keep-old            On merge conflict, keep old entry (default: prompt)
  --keep-new            On merge conflict, keep new entry (default: prompt)
  --check-alive         Check URL liveness via HEAD request (adds latency)
  --stats               Print statistics only, don't save
```

---

## Part of The Samaritan Project

This tool was built for and extracted from [The Samaritan Project](https://buymeacoffee.com/thesamaritanproject) — a build-in-public, self-hosted, multi-agent OSINT and intelligence research platform built on OpenClaw running on Parrot OS bare metal.

Samaritan runs 11 specialized investigation agents. Bookmark Ninja is the pipeline that feeds its 1,900+ entry intelligence source library across 246 categories. This is one of several internal tools extracted and open sourced from a production AI investigation platform.

Support The Samaritan Project at [buymeacoffee.com/thesamaritanproject](https://buymeacoffee.com/thesamaritanproject)

---

## License

MIT — free to use, modify, and distribute.

---

## About the developer

[**Harold Mansfield**](https://linkedin.com/in/haroldmansfield/) — AI Integration Consultant | AI Coaching | Agentic OSINT Automation | Sec+ CySA+
