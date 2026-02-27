# Requirements: LLM Hive Mind

**Defined:** 2026-02-24
**Source:** .planning/PRD.md
**Core Value:** Shared, topic-scoped memory bus across LLM CLI tools with source attribution

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Core CLI Commands

- [ ] **REQ-001**: CLI tool with `topic create <name>` command that initializes a topic-scoped context space *(Must)*
- [ ] **REQ-002**: `log <message> --source <llm>` command that writes an entry to the active topic with LLM attribution *(Must)*
- [ ] **REQ-003**: `read` command that displays all entries for the active topic, with optional `--from <llm>` filter *(Must)*
- [ ] **REQ-004**: `status` command that shows topic state: name, entry count, contributing LLMs *(Should)*
- [ ] **REQ-005**: `topic list` command to show all topics in the current repo *(Should)*

### GitHub Integration

- [ ] **REQ-010**: Each log entry creates a GitHub Issue in the current repo *(Must)*
- [ ] **REQ-011**: `topic create` creates a GitHub Project as the topic scope (e.g., "Zelda", "Crucible") *(Must)*
- [ ] **REQ-012**: Issues are labeled with `hive:source:<llm>` for LLM attribution *(Must)*
- [ ] **REQ-013**: Each log entry's Issue is added to the active topic's GitHub Project *(Must)*

### Configuration & Setup

- [ ] **REQ-100**: First-run setup validates `gh` CLI is authenticated and repo exists *(Should)*
- [ ] **REQ-101**: Topic configuration stored in `.hive/config.json` in repo root *(Should)*
- [ ] **REQ-102**: Sensible defaults — if `--source` omitted, use "unknown" or attempt env var detection *(Should)*

### Usability

- [ ] **REQ-200**: Help text for all commands (`hive --help`, `hive <command> --help`) *(Could)*
- [ ] **REQ-201**: Error messages are clear and suggest fixes (e.g., "gh not found — install with: brew install gh") *(Could)*

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Advanced Features

- **REQ-D01**: MCP server mode — expose hive as an MCP tool so LLMs can use it natively without shell commands
- **REQ-D02**: Auto-detection of running LLM via env vars (`CLAUDE_CODE`, `GEMINI_CLI`, etc.) or process inspection
- **REQ-D03**: `review <llm>` command — structured cross-LLM review workflow (pull another LLM's entries, prompt for critique)
- **REQ-D04**: File attachment support — attach code diffs, plan files, or screenshots to entries
- **REQ-D05**: Cross-repo topic linking — single topic spanning multiple repositories
- **REQ-D06**: `diff` command — show what changed since last read (incremental context)
- **REQ-D07**: GitHub Actions automation — webhook triggers on new entries
- **REQ-D08**: Topic archival — close/archive completed topics
- **REQ-D09**: Project board columns — add "Plan" and "Execution" columns to topic Projects for visual categorization

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Dependency tracking / task ordering | Memory bus, not PM tool — use Asana/Linear/GitHub Projects separately |
| Prioritization / sprint planning | Memory bus, not PM tool — use Asana/Linear/GitHub Projects separately |
| Cloud sync / hosted backend | Git IS the sync layer — no extra infrastructure |
| Authentication / user management | Single user tool, no multi-tenancy in v1 |
| Web UI dashboard | GitHub IS the UI — no custom frontend |
| Real-time notifications | Polling/read is sufficient for the use case |
| Chat-based LLM support | This is for CLI tools only (Claude Code, Gemini CLI, Codex CLI) |

## Traceability

Which phases cover which requirements. Updated by `/zelda:plan`.

| Requirement | Phase | Status |
|-------------|-------|--------|
| REQ-001 | Phase 1: Foundation | Pending |
| REQ-002 | Phase 2: Write Path | Pending |
| REQ-003 | Phase 3: Read Path | Pending |
| REQ-004 | Phase 3: Read Path | Pending |
| REQ-005 | Phase 1: Foundation | Pending |
| REQ-010 | Phase 2: Write Path | Pending |
| REQ-011 | Phase 1: Foundation | Pending |
| REQ-012 | Phase 2: Write Path | Pending |
| REQ-013 | Phase 2: Write Path | Pending |
| REQ-100 | Phase 1: Foundation | Pending |
| REQ-101 | Phase 1: Foundation | Pending |
| REQ-102 | Phase 2: Write Path | Pending |
| REQ-200 | Phase 4: Polish | Pending |
| REQ-201 | Phase 4: Polish | Pending |

**Coverage:**
- v1 requirements: 14 total
- Mapped to phases: 14
- Unmapped: 0

## Acceptance Criteria Reference

Detailed acceptance criteria for each requirement. Referenced during verification.

### REQ-001: Topic Create Command
- [ ] `hive topic create auth-refactor` creates a new topic named "auth-refactor"
- [ ] Running the command outputs confirmation with the topic name
- [ ] Creating a duplicate topic name returns a clear error
- [ ] Topic name is validated (no spaces, special chars — kebab-case only)
**PRD Reference**: Solution Detail > Core Capabilities

### REQ-002: Log Command
- [ ] `hive log "Plan: use JWT" --source claude` creates an entry attributed to Claude
- [ ] Entry includes timestamp, source LLM, and full message
- [ ] Message can be multi-line (quoted or piped from stdin)
- [ ] `--source` flag accepts any string (not restricted to known LLMs)
**PRD Reference**: Solution Detail > Core Capabilities

### REQ-003: Read Command
- [ ] `hive read` shows all entries for active topic in chronological order
- [ ] `hive read --from gemini` filters to only Gemini's entries
- [ ] Output includes: timestamp, source LLM, message preview
- [ ] Empty topic returns "No entries yet" message
**PRD Reference**: Solution Detail > Core Capabilities

### REQ-010: GitHub Issues Integration
- [ ] Each `hive log` command creates a GitHub Issue in the current repo
- [ ] Issue title is the first line of the log message
- [ ] Issue body contains full message, timestamp, and source metadata
- [ ] Issues are created via `gh issue create` (requires `gh` CLI)
**PRD Reference**: Technical Approach > Architecture Notes

### REQ-011: Topic as GitHub Project
- [ ] `hive topic create zelda` creates a GitHub Project named "Zelda" (or "hive: zelda")
- [ ] Project is created via `gh project create`
- [ ] Project number is stored in `.hive/config.json` for subsequent commands
- [ ] `hive read` retrieves items from the active topic's Project
**PRD Reference**: Technical Approach > Architecture Notes

### REQ-012: Source Label
- [ ] Each issue gets label `hive:source:<llm>` (e.g., `hive:source:claude`)
- [ ] Labels are auto-created if they don't exist
- [ ] `hive read --from claude` filters Project items by this label
**PRD Reference**: Technical Approach > Architecture Notes

### REQ-013: Issue-to-Project Assignment
- [ ] Each `hive log` command adds the created Issue to the active topic's GitHub Project
- [ ] Uses `gh project item-add` with the stored project number
- [ ] If no active topic is set, command errors with "No active topic — run `hive topic create <name>` first"
**PRD Reference**: Technical Approach > Architecture Notes

---
*Requirements defined: 2026-02-24*
*Last updated: 2026-02-24 — refined model: Topics = GitHub Projects (scoping), Labels = LLM attribution. Memory bus only (no PM features).*
