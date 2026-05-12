---
name: nodie-wiki
description: "Maintain durable markdown knowledge. Use when asked to remember, save, preserve, review/update repo architecture, document code, study source code, research a topic, organize notes, or build a long-lived wiki."
---

# /nodie-wiki

Maintain `.nodie/` with the minimum edit that keeps code, facts, and docs consistent.

## Layout

```
.nodie/
  MEMORY.md                    # one-sentence durable takeaways
  CLAUDE.md                    # tell Claude to read the relevant .nodie README first
  AGENTS.md                    # tell Codex to read the relevant .nodie README first
  repo/                        # current code repo architecture/module docs
    README.md                  # architecture map, flows, module boundaries
    <module>.md
    <area>/                    # only for large module groups
      README.md
      <module>.md
  ref/                         # third-party service/framework/library references
    README.md
    <dependency-or-tool>.md
  wiki/                        # user/domain knowledge not owned by the code repo
    README.md
    <topic>.md
    <topic>/
      <topic>.md
      01-<supporting-file>.md
```

## Ownership

- `.nodie/repo`: repo architecture, modules, data flows, public interfaces, implementation caveats.
- `.nodie/ref`: dependency/API/framework/tool facts.
- `.nodie/wiki`: user/domain knowledge, research, daily notes, records.
- `.nodie/MEMORY.md`: the only memory file.
- `.nodie/CLAUDE.md` and `.nodie/AGENTS.md`: root-level pointers only.

If ownership is unclear, ask which `.nodie/wiki/<category>` should own it before creating a new category.

## Structure Rules

- Put module docs directly under `.nodie/repo` by default.
- Create `.nodie/repo/<area>/README.md` only when a related group exceeds 5 module docs or needs its own architecture/data-flow map.
- `.nodie/wiki` is flat by default: simple topics use `<topic>.md`; larger topics use `<topic>/<topic>.md` plus numbered supporting files.
- Use date prefixes only when chronology matters: `YYYY-MM-DD-<topic>.md` or `YYYY-MM-DD-<topic>/YYYY-MM-DD-<topic>.md`.
- If existing docs use old names/locations, restructure them into this layout and preserve content.

## Maintenance Rules

- Read the nearest `README.md`, target page/section, and linked relevant sections before editing.
- Update canonical pages in place; merge stale or duplicate content into the canonical location.
- Keep nearby `README.md` navigation accurate when files or sections move.
- Treat source code as authoritative over docs unless the user says otherwise.
- During initial repo documentation, cover the core architecture and the 3-5 files/modules needed for current work; expand later as touched.

## Content Rules

- `.nodie/repo` docs include owned paths, core data structures, public interfaces, main flows, dependencies, and caveats.
- `.nodie/ref` docs emphasize exact interface facts: paths, methods, request/response shapes, config keys, commands, pitfalls, version constraints, source links.
- `.nodie/wiki` synthesis topics distill facts; record-style topics preserve user statements with minimal interpretation.

## Links

- Add stable anchors to important headings: `## Core Data Flows \[#core-data-flows]`.
- Prefer section links for specific facts: `[Core Data Flows](architecture.md#core-data-flows)`.
- When moving or renaming files/sections, update inbound links found with `rg`.

## Templates

Load only for new pages or major rewrites:

- `references/codebase-template.md` for `.nodie/repo` docs and external codebase studies.
- `references/ref-template.md` for `.nodie/ref` dependency/tool/API references.
- `references/knowledge-template.md` for `.nodie/wiki` topics and user-defined knowledge categories.
