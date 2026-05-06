---
name: recover-rst-content
description: Use this skill when recovering documentation content that was dropped during the rst → MkDocs Markdown conversion in this repo. Triggers on phrases like "missing content", "dropped during conversion", "restore from rst", "compare rst to md", or any request to reconcile current `.md` files in `./docs/` against the original `.rst` files in `../<repo>-pre-conversion/docs/`. Use whenever the user references the pre-conversion worktree.
---

# Recovering content dropped during rst → MkDocs conversion

## Context

Documentation in this repo was converted from reStructuredText to MkDocs Markdown in a single commit. The conversion **dropped substantial content from many files** — most often `.. csv-table::` blocks documenting parameters and data channels. Subsequent commits made further edits to the `.md` files that **must be preserved**.

Two trees are available side by side:

- `../<repo>-pre-conversion/docs/` — original `.rst` files (read-only reference, do not modify)
- `./docs/` — current `.md` files (edit target)

The exact directory names depend on how the user set up their git worktree. Confirm the paths in the first turn if not obvious.

## Procedure

For each file or batch the user specifies:

### 0. Build the deleted-pages list

Before triaging anything, list every `.rst` file in the pre-conversion tree that has **no** `.md` counterpart. These are pages the maintainers deliberately removed. Use this list throughout triage: any rst content that describes, links to, or supports a workflow centered on these pages is a strong candidate for "intentionally removed," not "dropped."

Confirm with the user once at the start of the session whether files with no md counterpart should be skipped entirely (typical) or restored as new pages (rare).

### 1. Locate the pair

Map each `.md` file to its `.rst` source. The conversion preserved relative paths and renamed `foo.rst` → `foo.md`. If a mapping isn't obvious (file renamed, split, or merged), list it and ask before proceeding.

### 2. Read both fully

Read the entire `.rst` and the entire current `.md` before comparing. Don't skim. Missing content is often a whole table dropped silently between two paragraphs that are present in both.

### 3. Identify what's missing

Categorize every chunk of `.rst` content as one of:

- **Dropped** — content present in the `.rst` but absent from the `.md`. This is what to restore. Common cases observed in this repo:
  - `.. csv-table::` parameter description tables
  - `.. csv-table::` output data channel tables
  - Notes, warnings, examples between sections
  - Cross-references that should have become MkDocs links
- **Transformed** — content that's present but reworded, restructured, or syntactically converted. **Do NOT restore.** Examples:
  - `:ref:\`label\`` already replaced with a Markdown link → already done
  - `.. code-block:: json` already converted to a fenced code block → already done
  - `.. csv-table:: :file: foo.csv` already replaced with `{{ read_csv('foo.csv') }}` → already done
- **Intentionally removed** — looks deliberately cut. Strong signals:
  - The chunk's purpose was to describe a feature documented on a *deleted reference page* (any `.rst` in the pre-conversion tree with no `.md` counterpart). When the reference page was deleted, the prose pointing users to it was usually deleted too — even if the prose itself is technically still accurate.
  - The chunk contains a `:doc:` cross-reference to a deleted page; the surrounding paragraph or section likely went with it.
  - Deprecated sections, outdated TODOs.

  Default to **flag for the user** when in doubt. Restoring the prose with the dead link silently rewritten or dropped is rarely correct — the prose existed to support the link.

### 4. Plan before editing

For each `.md` file, output:

- The path
- A list of missing items, each with: a one-line description, the rst line range, and where it belongs in the `.md` (which section, between which existing paragraphs)
- Anything ambiguous flagged for user input

Then **stop and wait for confirmation** unless the user has said "proceed without asking per-file." If batching, plan all files in the batch first, then wait for one confirmation covering the batch.

### 5. Apply edits — preserve existing `.md` content

When restoring:

- **Never revert post-conversion edits.** If the `.md` reworded a paragraph, keep the reworded version; only add what's actually absent.
- Convert `.rst` syntax to MkDocs Markdown using the conversion table below.
- Match the surrounding `.md` file's style (heading levels, code-fence language tags, link format).
- Place restored content in the same structural position it occupied in the `.rst` — same section, same relative order to neighboring content.

### 6. Leave changes unstaged

Edit files in place. **Do not run `git add`, `git commit`, or any staging commands.** The user reviews everything via `git diff` before staging.

### 7. Report

After each file (or batch), summarize in 1–3 lines what was added per file. No prose victory laps.

## rst → MkDocs conversion table

Tuned to conventions observed in this repo. When you encounter a construct not listed, **check how a similar construct was handled in already-converted `.md` files in this repo before inventing a translation** — repo consistency wins.

| rst construct | MkDocs Markdown equivalent |
|---|---|
| `===` / `---` / `~~~` underlines | `#` / `##` / `###` (use the level that matches surrounding `.md` headings) |
| `**bold**` | `**bold**` (unchanged) |
| `*italic*` | `*italic*` (unchanged) |
| `` ``code`` `` | `` `code` `` |
| `.. code-block:: <lang>` | ` ```<lang> ` fenced block |
| `.. note::` / `.. warning::` / `.. tip::` | `!!! note` / `!!! warning` / `!!! tip` (verify the repo uses Admonition extension; check existing `.md` files first) |
| `:doc:\`page-name\`` | `[page-name](page-name.md)` |
| `:ref:\`label\`` | `[link text](page.md#anchor)` — must look up where the label is defined; never guess the target |
| `` `Section Name`_ `` (internal link) | `[Section Name](#section-name)` (lowercased, hyphens for spaces) |
| `.. image:: path` with `:alt:` | `![alt](path)` |
| `.. csv-table::` with inline rows | Markdown pipe table: `\| col \| col \|` with `\|---\|---\|` separator. Preserve all rows. Strip the rst-specific `:widths:`, `:header:` directives. |
| `.. csv-table:: :file: foo.csv` | `{{ read_csv('foo.csv') }}` (uses mkdocs-table-reader-plugin) |
| Definition lists | Check existing `.md` files for the repo's convention before translating |

### csv-table → Markdown table specifics

This is the most common dropped construct in this repo. When converting:

1. The `:header:` directive line becomes the Markdown header row.
2. Each data row in the rst becomes one Markdown row. Cells in rst are comma-separated and may be quoted with `"..."`. **Preserve quoted commas inside cells** — these are easy to miss and corrupt the table.
3. Within cells, rst inline markup (`**bold**`, `:doc:\`...\``, etc.) must be converted using the rules above.
4. Drop the `:widths:` directive — Markdown tables don't take widths.
5. Pipe characters (`\|`) inside cell text must be escaped as `\\|`.

## Anti-patterns

- **Don't restore reworded content.** If the `.md` says the same thing in different words, that's not missing.
- **Don't "improve" prose while you're in there.** Restore what was dropped; leave everything else alone.
- **Don't translate rst syntax mechanically.** Always check how the rest of the repo handles the same construct first.
- **Don't process more than ~5 files without checkpointing** with the user, unless they've explicitly said to run a larger batch.
- **Don't stage or commit.** All edits stay unstaged for user review.
- **Don't guess at `:ref:` targets.** If you can't find where a label is defined, flag it and ask.
- **Don't paper over a dead link.** When rst content references a `:doc:` target that has no `.md` counterpart, the surrounding prose almost always existed *because of* that link. Restoring it with the link silently dropped or rewritten is not a fix — it's restoring content that was deliberately cut. Flag and ask.
