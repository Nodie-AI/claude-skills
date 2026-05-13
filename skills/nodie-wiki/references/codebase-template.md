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

| Module | Status | Doc | Owned paths | Boundary |
| --- | --- | --- | --- | --- |
| <module> | current | [<module>.md](<path>) | `<path>` | <responsibility> |
| <future module> | TODO | TODO: [<future module>.md](<future-module>.md) | `<path>` | <why it should exist later> |

## Cross-Module Pitfalls \[#cross-module-pitfalls]

| ID | Summary | Modules | Detail |
| --- | --- | --- | --- |
| `<PITFALL-ID>` | <one-sentence failure mode> | `<module>`, `<module>` | [pitfalls/<PITFALL-ID>.md](pitfalls/<PITFALL-ID>.md) |

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

## Regression Contract \[#regression-contract]

| ID | Case | Invariant | Test / Smoke | Detail |
| --- | --- | --- | --- | --- |
| `<PITFALL-ID>` | <short scenario> | <what must never regress> | `<test>` / <manual smoke> | [pitfalls/<PITFALL-ID>.md](pitfalls/<PITFALL-ID>.md) |

## Known Pitfalls \[#known-pitfalls]

| ID | One-line Summary | Affected Surface | Status | Detail |
| --- | --- | --- | --- | --- |
| `<PITFALL-ID>` | <one sentence describing the pitfall> | `<component/API/runtime>` | active / fixed / watch | [pitfalls/<PITFALL-ID>.md](pitfalls/<PITFALL-ID>.md) |
```

## Pitfall Page

Create under `.nodie/repo/pitfalls/<PITFALL-ID>.md`.

```markdown
---
title: "<PITFALL-ID> — <short title>"
description: "<one-line failure mode>"
author: agent
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [repo, pitfall, regression]
category: repo
status: active
---

# <PITFALL-ID> — <short title>

## Summary \[#summary]

<One sentence: what failed and why it matters.>

## Owning Module \[#owning-module]

- Module: [<module>](../<module>.md#known-pitfalls)
- Regression row: [<module> Regression Contract](../<module>.md#regression-contract)

## Symptom \[#symptom]

<What the user/developer saw. Include exact UI/CLI/runtime behavior when known.>

## Root Cause \[#root-cause]

<The underlying implementation/design failure. Avoid vague process blame.>

## Fix Pattern \[#fix-pattern]

<The durable implementation pattern or architecture rule that prevents recurrence.>

## Regression Checks \[#regression-checks]

| Check | Type | Command / Manual Step |
| --- | --- | --- |
| <check> | unit / integration / packaged smoke / manual | `<command>` or <step> |

## Source Links \[#source-links]

- [Owning module](../<module>.md#known-pitfalls)
- [Spec / handoff / issue](<relative-link>#section)
```
