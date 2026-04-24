---
name: emod-hub-doc-update
description: >
  Use this skill when updating documentation, docstrings, or Markdown files
  across EMOD-Hub repositories. Triggers include: any request to update, audit,
  or migrate docs for EMOD-Hub / IDM repos, fix outdated references, remove FAQ
  pages, update OS support language, fix package index URLs, or update GitHub
  org links. Covers .md, .rst, .txt, .py (docstrings), and similar text-based
  doc files. Do NOT use for code logic changes, CI/YAML pipelines, or
  non-documentation source files.
---

# EMOD-Hub Documentation Update Skill

This skill encodes the canonical rules for updating documentation, docstrings,
and Markdown files across EMOD-Hub repositories. Apply every rule below to
**every file you touch**. Rules are cumulative — a single file may require
several changes.

**Apply rules in order.** Rule 1 must run first — it creates `bib.md`, which
Rules 3 and 6 depend on to write new links correctly.

The user may provide one or more files, a repo path, or a description of the
docs to update. Produce clean, updated file content (or a diff) and clearly
summarise every change made.

---

## Rule 1 — Centralise All External Links into bib.md *(run this first)*

**What to do**

Every external URL in a documentation file should be defined once in a central
`bib.md` file at the root of the docs directory, then referenced by a short
label in each doc. This makes future link maintenance a single-file operation.
**This rule runs before all others** so that Rules 3 and 6 can write their new
links directly into `bib.md` as reference-style links rather than inline.

**Step 1 — Create or update `bib.md`**

If `bib.md` does not exist, create it at the root of the docs directory with
this structure:

```markdown
# Link Bibliography

All external URLs used across EMOD-Hub documentation.
Update links here; all docs reference this file by label.

[discussions]: https://github.com/orgs/EMOD-Hub/discussions
[emod-hub]: https://github.com/EMOD-Hub
<!-- add further entries below in alphabetical order by label -->
```

If `bib.md` already exists, append new entries — do not duplicate labels that
are already defined.

**Step 2 — Extract links from each doc file**

For each `.md` file being updated:
- Find every inline link of the form `[text](https://...)` where the URL is an
  external address (starts with `http://` or `https://`).
- Choose a short, descriptive, lowercase hyphenated label for the URL
  (e.g. `emod-hub`, `discussions`, `pypi-emod-api`, `ubuntu-releases`).
- Add an entry to `bib.md`: `[label]: https://full-url-here`
- Replace the inline link in the doc with a reference-style link: `[text][label]`

**Step 3 — Use reference-style links going forward**

Any new links introduced by subsequent rules (e.g. the discussions board URL
from Rule 3, the EMOD-Hub GitHub URL from Rule 6) must also be added to
`bib.md` and referenced by label — never written inline.

**What NOT to move to bib.md**

- Internal relative links (e.g. `[see here](../install.md)`) stay inline.
- Anchor-only links (e.g. `[top](#top)`) stay inline.
- Image links `![alt](url)` stay inline.
- URLs inside code blocks and comments are left untouched.

**Examples**

```
# BEFORE (inline links in README.md)
Please post questions on our
[discussion board](https://github.com/orgs/EMOD-Hub/discussions).
Clone the repo from [EMOD-Hub](https://github.com/EMOD-Hub).

# AFTER (README.md — reference-style)
Please post questions on our [discussion board][discussions].
Clone the repo from [EMOD-Hub][emod-hub].
```

```
# bib.md (created or appended)
[discussions]: https://github.com/orgs/EMOD-Hub/discussions
[emod-hub]: https://github.com/EMOD-Hub
```

**Label naming conventions**

- Lowercase hyphenated slugs: `emod-api`, `ubuntu-releases`, `pypi-home`.
- Include the repo name for repo-specific links: `emod-api-repo`.
- Include context for versioned or page-specific links: `ubuntu-22-04-release`.
- If a label already exists in `bib.md` for the same URL, reuse it — do not
  create a duplicate.

---

## Rule 2 — Remove FAQ Pages and All References to Them

**What to do**

- Delete any file whose name, title, or slug identifies it as an FAQ
  (e.g. `faq.md`, `FAQ.rst`, `frequently-asked-questions.md`).
- Search every remaining file for links, cross-references, nav-menu entries,
  `toctree` entries, or prose mentions that point to the deleted FAQ page and
  **remove those references entirely**.
- If a sentence only exists to say "see the FAQ", delete the whole sentence.
- If a section heading or paragraph is primarily about directing users to the
  FAQ, delete it.

**Examples**

```
# BEFORE
For common questions, see the [FAQ page](faq.md).

# AFTER
(line removed)
```

```
# BEFORE (toctree in index.rst)
.. toctree::
   install
   faq
   contributing

# AFTER
.. toctree::
   install
   contributing
```

---

