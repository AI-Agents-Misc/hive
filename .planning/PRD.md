# LLM Hive Mind — Cross-LLM Shared Memory CLI

*Generated: 2026-02-24*
*Type: Product / CLI Tool*
*Status: DRAFT*

**Complexity:** ⚫⚫⚪⚪⚪ Moderate
**Estimated Effort:** 1 week

---

## Problem Statement

Power users running multiple LLM CLI tools (Claude Code, Gemini CLI, Codex CLI) on the same project face complete context isolation. Plans, decisions, and code written by one LLM are invisible to another. Users manually copy-paste context between terminals, repeat themselves across sessions, and lose track of which LLM contributed what — wasting time and making cross-LLM collaboration impossible.

## Evidence

- Personal daily pain point — running multiple LLM CLIs simultaneously and losing context between them
- Existing tools (Supermemory, OpenMemory, Mem0) are memory stores, not collaboration buses — they lack topic scoping, source attribution, and structured cross-LLM review workflows
- The multi-LLM CLI workflow is now mainstream — Claude Code, Gemini CLI, and Codex CLI all shipped in 2025, creating a new power-user pattern with no shared coordination layer

## Proposed Solution

A CLI tool (`hive` or similar) that uses **GitHub Issues + Projects** as a shared, durable memory bus. Each "topic" maps to a **GitHub Project** (the natural scoping unit), and each entry is a GitHub Issue added to that project. Issues are labeled by source LLM (`hive:source:claude`, `hive:source:gemini`) for attribution. This is a **memory bus, not a project management tool** — it handles shared context and attribution, leaving prioritization and dependency tracking to whatever PM tool you already use. Zero new infrastructure — git is the sync, GitHub is the UI, and every LLM CLI already has `gh` access.

## Key Hypothesis

We believe a **topic-scoped, git-backed shared memory CLI** will **eliminate context silos between LLM CLI tools** for **power users running multiple LLMs simultaneously**.
We'll know we're right when **we can open a new terminal, ask any LLM about another's work, and get a coherent answer without copy-pasting**.

## What We're NOT Building

- **Project management** (dependencies, prioritization, sprint planning) — use Asana/Linear/GitHub Projects separately
- Cloud sync / hosted backend — git IS the sync layer
- Authentication / user management — single user, local tool
- Web UI dashboard — CLI only, GitHub is the UI
- Real-time notifications between terminals — polling/read is sufficient
- MCP server (v1) — deferred to v2
- Auto-detection of which LLM is running — deferred to v2

## Success Metrics

| Metric | Target | How Measured |
|--------|--------|--------------|
| Cross-LLM context retrieval | Can ask any LLM about another's contributions | Manual test: open new tab, query for plans from a different LLM |
| Session persistence | Context survives terminal close/reopen | Manual test: close terminal, reopen, verify context is accessible |
| Time to share context | < 10 seconds (one CLI command) | Timed test: `hive log` → verify in GitHub |

## Open Questions

- [ ] What should the CLI be named? (`hive`, `hivemind`, `topic`, `ctx`, `llm-sync`?)
- [ ] Should topic scoping be per-repo or global (cross-repo)?
- [ ] How to detect which LLM is running — env vars? Process name? Manual flag?

---

## Users & Context

**Primary User**
- **Who**: Developer running 2-3 LLM CLI tools simultaneously on the same project (e.g., Claude Code for architecture, Gemini for code review, Codex for implementation)
- **Current behavior**: Copy-pastes relevant output from one terminal to another, manually summarizes decisions across tools, loses attribution
- **Trigger**: Opens a new terminal tab with a different LLM and needs context from previous sessions/tools
- **Success state**: Runs one command and has full context of what all LLMs have planned/built, with clear attribution

**Job to Be Done**
When **I'm working with multiple LLM CLIs on the same project**, I want to **share plans, decisions, and code context between them with source attribution**, so I can **get cross-LLM review and maintain continuity across sessions and tools**.

