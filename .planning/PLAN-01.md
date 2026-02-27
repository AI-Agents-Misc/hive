# Phase 1 Plan: Foundation, Topics, Log & Read

## Context

Phase 1 of the LLM Hive Mind CLI. `hive` is a bash CLI that uses a dedicated GitHub "hub" repo as a shared coordination layer across LLM CLI tools, people, and repositories. Unlike competitors (Beads = single-repo, Hivemind = cloud dependency, Agent Mail = MCP-only), hive requires only `gh` CLI — zero new infrastructure.

**Requirements covered:** REQ-001, REQ-005, REQ-011, REQ-100, REQ-101

## Architecture

### Core Concept: Hub Repo + Org-Level Project

```
AI-Agents-Misc (org)
├── Project: "Hive Mind"              <-- org-level, visual dashboard
│   ├── Table/Board/Roadmap views
│   ├── Filter: label:topic:auth-refactor
│   └── Filter: label:topic:data-cleanup
│
└── Repo: hive-hub                    <-- Issues + Labels live here
    ├── Label: topic:auth-refactor
    ├── Label: topic:data-cleanup
    ├── Issue #1: "Started JWT middleware extraction"
    └── Issue #2: "Updated login page for new auth flow"
```

### Why This Architecture

| Requirement | How GitHub satisfies it |
|-------------|----------------------|
| Multi-repo | Issues live on hub repo, accessible from any working directory |
| Multi-person | GitHub auth handles identity; issue author tracks who logged |
| Multi-LLM | Any tool with shell access can run `gh` — no MCP/daemon needed |
| Zero new infra | GitHub is already in every team's stack |
| Cross-repo search | `gh search issues --repo org/hive-hub --label topic:X` |
| Conversation threading | Issue comments = agents can reply to each other's logs |

### Config Location

User-level config at `~/.config/hive/config.json` (NOT repo-level). This means `hive` works identically from any directory on the machine.

```json
{
  "hub": "AI-Agents-Misc/hive-hub",
  "active_topic": "auth-refactor"
}
```

### GitHub Primitives Used

| Primitive | hive concept | How |
|-----------|-------------|-----|
| Org Project v2 | Visual dashboard | Created during `hive init`, single board for all topics |
| Label | Topic | `gh label create "topic:name" -R hub --force` |
| Issue | Log entry | `gh issue create -R hub --label "topic:name"` |
| Issue list | Read | `gh issue list -R hub --label "topic:name"` |
| Issue comment | Reply/thread | `gh issue comment N -R hub` (future) |
| Project item | Dashboard entry | `gh project item-add` links Issues to the Project |

---

## gh CLI Research (Discovery)

| Operation | Command | Notes |
|-----------|---------|-------|
| Create label | `gh label create "topic:name" -R hub --force` | Idempotent with `--force` |
| List labels | `gh label list -R hub --json name -q '.[] \| select(.name \| startswith("topic:"))'` | Filter by prefix |
| Create issue | `gh issue create -R hub --title "msg" --label "topic:X" --body "body"` | Returns issue URL |
| List issues | `gh issue list -R hub --label "topic:X" --json number,title,author,createdAt --limit 20` | Structured output |
| Read issue | `gh issue view N -R hub` | Full issue with comments |
| Check auth | `gh auth status` | Verify logged in |
| Check repo | `gh repo view org/hub --json name` | Verify hub exists |

---

## Plan 01-01: CLI Scaffold + Config System

**Wave 1** | `depends_on: []` | `autonomous: true`

Rewrites the `hive` bash script with the new hub-repo architecture. Replaces the local `.hive/config.json` with user-level `~/.config/hive/config.json`.

**Files:**
- `hive` — Main CLI entrypoint (rewrite)

**Tasks:**

