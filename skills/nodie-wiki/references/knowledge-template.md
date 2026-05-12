# Knowledge Template

Use for `.nodie/wiki` topics and user-defined `.nodie/wiki/<category>` folders.

## Simple Topic

```markdown
---
title: "<topic>"
description: "<one-line factual summary>"
author: agent
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [knowledge]
category: wiki
status: active
---

# <topic>

## Summary \[#summary]

<3-6 concise bullets with conclusions rather than process notes.>

## Key Facts \[#key-facts]

| Fact | Source / Location | Notes |
| --- | --- | --- |
| <fact> | <source> | <notes> |

## Links \[#links]

- [Related section](../other-topic.md#section)
```

## Topic Folder Summary

```markdown
---
title: "<topic>"
description: "<one-line factual summary>"
author: agent
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [knowledge]
category: wiki
status: active
---

# <topic>

## Summary \[#summary]

<3-6 concise bullets with conclusions rather than process notes.>

## Key Facts \[#key-facts]

| Fact | Source / Location | Notes |
| --- | --- | --- |
| <fact> | <source> | <notes> |

## Files \[#files]

| File | Scope |
| --- | --- |
| [01-<file>.md](01-<file>.md) | <scope> |
| [02-<file>.md](02-<file>.md) | <scope> |
```
