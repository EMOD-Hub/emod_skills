---
name: emod-hub-doc-update
description: >
  Use this skill when updating documentation, docstrings, or Markdown files
  across EMOD-Hub repositories. Works with both Sphinx (conf.py) and MkDocs
  (mkdocs.yml) projects. Triggers include: any request to update, audit, or
  migrate docs for EMOD-Hub / IDM repos, fix outdated references, remove pages
  or links to deprecated/retired resources (FAQ, EMOD-InputData,
  docs-emod-scenarios), update OS support language, fix package index URLs, or
  update GitHub org links. Covers .md, .rst, .txt, .py (docstrings), and
  similar text-based doc files, plus adding any doc-build dependencies the
  edits require (e.g. pyproject.toml docs extras, pymdownx.snippets auto_append
  for MkDocs). Do NOT use for code logic changes, CI/YAML pipelines, or
  non-documentation source files.
---

# EMOD-Hub Documentation Update Skill

This skill encodes the canonical rules for updating documentation, docstrings,
and Markdown files across EMOD-Hub repositories. It supports both **Sphinx**
(`docs/conf.py`) and **MkDocs** (`mkdocs.yml`) doc generators. Apply every
rule below to **every file you touch**. Rules are cumulative — a single file
may require several changes.

**Apply rules in order.** Rule 1 must run first — it creates `bib.md`, which
Rules 3 and 6 depend on to write new links correctly. Rule 7 must run last —
it closes the loop by adding any build dependencies that earlier rules have
caused the doc toolchain to need.

The user may provide one or more files, a repo path, or a description of the
docs to update. Produce clean, updated file content (or a diff) and clearly
summarise every change made.

---

## Preamble — Detect the Doc Generator *(do this before Rule 1)*

Before applying any rule, identify which doc generator the project uses, and
locate the doc tree. The choice changes how Rule 1 centralises links and what
dependencies Rule 7 may need to add.

**Detection rules**

1. **Sphinx** — the project contains `docs/conf.py` (or any `conf.py` whose
   directory holds `.rst` / `.md` source pages). The doc tree is the directory
   containing `conf.py`.
2. **MkDocs** — the project contains `mkdocs.yml` (or `mkdocs.yaml`) at the
   repo root. The doc tree is the directory referenced by `docs_dir:` in
   `mkdocs.yml` (default: `docs/`).
3. **Both** — some repos historically had Sphinx and have migrated (or are
   migrating) to MkDocs. Treat each tree separately and apply Rule 1 once
   per tree, with its own `bib.md`.
4. **Neither** — no `conf.py` and no `mkdocs.yml`. The repo has only
   standalone Markdown (root `README.md`, etc.). Skip Rule 1 entirely; Rules
   2–6 still apply but they rewrite URLs inline (per the standalone-file
   guidance in Rule 1's Scope section). Rule 7 has nothing to do.

**Why the generator matters for Rule 1**

Markdown reference-style links (`[text][label]` resolved by `[label]: URL`)
behave differently across renderers:

| Renderer | How `bib.md` is consumed |
|---|---|
| Sphinx + MyST | MyST reads every `.md` page in the doc tree; reference definitions in any one file resolve across the whole project, so a single `bib.md` "just works" once it lives in the doc tree. |
| MkDocs (default) | Reference definitions are scoped **per file**. A `bib.md` at `docs/` root will NOT propagate to other pages unless you wire it in. |
| MkDocs + `pymdownx.snippets` with `auto_append` | The snippets extension appends the listed file(s) to every rendered page, restoring the cross-page resolution behaviour. **This is the supported mechanism for centralised links in MkDocs projects.** |
| GitHub (renders `README.md`, etc. standalone) | Reference definitions are scoped per file. `bib.md` is invisible to them. |

**MkDocs precondition for Rule 1**

If the project uses MkDocs, before Rule 1 can centralise links, `mkdocs.yml`
must include the `pymdownx.snippets` extension with `auto_append` listing
`bib.md`. Example:

```yaml
markdown_extensions:
  - pymdownx.snippets:
      auto_append:
        - docs/bib.md
```

If this configuration is **missing**, add it as part of Rule 1's setup (and
record the dependency under Rule 7 if `pymdownx` is not already pulled in by
`mkdocs-material` or similar). If the user has explicitly opted out of
centralised links, fall back to keeping links inline and note this in the
`## Items to verify` section.

**What "doc tree" means in the rest of this skill**

Throughout this document, "the doc tree" or "in-tree" means whichever of the
following applies for the project:

