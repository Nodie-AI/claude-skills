# claude-skills

Three Claude Code skills for shipping software with AI agents.

| Skill | Command | What it does |
|-------|---------|-------------|
| **lgtm** | `/lgtm` | *Looks Good To Me* — acceptance verification for agent work |
| **product-manager** | `/product-manager` | *PM* — own the product, make decisions, write PRDs |
| **rtfd** | `/rtfd` | *Read The F\*\*\*ing Docs* — build a wiki from anything |

## Why these three

AI agents are great at generating code. They're terrible at knowing when the work is actually done, what to build next, and writing documentation that humans can use.

These skills fill the gaps around the code generation loop:

```
         product-manager
         "what to build"
              │
              ▼
     ┌──────────────────┐
     │  Agent does work  │  ← your existing AI coding tool
     └──────────────────┘
         │           │
         ▼           ▼
       lgtm        rtfd
    "is it done?"  "write it down"
```

## Install

Copy to your Claude Code skills directory:

```bash
# Install all three
cp -r skills/lgtm ~/.claude/skills/
cp -r skills/product-manager ~/.claude/skills/
cp -r skills/rtfd ~/.claude/skills/

# Or just the ones you want
cp -r skills/lgtm ~/.claude/skills/
```

Then use them in any Claude Code session:

```
you> /lgtm
you> /product-manager
you> /rtfd compile
```

---

## /lgtm

**Looks Good To Me** — the engineering approval phrase, repurposed as a verification skill.

Answers one question: *is the current task actually done?*

- First call: reads your conversation, generates concrete acceptance criteria, saves to `.ai-verify/criteria.md`
- Subsequent calls: checks each criterion by running tests and inspecting code, shows progress
- Catches regressions: checks existing features still work before evaluating new ones
- Read-only: never touches your source code (`allowed-tools` enforced). Reports problems, doesn't fix them

```
you> /lgtm

Verification: User registration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

REGRESSION:
  ✓ Existing test suite passes (npm test → 42 passed)
  ✓ Build clean (tsc --noEmit → exit 0)

NEW CRITERIA:
  ✓ POST /register returns 201
    └ tests/auth.test.ts → test passes
  ✗ Passwords hashed with bcrypt
    └ src/models/user.ts:12 — plaintext storage
  ○ Verification email sent
    └ unchecked — needs manual test

Progress: ●●○ 2/3
```

When all criteria pass, it offers to save them to a regression suite (`.ai-verify/regression/`) so future changes can't silently break things.

**Why "lgtm"?** Because the whole point is to give you a confident "looks good to me" before you ship — backed by evidence, not vibes.

---

## /product-manager

A full PM agent. Not an assistant helping a PM — it *is* the PM.

Makes prioritization calls, writes PRDs, drives the roadmap, aligns stakeholders, pushes things forward. When a decision needs to be made, it makes it and explains why.

- Detects who it's talking to (solo founder vs team engineer vs business stakeholder) and adjusts output format
- Lightweight mode for solo founders: conclusion + 3 reasons + 2 next steps. No PRDs unless you ask.
- Full mode for teams: structured PRDs, stakeholder briefs, decision memos
- 22 reference files covering every PM workflow: requirements intake, prioritization, data analysis, launch readiness, competitive research, session handoff, and more

```
you> We need to decide whether to build export or notifications first

My call: export first.
1. Export is blocking 3 enterprise accounts (40% of Q2 ARR)
2. Notifications are nice-to-have, zero revenue impact
3. Export is 3 eng-days, notifications is 5

Next: I'll draft the export PRD by Thursday. Dev team — capacity estimate by Friday.
```

**Why "product-manager"?** It's the job title. This skill doesn't help you be a PM — it does the PM job.

---

## /rtfd

**Read The F\*\*\*ing Docs** — because nobody writes documentation until it's too late.

Maintains a structured, interlinked wiki from any input you throw at it. Inspired by [Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): instead of RAG (rediscovering knowledge on every query), the LLM **compiles** raw material into a persistent, interlinked knowledge base.

> *"Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass."*
> — Andrej Karpathy

Two modes, auto-detected from input:

| Input | Mode | Output |
|-------|------|--------|
| URLs, articles, meeting notes, transcripts | **Knowledge** | Distilled wiki entries with key takeaways |
| Git repos, source code | **Codebase** | Technical docs: architecture, modules, interfaces |

Four commands:

```bash
/rtfd compile     # Process raw/ inbox → wiki articles
/rtfd ingest URL  # Fetch a URL, save to raw/, compile
/rtfd clone REPO  # Clone repo, generate technical wiki
/rtfd audit       # Check wiki health: broken links, orphans, stale pages
```

The architecture follows Karpathy's three-layer model:

```
raw/          # Your inbox — drop material here (read-only for the agent)
wiki/         # Agent's domain — structured, interlinked markdown
  research/   # Knowledge entries (date-prefixed)
  codebases/  # Technical docs (folder per repo)
  memo/       # Meeting notes
output/       # Generated reports, comparisons
```

**Why "rtfd"?** Because when someone asks "where's the documentation?" the answer should be "run `/rtfd`" — not "we'll get to it later."

---

## Design principles

These skills follow the [Agent Skills spec](https://agentskills.io) and Anthropic's [skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices):

- **Progressive disclosure**: short SKILL.md (<120 lines) as entry point, detailed content in `references/`
- **Examples over rules**: show the output format, don't over-explain
- **Explain why, not just what**: Claude follows reasoning better than ALL_CAPS commands
- **Hardware guardrails**: `allowed-tools` in frontmatter, not prose promises
- **Don't teach Claude what it already knows**: no explanations of `npm test` or `git diff`

## License

MIT
