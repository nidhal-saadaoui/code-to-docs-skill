# Update Guide

## Goal

Update only the docs that are affected by changes on the current feature branch. Scope the work using `git diff` — do not re-read the entire codebase or regenerate docs that haven't changed.

## Step 1 — Identify what changed

```bash
git diff main...HEAD --name-only
```

If that fails (no `main` branch, or not on a feature branch), try in order:

```bash
git diff master...HEAD --name-only   # some repos use master
git diff HEAD~1 --name-only          # fallback: only the last commit
```

If there are uncommitted changes the user explicitly wants captured:

```bash
git diff --name-only          # unstaged
git diff --staged --name-only # staged
```

> If the diff returns more than 30 changed files, stop and warn the user: "Too many files changed for a targeted update — consider running the relevant subcommands directly (e.g. `/code-to-docs api`)." Don't attempt to diff-scope a full codebase rewrite.

## Step 2 — Map changed files to affected doc types

Apply these heuristics to the list of changed files:

| Changed file pattern | Doc to update |
|---|---|
| Route handlers (`routes/`, `*router*`, `*controller*`, `urls.py`, `handlers/`) | `docs/api.md` |
| Public API exports (`index.ts`, `lib/`, `src/`, `pub fn`, `public class`) | `docs/api.md` |
| Config / env loaders (`config.py`, `config.ts`, `cmd/root.go`, `.env.example`) | `DEPLOYMENT.md` env vars table |
| Dependency manifests (`requirements.txt`, `package.json`, `go.mod`, `Cargo.toml`, `pom.xml`) | `ONBOARDING.md` prerequisites |
| `docker-compose.yml`, `Dockerfile` | `DEPLOYMENT.md` deploy / build section |
| New top-level directories or major modules | `ARCHITECTURE.md` component descriptions |
| CI workflow changes (`.github/workflows/`) | `ONBOARDING.md` running tests section |

**README**: update only if a user-facing behavior changed — a new CLI command, a changed quick-start step, a new required env var in the setup flow. Do not update README for internal refactors, renamed variables, or implementation changes invisible to end users.

A changed file can affect more than one doc. Apply every matching row.

## Step 3 — Read and update each affected doc

For each doc identified in Step 2:

1. Read the current doc file
2. Read the changed source files relevant to that doc (from the `git diff main...HEAD -- <path>` output for those specific files)
3. Use `Edit` to update only the affected sections — do not rewrite sections that are still accurate

Be surgical: add the new endpoint, add the new env var row, update the command — nothing more.

## Step 4 — Handle missing docs

If a doc that should be updated doesn't exist yet, do not generate it as part of this update pass. Instead tell the user:

> `docs/api.md` doesn't exist yet. Run `/code-to-docs api` to generate it first, then re-run `update` to keep it in sync.

## Step 5 — Report what changed

After all edits, report specifically what was updated. Be concrete:

> Updated `docs/api.md`: added `POST /users/reset-password`.
> Updated `DEPLOYMENT.md`: added `EMAIL_SERVICE_URL` to env vars table.
> `README.md` — no user-facing changes detected, left untouched.

Vague summaries ("updated the docs") are not useful. Name the section and the change.
