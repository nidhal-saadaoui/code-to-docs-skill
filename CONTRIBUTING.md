# Contributing

Contributions are welcome — bug fixes, new stack support, improved reference guides, and additional eval cases are all useful.

## Before opening a PR

For anything beyond a small fix, open an issue first to discuss the change. This avoids wasted effort if the direction doesn't fit the project's goals.

## How the skill is structured

```
SKILL.md                — workflow entry point (Steps 1–5, subcommand table, stack detection)
references/             — one guide per subcommand, loaded on demand
evals/evals.json        — test cases with assertions (human-reviewed)
preview/                — example output for a fictional project
```

The key design principle: `SKILL.md` stays under 500 lines and loads reference files only when the relevant subcommand is requested. Keep it that way — don't add subcommand-specific logic to `SKILL.md` when it belongs in a reference file.

## Common contribution types

**Improving an existing reference guide** (`references/*.md`)
- Read the guide and identify what's missing or wrong
- Test your change against a real project before submitting
- Keep templates concrete — vague guidance produces vague output

**Adding a new stack**
- Add a detection row to the stack table in `SKILL.md`
- Add stack-specific commands to `references/readme-guide.md` (quick start table) and `references/deployment-guide.md` (build artifact table)
- Add single-test commands to `references/onboarding-guide.md` if applicable

**Adding an eval case** (`evals/evals.json`)
- Follow the existing format: `id`, `prompt`, `context`, `assertions`
- Assertions should be checkable by a human reading the output — no vague "output is good quality"
- Cover edge cases and failure modes, not just the happy path

**Adding a new subcommand**
- Add a row to the subcommands table in `SKILL.md`
- Add a row to the Step 2 reference file table in `SKILL.md`
- Create `references/<subcommand>-guide.md`
- Add at least one eval case
- Add example output to `preview/`

## Testing your changes

There is no automated test runner — evals are human-reviewed. To validate a change:

1. Install the skill in a Claude Code session: `git clone <your-fork> ~/.claude/skills/code-to-docs`
2. Run the affected subcommand against a real project
3. Check the output against the relevant assertions in `evals/evals.json`
4. If you changed `SKILL.md`, also verify the skill is still triggered by natural language (e.g. "document this project")

## PR checklist

- [ ] Change is tested against at least one real project
- [ ] If adding a stack: detection, quick start, and build commands are all updated consistently
- [ ] If adding a subcommand: reference guide, SKILL.md table, eval case, and preview output are all present
- [ ] CHANGELOG.md is updated under an `[Unreleased]` section
