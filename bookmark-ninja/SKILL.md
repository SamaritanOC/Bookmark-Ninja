---
name: Bookmark Ninja
version: 1.0.0
description: Universal bookmark-to-knowledge-base converter. Ingests browser HTML bookmark exports (Chrome, Firefox, Edge, Brave), parses hierarchical structure, and outputs machine-readable JSON/CSV indexes for agent consumption.


Supports incremental merging, conflict resolution, and optional URL liveness verification. Use for research libraries, OSINT source management, legal discovery archives, or any profession requiring organized bookmark collections as queryable knowledge bases.
homepage: https://github.com/SamaritanOC/Bookmark-Ninja
metadata:
  openclaw:
    requires:
      bins:
        - python3
    install:
      - kind: uv
        package: requests
        bins: []
---

# Bookmark Ninja — Universal Bookmark Knowledge Base

## Purpose Notice


This skill transforms browser bookmark exports into structured, machine-readable knowledge bases. 


While originally developed for OSINT source management, it's profession-agnostic: researchers organizing academic papers, lawyers managing case law databases, journalists tracking sources, developers cataloging technical references, or anyone who has accumulated bookmarks and wants their AI agent to search and use them intelligently.


The skill processes standard Netscape HTML bookmark format (exported by Chrome, Firefox, Edge, Brave) and produces JSON/CSV indexes with full category hierarchy, metadata, and optional liveness verification.

---

## When to Use This Skill


Use Bookmark Ninja whenever you need to:


* **Convert browser bookmarks to agent-queryable format** — Transform exported HTML into structured JSON/CSV
* **Build a living knowledge base** — Incrementally add new bookmarks or HTML exports without wiping existing data
* **Organize accumulated research** — Preserve folder hierarchy as category breadcrumb paths
* **Verify link validity** — Optional HEAD request checks to identify dead URLs
* **Merge multiple bookmark sources** — Combine exports from different browsers or time periods
* **Share curated resource lists** — Export portable indexes others can query programmatically
* **Archive before cleanup** — Snapshot current bookmarks before browser migration or folder reorganization


Always prefer this skill over manual bookmark management when you need:
* **Programmatic search across large bookmark collections
* **Deduplication and conflict resolution
* **Category-based filtering and organization
* **Historical tracking via date_added timestamps
* **Cross-browser bookmark consolidation

---

## Core Capabilities

### 1. Parse Standard Bookmark Exports

Reads Netscape HTML format (the universal export format):
```bash
python3 bookmark-parser.py bookmarks.html
```

Extracts:
- **URL** — The bookmark link
- **Title** — Page title or user-defined name
- **Category** — Full folder hierarchy as breadcrumb (e.g., "Research > AI > Papers")
- **Description** — Metadata attribute if present
- **Date Added** — Timestamp converted to ISO 8601
- **Icon** — Favicon URL if present

Output: `bookmarks-index.json` (default)

### 2. Incremental Merging

Add new bookmarks to existing index without wiping data:
```bash
# Initial import
python3 bookmark-parser.py bookmarks-2024.html -o my-index.json

# Later: merge new export
python3 bookmark-parser.py bookmarks-2025.html -o my-index.json --merge
```

**Conflict handling:**
- Detects when same URL exists with different title/category/description
- Prompts for resolution: keep old, keep new, or skip
- Automate with `--keep-old` or `--keep-new` flags

### 3. URL Liveness Check

Verify links are still accessible:
```bash
python3 bookmark-parser.py bookmarks.html --check-alive
```

Performs HEAD request to each URL, adds `"alive": true/false` field.
Requires `requests` library (auto-installed via skill metadata).

### 4. Multiple Output Formats
```bash
# JSON only (default)
python3 bookmark-parser.py bookmarks.html

# CSV only
python3 bookmark-parser.py bookmarks.html --format csv

# Both formats
python3 bookmark-parser.py bookmarks.html --format both
```

### 5. Statistics & Analysis

Preview without saving:
```bash
python3 bookmark-parser.py bookmarks.html --stats
```

Shows:
- Total entries
- Category count
- Top 10 categories by entry count
- Alive/dead URL counts (if `--check-alive` used)

---

## Usage Examples

### Basic Import
```bash
# Export bookmarks from Chrome: Menu > Bookmarks > Bookmark Manager > ⋮ > Export bookmarks
# Firefox: Menu > Bookmarks > Manage Bookmarks > Import and Backup > Export Bookmarks to HTML

python3 bookmark-parser.py ~/Downloads/bookmarks.html
# Output: bookmarks-index.json
```

### Custom Output Location
```bash
python3 bookmark-parser.py bookmarks.html -o ~/research/sources.json
```

