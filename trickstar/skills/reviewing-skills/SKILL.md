---
name: reviewing-skills
description: Reviews a SKILL.md against the Anthropic Skills best-practice rubric — frontmatter validity, conciseness, progressive disclosure, workflow clarity, anti-patterns — and proposes concrete improvements. Use when the user asks to evaluate, review, audit, critique, or improve an existing skill, including 「Skillを評価して」「skill をレビューして」「このSKILL.md を改善して」「skill のベストプラクティス確認」. Skip when creating a skill from scratch (use skill-creator), when stress-testing a fresh draft for ambiguity via a blank-slate executor (use empirical-prompt-tuning), or when running quantitative accuracy/variance benchmarks (use skill-creator's eval flow).
---

# Reviewing Skills

Audit an existing SKILL.md (and its bundled files) against the Anthropic Skills best-practice rubric, then propose and apply concrete improvements. The skill is purely a static lint pass — it reads the file, it does not execute the skill. For dynamic feedback, hand off:

- **Right after authoring / major revisions, to surface ambiguities a blank-slate executor stumbles on** → `empirical-prompt-tuning` (iterative: dispatch fresh subagent, collect unclear-points + trace, fix minimum diff, re-run with a new subagent until it plateaus)
- **To measure quantitative accuracy / variance over many runs** → `skill-creator`'s eval-viewer flow

## When to use

- A skill exists (draft or shipped) and the user wants a critique or polish pass
- Just authored a skill and want a lint pass before committing
- A skill is misfiring and you suspect structural causes (vague description, bloated body, deeply nested references)

When **not** to use:
- No SKILL.md exists yet → use `skill-creator`
- Just authored / heavily revised a skill and want to find the spots a fresh executor will trip over → use `empirical-prompt-tuning` (it dispatches a blank-slate subagent, captures unclear-points + trace, and iterates on minimum diffs). `reviewing-skills` is a static lint pass; `empirical-prompt-tuning` is a dynamic "does it actually run cleanly" pass — they compose well in that order.
- Want to measure quantitative accuracy / variance over many runs → use `skill-creator`'s eval-viewer
- The fix is obvious from a single reading — just edit it

## Workflow

Copy this checklist into your response and tick items as you progress:

```
Review Progress:
- [ ] Step 1: Locate target skill (SKILL.md path, bundled files, scripts)
- [ ] Step 2: Mechanical checks (frontmatter, line count, ref depth, paths)
- [ ] Step 3: Qualitative checks (description, conciseness, terminology, anti-patterns)
- [ ] Step 4: Script checks (only if the skill bundles executable code)
- [ ] Step 5: Produce findings report with severity-classified issues
- [ ] Step 6: Confirm scope of fixes with the user
- [ ] Step 7: Apply edits
- [ ] Step 8: Re-run mechanical checks to verify regressions are gone
```

**Step 1 — Locate target skill.** Ask the user for the SKILL.md path if not given. Then `ls` the skill directory and `cat` SKILL.md plus any referenced files. Note the full directory layout — the rubric scores reference structure, not just SKILL.md.

**Step 2 — Mechanical checks.** These are deterministic; do not skip. For each, record pass/fail with evidence:
- Frontmatter parses; `name` ≤ 64 chars, `[a-z0-9-]+` only, no reserved words (`anthropic`, `claude`)
- `description` non-empty, ≤ 1024 chars, no XML tags
- `wc -l SKILL.md` < 500
- All inline links resolve to existing files (`grep -oE '\[[^]]+\]\([^)]+\)' SKILL.md`)
- Reference depth: any file linked from SKILL.md must not introduce *new* primary references one more level down (skim each linked file for outbound links)
- Reference files > 100 lines have a Table of Contents near the top
- File paths use forward slashes (no `\` on relative paths)

**Step 3 — Qualitative checks.** Use the rubric in [rubric.md](rubric.md). Read SKILL.md against each item and write down what you observe — not whether it "feels good." Look especially for:
- Description in third person, includes both *what* and *when*, contains specific trigger terms
- Body avoids explaining things Claude already knows (PDFs, HTTP, common libraries)
- Degree of freedom matches task fragility (low for fragile/sequential ops, high for open-ended)
- Terminology is consistent (one word per concept throughout)
- No time-sensitive phrases ("until August 2025", "the new API"); legacy material lives in an "Old patterns" section
- Workflows have explicit steps (and a checklist for non-trivial flows)
- Feedback loops exist for quality-critical outputs (validate → fix → repeat)

**Step 4 — Script checks (skip if no scripts).** For each `.py`/`.sh`/`.ts` under the skill directory:
- Errors are handled, not punted to Claude (no bare `open(path).read()` style)
- No voodoo constants — magic numbers carry a brief justification comment
- Required packages are listed in SKILL.md and exist on the target runtime
- The instruction makes execution intent explicit ("Run X" vs "See X for the algorithm")

**Step 5 — Produce findings report.** Group by severity:
- **BLOCKER** — frontmatter invalid, reserved word in `name`, broken required references
- **HIGH** — vague description, body > 500 lines, time-sensitive content in main body, scripts that punt to Claude
- **MEDIUM** — verbose paragraphs Claude doesn't need, inconsistent terminology, missing ToC on long ref files, missing checklist on multi-step workflow
- **LOW** — naming style nits, formatting preferences, optional polish

For each finding: `path:line` + one-sentence rationale + suggested fix. Cite the rubric item.

**Step 6 — Confirm scope.** Before editing, summarize the BLOCKER + HIGH list and ask which findings to apply. Do not auto-rewrite the user's prose tone unless asked.

**Step 7 — Apply edits.** Use `Edit` for surgical changes; reserve `Write` for complete rewrites of small reference files. Preserve the author's voice — fix structure, not style, unless style was flagged.

**Step 8 — Re-verify.** Re-run Step 2's mechanical checks. Confirm no new regressions (e.g., a fix that pushes the body over 500 lines). If regressions appear, return to Step 7.

## Output format

Render the findings report as a markdown table the user can scan, then a follow-up question. Example:

```markdown
## Review of <skill-name>/SKILL.md

| Severity | Item | Location | Finding | Suggested fix |
|---|---|---|---|---|
| BLOCKER | frontmatter | SKILL.md:2 | `name` contains "claude" (reserved word) | Rename to `analyzing-prs` |
| HIGH | description | SKILL.md:3 | Written in 1st person ("I help you...") | Rewrite in 3rd person, add concrete trigger phrases |
| MEDIUM | conciseness | SKILL.md:42-58 | Paragraph explains what HTTP is | Delete; assume Claude knows |
| LOW | terminology | SKILL.md:60,72 | Mixes "endpoint" and "URL" | Pick one and replace |

**Apply BLOCKER + HIGH (4 fixes)?** Or pick a subset.
```

## Severity calibration

- A finding is **BLOCKER** only if the skill is structurally broken (won't load, frontmatter rejected, references are dead).
- A finding is **HIGH** if the skill loads but a typical user request will likely fail or mistrigger.
- Most readability/conciseness issues are **MEDIUM** — they cost tokens but the skill still works.
- Reserve **LOW** for genuine taste calls. If you find yourself filing many LOW items, you are nitpicking — tighten the bar.

## Reference

[rubric.md](rubric.md) — full evaluation criteria distilled from the Anthropic Skills best-practices guide. Read this before Step 3.
