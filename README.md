# AI Workflow Plugin

Most people use AI like a search engine: one question, one answer, start from scratch next time. This plugin builds a workflow system on top of Claude that works more like a trained employee. It remembers context across sessions, follows role-specific rules, and improves over time as you use it. Built for real professional use, it's designed to work across any knowledge-work role: research, client work, operations, or anything in between.

## Skills

### `/ai-setup` — Workspace Setup

Scaffolds a structured workspace with three modes:

- `/ai-setup cowork` — creates the root folder with `CLAUDE.md`, `MEMORY.md`, and a Resources folder
- `/ai-setup workstation` — creates a domain-specific folder inside Cowork for a particular type of work
- `/ai-setup project` — creates a project folder inside a workstation with its own context and session log

Run this once to set up your structure, then use `/onboard` at the start of every session.

### `/onboard` — Session Onboarding

Reads an existing project's workspace files and briefs the current session on what the project is, what has been done, and what comes next — so work resumes immediately without re-explaining context.

```
/onboard <project name or ID>
```

### `/review` — End-of-Session Review

Scans the current session for corrections, decisions, new facts, and patterns, then proposes targeted updates to `CLAUDE.md`, `MEMORY.md`, and `SESSION-LOG.md` at the right level — project, workstation, or root.

```
/review          # full scan
/review lite     # quick sweep when tokens are low
/review project  # explicitly targets the active project
```

### `/update-voice` — Voice Calibration

Refreshes a user's `voice-principles.md` by analysing their real sent emails and extracting genuine writing patterns.

## How it works

The plugin uses a three-level folder hierarchy:

```
Cowork/
├── CLAUDE.md          # global rules and routing map
├── MEMORY.md          # global facts and contacts
├── 00 Resources/
└── [Workstation]/
    ├── CLAUDE.md      # domain-specific rules
    ├── MEMORY.md      # domain-specific facts
    └── [Workstation] Projects/
        └── [Project]/
            ├── CLAUDE.md       # project-specific rules
            ├── MEMORY.md       # project facts and contacts
            └── SESSION-LOG.md  # running session history
```

Each level narrows scope. Project files only contain what's new for that project — everything above is inherited by context.

## Typical workflow

1. **First time:** Run `/ai-setup cowork` to scaffold your root folder, then `/ai-setup workstation` for each domain of work.
2. **New project:** Run `/ai-setup project` inside the relevant workstation.
3. **Every session:** Open the project folder, run `/onboard <project>` to brief the session.
4. **End of session:** Run `/review` to capture what was learned and log the session.

## Installation

Install via the Claude Code plugin system. Once installed, the four skills are available as slash commands in any Claude Code session opened inside your Cowork folder.
