# User Guide Documentation Guide

## Goal

A user-facing guide that explains what the system does and how to use it — written for the people who operate or consume the product, not the developers who build it.

This is the only subcommand where Step 3.5 questions are **required before writing anything**. The code shows what exists; only the user knows who reads this guide and what they are trying to accomplish.

## Output location

`docs/user-guide.md` — create `docs/` if it doesn't exist.

## Step 3.5 — always ask these before writing

Before reading source files, ask:

> Before I write the user guide, I need to understand the audience:
>
> 1. Who is this guide for? (e.g., end users of the web app, administrators, API consumers, data analysts)
> 2. What is the primary workflow a new user follows to get value from the product?
> 3. Are there multiple roles with different capabilities? (e.g., admin vs standard user)

If the user says "just document what you can see", proceed — but note at the top of the output that the audience and primary workflow were assumed from the code.

## Detect the product type

The product type determines the guide structure:

| Signal | Product type | Guide structure |
|---|---|---|
| Frontend components (`pages/`, `views/`, `components/`) | Web application | One section per major screen or feature |
| `[[bin]]` in Cargo.toml, `[project.scripts]`, `bin/` directory | CLI tool | One section per command or workflow |
| Route handlers only, no frontend | API / integration guide | One section per major use case with request examples |
| `roles`, `permissions`, `admin/` routes | Multi-role application | One section per role, each with its own workflow |
| Notebook-first (`notebooks/`, `.ipynb`) | Data / analytical tool | One section per analysis workflow |
| Airflow DAGs, scheduled jobs | Operations tool | One section per job or pipeline a user triggers |

## Web application structure

```markdown
# [Product Name] User Guide

## Getting started

Brief description of what the product does and who it is for.

## [Feature / Screen name]

What this section of the product does and when a user would use it.

### How to [primary action]

Step-by-step instructions. Use numbered lists for sequential steps.

1. Navigate to **Settings → Integrations**.
2. Click **Add connection**.
3. Enter your API key and click **Save**.

The connection status will change to **Active** within 30 seconds.

### [Secondary action]

...
```

## CLI tool structure

```markdown
# [Tool name] User Guide

## Overview

What the tool does in one sentence.

## Installation

\`\`\`bash
pip install tool-name
\`\`\`

## Commands

### tool-name run

[description of what it does]

\`\`\`bash
tool-name run --input data.csv --output report.json
\`\`\`

**Options**

| Flag | Default | Description |
|---|---|---|
| `--input` | — | Path to input file (required) |
| `--output` | `stdout` | Path to output file |
| `--verbose` | false | Print detailed progress |
```

## API / integration guide structure

```markdown
# [API name] Integration Guide

## Overview

What the API does and who should use this guide (developers integrating with the service).

## Authentication

How to obtain and use credentials.

## Common workflows

### [Workflow name] (e.g., "Create and track an order")

Narrative description + sequence of API calls with real example requests and responses.

\`\`\`bash
# Step 1 — create an order
curl -X POST https://api.example.com/orders \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"customer_id": "cust_123", "items": [...]}'
\`\`\`
```

## Writing style rules

- Write in plain language — no jargon, no code unless it is a CLI/API guide
- Use active voice: "Click Save" not "The Save button should be clicked"
- Use **bold** for UI element names (button labels, menu items, field names)
- Use `monospace` for values, file paths, and commands
- Describe what happens after an action: "The dashboard refreshes and shows your new data."
- Do not explain why the system works the way it does — that belongs in ARCHITECTURE.md

## Screenshot placeholders

If the guide would benefit from screenshots (web app, dashboard), insert placeholders rather than leaving the section without visual guidance:

```markdown
<!-- screenshot: the main dashboard showing the health score panel -->
```

This signals to the team where screenshots should be added without blocking the written content.

## Multi-role applications

If the product has distinct user roles, structure the guide with a top-level section per role:

```markdown
## For administrators

### Managing users
...

## For analysts

### Running a report
...
```

Detect roles from: route-level permission middleware, `Role` enums, `admin/` route prefixes, or permission constants.

## What NOT to include

- Implementation details, database names, class names
- Deployment or infrastructure information (that belongs in DEPLOYMENT.md)
- Code snippets unless the product is a CLI or API
- Internal URLs, staging environment details
- Anything the user's audience would never need to know
