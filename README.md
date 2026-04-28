# hila-quote-agent

A Claude Code / Claude Agent SDK skill for **Hila (הילה)** — a Hebrew-speaking customer service persona that handles textbook quote-request emails from school secretaries.

Built for the [n8nbooks](https://github.com/) pipeline: Gmail → Airtable catalog lookup → quote reply.

## What it does

Given an incoming email (with optional attachments), the skill drives the agent to:

1. **Classify** the email — quote request, continuation, or noise (no-reply, internal, supplier, ack-only Re).
2. **Read attachments deeply** when accessible (XLSX/PDF book lists across multiple sheets/pages).
3. **Identify the school** by `customerCode#` or fuzzy signature match.
4. **Identify each book** via `Sku# / mfrSku / דאנאקוד` (never ISBN as primary), with deep title-search fallbacks before giving up.
5. **Draft a Hebrew reply** in Hila's voice — short, warm, no tables, no emojis, signed `הילה`.
6. **Mark uncertain matches** with the line `מבקשת לוודא שזה המק״ט הרלוונטי.` instead of fabricating.
7. **Refuse to invent** prices, SKUs, titles, school names, or quantities.

## Raw content URLs (for n8n / HTTP fetch)

Pull these as plain text via any HTTP client:

| File | Raw URL |
|---|---|
| `SKILL.md` (the system prompt) | https://raw.githubusercontent.com/OzlevyQ/hila-quote-agent/main/SKILL.md |
| `references/n8nbooks-context.md` (Airtable schema + tools) | https://raw.githubusercontent.com/OzlevyQ/hila-quote-agent/main/references/n8nbooks-context.md |
| `README.md` (this file) | https://raw.githubusercontent.com/OzlevyQ/hila-quote-agent/main/README.md |

For stable pinning, replace `main` with a commit SHA from
`https://api.github.com/repos/OzlevyQ/hila-quote-agent/commits/main`.

## n8n integration

### HTTP Request node — `Fetch Hila Skill`

| Field | Value |
|---|---|
| Method | `GET` |
| URL | `https://raw.githubusercontent.com/OzlevyQ/hila-quote-agent/main/SKILL.md` |
| Authentication | `None` |
| Send Query / Headers / Body | Off |
| Timeout (Options) | `10000` |

The response body lands in `$json.data` as a plain markdown string.

### Wire to an AI Agent node

- **System Message:**
  `={{ $('Fetch Hila Skill').item.json.data }}`
- **User Message** (Gmail Trigger example):
  ```
  =נושא: {{ $('Gmail Trigger').item.json.subject }}
  מאת: {{ $('Gmail Trigger').item.json.from }}

  {{ $('Gmail Trigger').item.json.text }}
  ```

To avoid refetching on every run, run the HTTP Request once, copy the output into a `Set` node as `systemPrompt`, and refresh on a schedule.

## Install as a Claude Code skill

```bash
git clone https://github.com/OzlevyQ/hila-quote-agent.git ~/.claude/skills/hila-quote-agent
```

Restart Claude Code (or the Agent SDK runtime) to pick it up. The skill auto-triggers on phrases like `הצעת מחיר`, `quote request`, `הילה`, school order emails, or any task asking for a Hebrew reply as Hila.

## Files

- [`SKILL.md`](./SKILL.md) — the persona, decision pipeline, output format, and self-check.
- [`references/n8nbooks-context.md`](./references/n8nbooks-context.md) — Airtable schema, tool intents, and n8n quirks.

## License

MIT
