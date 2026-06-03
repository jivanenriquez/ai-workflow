---
name: onboard
version: 1.0.0
author: Jivan Enriquez
description: Brings a Cowork session up to speed on an existing project by reading its context and session log from the current working directory. Use this skill whenever the user types /onboard, says "pick up X", "continue X", "resume X", "get me up to speed on X", or opens a new session referencing a project previously created with /ai-setup.
---

# Onboard Skill

Reads an existing project's workspace files and briefs the current session on what the project is, what has been done, and what comes next — so work resumes immediately without re-explaining context.

---

## Invocation

```
/onboard <project name or ID>
```

Examples:
- `/onboard q4-reporting`
- `/onboard 260514-01`
- `/onboard 260514-01-q4-reporting`

If no name or ID is provided, list all available projects and ask which one to load.

---

## Workflow

### Step 1 — Locate the project folder

Search the current working directory (where the session was opened) for a folder matching the name or ID provided.

**Matching rules (in priority order):**
1. Exact folder name match
2. ID prefix match (e.g. `260514-01` matches `260514-01-q4-reporting`)
3. Partial name match (e.g. `reporting` matches `260514-01-q4-reporting`)

**If no match:** List all folders in the current working directory and ask which one to load.

**If no project folders exist:** Tell the user no projects exist yet and suggest running `/ai-setup` to create one.

---

### Step 2 — Read files in hierarchical order

Read in this sequence. Each layer narrows scope from global to project-specific.

| Order | File | Required | Purpose |
|---|---|---|---|
| 1 | Root `CLAUDE.md` | Yes | Global context and preferences for all sessions |
| 2 | Root `MEMORY.md` | Yes | Known facts and decisions to carry into this session |
| 3 | Workstation `CLAUDE.md` | If workstation applies | Context and preferences scoped to this workstation |
| 4 | Workstation `MEMORY.md` | If workstation applies | Known facts and decisions scoped to this workstation |
| 5 | Project `CLAUDE.md` | No | Context and preferences scoped to this project |
| 6 | Project `MEMORY.md` | No | Known facts and decisions about this project |
| 7 | Project `session_log.md` | No | History of sessions and decisions |

**Workstation:** determine which workstation the project belongs to using the Routing Map in the root `CLAUDE.md`. If no workstation applies, skip steps 3 and 4.

---

### Step 3 — Brief the session

Deliver a concise onboarding summary:

```
**Project:** <Project Name> (<folder ID>)

**What this is:** <1–2 sentence overview>

**Last session:** <1–2 sentence summary of the most recent log entry, or "No session log found">

**Next steps:** <Next steps from last log entry, or "Not recorded">

<Only include if applicable:>
**Heads up:** <Deadline, blocker, or other time-sensitive detail worth flagging>

---
Ready to continue — what would you like to work on?
```

Keep it tight — fast orientation, not a full read-back.

---

### Step 4 — Stay in project context

After onboarding, treat this session as operating within the project folder. Use the following folder structure for all work done in this session:

| Folder | Purpose |
|---|---|
| `resources/` | All files and materials uploaded or provided by the user |
| `outputs/` | All files and content produced by Claude during the session |

Create either folder if it doesn't already exist. Do not save files outside the project folder unless the user specifies otherwise.

---

## End of session

When the session is wrapping up, remind the user to run `/review` to log the session.

---

## Notes

- For partial name matches, prefer the most recently created folder (highest date prefix)
- Projects created with `/ai-setup lite` may have no session log — that's expected
