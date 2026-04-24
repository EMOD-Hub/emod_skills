# emod_skills

A shared library of Claude Code skills for EMOD-Hub contributors. Skills teach
Claude how to perform complex, repo-specific tasks consistently across all
projects in the org — without repeating instructions in every session.

## What is a skill?

A skill is a `SKILL.md` file that Claude Code reads before starting a task. It
contains rules, examples, and checklists that tell Claude exactly how to handle
a specific category of work. Once installed, Claude picks up the skill
automatically when your request matches it, or you can invoke it directly with
a slash command.

## Available skills

| Skill | Slash command | What it does |
|---|---|---|
| [emod-hub-doc-update](skills/emod-hub-doc-update/SKILL.md) | `/emod-hub-doc-update` | Updates `.md`, `.rst`, and docstring files across EMOD-Hub repos: removes FAQ pages, replaces the IDM email with the discussion board, updates the supported OS to Ubuntu 22.04, removes the old package index URL, and migrates GitHub org links from `InstituteforDiseaseModeling` to `EMOD-Hub`. |

---

## Installation

Skills are loaded from a `.claude/skills/` directory. You can install them
**per-project** (only active in one repo) or **globally** (active in every
repo you work in).

### Global install — recommended for EMOD-Hub contributors

Installs all skills once so they are available in every repo.

**macOS / Linux (bash/zsh):**

```bash
# Clone this repo somewhere on your machine
git clone https://github.com/EMOD-Hub/emod_skills.git ~/emod_skills

# Symlink the skills folder into your global Claude config
mkdir -p ~/.claude/skills
ln -s ~/emod_skills/skills/emod-hub-doc-update ~/.claude/skills/emod-hub-doc-update
```

**Windows (PowerShell, run as Administrator or with Developer Mode on):**

```powershell
# Clone this repo somewhere on your machine
git clone https://github.com/EMOD-Hub/emod_skills.git $env:USERPROFILE\emod_skills

# Create the global skills folder if it does not exist
New-Item -ItemType Directory -Force -Path $env:USERPROFILE\.claude\skills | Out-Null

# Symlink the skill into your global Claude config
New-Item -ItemType SymbolicLink `
  -Path  "$env:USERPROFILE\.claude\skills\emod-hub-doc-update" `
  -Target "$env:USERPROFILE\emod_skills\skills\emod-hub-doc-update"
```

> Windows note: creating symlinks requires either an elevated PowerShell
> session or Developer Mode enabled (Settings → Privacy & security → For
> developers). If you cannot use symlinks, `mklink /J` (a directory junction)
> from `cmd.exe` works as a drop-in replacement.

To pick up new or updated skills in future, just `git pull` inside the cloned
repo — the symlink means Claude always reads the latest version.

### Per-project install

Run this from the root of the repo you are working on.

**macOS / Linux:**

```bash
mkdir -p .claude/skills
ln -s /path/to/emod_skills/skills/emod-hub-doc-update .claude/skills/emod-hub-doc-update
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force -Path .\.claude\skills | Out-Null
New-Item -ItemType SymbolicLink `
  -Path  ".\.claude\skills\emod-hub-doc-update" `
  -Target "C:\path\to\emod_skills\skills\emod-hub-doc-update"
```

Add `.claude/skills/` to that repo's `.gitignore` if you do not want the
symlink committed.

---

## Usage

### Option 1 — Let Claude auto-detect the skill

Just describe what you want. If your request matches the skill's description,
Claude loads it automatically:

```
Update all the docs in this repo to use the new org links and remove the FAQ.
```

### Option 2 — Invoke the skill explicitly (most reliable)

Prefix your request with the slash command:

```
/emod-hub-doc-update update all .md files under docs/
```

```
/emod-hub-doc-update apply all rules to README.md and docs/installation.rst
```

### Typical session

```bash
cd ~/projects/emod-api      # navigate to any EMOD-Hub repo
claude                       # start Claude Code

# inside the session:
/emod-hub-doc-update apply all rules to every .md and .rst file in this repo
```

Claude will:
1. Create or update `bib.md` with all extracted links
2. Apply all six rules to each file
3. Report which rules fired on each file
4. Surface any items that need your verification at the end

---

## Keeping skills up to date

Skills evolve as org conventions change. To get the latest version:

```bash
# macOS / Linux
cd ~/emod_skills
git pull
```

```powershell
# Windows (PowerShell)
cd $env:USERPROFILE\emod_skills
git pull
```

Because the global install uses symlinks, all your projects immediately use
the updated skill — no reinstall needed.

---

## Contributing a new skill

1. Create a new folder under `skills/`:
   ```
   skills/
   └── your-skill-name/
       └── SKILL.md
   ```

2. Write your `SKILL.md` using the structure below, then open a PR.

### SKILL.md structure

```markdown
---
name: your-skill-name
description: >
  One or two sentences describing WHEN Claude should use this skill.
  Be specific about triggers and file types. Include "Do NOT use for X"
  to prevent false positives.
---

# Your Skill Title

Brief summary. State the execution order if rules depend on each other.

---

## Rule N — Rule Title

**What to do**
- Bullet-point instructions Claude must follow

**Examples**
\`\`\`
# BEFORE
...
# AFTER
...
\`\`\`

---

## Verification Checklist

| Check | Expected result |
|---|---|
| ... | ... |

---

## Output Format

Instructions on how Claude should format and structure its response.
```

**Tips for writing good skills:**
- Lead with the rule that other rules depend on
- Every rule needs at least one before/after example
- Verification checklists should be grep-able (literal strings to search for)
- State exceptions explicitly — Claude will not infer them
- Keep the file under 500 lines

---

## Repository structure

```
emod_skills/
├── README.md                          # this file
└── skills/
    └── emod-hub-doc-update/
        └── SKILL.md                   # doc migration skill
```

New skills each get their own folder under `skills/` following the same pattern.
