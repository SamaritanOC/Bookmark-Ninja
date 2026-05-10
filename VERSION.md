# Bookmark Ninja — Version History

## [1.1.0] — 2026-05-10
Anthropic Agent Skill standard compliance refactor.
- Renamed frontmatter name from "Bookmark Ninja" to "bookmark-ninja" (kebab-case required by standard)
- Restructured frontmatter: moved version and homepage into metadata, replaced openclaw-specific install metadata with standard compatibility field
- Moved bookmark-parser.py from root to scripts/bookmark-parser.py (Anthropic standard structure)
- Moved example-output.json from root to references/example-output.json
- Removed embedded full script source code from SKILL.md (was duplicating scripts/bookmark-parser.py — major token waste and drift risk)
- Removed OpenClaw-specific TOOLS.md "Wiring" section from README.md
- Removed hardcoded ~/.openclaw paths from documentation
- Added install instructions for Claude.ai, Cowork, Claude Code alongside existing OpenClaw methods
- No script behavior changes

## [1.0.0] — 2026-03-30
Initial release.
- Netscape HTML bookmark parsing
- JSON/CSV output
- Incremental merging with conflict resolution
- Optional URL liveness verification
- Category hierarchy preservation