- The directory containing `conf.py` (Sphinx).
- The directory referenced by `docs_dir:` in `mkdocs.yml`, default `docs/`
  (MkDocs).

Files **outside** the doc tree (root `README.md`, top-level
`tutorial_*.md`, `examples/**/README.md`, etc.) are "standalone" regardless
of generator — Rule 1 does not centralise their links.

---

## Rule 1 — Centralise External Links into bib.md (in-tree only) *(run this first)*

**What to do**

Every external URL in a documentation file inside the doc tree (Sphinx or
MkDocs — see the Preamble for how to locate it) should be defined once in a
central `bib.md` file at the root of that doc directory, then referenced by a
short label in each doc. This makes future link maintenance a single-file
operation. **This rule runs before all others** so that Rules 3 and 6 can
write their new links directly into `bib.md` as reference-style links rather
than inline.

**Generator-specific precondition**

- **Sphinx + MyST:** no extra setup needed. MyST resolves Markdown
  reference definitions across the whole tree natively.
- **MkDocs:** `mkdocs.yml` must include `pymdownx.snippets` with
  `auto_append: [docs/bib.md]` (or whatever path holds `bib.md`). If absent,
  add it as part of this rule. Without it, labels defined in `bib.md` will
  not resolve in any other page. The `pymdownx` extensions ship with
  `pymdown-extensions`, which is already a transitive dependency of
  `mkdocs-material` — so usually no new dependency is needed, but verify
  under Rule 7 if the repo is not on `mkdocs-material`.

**Scope — what `bib.md` covers and what it does not**

`bib.md` is **only consulted by files inside the doc tree** (the directory
containing `conf.py` for Sphinx, or `docs_dir:` for MkDocs). It does not
propagate to standalone Markdown files outside that tree, because Markdown
reference definitions resolve **per-file** when rendered by GitHub or any
other non-doc-generator renderer.

Apply Rule 1 to:

- All `.md` files inside the doc tree (typically `docs/**/*.md`).

Do **not** apply Rule 1 to:

- Top-level repo files: `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`,
  `getting_started.md`, `tutorial_*.md`, etc. These are rendered standalone
  by GitHub and have no access to `docs/bib.md`.
- READMEs inside non-doc directories: `examples/**/README.md`,
  `examples-container/**/README.md`, `.devcontainer/README.md`, etc.
- Any other `.md` file outside the directory containing `bib.md`.

For these standalone files, **keep links inline** (`[text](https://...)`).
If a standalone file already uses reference-style links, the reference
definitions must live at the bottom of that same file — never reach across
to `docs/bib.md`. Subsequent rules (3, 4, 5, 6) still apply to standalone
files; they just rewrite the URL in place rather than going through `bib.md`.

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

**Step 2 — Extract links from each in-scope `.md` file**

For each in-scope `.md` file (per the Scope section above — i.e. files inside
the doc tree):
- Find every inline link of the form `[text](https://...)` where the URL is an
  external address (starts with `http://` or `https://`).
- Choose a short, descriptive, lowercase hyphenated label for the URL
  (e.g. `emod-hub`, `discussions`, `pypi-emod-api`, `ubuntu-releases`).
- Add an entry to `bib.md`: `[label]: https://full-url-here`
- Replace the inline link in the doc with a reference-style link: `[text][label]`

For standalone `.md` files outside the doc tree, **leave the inline link
in place**. Do not move them to `bib.md` — the labels would not resolve
when GitHub renders the file (and on MkDocs they would not resolve either,
since standalone files are typically not in `docs_dir`).

**Step 3 — Use reference-style links going forward (in-tree only)**

Any new links introduced by subsequent rules in **in-scope** doc files
(e.g. the discussions board URL from Rule 3, the EMOD-Hub GitHub URL from
Rule 6) must also be added to `bib.md` and referenced by label — never
written inline.

In **standalone** `.md` files, the same subsequent rules just rewrite the
URL inline. Do not introduce new `[label]:` definitions in standalone files
unless that file already uses the reference-style convention with its own
bottom-of-file definitions; in that case, add the new definition to the
same file (not to `docs/bib.md`).

**What NOT to move to bib.md**

- Links in standalone `.md` files outside the Sphinx doc tree (root
  `README.md`, top-level `tutorial_*.md`, `examples/**/README.md`, etc.) —
  these stay inline (see Scope section above).
- Internal relative links (e.g. `[see here](../install.md)`) stay inline.
- Anchor-only links (e.g. `[top](#top)`) stay inline.
- Image links `![alt](url)` stay inline.
- URLs inside code blocks and comments are left untouched.

