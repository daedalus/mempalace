---
name: mempalace
description: Use MemPalace to store and retrieve memories, manage the palace database, mine project files and conversations, query the knowledge graph, and manage agent diaries. Triggers when the user wants to remember something, search past conversations, file new content, query knowledge about people/projects, write diary entries, or set up the MCP server. Also triggers for mining project directories or conversation exports into the palace.
---

# MemPalace — AI Memory System

MemPalace gives your AI a memory. It stores verbatim content and makes it instantly searchable. No API keys required. 100% local.

## Core Concepts

**Wing** — A broad category (person, project, or agent)
**Room** — A time-based or topic-based grouping within a wing
**Drawer** — A verbatim content chunk (your exact words, never summarized)
**Hall** — Emotional/topic categorization within rooms (hall_facts, hall_events, hall_discoveries, hall_preferences, hall_advice)
**Tunnel** — Cross-wing connections linking ideas across different domains

## MCP Tools (Primary Interface)

MemPalace exposes 30+ MCP tools. Use these for all memory operations:

### Read Tools
- `mempalace_status` — Palace overview (total drawers, wing/room breakdown)
- `mempalace_list_wings` — All wings with drawer counts
- `mempalace_list_rooms` — Rooms within a wing
- `mempalace_get_taxonomy` — Full wing → room → count tree
- `mempalace_search` — Semantic search with similarity scores
- `mempalace_check_duplicate` — Check if content exists before filing
- `mempalace_get_drawer` — Fetch a single drawer by ID
- `mempalace_list_drawers` — List drawers with pagination
- `mempalace_get_aaak_spec` — Get AAAK dialect specification

### Write Tools
- `mempalace_add_drawer` — File verbatim content into a wing/room
- `mempalace_delete_drawer` — Delete a drawer by ID
- `mempalace_update_drawer` — Update drawer content and/or metadata

### Knowledge Graph
- `mempalace_kg_query` — Query entity relationships (temporal, with validity dates)
- `mempalace_kg_add` — Add a fact (subject → predicate → object)
- `mempalace_kg_invalidate` — Mark a fact as no longer true
- `mempalace_kg_timeline` — Chronological timeline of facts
- `mempalace_kg_stats` — Knowledge graph overview

### Palace Graph (Tunnels)
- `mempalace_traverse` — Walk the graph from a room
- `mempalace_find_tunnels` — Find rooms connecting two wings
- `mempalace_create_tunnel` — Create cross-wing tunnel
- `mempalace_list_tunnels` — List explicit tunnels
- `mempalace_delete_tunnel` — Delete a tunnel
- `mempalace_follow_tunnels` — Follow tunnels from a room
- `mempalace_graph_stats` — Graph overview

### Agent Diary
- `mempalace_diary_write` — Write diary entry in AAAK format
- `mempalace_diary_read` — Read recent diary entries

### Maintenance
- `mempalace_reconnect` — Force reconnect after external writes
- `mempalace_hook_settings` — Configure hook behavior
- `mempalace_memories_filed_away` — Check latest checkpoint

## CLI Commands

For direct shell access:

```bash
# Initialize a project (detects rooms, entities)
mempalace init ~/projects/my_app

# Mine project files
mempalace mine ~/projects/my_app

# Mine conversations
mempalace mine ~/chats/ --mode convos

# Search
mempalace search "why did we switch to graphql"

# Status
mempalace status

# MCP setup
mempalace mcp
```

## Mining Project Files

The miner reads project directories and files content into drawers. Key features:

- **Room detection**: Automatic routing based on folder paths, filenames, and keyword scoring
- **Chunking**: 800 chars per drawer with paragraph-aware splitting
- **Entity extraction**: Auto-detects people and projects from content
- **Hall detection**: Categorizes content by emotional/topic type

### miner.py Workflow

1. Load `mempalace.yaml` from project directory for wing/room config
2. Scan files (respects `.gitignore` by default)
3. Route each file to a room based on path, filename, or content keywords
4. Chunk file content into 800-char drawers
5. File each chunk with metadata (wing, room, source_file, etc.)

