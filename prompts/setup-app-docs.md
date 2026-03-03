# Prompt: Setup App Documentation

Use this prompt in any Tinkun app repo to set up or update its documentation
so it integrates correctly with the [Tinkun Docs Hub](https://tinkun-tech.github.io/docs).

---

## Prompt

````
You are setting up documentation for a Tinkun app that publishes to the
Tinkun Docs Hub (https://tinkun-tech.github.io/docs).

Read the entire codebase first to understand:
- What the app does
- Its main features and how they work
- Its settings/configuration
- Its dependencies (Python version, Django version, etc.)
- Its existing git tags and GitHub releases (use `git tag` and `gh release list`)

Then do the following:

> **Repo structure summary:**
> - `docs/` — contains `index.md` and one `.md` per feature. Nothing else.
> - `CHANGELOG.md` — lives at the repo root only. The docs hub workflow
>   copies it automatically on each release. Do NOT put it inside `docs/`.

---

### 1. Create or update `docs/index.md`

This is the app overview page. Use this exact structure:

```markdown
# Quick Start

{One or two sentences describing what the app does and its main purpose
within the Tinkun ecosystem.}

## Installation

```bash
pip install {package-name}
````

### Setup

```python
# settings.py
INSTALLED_APPS = [
    ...
    "{app_module_name}",
]
```

{Any other required setup steps, environment variables, or minimal config.}

## Features

{Group features by category if there are many. For each feature that has
its own doc file, link to it.}

- **{Feature Name}** — {brief description} ([guide]({feature}.md))
- **{Feature Name}** — {brief description}

## Requirements

- Python {X.X}+
- Django {X.X}+
- {any other hard dependencies}

## Links

- [Changelog](changelog.md)
- [GitHub Releases](https://github.com/tinkun-tech/{repo-name}/releases)

````

---

### 2. Create a doc file for each feature in `docs/`

One `.md` file per significant feature (e.g. `health_check.md`,
`celery.md`, `jwt.md`). Skip trivial internals — only document things
a developer would need to configure or use.

Use this exact structure for each feature doc:

```markdown
# {Feature Name}

## Overview

{What this feature does and why it exists. 2–4 sentences.}

## How It Works

{Optional. Explain the flow or internal logic if it helps developers
understand usage or debug issues.}

## Configuration

{Required and optional settings. Always show code blocks.}

```python
# settings.py
SETTING_NAME = "value"  # explanation
````

## Usage

### Basic Usage

```python
{minimal working example}
```

### Advanced Usage

{Additional examples for non-trivial scenarios.}

## Options / Parameters

{Table or list of all options if applicable.}

````

---

### 3. Create or update `CHANGELOG.md` at the repo root

- Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
- Populate it from existing git tags and GitHub releases (`gh release list --limit 50`)
- Use categories: `Added`, `Changed`, `Fixed`, `Removed`, `Security`
- Use `---` as a separator between versions

Use this exact structure:

```markdown
# Changelog

All notable changes to `{package-name}` are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [Unreleased]

---

## [{version}] — {YYYY-MM-DD}

### Added

- {description}

### Changed

- {description}

### Fixed

- {description}

---
````

---

### 4. Update the release workflow

Find the existing release workflow (usually `.github/workflows/release.yml`
or similar). Replace or add the release publishing and docs trigger steps
with the following block at the end of the job:

```yaml
- name: Extract latest changelog entry
  run: |
    python - <<'EOF'
    import re, pathlib
    changelog = pathlib.Path("CHANGELOG.md").read_text()
    match = re.search(r'(## \[.*?)(?=\n## \[|\Z)', changelog, re.DOTALL)
    if match:
        pathlib.Path("CHANGELOG_LATEST.md").write_text(match.group(1).strip())
    else:
        pathlib.Path("CHANGELOG_LATEST.md").write_text("See CHANGELOG.md for details.")
    EOF

- name: Create GitHub Release
  uses: softprops/action-gh-release@v2
  with:
    generate_release_notes: false
    body_path: CHANGELOG_LATEST.md

- name: Trigger docs hub rebuild
  uses: peter-evans/repository-dispatch@v3
  with:
    token: ${{ secrets.GH_PAT }}
    repository: tinkun-tech/docs
    event-type: app-released
    client-payload: '{"app":"{app_folder_name}","version":"${{ github.ref_name }}"}'
```

Replace `{app_folder_name}` with the exact folder name used in the docs hub
(e.g. `utils_app`, `email_app`, `auth_app`, `code_app`, `instant_win_app`,
`marketplace_app`, `oidc_app`, `translation_app`).

The `GH_PAT` secret must have `repo` scope and be available in the app repo.

How it works:
- The Python script extracts the top entry from `CHANGELOG.md` (everything
  between the first and second `## [` headings) and writes it to a temp file
- `softprops/action-gh-release@v2` creates the GitHub Release using that
  extracted entry as the release body
- `repository-dispatch` triggers the docs hub to pull and publish the
  updated docs and changelog

---

## Consistency Rules

Follow these rules to keep all apps consistent:

- H1 in `index.md` is always `# Quick Start` (Zensical uses the section folder as the nav group title)
- H1 in feature docs is always the feature name
- Code blocks always specify a language (`python`, `bash`, `yaml`, etc.)
- Settings examples always go in `# settings.py` blocks
- Links to other docs use relative paths: `([guide](feature.md))`
- CHANGELOG versions use the format `[vX.Y.Z] — YYYY-MM-DD`
- Dates in ISO 8601 format (`YYYY-MM-DD`)
- Feature doc filenames are lowercase with underscores (e.g. `health_check.md`)

```

```
