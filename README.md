# trickstar0301 / skills

Personal Claude Code skill marketplace.

## Install

```
/plugin marketplace add trickstar0301/skills
/plugin install trickstar@trickstar0301
```

## Skills bundled in the `trickstar` plugin

- `trickstar:blog-style` — Japanese-language style guide and AI-tone detection rubric for blog posts (zenn / dev.to).
- `trickstar:reviewing-skills` — Reviews a SKILL.md against the Anthropic Skills best-practice rubric.
- `trickstar:html-deck` — Generate single-file HTML slide decks without frameworks (uses `frontend-design` for visual quality).

## Layout

```
.claude-plugin/marketplace.json     # marketplace manifest
trickstar/                          # the single bundled plugin
├── .claude-plugin/plugin.json
└── skills/
    ├── blog-style/SKILL.md
    ├── reviewing-skills/SKILL.md
    └── html-deck/
        ├── SKILL.md
        ├── template-full.html
        └── template-minimal.html
```

## Local development

```
# Iterate without reinstalling on every change:
claude --plugin-dir /Users/trickstar/alldata/github/trickstar0301/skills/trickstar
# Edit skills/<name>/SKILL.md, then in Claude: /reload-plugins
```