**Examples**

```
# BEFORE — docs/intro.md (in-scope, inside the doc tree)
Please post questions on our
[discussion board](https://github.com/orgs/EMOD-Hub/discussions).
Clone the repo from [EMOD-Hub](https://github.com/EMOD-Hub).

# AFTER — docs/intro.md (reference-style, labels resolve via docs/bib.md)
Please post questions on our [discussion board][discussions].
Clone the repo from [EMOD-Hub][emod-hub].
```

```
# docs/bib.md (created or appended)
[discussions]: https://github.com/orgs/EMOD-Hub/discussions
[emod-hub]: https://github.com/EMOD-Hub
```

```yaml
# mkdocs.yml — required wiring for MkDocs projects
markdown_extensions:
  - pymdownx.snippets:
      auto_append:
        - docs/bib.md
```

```
# BEFORE — README.md at repo root (out of scope, GitHub renders standalone)
Please email idm@gatesfoundation.org with questions.

# AFTER — README.md (Rule 3 still applies, but the link stays INLINE)
Please post your questions on our [discussion board](https://github.com/orgs/EMOD-Hub/discussions).
```

**Label naming conventions**

- Lowercase hyphenated slugs: `emod-api`, `ubuntu-releases`, `pypi-home`.
- Include the repo name for repo-specific links: `emod-api-repo`.
- Include context for versioned or page-specific links: `ubuntu-22-04-release`.
- If a label already exists in `bib.md` for the same URL, reuse it — do not
  create a duplicate.

---

## Rule 2 — Remove Pages and References to Deprecated or Retired Resources

**What to do**

Several documentation resources referenced by older EMOD-Hub docs are now
deprecated, retired, or archived. Delete any pages dedicated to them and remove
every link, `toctree` entry, include directive, `git clone` command, RST link
target (`.. _label:`), and prose mention that points at them.

This rule covers (at minimum) the resources in the table below. Treat the list
as additive — if you encounter another retired resource, follow the same
removal pattern.

| Resource | Status | What to find and remove |
|---|---|---|
| FAQ pages (`faq.md`, `FAQ.rst`, `frequently-asked-questions.*`) | Deprecated | Delete the file; remove every link, toctree entry, and "see the FAQ" sentence |
| `EMOD-InputData` GitHub repo | Archived 2026-02-19 — input files merged into main `EMOD` repo | Remove every URL containing `EMOD-InputData` (under any GitHub org), every `git clone .../EMOD-InputData.git` command, and prose paragraphs about downloading input data from this repo |
| `docs-emod-scenarios` GitHub repo | Retired — never migrated to EMOD-Hub | Remove every URL containing `docs-emod-scenarios` (under any GitHub org), every `.. _EMOD scenarios:` RST link target, and prose mentions of "EMOD scenarios releases" |

**How to remove cleanly**

- If a sentence exists only to point at one of these resources, delete the
  whole sentence.
- If a paragraph or section is primarily about one of these resources, delete
  the section heading and its body.
- If a `toctree` or `.. include::` directive references one of these, drop
  the entry. Do not leave a dangling include.
- If a sentence mixes a deprecated reference with still-relevant content, keep
  the relevant part and rewrite the sentence so it stands on its own.
- After removing an RST link target (`.. _label: URL`), grep for any remaining
  `` `label`_ `` references in the same file and remove or rephrase those too,
  otherwise Sphinx will warn about an unreferenced target.

**Examples — FAQ**

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

**Examples — EMOD-InputData**

```
# BEFORE (RST prose)
Clone the input data repository::

    git clone https://github.com/InstituteforDiseaseModeling/EMOD-InputData.git

# AFTER
(entire paragraph and code block removed; if the surrounding section was only
about input data, drop the section heading too)
```

```
# BEFORE (inline RST link)
See the GitHub `EMOD-InputData <https://github.com/InstituteforDiseaseModeling/EMOD-InputData>`_ repository for sample demographics files.

# AFTER
(sentence removed; sample demographics now live in the main EMOD repo, so
either drop the reference entirely or rewrite to point at the relevant
in-repo path if known)
```

**Examples — docs-emod-scenarios**

```
# BEFORE (RST link target)
.. _EMOD scenarios: https://github.com/InstituteforDiseaseModeling/docs-emod-scenarios/releases

# AFTER
(line removed; also remove every `EMOD scenarios`_ reference elsewhere in the
file — Sphinx will warn about unreferenced targets if you leave them)
```

```
# BEFORE (prose)
For example scenarios, see the `EMOD scenarios`_ releases on GitHub.

