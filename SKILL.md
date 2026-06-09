---
name: code-to-docs
description: Generate or update documentation from a codebase — README, architecture, API reference, onboarding guide, deployment guide, or ADRs. Use whenever the user wants docs written, updated, or generated from code — including casual requests like "document this", "I need docs", "document my code", "this project has no README", "write some docs", "explain the architecture", "write ADRs", "help onboard a new dev", "I finished a feature, update the docs", or "what does this project do."
---

# Code-to-Docs

Generate specific documentation files from an existing codebase using only the tools Claude Code already provides. No setup required.

## Subcommands

| Subcommand   | Generates                                                       |
|--------------|-----------------------------------------------------------------|
| `readme`     | `README.md` — project overview, usage, setup                   |
| `arch`       | `ARCHITECTURE.md` — system design, components, data flow       |
| `api`        | `docs/api.md` — API reference from routes/functions/types      |
| `onboard`    | `ONBOARDING.md` — contributor setup and workflow guide         |
| `deployment` | `DEPLOYMENT.md` — infrastructure, env vars, deploy steps       |
| `adr`        | `docs/adr/NNN-title.md` — one file per architectural decision  |
| `update`     | Edits existing docs to reflect changes on the current branch   |

If no subcommand is given, ask which doc type the user wants before proceeding.

---

## Workflow

> **`update` shortcut**: if the subcommand is `update`, skip Steps 1 and 3 entirely — go directly to Step 2 (load `references/update-guide.md`). The guide defines its own exploration via `git diff` and does not require stack detection first.

### Step 1 — Detect the stack

Read manifest files to identify the tech stack. Check in this order:

| File present                              | Stack              |
|-------------------------------------------|--------------------|
| `package.json` with `react`/`next`/`vite` | React / Next.js    |
| `package.json` (no UI framework)          | Node.js            |
| `pyproject.toml` / `requirements.txt`     | Python             |
| `go.mod`                                  | Go                 |
| `Cargo.toml`                              | Rust               |
| `pubspec.yaml`                            | Flutter / Dart     |
| `pom.xml` / `build.gradle` with `*.scala` sources | Scala / Maven or Gradle |
| `pom.xml` / `build.gradle` (no `.scala` files)    | Java / Kotlin           |
| `composer.json`                           | PHP                |
| `*.csproj` / `*.sln`                     | C# / .NET          |

If multiple stacks are detected (e.g. `pyproject.toml` + `package.json` in subdirectories), it's a multi-stack project. Note each stack and its directory. For `arch` docs, describe all components. For `readme` and `onboard`, lead with the primary entry point (the one a developer runs first to get the project working) and give each stack its own setup section. If it's unclear which is primary, ask before writing.

Also note these cross-cutting signals — they affect all doc types:

- `docker-compose.yml` / `Dockerfile` → containerized; mention in setup steps
- `terraform/` or `*.tf` → infrastructure-as-code
- `openapi.yml` / `swagger.yml` / `openapi.json` → existing API spec; read it for `api` docs
- `.github/workflows/` → CI/CD present; reference in onboarding and deployment docs
- `CHANGELOG.md` or `HISTORY.md` → project is mature; link from README

### Step 2 — Load the relevant guide

Read the reference file that matches the requested doc type:

| Doc type     | Reference file                      |
|--------------|-------------------------------------|
| `readme`     | `references/readme-guide.md`        |
| `arch`       | `references/architecture-guide.md`  |
| `api`        | `references/api-guide.md`           |
| `onboard`    | `references/onboarding-guide.md`    |
| `deployment` | `references/deployment-guide.md`    |
| `adr`        | `references/adr-guide.md`           |
| `update`     | `references/update-guide.md`        |

Each guide contains stack-specific section templates and quality rules. Read it before exploring the codebase.

### Step 3 — Explore the repo

Use Bash and Read to understand the codebase before writing anything:

```bash
find . -maxdepth 3 -not -path '*/.git/*' -not -path '*/node_modules/*' \
  -not -path '*/__pycache__/*' -type f | sort | head -80
```

**Read existing documentation first** — before touching source files. Existing docs are the highest-priority input:

- The target file itself (`README.md`, `ARCHITECTURE.md`, etc.) — read it unconditionally if it exists
- Any other docs in `docs/`, `wiki/`, or root-level markdown files
- Inline documentation: docstrings, JSDoc, Go doc comments, Rust `///` comments — treat these as authoritative, written by the author

Then read source files:
- Entry points: `main.py`, `index.ts`, `cmd/main.go`, `src/main.rs`, `app.js`, etc.
- Test files (to understand actual usage patterns and edge cases)
- Config files (for environment variables, ports, dependencies)

Only read what you need — avoid loading the entire codebase. Prioritize breadth over depth on first pass.

#### Handling existing documentation

Apply this decision tree to the target output file before writing:

```
Does the target file exist?
├── No → Write fresh
└── Yes → Read it, then:
    ├── Trivial (only a title, empty sections, or placeholder text)
    │   → Treat as fresh; replace entirely
    ├── Contradicts the current code (describes removed features, wrong commands, stale API)
    │   → Add to your Step 3.5 questions; flag the specific contradictions before editing
    │   → If multiple major components are contradicted (not just one stale line), stop entirely
    │     and ask the user to clarify the current state before writing anything — a partially
    │     wrong doc written confidently is worse than no doc
    ├── Partially complete (good sections exist alongside missing ones)
    │   → Edit: add the missing sections only; preserve what's already correct
    └── Looks complete and accurate
        → Tell the user what exists and ask what they want changed; don't regenerate unprompted
```

