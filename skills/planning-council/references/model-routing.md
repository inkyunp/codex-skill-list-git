# 모델 라우팅

## 목차

- [먼저 확인할 것](#먼저-확인할-것)
- [배치 원칙](#배치-원칙)
- [역할별 배치](#역할별-배치)
- [spawn_agent 호출 형식](#spawn_agent-호출-형식)
- [대체 순서](#대체-순서)

## 먼저 확인할 것

**모델 가용성은 세션·호출 시점에 따라 달라진다.** 카탈로그 파일에 있는 슬러그가 서브에이전트에 그대로 지정 가능하다고 가정하지 않는다.

확인 순서:

1. `spawn_agent` 도구 설명에 나열된 **available model overrides** 목록을 읽는다. 이것이 이번 세션의 실제 진실이다.
2. 목록이 보이지 않으면 `/mnt/c/Users/inkyunp/.codex/opencodex-catalog.json`을 참고하되, 지정 실패에 대비한다.
3. 지정한 모델이 거부되면 아래 대체 순서를 따르고, **실제로 사용한 모델을 보고서에 기록**한다.

카탈로그에는 상위 모델(예: `gpt-5.6-sol`, `anthropic/claude-opus-5`)이 있어도 서브에이전트 override 목록에는 빠져 있을 수 있다. 그 경우 override를 생략해 부모 모델을 상속시키는 편이, 없는 슬러그를 우겨넣어 실패하는 것보다 낫다.

## 검증된 모델 목록 (2026-07-27 실측)

**핵심 주의: 잘못된 슬러그를 넣었을 때 에러가 반환하는 "Available models" 목록은 불완전하다.**

그 에러는 `gpt-5.6-sol`과 `anthropic/claude-opus-5`를 누락한 채 5종만 반환하지만, 두 모델 모두 **실제로는 정상 spawn되고 응답한다**. 에러 메시지의 목록을 근거로 상위 모델을 배제하면 안 된다.

실측으로 확인된 5종과 각 모델의 추론 강도다.

| 슬러그 | 지원 reasoning_effort | 회사 |
|---|---|---|
| `gpt-5.6-sol` | low / medium / high / xhigh / max / ultra | OpenAI |
| `anthropic/claude-opus-5` | low / medium / high / xhigh / max / ultra | Anthropic |
| `anthropic/claude-sonnet-5` | low / medium / high / xhigh / max / ultra | Anthropic |
| `gpt-5.6-terra` | low / medium / high / xhigh / max / ultra | OpenAI |
| `gpt-5.6-luna` | low / medium / high / xhigh / max — **ultra 없음** | OpenAI |

`gpt-5.6-luna`에만 `ultra`가 없다. 여기에 `ultra`를 주면 spawn이 거부된다.

`google-antigravity/gemini-3.6-flash-low`와 `gpt-5.4-mini`는 런타임 에러 목록에 나타나지만 **현재 공인된 5종에는 없다**. 설정 변경 이전의 잔존 항목으로 보인다. 핵심 역할에 쓰지 마라.

### 가용성 확인 방법

슬러그 유효성은 **일부러 틀린 reasoning_effort를 넣어 확인한다**. 예를 들어 `reasoning_effort="supreme"`을 주면 런타임이 해당 모델의 지원 목록을 그대로 반환한다.

```
spawn_agent(model="anthropic/claude-opus-5", reasoning_effort="supreme", ...)
→ "Reasoning effort `supreme` is not supported for model `anthropic/claude-opus-5`.
   Supported reasoning efforts: low, medium, high, xhigh, max, ultra"
```

이 응답은 **모델이 유효하다는 증거**다. 검증이 모델별로 이루어지기 때문이다. 반면 잘못된 *모델* 슬러그는 `Unknown model` 에러를 낸다. 이 두 에러를 구분하면 슬러그 유효성을 확실히 판정할 수 있다.

모델의 자기 신고("나는 Sonnet이다")는 근거로 쓰지 마라. 모델은 자기 정체를 자주 틀리게 답한다. 실제로 `anthropic/claude-opus-5`로 띄운 에이전트가 자신을 Sonnet이라고 답한 사례가 있었지만, 위 방법으로 확인한 결과 슬러그 자체는 유효했다. 정체 확인이 필요하면 자기 신고 대신 위의 effort 검증을 쓴다.

## 배치 원칙

1. **품질선이 다양성보다 먼저다.** 서로 다른 회사 모델을 무조건 섞기보다, 최소 품질선을 넘는 모델로 독립 후보를 여러 개 만드는 편이 낫다. 약한 모델을 "관점이 다르다"는 이유로 넣으면 후보 풀이 오염된다.
2. **핵심 투표권과 기계적 작업을 분리한다.** 경량 모델은 중복 제거·형식 변환·자료 정리에만 쓰고, 생성·심사에는 넣지 않는다.
3. **심사자는 생성자와 다른 회사 모델로.** 자기 문체 선호 편향을 줄이기 위해서다. 다만 이것만으로 공정성이 보장되지는 않으므로 익명화·순서 셔플과 반드시 함께 건다.
4. **통합자와 최종 검증자는 다른 회사 모델로.** 같은 모델이 자기 통합안을 검증하면 검증이 형식화된다.

## 역할별 배치

검증된 5종 기준이다. 메인 에이전트는 `anthropic/claude-opus-5`로 동작하므로, 교차 회사 조건은 **메인(Anthropic) 대비 GPT 계열**을 섞어 만든다.

| 역할 | 모델 | 추론 강도 | 비고 |
|---|---|---|---|
| Brief Architect | 메인 (override 없음) | — | 메인 에이전트가 직접 수행 |
| strategic-ideator | `gpt-5.6-sol` | `xhigh` | 전제 재정의·장기 레버리지 |
| evidence-ideator | `anthropic/claude-opus-5` | `high` | 근거·사례 정합성 |
| operational-ideator | `anthropic/claude-sonnet-5` | `high` | 실행 구조·MVP |
| contrarian-ideator | `gpt-5.6-sol` | `xhigh` | 선택 |
| Clusterer | `gpt-5.6-terra` | `medium` | 기계적 정리, 판단은 메인 소유 |
| judge-A | `anthropic/claude-opus-5` | `high` | 생성자와 교차 |
| judge-B | `gpt-5.6-sol` | `xhigh` | judge-A와 다른 회사 |
| Lead Synthesizer | 메인 (override 없음) | — | 메인 에이전트가 직접 수행 |
| final-verifier | `gpt-5.6-sol` | `xhigh` | 메인(Opus)과 다른 회사 |

`gpt-5.6-luna`는 `ultra`를 지원하지 않으므로 `ultra`를 쓸 자리에 배치하지 마라. 경량 보조 작업에만 쓴다.

`max`·`ultra`는 비용이 급증한다. 기본은 `high`~`xhigh`로 두고, `depth: deep`에서 strategic-ideator와 judge-B에만 한 단계 올리는 것을 고려한다.

## spawn_agent 호출 형식

모델·추론강도 override는 **`fork_turns`가 `"none"` 또는 정수 문자열일 때만** 적용된다. 전체 히스토리 포크는 override를 거부하고 부모 설정을 상속한다.

```
spawn_agent(
  task_name="strategic_ideator",
  fork_turns="none",           # 필수. 맥락 상속 차단 + override 허용
  model="anthropic/claude-sonnet-5",
  reasoning_effort="high",
  message=<브리프 전문 + 역할 프롬프트 전문>
)
```

`fork_turns: "none"`이므로 메시지는 **완전히 자기완결적**이어야 한다. 생성자는 사용자 요청도, 다른 생성자의 존재도, 이 스킬의 구조도 모른다. 브리프 전문을 매번 통째로 넣는다.

생성자 3명은 한 번에 띄우고 `wait_agent`로 수확한다. 동시 실행 슬롯은 보통 4개이므로 메인 포함 3명 병렬이 안전하다. 4명이면 2+2로 나눈다.

## 대체 순서

지정 모델이 거부될 때:

```
gpt-5.6-sol                →  gpt-5.6-terra              →  override 생략(부모 상속)
anthropic/claude-opus-5    →  anthropic/claude-sonnet-5  →  override 생략
anthropic/claude-sonnet-5  →  anthropic/claude-opus-5    →  override 생략
gpt-5.6-terra              →  gpt-5.6-sol                →  override 생략
gpt-5.6-luna               →  gpt-5.6-terra              →  override 생략
```

대체 시 **회사를 유지**한다. 교차 회사 배치가 편향 통제의 근거이므로, GPT 자리를 Anthropic으로 메우면 심사 구조가 무너진다.

`ultra`를 쓰던 자리를 `gpt-5.6-luna`로 대체하면 안 된다. luna는 `ultra`를 지원하지 않아 spawn이 거부된다. 이 경우 effort를 `max`로 낮추거나 다른 모델을 쓴다.

대체가 발생하면 **교차 회사 조건이 깨졌는지** 확인한다. 심사자 둘이 같은 회사 모델로 수렴하면 `judges: 1`로 줄이고 메인이 직접 2차 검토를 맡는 편이, 같은 모델 두 개로 형식적 교차 심사를 흉내내는 것보다 낫다. 이 사실은 보고서에 명시한다.
