# knowledge/

Expert knowledge cards consumed by the agents. See `../KNOWLEDGE.md` for the design.

- `cards/<area>/*.md` — one card per file, YAML header + markdown body
- `card.schema.yaml` — header validation
- `conflicts.yaml` — contradicting pairs and the rule that resolves them

Add knowledge with `dsx learn add <url|file|->` (drafts a card for your review) or by writing a card by hand and running `dsx knowledge build`.