# AFTER
(sentence removed)
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
  does not (retired, renamed, or never migrated), **prefer removing the
  reference** under Rule 2's removal pattern rather than rewriting it or
  leaving a stale `InstituteforDiseaseModeling` URL behind. Only keep the
  original URL if the reference is still load-bearing for the doc and no
  EMOD-Hub equivalent exists; in that case, flag it in the `## Items to
  verify` section so the user can confirm. Do not leave `TODO` comments in
  the final file output.

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

---

## Rule 7 — Add Build Dependencies Required by Doc Changes *(run this last)*

**What to do**

Some edits introduced by Rules 1–6 can cause the doc toolchain (Sphinx + MyST
**or** MkDocs + plugins) to need a package it did not need before. If a rule
adds a new kind of source file (for example, the first `.md` file from Rule 1
in a previously RST-only Sphinx project) or begins to exercise a toolchain
feature that was configured but never actually triggered (e.g. a MyST
extension listed in `conf.py` or an MkDocs plugin listed in `mkdocs.yml`
whose backing package was never installed), the doc build will start failing
with a `ModuleNotFoundError` or similar missing-dependency error.

For **every rule that fires**, ask: "does this change cause the doc build to
need something new?" If yes, add the dependency in the same PR.

**Where to add the dependency**

Add it to the project's documentation dependency set, in this order of
preference (use whichever the repo already uses):

1. `pyproject.toml` under `[project.optional-dependencies].docs` — the canonical
   location in modern EMOD-Hub repos. CI installs via `pip install .[docs]`.
2. `setup.py` / `setup.cfg` under `extras_require={"docs": [...]}` — older
   repos.
3. `docs/requirements.txt` — if the repo uses a dedicated requirements file
   for docs.

Pin loosely (compatible-release `~=`) to match the repo's existing style.
Prefer the extras-style form (e.g. `myst-parser[linkify]`, `mkdocs-material[imaging]`)
over adding a separate top-level dependency, since extras keep related
packages co-versioned.

**When this rule fires**

Common triggers seen in practice:

*Sphinx / MyST projects*

| Earlier change | New dependency needed | Why |
|---|---|---|
| Rule 1 creates the first `.md` file in a previously RST-only project, and `conf.py` already enables the MyST `linkify` extension | `myst-parser[linkify]` (pulls in `linkify-it-py`) | MyST only processes Markdown, so the `linkify` extension was dormant until a `.md` file existed. First Markdown doc causes `ModuleNotFoundError: Linkify enabled but not installed.` |
| Adding a notebook (`.ipynb`) to the doc tree | `nbsphinx` or `myst-nb` | Sphinx needs a parser for notebook source files. |
| Adding Mermaid / PlantUML diagrams | `sphinxcontrib-mermaid` / `plantweb` | Diagram directives fail without their backing extension. |
| Adding dollar-math or AMS-math in `.md` files | `myst-parser[linkify]` is not enough — ensure `myst_enable_extensions` has `dollarmath`/`amsmath` and the MyST version supports them | Math extensions are built into `myst-parser` but must be enabled. No new package, but verify `conf.py`. |

*MkDocs projects*

| Earlier change | New dependency needed | Why |
|---|---|---|
| Rule 1 wires up `pymdownx.snippets` `auto_append` for the first time and the repo is **not** on `mkdocs-material` | `pymdown-extensions` | `pymdownx.*` extensions live in this package; `mkdocs-material` already depends on it transitively, but a bare `mkdocs` install does not. |
| Adding API auto-docs via `mkdocs-autoapi` | `mkdocs-autoapi`, `mkdocstrings`, `mkdocstrings-python` | Required to generate auto-API pages. |
| Adding admonitions / tabs / Mermaid in MkDocs | `mkdocs-material` (or `mkdocs-mermaid2-plugin`) | Material theme provides most extensions; bare MkDocs does not. |
| Adding Jupyter notebooks to the MkDocs doc tree | `mkdocs-jupyter` | Required to render `.ipynb` source pages. |
| Adding `include-markdown:` plugin usage (Rule 2 cleanups that previously inlined a removed file) | `mkdocs-include-markdown-plugin` | Plugin must be installed if it's listed under `plugins:` in `mkdocs.yml`. |
| Adding Mermaid / PlantUML diagrams to `.md` pages | `mkdocs-mermaid2-plugin` (or equivalent) | Plugin must be present for `mermaid` fenced blocks to render. |

