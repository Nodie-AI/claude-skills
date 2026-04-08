# Codebase Mode Template

Use this template when the input is a git repo or source code directory.

Create one folder per codebase under `wiki/codebases/<name>/`, matching the repo directory name.

```markdown
---
title: Module Name
description: "One-line summary"
author: agent
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [relevant, tags]
category: codebases
---

# Module Name

## Overview
What this module does and why it exists. 2-3 sentences.

## Architecture
Key components, data flow, dependencies. Diagrams welcome.

## Core Concepts
Important types, interfaces, patterns. Brief code snippets if they clarify.

## File Structure
Key files and their responsibilities. Not an exhaustive listing — focus on entry points and the files someone new would need to understand first.

## Module Relationships
How this module interacts with others. Use [[wiki-links]].

## Known Issues
Known limitations, planned improvements. Remove when resolved.
```

## Guidelines

- One file per module — keep files focused
- Start with `_index.md` that has a navigation table of all modules
- Structure-first: architecture → concepts → files → relationships
- When source code contradicts the wiki, update the wiki and note the change
- Write in the project's primary language (match existing docs)
- Technical depth should match the codebase complexity — don't over-document simple projects