**Inline docs (docstrings, JSDoc, etc.)**: use them as source material, not copy-paste targets. Paraphrase into the doc's voice rather than quoting verbatim. If a docstring already explains the *why*, that counts as knowledge — don't ask the user for something already documented inline.

### Step 3.5 — Ask before writing (when gaps are high-value)

After exploring, identify any gaps where the *why* behind a decision is both:
- **invisible from the code** (cannot be inferred from naming, structure, comments, or commit messages), and
- **worth explaining** (it would change how a reader understands or extends the system)

If you find any such gaps, **ask before writing** — not after. Surface at most **3** — the highest-value ones only. Note the rest as inline gaps in the doc rather than questions. List them all at once in a single message so the user can answer in one reply. **Exception for `adr`**: ask one question per surprising decision (where a reasonable engineer would have defaulted to something different), up to **5** across all ADRs in a session. Format clearly:

> Before I write the docs, a few things I couldn't determine from the code:
>
> 1. Why PostgreSQL over a document store? The schema is highly relational but there's no migration comment explaining the choice.
> 2. The `legacy/` directory — is it still in use or safe to omit from docs?
> 3. Auth uses both JWT and session cookies. Is one preferred for new endpoints?

If the user says "just document what you can see" or similar, skip the questions and proceed — noting the gaps inline in the doc instead.

**When NOT to ask:**
- Implementation details with no architectural consequence (why a loop is written a certain way, why a constant is a specific value)
- Anything you can state as a fact without context ("uses PostgreSQL" not "chose PostgreSQL because")
- Things already explained in comments, commit messages, or existing docs
- Gaps in existing docs where the code is the clear source of truth (missing setup step you can derive from `package.json`)

**Per doc type, the highest-value gaps to surface:**

| Doc type     | Ask about                                                                           |
|--------------|-------------------------------------------------------------------------------------|
| `arch`       | Key technology choices, why components are split this way                           |
| `readme`     | Target audience, non-obvious prerequisites                                          |
| `api`        | Auth flows not evident from middleware, deprecation plans                           |
| `onboard`    | Team conventions not encoded in tooling (PR process, etc.)                          |
| `deployment` | Non-obvious infra dependencies, secrets management approach                         |
| `adr`        | Rationale behind surprising choices (see adr-guide.md for what counts as surprising)|
| `update`     | Not applicable — scope is determined by `git diff`, not architectural ambiguity     |

### Step 4 — Write the documentation

Use `Write` to create new files, `Edit` to update specific sections in existing ones. Never overwrite an existing file wholesale if it has custom content.

Default output locations:
- `README.md` → repo root
- `ARCHITECTURE.md` → repo root
- `ONBOARDING.md` → repo root
- `DEPLOYMENT.md` → repo root
- `docs/api.md` → create `docs/` if it doesn't exist
- `docs/adr/NNN-title.md` → create `docs/adr/` if it doesn't exist; one file per decision
- `update` → edits existing files in place; does not create new doc files

#### Custom output directory

If the user specifies an output directory (e.g. "put everything in docs/", "write to wiki/", "use the docs folder"), use that as the root for all output paths instead of the defaults above.

Example: user says "put it all in docs/" → write `docs/README.md`, `docs/ARCHITECTURE.md`, `docs/ONBOARDING.md`, `docs/DEPLOYMENT.md`, `docs/api.md`, `docs/adr/NNN-title.md`.

If a subcommand has its own subdirectory convention (e.g. `api` → `api.md`, `adr` → `adr/NNN-title.md`), preserve that relative structure under the user-specified root.

#### GitHub Wiki format

If the user says "for our GitHub wiki", "wiki format", or "write to the wiki":

- Use `[[Page Name]]` for internal cross-references instead of `[text](./file.md)`
- Omit YAML front matter (GitHub Wiki ignores it)
- Use flat filenames — GitHub Wiki is a flat namespace with no subdirectories:
  - `ADR-NNN-Title.md` instead of `docs/adr/NNN-title.md`
  - `API-Reference.md` instead of `docs/api.md`
- Capitalise filenames to match GitHub Wiki's page title convention (`Architecture.md`, `Onboarding.md`, etc.)

Content, sections, and Mermaid diagrams are otherwise identical — GitHub Wiki renders standard Markdown.

### Step 5 — Confirm and offer to expand

After writing, tell the user what was generated. Offer to:
- Expand any section with more detail
- Add a Mermaid diagram (especially useful for `arch`)
- Add working code examples (for `api` and `onboard`)
- Generate a second doc type

---

## Output quality rules (apply to all doc types)

- Write for a reader who is unfamiliar with the codebase
- Prefer concrete examples over abstract descriptions
- Never fabricate function signatures, endpoints, or behaviors — read the source
- If something is unclear in the code, say so in the doc rather than guessing
- Use Mermaid (`graph TD`, `sequenceDiagram`) for non-trivial architecture — keep diagrams under 15 nodes
- Use fenced code blocks with language identifiers for all code samples
- Keep sentences short; use headers to aid navigation