**How to verify**

After adding the dependency:

1. Confirm the dependency file change (show the diff).
2. Confirm the doc-build CI workflow installs via the same extras set you
   updated. Common workflow files:
   - Sphinx: `.github/workflows/sphinx.yml`, `.github/workflows/docs.yml`,
     ReadTheDocs `.readthedocs.yaml` (where `python.install` references
     `extra_requirements: [docs]`).
   - MkDocs: `.github/workflows/mkdocs_build.yml`,
     `.github/workflows/mkdocs_deploy.yml`.

   If the workflow uses `pip install .[docs]` the change flows through
   automatically; if it uses a pinned `requirements.txt`, that file needs
   updating too.
3. If you can run the build locally, do so and verify no `ModuleNotFoundError`:
   - Sphinx: `sphinx-build -b html docs/ docs/_build/html` (or `make html`).
   - MkDocs: `mkdocs build --strict` (the `--strict` flag turns warnings into
     errors and catches missing-plugin issues immediately).

**Example A — Sphinx Linkify**

A repo that previously had only `.rst` docs adds `docs/bib.md` as part of
Rule 1. `docs/conf.py` already contains:

```python
myst_enable_extensions = [
    ...,
    "linkify",
    ...,
]
```

The next doc build fails on GitHub Actions with:

```
ModuleNotFoundError: Linkify enabled but not installed.
```

Because the `linkify` MyST extension requires the `linkify-it-py` package,
which ships as an optional extra of `myst-parser`. Fix in `pyproject.toml`:

```diff
 docs = [
     ...
-    "myst-parser~=2.0",
+    "myst-parser[linkify]~=2.0",
     ...
 ]
```

No workflow change is needed because the docs CI installs via
`pip install .[docs]`. The next build succeeds.

**Example B — MkDocs `pymdownx.snippets` for centralised links**

A bare MkDocs repo (no `mkdocs-material`) adopts Rule 1 by adding to
`mkdocs.yml`:

```yaml
markdown_extensions:
  - pymdownx.snippets:
      auto_append:
        - docs/bib.md
```

The next `mkdocs build` fails with:

```
ModuleNotFoundError: No module named 'pymdownx'
```

Fix in `pyproject.toml`:

```diff
 docs = [
     "mkdocs",
+    "pymdown-extensions",
     ...
 ]
```

If the project already uses `mkdocs-material`, this is unnecessary — the
material theme pulls `pymdown-extensions` in transitively.

**What NOT to do**

- Do not add dependencies speculatively. Only add what an actually-failing
  build (or a build you can predict will fail from the configuration) needs.
- Do not pin exact versions unless the repo already uses exact pins.
- Do not add a runtime dependency (top-level `dependencies` in
  `pyproject.toml`) for a docs-only package — keep it scoped to the `docs`
  extra.

---

## Verification Checklist

After applying all rules, confirm the following before returning the updated
files:

| Check | Expected result |
|---|---|
| Doc generator detected (Sphinx `conf.py` or MkDocs `mkdocs.yml`) | Generator identified; doc-tree root located before applying Rule 1 |
| `bib.md` exists at doc-tree root | Present and contains all extracted labels |
| (MkDocs only) `pymdownx.snippets` `auto_append` includes `bib.md` | Configured in `mkdocs.yml`, otherwise reference labels will not resolve |
| Search for `faq` (case-insensitive) | No remaining links or toctree entries pointing to an FAQ page |
| Search for `EMOD-InputData` | Zero occurrences (repo is archived; references should be removed per Rule 2) |
| Search for `docs-emod-scenarios` | Zero occurrences (repo retired; references should be removed per Rule 2) |
| Search for `idm@gatesfoundation.org` | Zero occurrences |
| Search for `CentOS` (case-insensitive) | Zero occurrences |
| Search for `packages.idmod.org` | Zero occurrences |
| Search for `InstituteforDiseaseModeling` | Zero occurrences (except `/issues/` and `/pull/` URLs) |
| Search for inline external links `](http` in in-tree `.md` files (the doc tree only — Sphinx or MkDocs) | Zero occurrences (all moved to `bib.md`). Standalone `.md` files outside the doc tree are exempt — inline links there are correct. |
| All surviving links resolve logically | No broken anchors introduced by removals |
| Doc build dependencies cover all new file types / toolchain features (Rule 7) | `pyproject.toml` (or equivalent) lists every package the doc build now needs (Sphinx + MyST extras OR MkDocs plugins) |

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
