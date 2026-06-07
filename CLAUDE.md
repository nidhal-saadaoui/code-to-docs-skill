# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`code-to-docs-skill` — a Claude Code skill for generating documentation from source code.

## Structure

```
code-to-docs/
├── SKILL.md                        — entry point: subcommands, stack detection, workflow
└── references/                     — loaded on-demand per doc type
    ├── readme-guide.md             — README generation rules and stack-specific templates
    ├── architecture-guide.md       — architecture doc patterns and Mermaid diagram rules
    ├── api-guide.md                — REST, GraphQL, and library API doc formats
    ├── onboarding-guide.md         — contributor setup and workflow doc guide
    └── deployment-guide.md         — deployment model detection and operator doc guide
```

## Skill design principles

- No external infrastructure — uses only Claude Code's built-in tools (Bash, Read, Write, Edit)
- Progressive disclosure: `SKILL.md` loads the workflow; `references/` files load only when the relevant doc type is requested
- Stack detection via manifest files (no ML, no classifiers)
- Distributes as a folder — zero setup for the user
