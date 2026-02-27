# Project: LLM Hive Mind

**Source**: .planning/PRD.md
**Generated**: 2026-02-24

## Vision
A CLI tool that creates a shared, topic-scoped memory layer across different LLM CLI tools using Git + GitHub Projects — enabling any LLM to read, write, and review what other LLMs have planned or built.

## Problem
LLMs running in different terminal tabs are completely siloed. Plans, decisions, and code from one LLM are invisible to another. Users waste time copy-pasting context and lose track of which LLM contributed what.

## Core Value
Open a new terminal with any LLM, run one command, and have full context of what all other LLMs have planned and built — with clear source attribution.

## Success Metrics
- Cross-LLM context retrieval works (ask any LLM about another's work)
- Context persists across sessions (survives terminal close/reopen)
- Time to share context < 10 seconds

## Constraints
- Must work without any cloud service beyond GitHub
- Must work across all major LLM CLIs (Claude Code, Gemini CLI, Codex CLI)
- Must attribute entries to their source LLM
- Single user tool (v1)
- CLI only — no web UI

## Out of Scope
- Cloud sync / hosted backend
- Authentication / user management
- Web UI dashboard
- Real-time notifications between terminals
- MCP server mode (v2)
- Auto-detection of running LLM (v2)
