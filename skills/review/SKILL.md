---
name: review
version: 1.0.0
author: Jivan Enriquez
description: End-of-session skill that scans the conversation for corrections, new facts, decisions, and patterns, then proposes targeted updates to CLAUDE.md, MEMORY.md, and SESSION-LOG.md at the appropriate level — project, workstation, or root. Use this skill whenever the user types /review, /review lite, "review project", "review workstation [name]", "review root", or says things like "update the project", "save what we learned", "capture decisions", "log this session", "update my memory", or "end of session update". Also trigger when the user finishes a meaningful work session and wants to preserve context before closing — even if they don't use the word review. Default to project-level if a project was active during the session. Escalate to workstation or root only if the user specifies, or if the learning is clearly domain-wide or global.
---

# Review Skill

Scans the current session, detects what was learned, corrected, or decided, and writes targeted updates to CLAUDE.md, MEMORY.md, and SESSION-LOG.md — at the right level — so the next session picks up exactly where this one left off.

---

## Invocation modes

| Command | Behaviour |
|---|---|
| `/review` | Full scan — infers the right level, produces numbered recommendations |
| `/review lite` | Quick sweep — surfaces only the most important corrections and decisions. Use when tokens are low or the session was short. |
| `/review project` | Explicitly targets the active project's files |
| `/review workstation [name]` | Explicitly targets a named workstation's files |
| `/review root` | Explicitly targets the root CLAUDE.md and MEMORY.md |

---

## Step 1 — Determine the calibration level

The level determines which files get updated. Resolve it in this order:

**If the user specified a level explicitly** (e.g. "calibrate workstation Research", "calibrate root") — use that. No need to infer.

**If the user typed `/review` or `/review lite` with no qualifier** — infer the level:
- If a named project was actively worked on during the session → **project level**
- If the session touched a workstation broadly but no specific project → **workstation level**
- If the session produced learnings that apply across all workstations (e.g. global preferences, new routing rules, new resource files) → **root level**
- If multiple levels are relevant, propose updates at each level and let the user confirm

**File targets by level:**

| Level | CLAUDE.md | MEMORY.md | SESSION-LOG.md |
|---|---|---|---|
| Project | `[Workstation]/[Workstation] Projects/[Project]/CLAUDE.md` | `…/[Project]/MEMORY.md` | `…/[Project]/SESSION-LOG.md` |
| Workstation | `[Workstation]/CLAUDE.md` | `[Workstation]/MEMORY.md` | *(no session log at workstation level)* |
| Root | `CLAUDE.md` | `MEMORY.md` | *(no session log at root level)* |

If the level is ambiguous, ask the user to confirm before scanning.

---

## Step 2 — Scan the session

Review the full conversation (or recent exchanges for lite mode) and look for:

**Corrections**
- Outputs that were wrong and had to be revised — capture the correct version and why
- Assumptions that turned out to be unsourced or inaccurate
- Misunderstandings that needed clarification

**Decisions made**
- Analytical choices and their reasoning
- Scope decisions — what's in, what's out
- Approach or methodology choices

**New facts**
- Data points, figures, and sources discovered
- New contacts, stakeholders, or organisations
- Deadlines, constraints, or milestones confirmed or updated

**Patterns and preferences expressed**
- How the user wants work structured or presented
- Tools, sources, or approaches preferred
- Things asked to stop or change

**Files created or updated**
- Documents or outputs produced — capture location and status

For `/review lite`, only surface corrections, key decisions, and file locations.

---

## Step 3 — Generate numbered recommendations

Present recommendations before writing anything. Default to exactly 5 items — no more, no fewer — unless the user explicitly requests a different number (e.g. "/calibrate 10").

```
**Review Recommendations — [Scope: Project / Workstation / Root] — [Name if applicable] — YYMMDD**

CLAUDE.md updates:
1. [CLAUDE.md — <Section>] <What to add or change>
   Why: <One sentence referencing what happened>
   Update: <Exact text>

2. [CLAUDE.md — <Section>] <What to add or change>
   Why: <One sentence>
   Update: <Exact text>

MEMORY.md updates:
3. [MEMORY — <Section>] <What to record>
   Why: <One sentence>
   Update: <Exact text to add>

4. [MEMORY — <Section>] <What to record>
   Why: <One sentence>
   Update: <Exact text to add>

SESSION LOG (project level only):
5. [SESSION-LOG] Append session entry
   Update: <Full session log entry — see format below>
```

Pick the 2 highest-value CLAUDE.md updates and 2 highest-value MEMORY.md updates. If there is nothing meaningful to write for a slot, leave it out rather than padding with a weak entry — but still cap the total at 5.

---

## Step 4 — Wait for instruction, then apply

Accepted formats:
- **"Apply all"** — apply every recommendation
- **"Accept 1, 3, 7"** — apply only those items
- **"Skip 2"** — apply all except that item

Read each target file before editing. Apply changes precisely — don't rewrite sections that aren't changing.

**MEMORY.md** — Add to the relevant section (Contacts, Key Decisions, or Context). Don't overwrite existing entries unless explicitly correcting one.

**CLAUDE.md** — Add rows to Resources, rules to Rules, or rows to the Routing Map (root only). Don't touch Identity or Workflow unless the user specifically asked for a change there.

**SESSION-LOG.md** (project level only) — Prepend a new entry at the top (most recent first):

```markdown
## YYYY-MM-DD — [One-line summary of what was done]

- [Key output, decision, or finding]
- [Key output, decision, or finding]
- [Key output, decision, or finding]

**Next steps:**
- [Specific actionable next step]
- [Specific actionable next step]
```

Keep entries to 5–8 bullets — enough to brief someone cold, not a full transcript. Next steps must be specific and actionable.

---

## Tips

**Be specific in Key Decisions.** Include the reasoning and what not to revert to. Vague entries don't help the next session.

**If a file was created, add it to Resources.** Include a clear "read when" trigger so future sessions load it at the right moment.

**Record both sides of a correction.** Capture what was wrong and what's right — so the same mistake doesn't happen next session.

**Match the level to the learning.** Project-specific facts belong in the project folder. Global preferences belong at root. Don't mix them.

**The SESSION-LOG entry is the most important output at project level.** It's what the next session reads first when onboarding. Write it as a briefing, not a changelog.
