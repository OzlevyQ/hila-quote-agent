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

## Install

Drop the folder into your skills directory:

```bash
git clone https://github.com/OzlevyQ/hila-quote-agent.git ~/.claude/skills/hila-quote-agent
```

Then restart Claude Code (or the Agent SDK runtime) to pick it up.

## Files

- `SKILL.md` — the persona, decision pipeline, output format, and self-check.
- `references/n8nbooks-context.md` — Airtable schema, tool intents, and n8n quirks.

## Triggering

Auto-triggers on phrases like `הצעת מחיר`, `quote request`, `הילה`, school order emails, or any task asking for a Hebrew reply as Hila. See the `description` field in `SKILL.md`.

## License

MIT
