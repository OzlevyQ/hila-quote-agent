# n8nbooks operational context

Reference notes for the Hila skill. Read this only when you need schema details, tool semantics, or pipeline quirks. The main behavior is in `SKILL.md`.

## Repo

`/Users/mymac/code/n8nbooks` — operations home for a single n8n workflow: **Quote Request → Airtable** (`Q8CQ8nFyMMLLbLoV`). Gmail trigger → parse line items → Airtable lookup → write `requests` rows → reply with quote.

Source data files at repo root:
- `catalog.csv` / `ספרי לימוד ...csv` → feeds `catalog`.
- `schools.xls` / `רשימת בתי ספר ...xls` → feeds `schools`.

## Airtable base `appu3b1o75q2NVAWB`

| Table | ID | Fields | Mode |
|---|---|---|---|
| `catalog` | `tblSaRPAGwg8J3P4B` | `Sku#`, `mfrSku·`, `title·`, `group`, `price#` | read-only |
| `schools` | `tblbyxYpswsaIC9Sw` | `customerCode#`, `name·`, `requests↗` | read-only |
| `requests` | `tblpyAUMx0Yrw6Krf` | `messageId, from, subject, receivedAt, schoolName, school↗, book↗, title, isbn#, sku#, quantity, unitPrice, status, rawLine, quoteSentAt, quoteNumber` | write target |

Search results are `{id, fields: {...}}` — read `.fields.X`.

The workspace is on Free/trial — base-scoped REST (`/v0/meta/bases/{id}/*`, `/v0/{id}/{table}`) returns 422. Use the n8n Airtable node for data ops; do schema edits in the UI.

## Tool intents (logical)

The Hila persona reasons in terms of these intents — actual tool names may vary by environment:

- `lookup_school_by_code(code)` — exact lookup on `customerCode#`.
- `search_school_by_name(query)` — fuzzy on `name·`. Skip generic words ("בית ספר", "תיכון", "יסודי").
- `lookup_book_by_id(id)` — exact lookup on `Sku#` or `mfrSku·`.
- `search_book_by_title(query)` — fuzzy on `title·`.
- MCP `search_records` (Airtable MCP) — fallback only when local cache is empty across all attempts. Precede with `list_tables_for_base` to confirm field names. **Never** call `list_records_for_table` with `recordIds`.

## Identifier policy

`Sku# / mfrSku· / דאנאקוד` are the real identifiers. ISBN exists as a column on `requests` for legacy reasons but is **not** a reliable lookup key in `catalog`. Don't lead with ISBN.

## Known pipeline bugs (already fixed — do not reintroduce)

- Gmail Trigger uses `simple: true`.
- Build Record reads `catalogMatch.fields.title` / `.fields.price`.
- Build Quote creates the `quote` binary on the `hasIssues` path too.
- Parse SKU regex is `\d{4,12}`.
- Lookup nodes have `alwaysOutputData: true`.
- Send Email is static `operation: "send"` — split reply/send via an If node (Gmail v2.1 doesn't accept expressions on `operation`).

## n8n SDK quirks

- No top-level arrow functions in Code nodes — use `function(){}` or pre-extract to `const`.
- `fromAi(...)` must be passed bare, not wrapped.
- Always `validate_workflow` before `update_workflow`.
- `update_workflow` wipes credential bindings on these 6 nodes: Gmail Trigger, Send Email, Lookup Catalog, Lookup School, Create Request, Mark as Quoted. After every `update_workflow`, prompt the user to reconnect.

## Sticky notes

Every workflow edit must add/update `stickyNote` nodes in the same `update_workflow` call. The user reads them as in-canvas context.
