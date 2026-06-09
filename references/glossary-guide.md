# Glossary Documentation Guide

## Goal

A `GLOSSARY.md` that defines the domain-specific terms used in the codebase — written for anyone joining the project (developer, analyst, or stakeholder) who needs to understand the business language before reading the code or documentation.

Do not define general programming terms (API, HTTP, JSON) or framework names. Only document terms with domain-specific meaning in this project.

## Output location

`GLOSSARY.md` — repo root.

## Detecting domain terms

Read these signals to find terms worth defining:

| Signal | What to extract |
|---|---|
| Model / entity class names (`Order`, `Invoice`, `Shipment`) | Core domain entities |
| Enum values with business meaning (`OrderStatus.PENDING`, `Role.ADMIN`) | State values and their transitions |
| Constants with business names (`MAX_RETRY_ATTEMPTS`, `GRACE_PERIOD_DAYS`) | Business rules encoded as values |
| Comments that explain a concept (`# A fulfillment is distinct from an order`) | Author-defined definitions — use these directly |
| Field names that aren't self-evident (`ltv`, `churn_score`, `cac`) | Abbreviations needing expansion |
| Multiple names for the same concept (`customer` vs `client` vs `user` in different files) | Synonyms — document the canonical term |
| State machine transitions in code or config | Entity lifecycle |

Read source files breadth-first: models/entities first, then service layer, then API layer. Domain language concentrates at the model layer.

## Format

Alphabetical, grouped by letter. Each entry:

```markdown
# Glossary

## C

### Customer
An individual or organisation that has placed at least one order. Distinct from a **Lead** (who has not yet purchased) and a **Contact** (a person associated with a customer account).

See also: [Lead](#lead), [Contact](#contact)

---

### Churn Score
A value between 0 and 1 representing the predicted probability that a customer will not renew within the next 90 days. Computed nightly by the retention model. A score above `0.7` triggers an automatic outreach task.

---

## O

### Order
A confirmed purchase request from a customer. An order transitions through the following states:

`draft → pending → confirmed → processing → shipped → delivered`

Cancelled orders move to `cancelled` from any state except `delivered`.

---

### Order Line
A single product SKU within an order, with quantity and agreed unit price at the time of purchase. Changing a line after confirmation requires a new order revision.
```

## State machines

If an entity has a defined set of states (from an enum, a `status` field, or an explicit state machine library), always document the transitions:

```markdown
**States**: `draft` → `pending` → `confirmed` → `processing` → `shipped` → `delivered`

Terminal states: `delivered`, `cancelled`

`cancelled` is reachable from any non-terminal state.
```

Read the transition logic from the code (validators, service methods, state machine config) — do not invent transitions.

## Synonyms and canonical terms

If the codebase uses multiple names for the same concept (e.g., `user` in the auth module vs `customer` in the billing module), document both and state which is canonical:

```markdown
### User
See **Customer**. The term `user` appears in authentication code; `customer` is the canonical business term used everywhere else.
```

## Step 3.5 questions for glossary

Ask before writing when:
- A term is abbreviated or abbreviated differently in different files (`ltv` vs `lifetime_value`)
- Two different concepts share the same name in different modules
- A business rule encoded as a constant needs context to define (e.g., why is `GRACE_PERIOD_DAYS = 14`?)

Cap at 3 questions — the rest can be noted as `[definition unclear — fill in]` placeholders.

## What NOT to include

- Framework terms: `middleware`, `controller`, `migration`, `fixture`
- Generic technical terms: `API`, `REST`, `JSON`, `webhook`
- Obvious field names: `created_at`, `updated_at`, `id`, `name`
- Internal implementation details with no business meaning: `_cache`, `_raw`, `__computed`