## Rule 3 — Replace the IDM Support Email with the Discussion Board URL

**What to do**

- Find every occurrence of the email address `idm@gatesfoundation.org`
  (in any form: bare address, mailto link, inline text, docstring, comment).
- Replace it with a reference-style link using the `[discussions]` label
  already defined in `bib.md` by Rule 1.
- Reword the surrounding sentence naturally so it reads as a pointer to the
  discussion board, not an email address. Do not leave "email us at" or
  "contact us at" language attached to the new link.

**Examples**

```
# BEFORE
Please email idm@gatesfoundation.org with questions.

# AFTER
Please post your questions on our [discussion board][discussions].
```

```
# BEFORE
For support, contact us at idm@gatesfoundation.org.

# AFTER
For support, visit our [discussion board][discussions].
```

---

## Rule 4 — Update Supported Operating System from CentOS to Ubuntu 22.04

**What to do**

- Find every mention of CentOS (any version, any capitalisation: `CentOS`,
  `centos`, `CentOS 7`, `CentOS Linux`, etc.) used to describe the
  **officially supported** or **recommended** OS.
- Replace with **Ubuntu 22.04 (Jammy Jellyfish)**.
- Also replace any older Ubuntu versions (e.g. `Ubuntu 20.04`, `Ubuntu 18.04`)
  used as the recommended OS with `Ubuntu 22.04 (Jammy Jellyfish)`.
- Update any version-specific language (e.g. "CentOS 7 or later") to simply
  reference Ubuntu 22.04 unless the docs explicitly discuss multiple distros.
- If a doc lists CentOS alongside other distros for informational purposes,
  remove CentOS from that list and add Ubuntu 22.04 if it is not already there.
- Update any related instructions (package manager commands, paths, etc.) that
  were CentOS/RHEL-specific to their Ubuntu/Debian equivalents where possible.

**Examples**

```
# BEFORE
EMOD is officially supported on CentOS 7.

# AFTER
EMOD is officially supported on Ubuntu 22.04 (Jammy Jellyfish).
```

```
# BEFORE
Tested on: CentOS 7, Ubuntu 18.04

# AFTER
Tested on: Ubuntu 22.04 (Jammy Jellyfish)
```

```
# BEFORE (install command)
sudo yum install python3

# AFTER
sudo apt-get install python3
```

---

## Rule 5 — Remove the Custom Package Index URL (packages.idmod.org)

**What to do**

- Find every reference to `https://packages.idmod.org/` (or any sub-path of
  it) used as a pip `--index-url`, `--extra-index-url`, `-i` flag, or in
  `pip.conf` / `requirements.txt` / `setup.cfg` / `pyproject.toml` docs.
- **Remove** the flag and URL entirely. All packages are now on PyPI; no
  custom index is needed.
- If the surrounding sentence or code block only existed to explain the custom
  index, remove that sentence/block too.
- Do not replace the URL with PyPI's default URL — simply omit it, since pip
  uses PyPI by default.

**Examples**

```
# BEFORE
pip install emod-api --index-url=https://packages.idmod.org/api/pypi/pypi-production/simple

# AFTER
pip install emod-api
```

```
# BEFORE
[global]
index-url = https://packages.idmod.org/api/pypi/pypi-production/simple

# AFTER
(section removed — pip uses PyPI by default)
```

```
# BEFORE
Packages are hosted at https://packages.idmod.org/. Install using:
pip install emod-api -i https://packages.idmod.org/api/pypi/pypi-production/simple

# AFTER
Install using:
pip install emod-api
```

---

## Rule 6 — Update GitHub Organisation from InstituteforDiseaseModeling to EMOD-Hub

**What to do**

- Find every URL or reference that uses the old GitHub organisation path
  `github.com/InstituteforDiseaseModeling/` and replace it with
  `github.com/EMOD-Hub/`.
- This applies to:
  - Hyperlinks in Markdown (`[text](https://github.com/InstituteforDiseaseModeling/repo)`)
  - RST hyperlinks and `.. _label: URL` directives
  - Bare URLs in prose or docstrings
  - `git clone` commands
  - `pip install git+https://...` URLs
  - Badge URLs (shields.io, readthedocs, etc.) that embed the org name
- After updating the URL, add it to `bib.md` (per Rule 1) and replace any
  Markdown inline link with a reference-style link using the new label.
- Preserve the repository name (the part after the org) exactly as-is; only
  the org segment changes.
- **Exception — issue links:** If the URL points to a specific issue or pull
  request (i.e. the path contains `/issues/` or `/pull/`), leave the URL
  completely unchanged. Issue numbers are tied to the original repository
  history and may not map correctly after an org move.
