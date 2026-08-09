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

## `subagent-delegation` 스킬

여러 파일 조사·구현·독립 검토가 함께 필요한 작업을 역할별 서브에이전트로 나누는
워크플로입니다. 단순 질문, 오탈자, 한 파일 확인처럼 한 번의 검증으로 끝나는 일에는
쓰지 않습니다.

| 등급 | 적용 상황 | 진행 방식 |
| --- | --- | --- |
| D0 | 단순 작업 | 메인 에이전트가 직접 수행 |
| D1 | 읽기 전용 조사 | Explorer를 병렬로 실행하고 메인이 근거를 종합 |
| D2 | 구현 또는 지속 산출물 | Explorer → Executor → 다른 모델의 Reviewer → 필요 시 1회 재검토 |
| D3 | 보안·권한·삭제·마이그레이션·공개 API 등 고위험 작업 | 제한된 Explorer → 계획 확정 → Executor → Reviewer → 충돌 시 Arbiter |

메인 에이전트는 요구사항 해석, 작업 분해, 충돌 판정, 최종 결과를 계속 소유합니다.
각 위임에는 `OBJECTIVE`, `SCOPE`, `WRITE_PERMISSION`, `DONE_WHEN`,
`RETURN_FORMAT`을 명시해 범위를 통제합니다. Explorer와 Reviewer는 읽기 전용,
Executor만 지정된 경로를 수정하며 같은 파일을 동시에 수정하지 않습니다. 자동 테스트와
정적 검증을 먼저 실행한 뒤, 작성자와 다른 모델의 Reviewer가 요구사항·최종 diff·검증
결과만 바탕으로 검토합니다.

사용 예시:

```text
$subagent-delegation 이 변경을 조사·구현·교차 리뷰로 나눠 처리해줘.
```
