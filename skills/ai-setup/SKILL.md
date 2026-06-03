---
name: ai-setup
version: 1.1.0
author: Jivan Enriquez
description: Sets up a Cowork workspace, workstation, or project depending on the argument provided. Use this skill whenever the user types /ai-setup, /ai-setup cowork, /ai-setup workstation, or /ai-setup project — or says things like "set up my cowork folder", "create a new workstation", "set up a project", "start a new project workspace", "I need a workstation for X", or "create a project under [workstation]". If no argument is given, ask the user what they want to set up before proceeding. Do not trigger for onboarding an existing project — that belongs to /onboard.
---

# Setup Skill

One skill, three modes. Routes to the right setup flow based on the argument provided.

---

## Invocation

| Command | What it does |
|---|---|
| `/ai-setup cowork` | Scaffolds the root Cowork folder structure — CLAUDE.md, MEMORY.md, and a Resources folder |
| `/ai-setup workstation` | Creates a new workstation inside the Cowork folder |
| `/ai-setup project` | Creates a new project inside an existing workstation |
| `/ai-setup` (no argument) | Asks the user what they want to set up, then routes accordingly |

---

## Routing logic

When the skill fires, check the argument:

- Contains "cowork" → run **Cowork Setup**
- Contains "workstation" → run **Workstation Setup**
- Contains "project" → run **Project Setup**
- No argument → ask: *"What would you like to set up? Choose from: Cowork (your root folder), Workstation (a domain of work), or Project (a specific piece of work inside a workstation)."* Then route based on the answer.

---

## Mode A: Cowork Setup (`/setup cowork`)

Sets up the root Cowork folder — the foundation everything else sits inside.

### Step 1: Confirm the folder location

Ask the user to confirm the folder they've selected in Cowork (or have them select one if not already done). This is where all files will be created.

### Step 2: Ask clarifying questions

1. **What's your name and role?** Used to personalise the root CLAUDE.md.
2. **How do you prefer to work?** Any communication defaults, formatting preferences, or things Claude should always or never do?
3. **What types of work will you route through Cowork?** This shapes the initial Routing Map.

Wait for answers before proceeding.

### Step 3: Create the folder structure

```
[Cowork Folder]/
├── CLAUDE.md
├── MEMORY.md
└── 00 Resources/
```

### Step 4: Write root CLAUDE.md

Write a root CLAUDE.md using the user's answers. Follow this structure exactly:

```markdown
# CLAUDE.md

## Memory System

At the start of every session, read MEMORY.md before responding — this is mandatory and must happen before any other action, including answering the first message. Use what you find to inform your work. Don't announce what you found, just be informed by it.

When I say "remember this," write the information to MEMORY.md immediately and confirm you've done it.

**Where things go:** Apply two tests when deciding where to save something. Test 1: Does it prescribe behavior? Look for words like "always," "never," "before doing X, do Y." If yes, add it to CLAUDE.md. Test 2: Does it describe a fact about the world that could change? Contact details, project status, decisions. If yes, add it to MEMORY.md. When unsure, suggest which file it belongs in and ask to confirm.

## Preferences

[Populated from user's answers — tone, formatting, communication defaults]

## Rules

- Ask clarifying questions before starting any task where more context would improve the output.
- If you're not sure about something, say so. Don't guess.
- Before producing any written content, read voice-principles.md in the 00 Resources folder.

[Add any additional rules from user's answers]

## Routing Map

*Add rows as you create new workstations.*

| Workstation | Route here when I... |
| :---- | :---- |

## References

| Resource | Read when... |
| :---- | :---- |
| voice-principles.md | Writing any content on my behalf |
```

### Step 5: Write root MEMORY.md

```markdown
# Memory

## Contacts

*People and organisations I work with. Populated over time.*

| Name | Role | Organisation | Notes |
| :---- | :---- | :---- | :---- |

## Key Decisions

*Reasoning behind important choices. Populated over time.*

## Context

*Standing facts — preferences, constraints, recurring patterns.*
```

### Step 6: Confirm

Tell the user what was created and suggest the next step: *"Run /ai-setup workstation to create your first domain of work."*

---

## Mode B: Workstation Setup (`/setup workstation`)

Creates a new workstation inside the Cowork folder.

### Step 1: Ask clarifying questions

1. **What is this workstation for?** What kind of tasks or topics will route here?
2. **What should *not* route here?** Adjacent topics that belong elsewhere.
3. **What does a typical task look like?** A quick example helps shape the Workflow.
4. **Is there any standing context Claude should always have in mind here?** Recurring constraints, tools, methodologies.
5. **Are there reference files Claude should consult in this workstation?**

Wait for answers before proceeding.

### Step 2: Read the root CLAUDE.md