**Non-Users**
- Teams collaborating on shared codebases (this is a single-user tool in v1)
- Users of chat-only LLMs (ChatGPT web, Claude.ai) — this is for CLI power users
- Users wanting a general knowledge base / second brain — this is project-scoped, not personal

---

## Solution Detail

### Core Capabilities (MoSCoW)

| Priority | Capability | Rationale |
|----------|------------|-----------|
| Must | `hive topic create <name>` — Create a GitHub Project as the topic scope | Foundation for all sharing |
| Must | `hive log <message> [--source <llm>]` — Write entry as Issue, add to Project | Core write path |
| Must | `hive read [--from <llm>]` — Read entries from Project, filter by source label | Core read path |
| Must | GitHub Issues + Projects — Issues for entries, Projects for topic scoping, Labels for attribution | Persistence + UI for free |
| Should | `hive status` — Show topic state, contributor summary | Orientation for LLMs |
| Should | Auto-label source LLM when detectable | Reduce friction |
| Could | `hive diff` — Show what changed since last read | Incremental context |
| Won't | Project management (dependencies, priorities, task ordering) | Use existing PM tools |
| Won't (v2) | MCP server mode | Native LLM integration |
| Won't (v2) | `hive review <llm>` — Structured cross-LLM review | Advanced workflow |
| Won't (v2) | Auto-detection of running LLM | Magic attribution |

### MVP Scope

The minimum to validate the hypothesis:
1. A CLI that creates GitHub Issues labeled by source LLM as a shared memory log
2. Read/write commands that any LLM can use via shell
3. Topic scoping via labels so different contexts don't collide
4. Enough structure that asking "what did Gemini plan?" returns a useful answer

**Explicitly NOT in MVP**: dependency tracking, task ordering, prioritization — those belong in your PM tool.

### User Flow (Critical Path)

```
Terminal 1 (Claude Code):
  $ hive topic create "auth-refactor"
  $ hive log "Plan: migrate from session to JWT tokens" --source claude

Terminal 2 (Gemini CLI):
  $ hive read --from claude
  → Shows Claude's plan
  $ hive log "Review: JWT is good but consider refresh token rotation" --source gemini

Terminal 3 (Codex CLI):
  $ hive status
  → Topic: auth-refactor | Entries: 2 | Contributors: claude, gemini
  $ hive read
  → Shows all entries with source attribution
  → Codex now has full context to implement
```

---

## Non-Negotiables

**Required Technologies:**
- Bash/Shell — must work in any terminal
- `gh` CLI — GitHub API access
- Git — version control and sync

