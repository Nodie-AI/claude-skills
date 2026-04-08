---
name: rtfd
description: "Read The F***ing Docs — maintain a structured wiki knowledge base from any input. Feed it URLs, articles, meeting notes, or source code repos and it produces organized, interlinked wiki pages. Use when documentation is missing, outdated, or scattered. Commands: compile, ingest, clone, audit."
---

# /rtfd

Turn chaos into documentation. Feed this skill raw material — URLs, articles, meeting notes, or codebases — and it produces structured, interlinked wiki pages.

Two modes, auto-detected from input:

- **Knowledge mode**: articles, notes, URLs → distilled wiki entries with key takeaways
- **Codebase mode**: source code repos → technical docs (architecture, modules, interfaces)

## Directory layout

```
raw/                           # User's inbox — drop raw material here
  codebases/                   # Git-cloned repos
  *.md                         # Articles, notes, transcripts

wiki/                          # Agent's domain — maintained by this skill
  _master-index.md             # Entry point, one-line per topic
  research/                    # Knowledge entries (date-prefixed)
    _index.md
    YYYY-MM-DD-<topic>.md
  codebases/                   # Technical docs (folder per repo)
    <name>/
      _index.md
      architecture.md
      ...
  memo/                        # Meeting notes
  projects/                    # Project docs

output/                        # Generated reports, comparisons
```

Paths are project-relative. Adapt to whatever structure the project uses — the pattern matters more than the exact paths.

## Commands

### compile
Process everything in `raw/` not yet in the wiki:
1. Read each raw file
2. Pick the right mode (knowledge vs codebase) and template
3. Write wiki article, update topic `_index.md`, update `_master-index.md`
4. Cross-link related topics with `[[wiki-links]]`

### ingest \<URL\>
Fetch a URL, save to `raw/`, optionally compile immediately.

### clone \<repo-url\>
Clone repo to `raw/codebases/<name>/`, generate technical wiki in `wiki/codebases/<name>/`.

### audit
Check wiki health:
- Broken `[[wiki-links]]`
- Missing `_index.md` files
- Orphan pages not linked from any index
- Stale entries (source changed, wiki didn't)

## Mode detection

| Input | Mode | Template |
|-------|------|----------|
| URL, article, transcript, meeting notes | Knowledge | `references/knowledge-template.md` |
| Git repo, source code directory | Codebase | `references/codebase-template.md` |
| Mixed / unclear | Ask the user | — |

## Conventions

- **`[[wiki-links]]`** for cross-references between topics
- **Every article** gets YAML frontmatter (title, description, author, created, modified, tags, category)
- **Complex topic** (3+ files): folder with `_index.md`
- **Simple topic**: single `.md` file
- **Date-prefixed** entries in research/ and memo/
- **Don't delete** — update or annotate as stale
- **raw/ is read-only** (except when ingesting URLs)
- **wiki/ is your domain** — you own everything in it