1. **Rewrite `hive` bash script** with command routing (`init`, `topic`, `log`, `read`, `status`, `--help`). Use case-based argument parsing. Include `check_gh()` that validates: (a) `gh` is installed, (b) `gh auth status` succeeds. Remove the `project` scope check (no longer needed — Issues don't require it).

2. **Rewrite config system** — Config lives at `~/.config/hive/config.json`. `init_config()` creates the directory and file. `load_config()` reads it. `save_config()` writes it. Schema:
   ```json
   {
     "hub": null,
     "active_topic": null,
     "project_number": null
   }
   ```

3. **Implement `hive init <owner/repo>`** — Sets the hub repo. Validates the repo exists via `gh repo view`. If repo doesn't exist, offer to create it: `gh repo create <owner/repo> --public --description "Hive Mind coordination hub"`. Creates an org-level Project named "Hive Mind" via GraphQL API (`gh project create` is broken in gh v2.87+). Stores hub and project number in config. This is the only setup step — everything else derives from the hub.

4. **Implement `require_hub()`** — Helper that checks if hub is configured. Called by all commands except `init` and `--help`. If not set, prints: "Run: hive init <owner/repo>".

**Verify:** `./hive --help` shows usage. `./hive init AI-Agents-Misc/hive-hub` sets hub. `./hive topic create test` routes correctly.

---

## Plan 01-02: Topic Create & List

**Wave 2** | `depends_on: ["01-01"]` | `autonomous: true`

Implements `hive topic create <name>` and `hive topic list`. Topics are GitHub labels on the hub repo with a `topic:` prefix.

**Files:**
- `hive` — Add `cmd_topic_create()` and `cmd_topic_list()` functions

**Tasks:**

1. **Implement `cmd_topic_create()`** — Validates topic name (kebab-case, regex `^[a-z0-9][a-z0-9-]*$`). Creates label on hub repo: `gh label create "topic:$name" --description "Hive topic" --color 0E8A16 -R $hub --force`. Sets as active topic in config. Outputs confirmation.

2. **Implement `cmd_topic_list()`** — Lists all `topic:*` labels from hub repo: `gh label list -R $hub --json name,description --jq '.[] | select(.name | startswith("topic:"))'`. Marks the active topic with `*`. If no topics, shows "No topics yet — run: hive topic create <name>".

3. **Implement `hive topic set <name>`** — Switches the active topic without creating a new one. Validates the topic exists as a label on the hub.

**Verify:** `./hive topic create auth-refactor` creates a label. `./hive topic list` shows it. `./hive topic create data-cleanup` creates second topic and switches active. `./hive topic set auth-refactor` switches back.

---

## Plan 01-03: Log Command

**Wave 3** | `depends_on: ["01-02"]` | `autonomous: true`

Implements `hive log <message>` — creates a GitHub Issue on the hub repo, labeled with the active topic.

**Files:**
- `hive` — Add `cmd_log()` function

**Tasks:**

1. **Implement `cmd_log()`** — Requires active topic. Creates an Issue on the hub repo:
   ```bash
   gh issue create -R $hub \
     --title "$message" \
     --label "topic:$active_topic" \
     --body "$(build_log_body)"
   ```
   The body includes metadata:
   ```markdown
   **Agent:** $(whoami)@$(hostname)
   **Repo:** $(gh repo view --json nameWithOwner -q .nameWithOwner 2>/dev/null || echo "none")
   **Branch:** $(git branch --show-current 2>/dev/null || echo "none")
   **Timestamp:** $(date -u +"%Y-%m-%dT%H:%M:%SZ")
   ```
   This metadata lets agents know WHO logged, FROM WHERE, and WHEN — critical for multi-repo, multi-person coordination.

2. **Add Issue to Project** — After creating the Issue, add it to the org-level Project so it appears on the visual dashboard: `gh project item-add $project_number --owner $org --url $issue_url`. If this fails (e.g., project scope missing), log a warning but don't block — the Issue is still created and queryable.

3. **Handle edge cases** — No active topic: error with hint. Empty message: error. Hub not configured: error via `require_hub()`. Not in a git repo: still works (repo/branch fields show "none").

**Verify:** `./hive log "Completed auth refactor, moved JWT validation to middleware"` creates an Issue on the hub repo with the right label and metadata, and the Issue appears on the Project board.

---

## Plan 01-04: Read Command

**Wave 3** | `depends_on: ["01-02"]` | `autonomous: true`

Implements `hive read` — shows recent log entries for the active topic.

**Files:**
- `hive` — Add `cmd_read()` function

**Tasks:**

1. **Implement `cmd_read()`** — Requires active topic. Lists recent Issues:
   ```bash
   gh issue list -R $hub \
     --label "topic:$active_topic" \
     --json number,title,author,createdAt,body \
     --limit 20 \
     --state all
   ```
   Format output as a clean, readable log:
   ```
   Topic: auth-refactor (20 entries)

   #12  2h ago  @nicksonnenberg
        Completed auth refactor, moved JWT validation to middleware
        repo: Alpha-Bet-New/ws-backend  branch: feat/auth

   #11  5h ago  @claude-agent
        Updated JWT expiry from 1h to 15min, added refresh token flow
        repo: Alpha-Bet-New/ws-backend  branch: feat/auth

   #10  1d ago  @cursor-agent
        Researched OAuth2 PKCE flow, documented in .planning/RESEARCH-auth.md
        repo: Alpha-Bet-New/webfrontend  branch: main
   ```

2. **Support `hive read --all`** — Shows entries across ALL topics (omit `--label` filter).

3. **Support `hive read --limit N`** — Override default 20-entry limit.

**Verify:** After logging entries, `./hive read` shows them in reverse chronological order with agent/repo/branch metadata.

---

## Execution Order

```
Wave 1: 01-01 (scaffold + config + init)
Wave 2: 01-02 (topic create/list/set)
Wave 3: 01-03 (log) + 01-04 (read) — can be parallel since they're independent functions
```

Effective execution: 01-01 → 01-02 → 01-03 → 01-04

---

## Must-Haves (Goal-Backward Verification)

**Truths:**
- `hive init AI-Agents-Misc/hive-hub` connects hive to a GitHub repo
- `hive topic create auth-refactor` creates a label `topic:auth-refactor` on the hub
- `hive topic list` shows all topics with active marker
- `hive log "message"` creates an Issue with topic label, agent identity, and source repo metadata
- `hive read` shows recent log entries for the active topic in a clean format
- Works from ANY directory on the machine (config is user-level)
- Works for ANY LLM CLI tool that has shell access (no MCP required)
- Works across repos (all state lives on the hub repo)

**Artifacts:**
- `hive` — Bash script, executable, ~300-400 lines
- `~/.config/hive/config.json` — User-level config (hub + active topic)
- `topic:*` labels on hub repo — Topic registry
- Issues on hub repo — Log entries

**Key links:**
- `hive` → `gh issue create/list` (via shell commands)
- `hive` → `~/.config/hive/config.json` (read/write via jq)
- `hive` → hub repo labels (topic registry)

---

## End-to-End Verification

```bash
# Setup
chmod +x hive
./hive init AI-Agents-Misc/hive-hub

# Create topics
./hive topic create auth-refactor
./hive topic create data-cleanup
./hive topic list                        # Shows both, data-cleanup is active

# Switch topic
./hive topic set auth-refactor

# Log from different directories
cd ~/projects/ws-backend
hive log "Started JWT middleware extraction"

cd ~/projects/webfrontend
hive log "Updating login page to use new auth flow"

# Read the conversation
hive read                                # Shows both entries with repo context

# Read all topics
hive read --all
```

---

## Competitive Differentiators

| vs Beads | hive is cross-repo; Beads is single-repo |
| vs Hivemind | hive needs no cloud account; Hivemind is a hosted service |
| vs Agent Mail | hive needs no MCP server; Agent Mail requires MCP |
| vs claude-flow | hive is LLM-agnostic; claude-flow is Claude-only |
| vs all | hive requires only `gh` CLI — literally zero new infrastructure |
