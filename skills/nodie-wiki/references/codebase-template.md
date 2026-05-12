# Codebase Template

Use for `.nodie/repo` architecture/module docs or external repo studies under a user-defined `.nodie/wiki/<category>/<repo>/` folder.

## Root README

```markdown
---
title: "<repo> repo docs"
description: "Current architecture, data flows, module boundaries, and reference docs"
author: agent
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [repo, architecture]
category: repo
---

# <repo> Repo Docs

Current system reference.

## Architecture \[#architecture]

<One compact diagram or bullet map of major components and runtime boundaries.>

## Core Data Flows \[#core-data-flows]

| Flow | Path | Notes |
| --- | --- | --- |
| <name> | `<entry>` -> `<service>` -> `<store>` | <facts> |

## Modules \[#modules]

| Module | Doc | Owned paths | Boundary |
| --- | --- | --- | --- |
| <module> | [<module>.md](<path>) | `<path>` | <responsibility> |

## References \[#references]

| File | Scope |
| --- | --- |
| [<ref>.md](../ref/<ref>.md) | <scope> |
```

## Area README

```markdown
---
title: "<area>"
description: "<area boundary and owned responsibilities>"
author: agent
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [repo, <area>]
category: repo
---

# <area>

## Boundary \[#boundary]

<What this area owns and which adjacent responsibilities belong elsewhere.>

## Architecture \[#architecture]

<Components, dependencies, and important runtime boundaries.>

## Data Flows \[#data-flows]

| Flow | Steps | Notes |
| --- | --- | --- |
| <flow> | `<file/function>` -> `<file/function>` | <facts> |

## Modules \[#modules]

| Module | File | Purpose |
| --- | --- | --- |
| <module> | [<module>.md](<module>.md) | <purpose> |
```

## Module Page

```markdown
---
title: "<module>"
description: "<one-line factual summary>"
author: agent
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [repo, module]
category: repo
---

# <module>

## Boundary \[#boundary]

<Owned responsibility. Explicit non-goals or adjacent modules.>

## Owned Paths \[#owned-paths]

| Path | Purpose |
| --- | --- |
| `<path>` | <purpose> |

## Core Data Structures \[#core-data-structures]

| Name | Location | Fields / Shape | Notes |
| --- | --- | --- | --- |
| `<type/table/schema>` | `<path>` | `<fields>` | <facts> |

## Functional Flow \[#functional-flow]

1. `<entry point>` receives `<input>`.
2. `<component>` transforms or validates `<data>`.
3. `<component>` writes/emits `<output>`.

## Public Interfaces \[#public-interfaces]

| Interface | Location | Shape |
| --- | --- | --- |
| `<CLI/API/event/component>` | `<path>` | `<signature/schema>` |

## Dependencies \[#dependencies]

| Dependency | Use | Reference |
| --- | --- | --- |
| `<service/library>` | <use> | `.nodie/ref/<name>.md` |

## Caveats \[#caveats]

- <Known sharp edge, invariant, failure mode, migration note, or operational concern.>
```
