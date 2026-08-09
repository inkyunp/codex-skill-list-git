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

## Codex에 스킬 설치하기

> 현재 Codex의 로컬 스킬 탐색 경로는 `.codex/skills`가 아니라
> `.agents/skills`입니다. `.codex`는 주로 Codex 설정을 위한 디렉터리이므로,
> 그 아래에 스킬을 복사해도 자동으로 발견된다고 보장할 수 없습니다.

### 모든 프로젝트에서 쓰기

```bash
git clone https://github.com/inkyunp/codex-skill-list-git.git ~/src/codex-skill-list-git
mkdir -p ~/.agents/skills
cp -a ~/src/codex-skill-list-git/skills/. ~/.agents/skills/
```

### 특정 프로젝트에서만 쓰기

프로젝트 루트에서 아래를 실행합니다. Git으로 관리되는 팀 프로젝트에는 이
방법이 적합합니다.

```bash
mkdir -p .agents/skills
cp -a /path/to/codex-skill-list-git/skills/. .agents/skills/
```

각 스킬은 `SKILL.md`가 있는 디렉터리 하나로 설치됩니다. Codex는 변경을
자동 감지하며 목록에 바로 보이지 않으면 Codex를 다시 시작합니다. CLI/IDE에서는
`$skill-name`으로 명시 호출하거나, 요청이 스킬의 `description`과 맞으면 자동으로
선택됩니다.
