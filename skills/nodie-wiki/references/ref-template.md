# Reference Template

Use for `.nodie/ref/<dependency-or-tool>.md`.

```markdown
---
title: "<dependency/tool> reference"
description: "<one-line scope>"
author: agent
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [reference, <dependency-or-tool>]
category: reference
status: active
---

# <dependency/tool> Reference

## TL;DR \[#tldr]

| Item | Value |
| --- | --- |
| Package / service | `<name>` |
| Version / date | `<version or checked date>` |
| Base URL / entry point | `<url/path/import>` |
| Auth / config | `<headers/env/config>` |

## Interfaces \[#interfaces]

| Interface | Location | Shape |
| --- | --- | --- |
| `<method/endpoint/command/component>` | `<path/url/package>` | `<signature/request/response>` |

## Integration Notes \[#integration-notes]

- <Exact fact needed by agents implementing against this dependency.>

## Pitfalls \[#pitfalls]

| Pitfall | Correct behavior |
| --- | --- |
| <common wrong assumption> | <verified correct path/shape/name> |

## Sources \[#sources]

- [<source title>](<url>)
```
