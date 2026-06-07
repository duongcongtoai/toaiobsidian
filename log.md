# Vault Operations Log

This file is a pointer. Vault operation logs are stored as append-only files per day in `Logs/`.

Format for daily log entries:
```markdown
---
type: log
date: YYYY-MM-DD
ai-first: true
---
**HH:MM** - action | description
```