### Merge New Bookmarks
```bash
# Import initial collection
python3 bookmark-parser.py old-bookmarks.html -o index.json

# Later: add new bookmarks (prompts on conflicts)
python3 bookmark-parser.py new-bookmarks.html -o index.json --merge

# Auto-keep newer data
python3 bookmark-parser.py new-bookmarks.html -o index.json --merge --keep-new
```

### Verify Links Are Alive
```bash
python3 bookmark-parser.py bookmarks.html --check-alive -o verified.json
# Adds "alive": true/false to each entry
# Note: Adds latency (5s timeout per URL)
```

### Export as CSV
```bash
python3 bookmark-parser.py bookmarks.html --format csv
# Output: bookmarks-index.csv
```

### Quick Statistics
```bash
python3 bookmark-parser.py bookmarks.html --stats
```

---

## Output Format

### JSON Structure
```json
[
  {
    "url": "https://example.com",
    "title": "Example Site",
    "category": "Research > Web Dev > Tools",
    "description": "Useful web development tool",
    "date_added": "2024-03-15T10:30:00",
    "icon": "https://example.com/favicon.ico",
    "alive": true
  }
]
```

### CSV Columns
```
url,title,category,description,date_added,icon,alive
https://example.com,"Example Site","Research > Web Dev > Tools","Useful tool",2024-03-15T10:30:00,https://example.com/favicon.ico,True
```

---

## Agent Integration Patterns

### Search by Keyword
```python
import json

with open("bookmarks-index.json") as f:
    bookmarks = json.load(f)

# Search titles and descriptions
keyword = "machine learning"
matches = [
    b for b in bookmarks 
    if keyword.lower() in b["title"].lower() 
    or keyword.lower() in b["description"].lower()
]
```

### Filter by Category
```python
category = "OSINT > Email Search"
matches = [b for b in bookmarks if category in b["category"]]
```

### Get All Categories
```python
categories = sorted(set(b["category"] for b in bookmarks))
```

### Find Dead Links
```python
dead = [b for b in bookmarks if b.get("alive") == False]
```

### Recent Bookmarks
```python
from datetime import datetime, timedelta

recent_date = datetime.now() - timedelta(days=30)
recent = [
    b for b in bookmarks 
    if b["date_added"] and datetime.fromisoformat(b["date_added"]) > recent_date
]
```

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

## Workflow: Building a Research Library

1. **Initial import**
```bash
   python3 bookmark-parser.py research-bookmarks.html -o research-library.json
```

2. **Add new sources monthly**
```bash
   python3 bookmark-parser.py bookmarks-march.html -o research-library.json --merge --keep-new
```

3. **Verify links quarterly**
```bash
   python3 bookmark-parser.py bookmarks-current.html --check-alive -o research-library.json --merge --keep-new
```

4. **Agent queries the index**
```bash
   # Using jq for CLI queries
   cat research-library.json | jq '.[] | select(.category | contains("AI")) | .title'
   
   # Or load into Python/agent for programmatic search
```

---

## Troubleshooting

**"requests library not available" warning**
- Liveness check requires `requests` package
- Skill metadata auto-installs via `uv` if available
- Manual install: `pip install requests`


**Encoding errors on import**
- Parser uses `errors="ignore"` for malformed HTML
- If issues persist, re-export bookmarks from browser


**Category hierarchy looks wrong**
- Bookmark format nests categories via `<DL>` tags
- Verify export includes folder structure (not flat list)
- Chrome/Firefox: ensure "Include folder structure" is checked during export


**Merge conflicts on every entry**
- Date/time format changes trigger conflicts
- Use `--keep-new` to auto-accept updated metadata

---

## Related Skills

- **google-dorks** — Precision web search operators for research


- Use Bookmark Ninja to organize discovered sources from dork queries
- Export curated source lists as shareable bookmark HTML

---

## Version History

**1.0.0** (2026-03-30)
- Initial release
- Netscape HTML bookmark parsing
- JSON/CSV output
- Incremental merging with conflict resolution
- Optional URL liveness verification
- Category hierarchy preservation

## Included Script: bookmark-parser.py

