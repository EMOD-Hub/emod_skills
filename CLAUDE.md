# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This repo contains **no executable code**. It is a shared library of Claude Code *skills* — each skill is a `SKILL.md` prompt file that teaches Claude how to perform a specific repo-wide task (e.g. doc migration) for EMOD-Hub projects. Edits here are almost always to Markdown prompt content, not to software.

The repo is designed to hold **many skills side by side**. Today it contains one (`emod-hub-doc-update`), but additional skills will be added over time — each gets its own sibling folder under `skills/`. When you add or substantially modify a skill, expect to touch both the skill folder and this file (see "Keeping CLAUDE.md in sync" below).

Consumers install skills by symlinking an individual skill folder into `~/.claude/skills/<name>` (global) or `<repo>/.claude/skills/<name>` (per-project). Because of the symlink model, any change made in this repo propagates to every installation on `git pull` — there is no packaging or release step, and backwards compatibility of skill *behavior* matters when you modify an existing `SKILL.md`.

## Repository layout

```
skills/
├── emod-hub-doc-update/
│   └── SKILL.md
└── <future-skill-name>/     # add new skills as sibling folders
    └── SKILL.md
```

Each skill lives in its own folder so it can be symlinked independently. Do not add shared helpers across skills — a skill must be self-contained in its own folder so a user symlinking just that folder gets everything it needs. If two skills end up sharing non-trivial instructions, duplicate the text rather than introducing cross-folder references.

## SKILL.md authoring rules

Every `SKILL.md` must follow the structure documented in `README.md` (under "Contributing a new skill"). The two parts that are load-bearing:

1. **YAML frontmatter `description`.** This is what Claude reads to decide whether to auto-invoke the skill for a given user request. Be specific about *when* the skill applies, include concrete trigger phrases, and add a `Do NOT use for X` clause to prevent false positives. The existing `emod-hub-doc-update/SKILL.md` is the reference example.
2. **Rule ordering.** When a skill has multiple rules, state the execution order explicitly at the top and lead with rules that others depend on. In `emod-hub-doc-update`, Rule 1 creates `bib.md` and Rules 3 and 6 write into it — reordering them breaks the skill. Preserve this kind of dependency when editing.

Other conventions from `README.md` to honor when adding or editing a skill: every rule needs a before/after example; verification checklists should use literal grep-able strings; exceptions must be stated explicitly (Claude will not infer them); keep the file under ~500 lines.

## Editing the existing `emod-hub-doc-update` skill

This skill encodes six cumulative rules for migrating docs across EMOD-Hub repos. When modifying it, be aware:

- Rule 1 (centralize links into `bib.md`) is a prerequisite for Rules 3 and 6, which add new links that must be written as reference-style links against `bib.md`. Do not change Rule 1's ordering or its output format without updating Rules 3 and 6 in lockstep.
- Rule 6 (GitHub org rewrite `InstituteforDiseaseModeling` → `EMOD-Hub`) has two explicit exceptions — `/issues/` and `/pull/` URLs are left untouched, and repos not migrated to `EMOD-Hub` (documented case: `docs-emod-scenarios`) are left untouched. If you add another non-migrated repo, list it in the same exception block so Claude does not rewrite it.
- The "Verification Checklist" and "Output Format" sections at the bottom are what consuming Claude sessions use to self-check their work. If you add a new rule, add a corresponding grep-able row to the checklist.

## Keeping CLAUDE.md in sync

This file is the entry point for every future Claude session that opens the repo, so it must stay accurate as the skill library grows. **Update `CLAUDE.md` in the same change whenever you:**

- **Add a new skill.** Add it to the "Repository layout" tree and add a short section (one or two paragraphs, mirroring the style of "Editing the existing `emod-hub-doc-update` skill") describing its purpose, any rule-ordering constraints, and known exceptions future editors must preserve.
- **Make a major update to an existing skill** — adding/removing/reordering rules, changing a rule's inputs or outputs, introducing a new dependency between rules, or adding a new documented exception. Revise the relevant per-skill section so the invariants described here match what the `SKILL.md` actually enforces.
- **Remove or rename a skill.** Delete or rename its section here and update the layout tree.

Pure wording fixes inside a rule (typos, clearer examples, tightened prose) do not require a CLAUDE.md update. The bar is "would a future Claude session be misled by the old description?" — if yes, update this file.

## Testing changes

There is no automated test suite. To validate a skill edit:

1. Symlink the edited skill into `~/.claude/skills/<name>` (see `README.md` for the exact command).
2. Run `claude` inside a target EMOD-Hub repo and invoke the skill by its slash command (e.g. `/emod-hub-doc-update`) or by a natural-language request that should trigger its `description`.
3. Confirm the skill fires, applies rules in the stated order, and produces the output format declared in its `## Output Format` section.

The symlink means no reinstall is needed between iterations — edit `SKILL.md`, re-run the skill in a fresh Claude session.