Check what's already defined globally — preferences, rules, memory instructions. The workstation CLAUDE.md should only add what's new for this domain. Also check the Routing Map for any existing placeholder rows.

### Step 3: Create the folder structure

```
[Workstation Name]/
├── CLAUDE.md
├── MEMORY.md
├── [Workstation Name] Resources/
└── [Workstation Name] Projects/
```

### Step 4: Write workstation CLAUDE.md

Write a CLAUDE.md specific to this domain. Don't repeat anything already in root CLAUDE.md.

```markdown
# [Workstation Name]

## Identity

[One paragraph. Who is Claude in this workstation? What routes here? What explicitly does not?]

## Resources

| Resource | Read when... |
| :---- | :---- |

## Workflow

1. [First step — based on user's example]
2. [Continue as needed]
3. Flag anything uncertain before finalising.

## Rules

[Domain-specific rules only. If none yet: "No domain-specific rules yet — add these as patterns emerge."]
```

### Step 5: Write workstation MEMORY.md

```markdown
# [Workstation Name] Memory

## Contacts

*People relevant to this domain. Populated over time.*

| Name | Role | Relationship | Notes |
| :---- | :---- | :---- | :---- |

## Key Decisions

*Reasoning behind choices made in this workstation. Populated over time.*

## Context

*Standing facts — client preferences, constraints, recurring patterns.*
```

### Step 6: Update the root Routing Map

Add a row to the Routing Map in root CLAUDE.md:

```
| [Workstation Name] | ...need to [domain-appropriate trigger phrase] |
```

Replace any existing placeholder row for this workstation rather than adding a duplicate.

### Step 7: Confirm

Short plain-prose summary of what was created. Call out any assumptions made. Suggest next step: *"Run /ai-setup project to start your first project inside this workstation."*

---

## Mode C: Project Setup (`/setup project`)

Creates a new project folder inside an existing workstation.

### Step 1: Ask clarifying questions

1. **Which workstation does this project live in?** List from root Routing Map if helpful.
2. **What is this project about?** One sentence — what is the outcome or deliverable?
3. **Who else is involved?** Names, roles, or organisations.
4. **Any reference files, source documents, or constraints to know about?**
5. **Is there a deadline or key milestone?**

Wait for answers before proceeding.

### Step 2: Read the workstation CLAUDE.md

Check what's already defined so the project CLAUDE.md only adds what's new. Confirm the workstation path so files are created in the right location.

If the named workstation doesn't exist, let the user know and offer to run `/setup workstation` first.

### Step 3: Create the folder structure

```
[Workstation Name]/
└── [Workstation Name] Projects/
    └── [Project Name]/
        ├── CLAUDE.md
        ├── MEMORY.md
        ├── SESSION-LOG.md
        └── [Project Name] Resources/
```

### Step 4: Write project CLAUDE.md

```markdown
# [Project Name]

## Identity

[One paragraph. What is this project? Expected outcome? Key stakeholders? What does and doesn't belong here?]

## Resources

| Resource | Read when... |
| :---- | :---- |

## Workflow

1. [First step]
2. [Continue as needed]
3. Flag anything uncertain before finalising.

## Rules

[Project-specific rules only. If none yet: "No project-specific rules yet — add these as patterns emerge."]
```

Keep the project CLAUDE.md to around 300 words. It should be a focused briefing, not a manual.

### Step 5: Write project MEMORY.md

```markdown
# [Project Name] Memory

## Contacts

*People and organisations relevant to this project. Populated over time.*

| Name | Role | Organisation | Notes |
| :---- | :---- | :---- | :---- |

## Key Decisions

*Reasoning behind choices made in this project. Populated over time.*

## Context

*Standing facts — deadlines, constraints, recurring patterns, client preferences.*
```

If the user provided contacts or a deadline in their answers, pre-populate those fields now.

### Step 6: Write SESSION-LOG.md

```markdown
# [Project Name] — Session Log

*A running record of what was done in each session. Most recent entry at the top.*

---
```

At the end of any working session on this project, append an entry:

```markdown
## [Date] — [One-line summary]

- [Key output or decision]
- [Key output or decision]

**Next steps:**
- [Specific actionable next step]
```

### Step 7: Confirm

Short plain-prose summary. Call out any pre-populated fields or assumptions. Remind the user: *"Start any new chat on this project with /onboard [project name]. Run /ai-setup project again for any additional projects."*

---

## General tips and edge cases

- If a folder with the same name already exists, ask before overwriting. Never silently replace an existing file.
- If the user gives minimal answers, make reasonable inferences from the name and context — then flag what was assumed so they can correct it.
- For workstation setup: if the user wants multiple workstations at once, handle them one at a time.
- For project setup: if the user doesn't specify a workstation, check the root Routing Map and ask them to confirm — don't guess.
