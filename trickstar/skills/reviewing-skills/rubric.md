# Skill Review Rubric

Distilled from the Anthropic Skills best-practices guide. Use this as the checklist for Step 3 of the workflow in SKILL.md. Each section lists what to inspect, the failure modes to look for, and the rationale so you can judge edge cases instead of mechanical pattern-matching.

## Contents

- [1. Frontmatter](#1-frontmatter)
- [2. Description](#2-description)
- [3. Naming](#3-naming)
- [4. Conciseness](#4-conciseness)
- [5. Degrees of freedom](#5-degrees-of-freedom)
- [6. Progressive disclosure & file layout](#6-progressive-disclosure--file-layout)
- [7. Workflows & feedback loops](#7-workflows--feedback-loops)
- [8. Content hygiene](#8-content-hygiene)
- [9. Common patterns](#9-common-patterns)
- [10. Anti-patterns](#10-anti-patterns)
- [11. Executable code](#11-executable-code)
- [12. MCP & dependencies](#12-mcp--dependencies)

---

## 1. Frontmatter

**Inspect:** YAML at top of SKILL.md.

- `name` present, ≤ 64 chars, matches `^[a-z0-9-]+$`
- `name` contains no reserved words: `anthropic`, `claude`
- `name` contains no XML tags
- `description` present, non-empty, ≤ 1024 chars, no XML tags

**Common failures:** uppercase or underscores in `name`; missing `description`; trailing XML-like markers from copy/paste.

---

## 2. Description

The description is the single most important field. It is pre-loaded into the system prompt and decides whether Claude triggers the skill at all.

- **Third person.** "Processes Excel files…" not "I can help you…" / "You can use this to…".
- **Includes *what***: the capability.
- **Includes *when***: trigger phrases, contexts, file types, user intents.
- **Specific terms** appear (filenames, library names, domain words) that real user requests would also contain.
- **Skip / negative scope** is included when the skill could be confused with a sibling (e.g. "Skip when X — use Y instead").

**Failure modes:** "Helps with documents", "Does stuff with files", first-person voice, missing trigger phrases, omits the *when*.

**Rationale:** Claude sees only `name` + `description` until the skill is triggered. A vague description means the skill never activates, no matter how good the body is.

---

## 3. Naming

- Prefer **gerund form**: `processing-pdfs`, `analyzing-spreadsheets`, `reviewing-skills`.
- Acceptable alternatives: noun phrase (`pdf-processing`) or imperative (`process-pdfs`) — but be consistent across a collection.
- Avoid: `helper`, `utils`, `tools`, `documents`, `data`, `files`, anything containing `anthropic` or `claude`.

---

## 4. Conciseness

The default assumption is that **Claude is already smart**. Every paragraph must justify its tokens.

For each paragraph in SKILL.md, ask:
- Does Claude really need this?
- Is this explaining something Claude already knows (HTTP, JSON, what a PDF is, how libraries work)?
- Could this be one sentence instead of one paragraph?

**Bad:** "PDF (Portable Document Format) files are a common file format that contains text, images, and other content. To extract text from a PDF, you'll need to use a library…"

**Good:** "Use pdfplumber for text extraction:" + 3-line code snippet.

**Note:** Conciseness in SKILL.md still matters even though it's loaded on demand — once loaded, it competes with conversation history and other context.

---

## 5. Degrees of freedom

Match specificity to fragility:

- **Low freedom** (exact commands, no flags): operations that must run in sequence, are destructive, or have a single correct invocation. Use imperative "Run exactly this:" framing.
- **Medium freedom** (templates with parameters): a preferred pattern exists but configuration varies.
- **High freedom** (textual guidance, no code): open-ended judgment tasks where context decides the approach.

**Failure modes:** rigid scripts for open-ended tasks (over-prescribed); free-form prose for fragile sequential ops (under-specified).

**Heuristic — the bridge vs. field test:** if there are cliffs on either side of the path, give exact instructions; if it's an open field, give direction and trust the agent.

---

## 6. Progressive disclosure & file layout

- **SKILL.md body < 500 lines.** Over → split.
- **References one level deep.** SKILL.md → `foo.md` is fine. SKILL.md → `foo.md` → `bar.md` (where `bar.md` holds *new primary content*) is bad. Claude may partial-read deep chains and miss content.
- **Long reference files (> 100 lines) include a ToC** at the top so partial reads still surface the structure.
- **Domain-organized references** (`reference/finance.md`, `reference/sales.md`) beat generic `docs/file1.md`, `docs/file2.md`.
- **Descriptive filenames.** `form_validation_rules.md` not `doc2.md`.
- **No context penalty for unread files** — bundle generously, but ensure the link from SKILL.md is unambiguous.

**Failure modes:** SKILL.md is one giant wall of 800 lines; deeply nested chains; reference files named after letters/numbers; references mentioned in prose but never linked.

---

## 7. Workflows & feedback loops

For multi-step tasks:
- Explicit numbered steps.
- A copy-able checklist for the agent to track progress (especially for 4+ steps).
- A validation/feedback loop where stakes are high: "validate → fix errors → re-validate → only proceed when clean."

**Failure modes:** "do X, Y, and Z" in a single paragraph; no checkpoint between destructive steps; missing "what to do if validation fails" branch.

---

## 8. Content hygiene

- **No time-sensitive content in main body.** "After August 2025…" rots. Move legacy info into a collapsible "Old patterns" / "Legacy" section.
- **Consistent terminology.** Pick one of {endpoint, URL, route, path}, {field, box, element}, {extract, pull, get} and use it throughout.
- **Concrete examples** beat abstract description. If the skill produces a specific format, show input → output pairs.

---

## 9. Common patterns

When present, verify they're used correctly:

- **Template pattern:** strict templates ("ALWAYS use this exact structure") vs. flexible templates ("a sensible default — adapt as needed"). Strictness should match downstream tolerance.
- **Examples pattern:** input → output pairs are clearer than prose for style/format skills.
- **Conditional workflow:** branching ("Creating? → A. Editing? → B.") for skills that handle distinct subtasks.

If the skill *should* use one of these but doesn't (e.g. a format-output skill with no example), file it as a finding.

---

## 10. Anti-patterns

Flag any of these:

- Windows-style backslash paths (`scripts\helper.py`)
- "You can use library A, or B, or C, or D…" — pick a default and add an escape hatch only if needed
- First-person or second-person description
- Meta-commentary ("This skill will help you…") instead of direct instructions
- Time-bound dates outside an Old-patterns section
- Inconsistent terminology
- Reference chains > 1 level deep
- Files named `doc1.md`, `notes.md`, `misc.md`

---

## 11. Executable code

Only applies if the skill bundles `.py` / `.sh` / `.ts` etc.

- **Solve, don't punt.** Scripts handle expected error conditions instead of throwing back to Claude. Bad: `return open(path).read()`. Good: catch `FileNotFoundError`, log, return a sensible default.
- **No voodoo constants.** Magic numbers (timeouts, retry counts) carry a one-line justification comment. If the author can't justify the value, neither can Claude.
- **Self-documenting scripts.** A short docstring or top-of-file comment explains purpose, inputs, outputs.
- **Execution intent is explicit in SKILL.md.** "Run `analyze_form.py`" (execute) vs. "See `analyze_form.py` for the algorithm" (read as reference). Ambiguity here causes Claude to load the script into context unnecessarily.
- **Validation/verification scripts** exist for fragile multi-step operations (the plan-validate-execute pattern).
- **Verbose error messages** with actionable hints, not just stack traces.

---

## 12. MCP & dependencies

- **MCP tool references are fully qualified**: `ServerName:tool_name` (e.g. `BigQuery:bigquery_schema`). Bare `bigquery_schema` will fail when multiple servers are present.
- **Required packages are listed** in SKILL.md and verified to exist in the target runtime (claude.ai allows pip/npm install; the bare Claude API runtime does not).
- **No assumed-installed libraries** without a setup line.

---

## Severity assignment cheat-sheet

| Severity | Examples |
|---|---|
| BLOCKER | invalid `name` chars, reserved word in `name`, missing/oversize `description`, broken required reference, frontmatter fails to parse |
| HIGH | first-person description, body > 500 lines, time-sensitive info in main body, scripts that punt to Claude, missing `when-to-use` triggers |
| MEDIUM | unnecessary explanation paragraphs, inconsistent terminology, missing ToC on long ref file, missing checklist on 4+ step workflow, voodoo constants |
| LOW | naming style preference, formatting nits, optional examples |

If unsure between two severities, pick the lower one — over-flagging trains the user to ignore the report.
