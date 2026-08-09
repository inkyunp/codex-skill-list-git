# Personal Codex skills

This public repository tracks selected editable personal skill sources copied
from `/mnt/c/Users/inkyunp/.codex/skills` on 2026-08-09.

Tracked skills:

- `api-security`
- `browser-automation`
- `humanize-korean`
- `planning-council`
- `subagent-delegation`

Excluded intentionally:

- `.system/` bundled skills, because they are platform-managed distribution files.
- Plugin skill caches, because they are recreated by plugin installation.
- `10x-web-summary-download` and `lims-order-note`, because they are LIMS-MCP-related.
- Codex configuration, authentication, cookies, sessions, logs, and all generated caches.

To refresh a tracked skill after editing its live source, copy only that skill
directory into `skills/<name>/`, preserving the exclusions in `.gitignore`.