**Existing Infrastructure:**
- GitHub (user's existing account and repos)

**Compliance:**
- None

**Absolute Must-Haves:**
- Must work without any cloud service beyond GitHub
- Must work across all LLM CLIs (Claude Code, Gemini CLI, Codex CLI)
- Must attribute entries to their source LLM

---

## Technical Approach

**Feasibility**: HIGH

**Architecture Notes**
- CLI tool written as a shell script (maximum portability) or lightweight Python/Node script
- Uses `gh` CLI for all GitHub API operations (issues, projects, labels)
- **Two orthogonal primitives:**
  - **Projects = Topic scoping** — `hive topic create zelda` → GitHub Project "Zelda"
  - **Labels = LLM attribution** — `hive:source:claude`, `hive:source:gemini`
- Each log entry = GitHub Issue with:
  - Title: first line of message
  - Body: full message content + metadata (timestamp, source)
  - Labels: `hive:source:<llm>`
  - Added to the active topic's GitHub Project
- Read operations = `gh project item-list` + optional label filter
- Config stored in `.hive/config.json` in repo root (tracks active topic, project numbers)
- **Why Projects for topics:** In monorepos (e.g., `claude-skills` with zelda + crucible), each sub-project IS a topic. GitHub Projects are the natural organizational unit — not labels.

**Technical Risks**

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| GitHub API rate limits (5000/hr) | Low | Batched operations, caching reads |
| `gh` CLI not installed | Medium | Check on first run, provide install instructions |
| Multiple repos for same topic | Low | v1: single repo per topic. v2: cross-repo linking |
| Issue volume gets unwieldy | Medium | Use milestones or date filtering for topic archival |

---

## Implementation Phases

| # | Phase | Description | Complexity | Status | Depends |
|---|-------|-------------|------------|--------|---------|
| 1 | Core CLI + GitHub | `topic create`, `log`, `read`, `status` — all wired to GitHub Issues + Labels via `gh` | ⚫⚫⚪ | pending | - |
| 2 | Polish & Docs | Error handling, help text, README, install script | ⚫⚪⚪ | pending | 1 |

### Verification Checkpoints

| Phase | Verification | How to Test |
|-------|--------------|-------------|
| 1 | Full user flow works end-to-end | `hive topic create test && hive log "hello" --source claude && hive read && gh issue list --label "hive:topic:test"` |
| 2 | New user can install and use in < 5 minutes | Fresh terminal, follow README, complete user flow |

---

## Research Summary

**Audience Type**: Internal

**Market Context**
- Three tools in the space (Supermemory, OpenMemory, Mem0) — all are memory stores, none solve topic-scoped collaboration with LLM attribution
- The core gap: no tool provides structured cross-LLM collaboration where you can ask one LLM about another's work with source attribution
- Multi-LLM CLI usage is a new and growing pattern (Claude Code, Gemini CLI, Codex CLI all launched 2025)

**Technical Context**
- `gh` CLI provides full GitHub API access from any terminal
- GitHub Projects API supports board views, custom fields, and automation
- GitHub Issues support labels, milestones, and rich markdown — perfect for structured entries
- Every major LLM CLI (Claude Code, Gemini CLI, Codex CLI) can execute shell commands

---

## Considered Alternatives

| Decision | Choice | Alternatives Considered | Why This Approach |
|----------|--------|------------------------|-------------------|
| Storage layer | GitHub Issues + Projects | Local files, SQLite, Redis, Cloud service | Issues for entries, Projects for topic scoping. Zero infrastructure. Built-in UI. |
| Topic scoping | GitHub Projects | Labels, directory structure, SQLite tables | Projects are the natural unit for monorepos (zelda, crucible = separate projects). Labels don't scale for topic grouping. |
| LLM attribution | Labels (`hive:source:<llm>`) | Env var detection, process inspection | Simple, reliable, no magic. Auto-detection in v2 |
| CLI language | Shell script (bash) | Python, Node.js, Go, Rust | Maximum portability, no dependencies, works in any terminal |
| Scope | Memory bus only | Memory bus + PM (deps, priorities) | PM is a separate concern. Hive handles context sharing. Use Asana/Linear for task management. |

---

## Known Issues & Troubleshooting

### Prerequisites

- [ ] GitHub account with `gh` CLI authenticated (`gh auth login`)
- [ ] Repository exists (issues will be created in current repo)
- [ ] `gh` CLI version 2.0+ installed

### Anticipated Issues

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| `gh` not authenticated | "You are not logged in" error | Run `gh auth login` |
| No repo in current directory | "not a git repository" error | Initialize repo or navigate to one |
| Rate limiting | 403 errors after many operations | Wait 1 hour or use GitHub token with higher limits |

### Risk Warnings

> **Data is public by default**: If the repo is public, all `hive` entries (plans, code decisions) will be publicly visible as GitHub Issues. Use a private repo for sensitive projects.

### Fallback Plan

If GitHub Issues approach doesn't scale or feels too heavy:
1. Fall back to structured markdown files in `.hive/` directory
2. Use git commits for sync instead of GitHub API
3. Keep the same CLI interface — only the storage backend changes