- **Exception — repo not migrated to EMOD-Hub:** Before rewriting a repo URL,
  verify the repo actually exists under `github.com/EMOD-Hub/<repo>`. If it
  does not (e.g. it was retired, renamed, or never migrated), **keep the
  original `InstituteforDiseaseModeling` URL unchanged** — do not rewrite it
  and do not leave a `TODO` comment in the file. A known case is
  `docs-emod-scenarios`, which remains at
  `https://github.com/InstituteforDiseaseModeling/docs-emod-scenarios`.
  When in doubt during an initial pass, you may insert a `TODO: verify` note
  so the user can confirm, but once verified the TODO must be removed and the
  URL settled one way or the other — no TODO comments should remain in
  final output.

**Examples**

```
# BEFORE
git clone https://github.com/InstituteforDiseaseModeling/emod-api.git

# AFTER
git clone https://github.com/EMOD-Hub/emod-api.git
```

```
# BEFORE (inline Markdown link)
See the [emod-api](https://github.com/InstituteforDiseaseModeling/emod-api) repo.

# AFTER (reference-style, bib.md updated)
See the [emod-api][emod-api-repo] repo.
```

```
# BEFORE
[![Build](https://github.com/InstituteforDiseaseModeling/emod-api/actions/workflows/test.yml/badge.svg)]
(https://github.com/InstituteforDiseaseModeling/emod-api/actions/workflows/test.yml)

# AFTER
[![Build](https://github.com/EMOD-Hub/emod-api/actions/workflows/test.yml/badge.svg)]
(https://github.com/EMOD-Hub/emod-api/actions/workflows/test.yml)
```

```
# BEFORE (issue link — leave untouched)
See https://github.com/InstituteforDiseaseModeling/emod-api/issues/42 for context.

# AFTER (no change)
See https://github.com/InstituteforDiseaseModeling/emod-api/issues/42 for context.
```

```
# BEFORE (repo that does not exist under EMOD-Hub — leave untouched)
.. _EMOD scenarios: https://github.com/InstituteforDiseaseModeling/docs-emod-scenarios/releases

# AFTER (no change — docs-emod-scenarios was not migrated to EMOD-Hub)
.. _EMOD scenarios: https://github.com/InstituteforDiseaseModeling/docs-emod-scenarios/releases
```

---

## Verification Checklist

After applying all rules, confirm the following before returning the updated
files:

| Check | Expected result |
|---|---|
| `bib.md` exists at docs root | Present and contains all extracted labels |
| Search for `faq` (case-insensitive) | No remaining links or toctree entries pointing to an FAQ page |
| Search for `idm@gatesfoundation.org` | Zero occurrences |
| Search for `CentOS` (case-insensitive) | Zero occurrences |
| Search for `packages.idmod.org` | Zero occurrences |
| Search for `InstituteforDiseaseModeling` | Zero occurrences (except `/issues/` and `/pull/` URLs) |
| Search for inline external links `](http` in `.md` files | Zero occurrences (all moved to `bib.md`) |
| All surviving links resolve logically | No broken anchors introduced by removals |

---

## Output Format

When returning updated files or diffs:

1. **State which rules fired** on each file (e.g. "Applied rules 3, 5, 6").
2. Provide the **full updated file** or a clearly readable **unified diff**.
3. Always include the final `bib.md` in the output, even if only a few entries
   were added.
4. If a file required no changes, say so explicitly — do not silently skip it.
5. If any ambiguous case required a judgment call (e.g. a repo name that does
   not appear under EMOD-Hub), **flag it** with a `<!-- TODO: verify -->` HTML
   comment or an inline note, and explain the ambiguity to the user.
6. **Surface all open items at the end of the output under a `## Items to
   verify` section.** If you left any `TODO`, `<!-- TODO: verify -->`, or
   other "needs user confirmation" markers in the files, or made any judgment
   call that the user should double-check, list every one of them in a
   bulleted checklist at the bottom of the response. Each bullet must
   include:
   - the **file path and line number** (or a clear locator),
   - the **exact marker / snippet** left behind,
   - a **one-line question or action** for the user (e.g. "confirm repo
     `<name>` exists under EMOD-Hub; if not, revert URL to
     `InstituteforDiseaseModeling`").

   If there are zero open items, write `## Items to verify` followed by
   `None — all rules applied cleanly.` so the user knows nothing was deferred.
   Never leave TODO markers buried silently in the files without surfacing
   them here.

   **Example**

   ```markdown
   ## Items to verify

   - `docs/emod/foo.rst:42` — left `<!-- TODO: verify -->` on
     `https://github.com/EMOD-Hub/some-repo`. Please confirm `some-repo`
     exists under EMOD-Hub; if not, revert to `InstituteforDiseaseModeling`.
   - `docs/install.md:17` — replaced `CentOS 7` with `Ubuntu 22.04`, but the
     surrounding paragraph still references a CentOS-specific package path.
     Please confirm the Ubuntu equivalent is correct.
   ```
