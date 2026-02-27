# Materials

**Generated**: 2026-02-24
**Mode**: CREATE (fresh start)

---

## Existing PRD

**Status**: Not provided
**Path**: N/A

---

## Codebases

**Status**: Greenfield
**Count**: 0

No existing codebases — greenfield project.

---

## Inspiration Products

**Count**: 3 sources analyzed

### Supermemory MCP
**Type**: URL
**Source**: https://github.com/supermemoryai/supermemory-mcp
**Analyzed**: 2026-02-24

**What It Is**: Universal memory API accessible via MCP. Makes memories available across MCP-compatible clients.

**Problem It Solves**: Memories are trapped in individual LLM apps (ChatGPT, Claude, etc.).

**Key Capabilities**:
- Universal memory layer via MCP
- Cloud-hosted API
- No authentication required for MCP clients

**What User Liked**:
- Concept of cross-LLM memory sharing

**How To Apply**:
- Validated the market need for cross-LLM context sharing
- Identified gaps: no topic scoping, no LLM attribution, cloud-only

**Gaps Identified**:
- Cloud-hosted (not local-first)
- No topic/project scoping
- No source attribution (which LLM wrote what)
- MCP-only (Gemini CLI doesn't have MCP)

---

### OpenMemory (CaviraOSS)
**Type**: URL
**Source**: https://github.com/CaviraOSS/OpenMemory
**Analyzed**: 2026-02-24

**What It Is**: Self-hosted cognitive memory engine with MCP server, multi-sector memory, temporal reasoning.

**Key Capabilities**:
- Local-first architecture
- MCP integration
- CLI tool (`opm`)
- Multi-sector memory (episodic, semantic, procedural)

**Gaps Identified**:
- No automatic cross-tool sync
- Topic scoping via tags only (not first-class)
- No structured cross-LLM review workflow

---

### Mem0 OpenMemory MCP
**Type**: URL
**Source**: https://mem0.ai/blog/introducing-openmemory-mcp
**Analyzed**: 2026-02-24

**What It Is**: Local-first persistent memory layer with MCP server.

**Key Capabilities**:
- Cross-tool context sharing ("store in Cursor, retrieve in Claude")
- Local-first
- MCP server

**Gaps Identified**:
- No topic scoping
- No LLM attribution
- MCP-only

---

## Summary for Planning

### What the Workflow Should Know

**Codebase Analysis**: Skip — greenfield

**Patterns to Follow**:
- CLI tool pattern: simple commands, `--flag` options, clear help text
- GitHub CLI (`gh`) usage patterns for issues and projects
- Label-based organization (how GitHub uses labels for scoping)

**Patterns to Avoid**:
- Cloud dependency — stay local-first with GitHub as the only external service
- MCP-only approach — must work via shell commands for universal compatibility
- Over-engineering memory systems — keep it simple (issues + labels)

**Technical Decisions Already Made**:
- GitHub Issues + Projects as storage/UI layer
- `gh` CLI for all GitHub API operations
- Shell script or lightweight scripting language for CLI
- Labels for topic scoping and LLM attribution

**Research Already Done** (can skip):
- Competitive landscape (Supermemory, OpenMemory, Mem0)
- GitHub API capabilities for issues, labels, projects

**Research Still Needed** (zelda should do):
- GitHub Projects API specifics (creating boards, columns via `gh`)
- Best practices for label management at scale
- Shell script argument parsing patterns (getopts vs manual)