### convo_miner.py — Conversation Mining

Ingests chat exports from:
- Claude Code JSONL
- Claude.ai JSON
- ChatGPT conversations.json
- Slack exports
- Plain text with `>` markers

Two extract modes:
- `exchange` — Default, chunks by Q&A pairs
- `general` — Extracts 5 memory types (decisions, preferences, milestones, problems, emotional)

### normalize.py — Format Detection

Auto-detects and converts formats:
- Plain text with `>` markers (pass through)
- Claude Code JSONL
- Claude.ai JSON export
- ChatGPT conversations.json
- Slack JSON
- OpenAI Codex CLI JSONL

Strips noise: system tags, hook output, Claude Code UI chrome.

### general_extractor.py — Memory Type Extraction

Extracts 5 types from any text:
1. **DECISIONS** — "we went with X because Y"
2. **PREFERENCES** — "always use X", "never do Y"
3. **MILESTONES** — breakthroughs, things that finally worked
4. **PROBLEMS** — what broke, what fixed it
5. **EMOTIONAL** — feelings, vulnerability

Pure regex/keyword heuristics. No LLM required.

## AAAK Dialect

Compressed format for efficient storage:

```
ENTITIES: 3-letter codes (ALC=Alice, JOR=Jordan)
EMOTIONS: *markers* (*warm*=joy, *fierce*=determined)
STRUCTURE: Pipe-separated (FAM: | PROJ: | ⚠:)
DATES: ISO format (2026-03-31)
COUNTS: Nx = N mentions
IMPORTANCE: ★ to ★★★★★
HALLS: hall_facts, hall_events, hall_discoveries, hall_preferences, hall_advice
WINGS: wing_user, wing_agent, wing_code, wing_myproject
ROOMS: Hyphenated slugs (chromadb-setup, gpu-pricing)
```

Example:
```
FAM: ALC→♡JOR | 2D(kids): RIL(18,sports) MAX(11,chess+swimming)
```

## MCP Server Setup

```bash
# Add to Claude Code
claude mcp add mempalace -- python -m mempalace.mcp_server

# With custom palace path
claude mcp add mempalace -- python -m mempalace.mcp_server --palace /path/to/palace

# Run standalone
python -m mempalace.mcp_server --palace ~/.mempalace/palace
```

## Memory Protocol (Critical)

ALWAYS follow this protocol:

1. **ON WAKE-UP**: Call `mempalace_status` and `mempalace_get_aaak_spec` to load palace overview
2. **BEFORE RESPONDING** about any person, project, or past event: Query the palace first. Never guess.
3. **IF UNSURE** about a fact (name, gender, relationship): Say "let me check" and query
4. **AFTER EACH SESSION**: Call `mempalace_diary_write` to record what happened
5. **WHEN FACTS CHANGE**: Invalidate old fact, add new one

## Best Practices

- **Verbatim always**: Store exact words, never summarize
- **Entity-first**: Key by real names with disambiguation
- **Query before responding**: Never guess about past events
- **File incrementally**: Append-only after initial build
- **Use tunnels**: Connect related ideas across wings
- **Write diaries**: Record observations in AAAK format
- **Daily checkpoint**: Use the stop hook for auto-save

## File Paths

- Palace: `~/.mempalace/palace` (default) or custom `--palace` path
- Config: `~/.mempalace/config.json`
- Knowledge Graph: `~/.mempalace/knowledge_graph.sqlite3`
- Entities: `~/.mempalace/known_entities.json`
- WAL (audit log): `~/.mempalace/wal/write_log.jsonl`

## Error Handling

- "No palace found": Run `mempalace init <dir>` then `mempalace mine <dir>`
- "Drawer not found": Verify the drawer_id, use `mempalace_list_drawers` to find it
- Stale index: Call `mempalace_reconnect` after external CLI writes