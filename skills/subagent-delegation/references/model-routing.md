# 모델 라우팅

## 목차

- [먼저 확인할 것](#먼저-확인할-것)
- [사용 가능한 모델](#사용-가능한-모델)
- [배치 원칙](#배치-원칙)
- [난도별 배치 T0~T5](#난도별-배치-t0t5)
- [교차 리뷰 배치](#교차-리뷰-배치)
- [승격 규칙](#승격-규칙)
- [강등 규칙](#강등-규칙)
- [대체 순서](#대체-순서)

## 먼저 확인할 것

**모델 가용성은 세션마다 다를 수 있다.** 이 문서의 목록을 무조건 신뢰하지 말고, `spawn_agent` 도구 설명에 나열된 available model overrides가 있으면 그것을 우선한다.

`model`과 `reasoning_effort`는 `fork_turns`가 `"none"` 또는 정수 문자열(`"3"` 등)일 때만 적용된다. 전체 히스토리 포크는 override를 거부하고 부모 설정을 상속한다. **override가 무시된 것 같으면 가장 먼저 `fork_turns`를 확인한다.**

슬러그 유효성이 의심되면 일부러 틀린 `reasoning_effort`(예: `"supreme"`)를 넣어본다. 런타임이 해당 모델의 지원 목록을 반환하면 **슬러그는 유효하다**. 잘못된 슬러그는 `Unknown model`을 낸다. 이 두 에러를 구분하면 확실히 판정된다.

모델의 자기 신고("나는 Sonnet이다")는 근거로 쓰지 않는다. 모델은 자기 정체를 자주 틀리게 답한다.

## 사용 가능한 모델

| 슬러그 | 지원 reasoning_effort | 회사 | 성격 |
|---|---|---|---|
| `gpt-5.6-luna` | low / medium / high / xhigh / max — **ultra 없음** | OpenAI | 빠르고 싸다. 기본 Scout이자 가벼운 Worker |
| `gpt-5.6-terra` | low / medium / high / xhigh / max / ultra | OpenAI | 균형형. 지속적 추론이 필요한 구현 |
| `anthropic/claude-sonnet-5` | low / medium / high / xhigh / max / ultra | Anthropic | 코드 관례 준수·지시 이행에 강함 |
| `gpt-5.6-sol` | low / medium / high / xhigh / max / ultra | OpenAI | 최상위. 판정·고위험 전용 |
| `anthropic/claude-opus-5` | low / medium / high / xhigh / max / ultra | Anthropic | 최상위. 판정·고위험 전용 |

`gpt-5.6-luna`에 `ultra`를 주면 spawn이 거부된다.

Haiku·Gemini Flash·오픈웨이트(DeepSeek, GLM, Kimi)는 **현재 서브에이전트로 지정할 수 없다.** 이 계층의 저가 작업은 `gpt-5.6-luna low`가 대신한다. 나중에 추가되면 T0·T1의 primary만 교체하면 된다.

## 배치 원칙

1. **역할과 모델을 1:1로 묶지 않는다.** "Explorer = 무조건 luna"가 아니라 조사 난도에 따라 luna low / luna medium / terra medium으로 나뉜다.
2. **모델명은 이 문서에만 쓴다.** 역할 프롬프트에 모델명을 박아두면 교체 비용이 커진다.
3. **위에서 시작하지 않고 아래에서 올라간다.** 처음부터 최고 모델을 쓰지 않되, 싼 모델을 무한 재시도하지도 않는다. 재시도가 승격보다 비싸지는 지점이 있다.
4. **리뷰어는 작성자와 다른 회사.** 맹점이 덜 겹친다는 것이 유일한 근거이므로, 익명화와 함께 걸어야 의미가 있다.
5. **`max`·`ultra`는 기본 배치에 쓰지 않는다.** 비용과 지연이 급증한다. D3 고위험 판정에서만 고려한다.

## 난도별 배치 T0~T5

### T0 — Mechanical

파일·함수 위치 찾기, 검색 결과에서 제목·날짜·URL 추출, 한 문서 짧은 요약, 로그에서 오류 줄 추출, 형식 변환, 중복 제거, 템플릿 채우기.

```yaml
primary:
  model: gpt-5.6-luna
  reasoning: low
```

T0에는 sonnet·terra·sol·opus를 쓰지 않는다. 추론이 거의 필요 없고, 요구되는 것은 반환 형식 준수뿐이다.

### T1 — Bounded Research

공식 문서 2~5개 비교, 특정 코드 호출 경로 추적, 검색 결과를 근거와 함께 정리, 기존 구현 관례 찾기, 긴 문서에서 관련 부분 선별.

```yaml
primary:
  model: gpt-5.6-luna
  reasoning: medium
fallback:
  model: gpt-5.6-terra
  reasoning: low
```

T2로 승격하는 조건: 출처가 서로 충돌함 / 코드와 외부 문서를 함께 해석해야 함 / 조사 결과가 아키텍처 결정을 바꿈 / 첫 모델이 핵심 근거를 못 찾음.

### T2 — Synthesis and Light Implementation

여러 문서·코드 결과 합성, 모호한 버그의 원인 후보 분석, 구현 계획 작성, 단일 파일 범위의 코드 수정, 기존 패턴을 따르는 테스트 추가.

```yaml
primary:
  model: gpt-5.6-luna
  reasoning: high
alternatives:
  - model: gpt-5.6-terra
    reasoning: medium
  - model: anthropic/claude-sonnet-5
    reasoning: medium
```

luna high를 첫 번째 표준 Worker로 쓴다. 여러 파일을 일관되게 고쳐야 하거나, luna가 범위를 반복해서 벗어나거나, API·상태·데이터 계약을 바꾸면 terra/sonnet으로 올린다.

### T3 — Standard Implementation

여러 파일을 수정하는 기능 구현, 리팩터링, 복잡한 버그 수정, 테스트와 구현 동시 변경, 장시간 도구 사용.

```yaml
convention_heavy:          # 기존 관례를 촘촘히 따라야 하는 경우
  model: anthropic/claude-sonnet-5
  reasoning: medium

sustained_reasoning:       # 긴 추론 사슬이 필요한 경우
  model: gpt-5.6-terra
  reasoning: medium

fast_bounded:              # 범위가 명확하고 빨라야 하는 경우
  model: gpt-5.6-luna
  reasoning: high
```

terra를 모든 구현의 기본값으로 두지 않는다. luna high가 실패한 이력이 있거나 도구 사용이 불안정할 때 선택한다.

### T4 — Adversarial Review

[교차 리뷰 배치](#교차-리뷰-배치) 참조.

### T5 — Arbitration

Executor와 Reviewer의 사실 판단이 충돌 / 두 Reviewer의 결론이 다름 / 보안·데이터 손실·배포 위험이 큼 / Reviewer 지적이 실제 결함인지 판정하기 어려움.

```yaml
primary:
  model: gpt-5.6-sol
  reasoning: xhigh
alternative:
  model: anthropic/claude-opus-5
  reasoning: high
```

Arbiter는 **메인 에이전트와 다른 회사** 모델로 고른다. 메인이 Anthropic 계열이면 `gpt-5.6-sol`, OpenAI 계열이면 `anthropic/claude-opus-5`다.

Arbiter는 항상 부르지 않는다. 실제 충돌이 발생했을 때만 부른다.

## 교차 리뷰 배치

Reviewer는 Executor와 **다른 회사** 모델을 쓴다.

| Executor | 최종 Reviewer |
|---|---|
| `gpt-5.6-luna` | `anthropic/claude-sonnet-5` high |
| `gpt-5.6-terra` | `anthropic/claude-sonnet-5` high |
| `anthropic/claude-sonnet-5` | `gpt-5.6-terra` high |
| 메인이 직접 구현 | 메인과 다른 회사, high |

저위험 변경이면 `gpt-5.6-luna high`로 가벼운 리뷰만 돌려도 된다. 다만 다음에는 luna 리뷰로 끝내지 않는다.

```
보안·인증·권한 검토
데이터 삭제·마이그레이션
공개 API 계약 변경
동시성 문제
대규모 아키텍처 변경
```

이 경우 sonnet high 또는 terra high 이상을 쓴다.

## 승격 규칙

승격은 다음 다섯 가지에만 허용한다.

```
1. 필수 근거를 찾지 못함
2. 반환 형식을 반복해서 위반함
3. 도구 호출을 잘못 사용함
4. 충돌하는 정보를 판정하지 못함
5. 검증 실패의 원인을 설명하지 못함
```

동일 모델·동일 추론 강도 재시도는 **최대 1회**다. 그 다음 순서로 올린다.

```
low → medium → high → xhigh
luna → terra 또는 sonnet → sol 또는 opus
```

"답이 마음에 들지 않는다"는 승격 사유가 아니다. **실패 유형을 기록한 뒤** 올린다.

## 강등 규칙

아래를 모두 만족하면 다음 작업부터 한 단계 싼 모델을 시험한다.

```
- 최근 동일 유형 작업 5건 이상 성공
- 반환 형식 위반 없음
- Reviewer blocker 없음
- 자동 검증 통과
- 메인의 수정 개입 없음
```

강등은 한 번에 한 단계만, 한 축(모델 또는 추론 강도)만 내린다. 둘을 동시에 내리면 실패 원인을 분리할 수 없다.

## 대체 순서

지정 모델이 거부될 때:

```
gpt-5.6-luna              →  gpt-5.6-terra              →  override 생략(부모 상속)
gpt-5.6-terra             →  gpt-5.6-sol                →  override 생략
gpt-5.6-sol               →  gpt-5.6-terra              →  override 생략
anthropic/claude-sonnet-5 →  anthropic/claude-opus-5    →  override 생략
anthropic/claude-opus-5   →  anthropic/claude-sonnet-5  →  override 생략
```

대체 시 **회사를 유지**한다. 교차 회사 배치가 리뷰 독립성의 근거이므로, GPT 자리를 Anthropic으로 메우면 교차 구조가 무너진다.

`ultra`를 쓰던 자리를 `gpt-5.6-luna`로 대체하면 안 된다. luna는 `ultra`를 지원하지 않아 spawn이 거부된다. effort를 `max`로 낮추거나 다른 모델을 쓴다.

Executor와 Reviewer가 같은 회사로 수렴하면, 같은 계열로 형식적 교차 리뷰를 흉내내지 말고 **메인이 직접 2차 검토**를 맡는다. 이 사실은 최종 보고에 명시한다.