```python
#!/usr/bin/env python3
"""
bookmark-parser.py — Universal bookmark-to-knowledge-base converter

Ingests browser HTML bookmark exports, parses structure, and outputs
machine-readable JSON index for agent consumption.

Supports: Chrome, Firefox, Edge, Brave (Netscape bookmark format)
"""

import sys
import json
import csv
from html.parser import HTMLParser
from pathlib import Path
from datetime import datetime
from urllib.parse import urlparse
import argparse

try:
    import requests
    REQUESTS_AVAILABLE = True
except ImportError:
    REQUESTS_AVAILABLE = False


class BookmarkParser(HTMLParser):
    """Parses Netscape HTML bookmark format."""

    def __init__(self):
        super().__init__()
        self.entries = []
        self.category_stack = []
        self.current_href = None
        self.current_attrs = {}
        self.capture_title = False
        self.current_title = ""

    def handle_starttag(self, tag, attrs):
        attrs_dict = dict(attrs)

        if tag == "h3":
            # Category header
            self.capture_title = True
            self.current_title = ""

        elif tag == "a":
            # Bookmark link
            self.current_href = attrs_dict.get("href", "")
            self.current_attrs = attrs_dict
            self.capture_title = True
            self.current_title = ""

        elif tag == "dl":
            # Descending into a category
            pass

    def handle_endtag(self, tag):
        if tag == "h3":
            # End of category header - push to stack
            category = self.current_title.strip()
            if category:
                self.category_stack.append(category)
            self.capture_title = False

        elif tag == "a":
            # End of bookmark link - record entry
            if self.current_href:
                category_path = " > ".join(self.category_stack) if self.category_stack else "Uncategorized"

                # Parse ADD_DATE (Unix timestamp)
                add_date = self.current_attrs.get("add_date", "")
                date_added = ""
                if add_date:
                    try:
                        date_added = datetime.fromtimestamp(int(add_date)).isoformat()
                    except (ValueError, OSError):
                        pass

                self.entries.append({
                    "url": self.current_href,
                    "title": self.current_title.strip(),
                    "category": category_path,
                    "description": self.current_attrs.get("description", ""),
                    "date_added": date_added,
                    "icon": self.current_attrs.get("icon", ""),
                })

            self.current_href = None
            self.current_attrs = {}
            self.capture_title = False

        elif tag == "dl":
            # Ascending from a category
            if self.category_stack:
                self.category_stack.pop()

    def handle_data(self, data):
        if self.capture_title:
            self.current_title += data


def parse_bookmarks(html_file):
    """Parse HTML bookmark file and return entries."""
    with open(html_file, "r", encoding="utf-8", errors="ignore") as f:
        content = f.read()

    parser = BookmarkParser()
    parser.feed(content)
    return parser.entries


def check_url_alive(url, timeout=5):
    """Check if URL is accessible via HEAD request."""
    if not REQUESTS_AVAILABLE:
        return None

    try:
        response = requests.head(url, timeout=timeout, allow_redirects=True)
        return response.status_code < 400
    except:
        return False


def merge_entries(existing, new, conflict_resolution="prompt"):
    """
    Merge new entries into existing index.

    Args:
        existing: list of existing entries
        new: list of new entries
        conflict_resolution: "keep-old", "keep-new", "prompt"

    Returns:
        merged list, conflict count
    """
    existing_map = {e["url"]: e for e in existing}
    conflicts = []

    for entry in new:
        url = entry["url"]

        if url in existing_map:
            old_entry = existing_map[url]

            # Check if anything changed
            if (old_entry["title"] != entry["title"] or
                old_entry["category"] != entry["category"] or
                old_entry["description"] != entry["description"]):

                conflicts.append({
                    "url": url,
                    "old": old_entry,
                    "new": entry
                })
        else:
            existing_map[url] = entry

    # Handle conflicts
    if conflicts and conflict_resolution == "prompt":
        print(f"\n⚠️  Found {len(conflicts)} conflicting entries:\n")

        for i, conflict in enumerate(conflicts, 1):
            print(f"Conflict {i}/{len(conflicts)}: {conflict['url']}")
            print(f"  OLD: [{conflict['old']['category']}] {conflict['old']['title']}")
            print(f"  NEW: [{conflict['new']['category']}] {conflict['new']['title']}")

            choice = input("  Keep (o)ld, (n)ew, or (s)kip? [o/n/s]: ").strip().lower()

            if choice == "n":
                existing_map[conflict["url"]] = conflict["new"]
            elif choice == "s":
                pass  # Keep old by default
            # else keep old

    elif conflicts and conflict_resolution == "keep-new":
        for conflict in conflicts:
            existing_map[conflict["url"]] = conflict["new"]

    # conflict_resolution == "keep-old" does nothing

    return list(existing_map.values()), len(conflicts)


def save_json(entries, output_file):
    """Save entries as JSON."""
    with open(output_file, "w", encoding="utf-8") as f:
        json.dump(entries, f, indent=2, ensure_ascii=False)
    print(f"✓ Saved JSON: {output_file} ({len(entries)} entries)")


def save_csv(entries, output_file):
    """Save entries as CSV."""
    if not entries:
        return

    fieldnames = ["url", "title", "category", "description", "date_added", "icon", "alive"]

    with open(output_file, "w", encoding="utf-8", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(entries)

    print(f"✓ Saved CSV: {output_file} ({len(entries)} entries)")


def main():
    parser = argparse.ArgumentParser(
        description="Convert browser bookmarks to machine-readable index",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  # Parse bookmark file
  %(prog)s bookmarks.html

  # Specify output location
  %(prog)s bookmarks.html -o output.json

  # Merge into existing index
  %(prog)s bookmarks.html -o index.json --merge

  # Check URL liveness (slow)
  %(prog)s bookmarks.html --check-alive

  # Handle conflicts automatically
  %(prog)s bookmarks.html -o index.json --merge --keep-new
        """
    )

    parser.add_argument("input", help="HTML bookmark file to parse")
    parser.add_argument("-o", "--output", help="Output file path (default: bookmarks-index.json)")
    parser.add_argument("--format", choices=["json", "csv", "both"], default="json",
                        help="Output format (default: json)")
    parser.add_argument("--merge", action="store_true",
                        help="Merge with existing index file")
    parser.add_argument("--keep-old", action="store_true",
                        help="On merge conflict, keep old entry (default: prompt)")
    parser.add_argument("--keep-new", action="store_true",
                        help="On merge conflict, keep new entry (default: prompt)")
    parser.add_argument("--check-alive", action="store_true",
                        help="Check URL liveness via HEAD request (adds latency)")
    parser.add_argument("--stats", action="store_true",
                        help="Print statistics only, don't save")

    args = parser.parse_args()

    # Validate input file
    if not Path(args.input).exists():
        print(f"❌ Error: Input file not found: {args.input}")
        sys.exit(1)

    # Determine output file
    if args.output:
        output_file = args.output
    else:
        output_file = "bookmarks-index.json"

    # Parse bookmarks
    print(f"📖 Parsing: {args.input}")
    entries = parse_bookmarks(args.input)
    print(f"✓ Parsed {len(entries)} entries")

    # Check liveness if requested
    if args.check_alive:
        if not REQUESTS_AVAILABLE:
            print("⚠️  Warning: requests library not available, skipping liveness check")
        else:
            print(f"🔍 Checking URL liveness (this may take a while)...")
            for i, entry in enumerate(entries, 1):
                alive = check_url_alive(entry["url"])
                entry["alive"] = alive
                if (i % 10 == 0) or (i == len(entries)):
                    print(f"  Progress: {i}/{len(entries)}")

    # Handle merge
    if args.merge and Path(output_file).exists():
        print(f"🔄 Merging with existing index: {output_file}")

        with open(output_file, "r", encoding="utf-8") as f:
            existing = json.load(f)

        conflict_resolution = "keep-old" if args.keep_old else ("keep-new" if args.keep_new else "prompt")
        entries, conflict_count = merge_entries(existing, entries, conflict_resolution)

        if conflict_count > 0:
            print(f"✓ Resolved {conflict_count} conflicts")

    # Print statistics
    categories = {}
    for entry in entries:
        cat = entry["category"]
        categories[cat] = categories.get(cat, 0) + 1

    print(f"\n📊 Statistics:")
    print(f"  Total entries: {len(entries)}")
    print(f"  Categories: {len(categories)}")

    if args.check_alive and REQUESTS_AVAILABLE:
        alive_count = sum(1 for e in entries if e.get("alive") == True)
        dead_count = sum(1 for e in entries if e.get("alive") == False)
        print(f"  Alive URLs: {alive_count}")
        print(f"  Dead URLs: {dead_count}")

    if args.stats:
        print("\nTop categories:")
        for cat, count in sorted(categories.items(), key=lambda x: -x[1])[:10]:
            print(f"  {cat}: {count}")
        sys.exit(0)

    # Save output
    print(f"\n💾 Saving output...")

    if args.format == "json" or args.format == "both":
        save_json(entries, output_file)

    if args.format == "csv" or args.format == "both":
        csv_file = output_file.replace(".json", ".csv") if output_file.endswith(".json") else output_file + ".csv"
        save_csv(entries, csv_file)

    print(f"\n✅ Complete!")


if __name__ == "__main__":
    main()
```

## Example Output

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
  },
  {
    "url": "https://hunter.io/",
    "title": "Hunter - Find email addresses in seconds",
    "category": "OSINT > Email Search",
    "description": "Finds email addresses, account clues, or email-linked identity data.",
    "date_added": "2026-03-14T09:20:00",
    "icon": "",
    "alive": true
  },
  {
    "url": "https://pimeyes.com/en",
    "title": "PimEyes: Face Recognition Search Engine",
    "category": "OSINT > Facial Rec/Image Search",
    "description": "Matches images or faces to duplicates, profiles, or source pages.",
    "date_added": "2026-03-14T09:20:00",
    "icon": "",
    "alive": true
  }
]
```
