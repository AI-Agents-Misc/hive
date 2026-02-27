# Roadmap: LLM Hive Mind

## Overview

Build a CLI tool (`hive`) that uses GitHub Issues + Projects as a shared memory bus across LLM CLI tools. Phase 1 establishes the CLI scaffold and topic management (GitHub Projects). Phase 2 wires the write path (log entries as Issues with source labels). Phase 3 completes the read path (read + status commands). Phase 4 polishes help text and error messages.

## Phases

- [ ] **Phase 1: Foundation & Topic Management** ⚫⚫⚪ - CLI scaffold, config, `gh` validation, `topic create/list`
- [ ] **Phase 2: Write Path** ⚫⚫⚪ - `log` command creating labeled Issues added to Projects
- [ ] **Phase 3: Read Path** ⚫⚪⚪ - `read` and `status` commands for retrieving shared context
- [ ] **Phase 4: Polish & Docs** ⚫⚪⚪ - Help text, error messages, README

**Complexity Key:** ⚫⚪⚪ Simple | ⚫⚫⚪ Moderate | ⚫⚫⚫ Complex

## Phase Details

### Phase 1: Foundation & Topic Management ⚫⚫⚪
**Goal**: CLI scaffold with config system, `gh` CLI validation, and `topic create/list` commands wired to GitHub Projects
**Depends on**: Nothing (first phase)
**Requirements**: REQ-001, REQ-005, REQ-011, REQ-100, REQ-101
**Success Criteria** (what must be TRUE):
  1. `hive topic create auth-refactor` creates a GitHub Project named "hive: auth-refactor"
  2. `hive topic list` shows all hive topics in the current repo
  3. First run validates `gh` is installed and authenticated with `project` scope
  4. `.hive/config.json` stores active topic name and project number
**Verification**: `hive topic create test-topic && hive topic list | grep test-topic`
**Research**: Likely (gh project API specifics, bash argument parsing)
**Research topics**: `gh project create` flags, `gh project list` filtering, getopts vs manual arg parsing
**Plans**: 3 plans

Plans:
- [ ] 01-01: CLI scaffold + config system (`hive` script, `.hive/config.json`, `gh` validation)
- [ ] 01-02: Topic create command (wired to `gh project create`, stores project number)
- [ ] 01-03: Topic list command (reads hive projects from GitHub)

**Risks**: `gh auth` may not have `project` scope — need to detect and prompt `gh auth refresh -s project`

### Phase 2: Write Path ⚫⚫⚪
**Goal**: `hive log` command that creates GitHub Issues with `hive:source:<llm>` labels and adds them to the active topic's Project
**Depends on**: Phase 1
**Requirements**: REQ-002, REQ-010, REQ-012, REQ-013, REQ-102
**Success Criteria** (what must be TRUE):
  1. `hive log "Plan: use JWT" --source claude` creates a GitHub Issue with label `hive:source:claude`
  2. Issue is automatically added to the active topic's GitHub Project
  3. Issue title = first line of message, body = full message + timestamp + source metadata
  4. Omitting `--source` defaults to "unknown"
**Verification**: `hive log "test entry" --source claude && gh issue list --label "hive:source:claude" | head -1`
**Research**: Unlikely (gh issue create and gh project item-add are well-documented)
**Plans**: 2 plans

Plans:
- [ ] 02-01: Log command + Issue creation (Issue with title/body/label, auto-create labels)
- [ ] 02-02: Issue-to-Project assignment (add Issue to active topic's Project)

**Risks**: Label auto-creation requires `gh label create` — may fail silently if label exists

### Phase 3: Read Path ⚫⚪⚪
**Goal**: `hive read` and `hive status` commands for retrieving and summarizing shared context
**Depends on**: Phase 2
**Requirements**: REQ-003, REQ-004
**Success Criteria** (what must be TRUE):
  1. `hive read` shows all entries for active topic in chronological order
  2. `hive read --from gemini` filters to only Gemini's entries (by label)
  3. `hive status` shows topic name, entry count, list of contributing LLMs
  4. Empty topic returns "No entries yet" message
**Verification**: `hive read && hive read --from claude && hive status`
**Research**: Unlikely (gh project item-list with label filtering)
**Plans**: 2 plans

Plans:
- [ ] 03-01: Read command (retrieve Project items, optional `--from` label filter)
- [ ] 03-02: Status command (topic state summary: name, count, contributors)

**Risks**: None identified

### Phase 4: Polish & Docs ⚫⚪⚪
**Goal**: Help text for all commands, clear error messages with fix suggestions, README for installation
**Depends on**: Phase 3
**Requirements**: REQ-200, REQ-201
**Success Criteria** (what must be TRUE):
  1. `hive --help` and `hive <command> --help` display useful help text
  2. Error messages suggest fixes (e.g., "gh not found — install with: brew install gh")
  3. New user can install and use hive in < 5 minutes following README
**Verification**: `hive --help && hive topic --help && hive log --help`
**Research**: Unlikely (standard CLI patterns)
**Plans**: 1 plan

Plans:
- [ ] 04-01: Help text, error messages, and README

**Risks**: None identified

## Progress

**Execution Order:**
Phase 1 → Phase 2 → Phase 3 → Phase 4

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & Topic Management | 0/3 | Not started | - |
| 2. Write Path | 0/2 | Not started | - |
| 3. Read Path | 0/2 | Not started | - |
| 4. Polish & Docs | 0/1 | Not started | - |
