---
name: goal-setting
description: Designs a complete, paste-ready `/goal` prompt for Claude Code by enforcing the Three Pillars (Feasibility, Demonstrability, Boundedness) and the 4-Element Structure (Goal, Verification, Constraints, Stop Clause). Produces a single-line `/goal ...` string the user can paste directly into Claude Code.
when_to_use: Use when the user wants to build a `/goal` prompt, talks about long-running multi-turn agent work, mentions "goal-setting", "/goal", "완료 조건", "장기 실행", "에이전트 목표 설계", or whenever a user request is too vague to hand to `/goal` directly.
---

# Goal Setting Operating Rules

## 역할

이 스킬은 `/goal`을 **실행하지 않는다**. 사용자가 Claude Code의 `/goal` 명령에 바로 붙여넣을 수 있는 **완성된 한 줄 프롬프트**를 함께 설계한다.

```
사용자의 모호한 요구 → [goal-setting] → 측정·증명·경계·중단이 명시된 /goal 프롬프트
```

`/goal-setting`의 산출물은 항상 `/goal `로 시작하는 단일 코드블록이다.

---

# 기본 원칙

1. 모든 `/goal` 프롬프트는 **4-Element Structure**를 갖춰야 한다.
2. 모든 `/goal` 프롬프트는 **Three Pillars** 검증을 통과해야 한다.
3. 평가자는 도구를 실행하지 않고 파일도 직접 읽지 않는다 — 증거는 **대화에 드러나야** 한다.
4. 중단 절(`or stop after ...`)이 없으면 출력하지 않는다.
5. 측정 불가능한 형용사(`better`, `cleaner`, `perfect`, `fully`)는 거부하고 환원한다.
6. `/goal` 프롬프트는 한 줄 단위의 단일 목표(atomic)로 유지한다. 복합 목표는 분리한다.
7. 사용자 의도 외 작업은 만들지 않는다. 검증 명령은 사용자 프로젝트에 실제 존재하는 것만 제안한다.

---

# Three Pillars (필수 통과 기준)

모든 `/goal` 프롬프트는 아래 세 기둥을 동시에 만족해야 한다.

## 1. Feasibility — 측정 가능한가?

완료 상태가 boolean(yes/no)으로 환원되는가?

| 표현 | 판정 | 환원 방향 |
|------|------|----------|
| `make the app perfect` | ❌ Infeasible | 정의 불가, 거부 |
| `improve the payment module` | ❌ Infeasible | "어떤 명령이 통과하면 개선된 것인가?"로 환원 |
| `all tests in test/auth pass` | ✅ Feasible | exit code로 판정 |
| `each file under 300 lines in src/payment/parts` | ✅ Feasible | wc -l로 판정 |

## 2. Demonstrability — 대화로 증명 가능한가?

평가자는 transcript에 surface된 내용만 본다. 다음을 항상 확인:

- Claude가 명령을 **실제로 실행**하고 **출력을 대화에 남길 수 있는가**?
- 그 출력만 보고 evaluator가 `yes`라고 말할 수 있는가?

| 표현 | 판정 | 환원 방향 |
|------|------|----------|
| `all files are correct` | ❌ Non-demonstrable | 평가자가 파일을 못 봄 |
| `the bug is fixed` | ❌ Non-demonstrable | "고쳤다"를 명령 출력으로 환원 |
| `npm test exits 0 and the result is shown in the conversation` | ✅ Demonstrable | exit code가 대화에 남음 |
| `grep -rn legacyAuth src/ returns 0 matches in the conversation` | ✅ Demonstrable | grep 결과가 대화에 남음 |

핵심 패턴: **`<command>` exits 0 / returns empty / shows [pattern] in the conversation**.

## 3. Boundedness — 중단 조건이 있는가?

모든 `/goal`은 다음 중 하나의 중단 절을 반드시 포함:

- `or stop after N turns`
- `or stop after T minutes`
- 또는 추가 명시적 한도(예: `or stop after 3 failed evaluations`)

권장 기본값:

| 작업 유형 | 권장 N |
|----------|--------|
| 단순 버그 수정 | 10–15 turns |
| 리팩터링 | 15–25 turns |
| 마이그레이션 | 20–30 turns |
| 문서 작업 | 8–12 turns |

사용자가 더 짧게 끝내고 싶다고 하면 그대로 따르되, **0/없음은 허용하지 않는다**.

---

# 4-Element Structure (필수 4요소)

모든 `/goal` 프롬프트는 반드시 다음 네 요소를 포함한다.

| 요소 | 질문 | 절(clause) 형태 |
|------|------|----------------|
| **Goal** | 무엇을 끝내야 하는가? | `[목표 동사 + 구체적 대상]` |
| **Verification** | 무엇으로 완료를 확인하는가? | `prove it by [command(s)] and show [result] in the conversation` |
| **Constraints** | 무엇은 건드리면 안 되는가? | `do not [금지 범위]` |
| **Stop Clause** | 언제까지 안 되면 멈출 것인가? | `or stop after [N turns / T minutes]` |

요소 중 하나라도 비어 있으면 **출력하지 않는다**. 사용자에게 한 번에 하나씩 물어 채운다.

---

# Standard Template

```
/goal [GOAL],
prove it by [VERIFICATION COMMANDS] and show the result in the conversation,
do not [CONSTRAINTS],
or stop after [N turns / T minutes]
```

쉼표로 절을 구분하되, 최종 출력은 **한 줄로 합쳐도 동작**한다. 가독성을 위해 줄바꿈해 보여줄 수 있지만, 사용자에게는 "한 줄로 붙여넣어도 동일하게 동작한다"고 안내한다.

---

# Workflow

## Step 1 — Intake (요구 수집)

사용자가 `/goal-setting`을 호출하면 다음 4개 질문을 명시적으로 던진다. 사용자가 이미 일부를 제공했다면, 누락된 것만 묻는다.

```
1) 무엇이 "완료"인가? (한 문장으로)
2) 완료를 어떤 명령으로 확인할 수 있는가? (없다면 가장 가까운 명령을 함께 찾자)
3) 변경하면 안 되는 파일·디렉터리·범위는?
4) 몇 턴(또는 몇 분) 안에 끝나야 하는가?
```

질문은 한 번에 묶어 던지되, 4번이 비어 있으면 **권장 기본값을 제시**(위 Boundedness 표).

## Step 2 — Three-Pillar Audit (검증)

사용자 답변을 받으면 Three Pillars 각각으로 점검:

- Feasibility: 모호한 형용사가 있는가? → 명령 기반 조건으로 환원 제안
- Demonstrability: 검증 결과가 대화 transcript에 남는가? → 남지 않으면 명령 기반으로 재작성
- Boundedness: stop 절이 있는가? → 없으면 권장값 추가

검증 중 거절이 발생하면, **거절 이유를 1줄로 명시**한 뒤 환원 제안을 제시한다.

## Step 3 — Verification Mapping (검증 명령 매핑)

추상 표현을 아래 카탈로그의 검증 동사로 환원한다.

### 테스트
- `npm test`, `npm test -- <pattern>`
- `pytest`, `pytest tests/<dir>`
- `cargo test`, `go test ./...`

### 타입체크 / 린트
- `npm run typecheck`, `tsc --noEmit`
- `npm run lint`, `eslint .`
- `mypy .`, `ruff check .`

### 빌드
- `npm run build`, `vite build`
- `cargo build --release`

### Scope / 변경 범위
- `git diff --name-only origin/main | grep -v ^src/auth/ returns empty`
- `git status --porcelain shows only files under <path>`

### 검색 기반
- `grep -rn <pattern> src/ returns 0 matches`
- `gh issue list --label bug --milestone current returns 0 items`

### 파일 형태
- `wc -l <file> shows under N`
- `find <dir> -type f -name '*.ts' | wc -l shows N`

검증 명령은 사용자 프로젝트에 실제 존재하는 것만 사용한다. 존재 여부가 불확실하면 **사용자에게 확인**한 뒤 제안한다.

## Step 4 — Anti-Pattern Block

다음 패턴이 출력 직전이라면 자동 차단하고 환원 제안:

| Anti-Pattern | Why Bad | 환원 예시 |
|--------------|---------|----------|
| `make X better` | 측정 불가 | `X passes <command>` |
| `fix all bugs` | 무경계 | `gh issue list --label bug returns empty` |
| `all files are correct` | 평가자가 못 봄 | `npm run typecheck exits 0 shown in conversation` |
| `complete the migration` | 어디까지? | `grep -rn legacyAuth src/ returns 0 matches shown in conversation` |
| 중단 절 없음 | 무한 반복 위험 | `or stop after 20 turns` 추가 |
| 복합 목표 | atomicity 위반 | 두 개의 `/goal`로 분리 |

## Step 5 — Output (출력)

다음 형식을 **반드시** 따른다.

### 5-1. 완성된 `/goal` 프롬프트 (코드블록)

```
/goal [정제된 목표],
prove it by [검증 명령] and show the exit code / output in the conversation,
do not modify [범위 외 파일],
or stop after [N turns]
```

### 5-2. Self-Check 결과

```
Self-Check:
- [x] Feasible — 측정 가능한 boolean 종료 상태
- [x] Demonstrable — 검증 결과가 대화 transcript에 남음
- [x] Bounded scope — 변경 범위 제한
- [x] Stop clause — or stop after N turns 포함
- [x] Atomic — 한 줄 단일 목표
```

각 항목은 실제로 통과한 경우에만 `[x]`로 표시한다. 통과 못 한 항목은 `[ ]`로 두고 **이유와 다음 단계**를 짧게 설명한다.

### 5-3. 사용 안내 1줄

```
이 한 줄을 그대로 Claude Code에 붙여넣어 실행하세요. `/goal`만 입력하면 현재 진행 상태와 평가자 판단 이유를 확인할 수 있습니다.
```

---

# 시나리오 예시

## 시나리오 1 — 버그 수정

**User:** "로그인 버그 고치고 싶어"

**Goal-setting 출력:**

```
/goal fix the login bug described in issue #42, prove it by running npm test -- test/auth and showing exit 0 in the conversation, do not modify files outside src/auth, or stop after 15 turns
```

```
Self-Check:
- [x] Feasible — 테스트 통과 여부로 판정
- [x] Demonstrable — exit code가 대화에 남음
- [x] Bounded scope — src/auth 외 수정 금지
- [x] Stop clause — 15 turns
- [x] Atomic — 단일 목표
```

## 시나리오 2 — 리팩터링

**User:** "payment 모듈 큰 거 쪼개고 싶어"

**Goal-setting 출력:**

```
/goal split src/payment/index.ts into smaller modules under src/payment/parts where each file is under 300 lines, prove it by running npm run typecheck and npm test -- test/payment, both exiting 0 with output shown in the conversation, do not modify code outside src/payment, or stop after 20 turns
```

```
Self-Check:
- [x] Feasible — 파일 크기와 두 명령의 exit 0로 판정
- [x] Demonstrable — 두 명령 출력이 대화에 남음
- [x] Bounded scope — src/payment 외 수정 금지
- [x] Stop clause — 20 turns
- [x] Atomic — 단일 리팩터링 목표
```

## 시나리오 3 — 마이그레이션

**User:** "legacyAuth을 newAuth로 전부 마이그레이션"

**Goal-setting 출력:**

```
/goal migrate all usages of legacyAuth to newAuth in src/auth, prove it by running grep -rn legacyAuth src/ and showing 0 matches AND running npm test exits 0, both results shown in the conversation, do not modify tests/legacy or database migration files, or stop after 25 turns
```

```
Self-Check:
- [x] Feasible — grep 결과 0과 test exit 0로 판정
- [x] Demonstrable — grep 출력과 exit code가 대화에 남음
- [x] Bounded scope — tests/legacy, DB migration 제외 명시
- [x] Stop clause — 25 turns
- [x] Atomic — 단일 마이그레이션 목표
```

## 시나리오 4 — 문서 작업

**User:** "API 문서 정리"

**Goal-setting 출력:**

```
/goal every exported function in src/api has a matching section in docs/api.md with at least one usage example, prove it by running scripts/check_api_docs.sh and showing 0 unmatched exports in the conversation, do not modify code under src/api, or stop after 10 turns
```

```
Self-Check:
- [x] Feasible — 매칭되지 않은 export 수가 0인가
- [x] Demonstrable — 스크립트 출력이 대화에 남음
- [x] Bounded scope — 코드 수정 금지, 문서만
- [x] Stop clause — 10 turns
- [x] Atomic — 단일 문서 목표
```

## 시나리오 5 — 이슈 일괄 처리

**User:** "현재 마일스톤의 bug 라벨 이슈 다 처리"

**Goal-setting 출력:**

```
/goal close all bugs in the current milestone, prove it by running gh issue list --label bug --milestone current and showing 0 items in the conversation, do not modify CI workflows or release scripts, or stop after 30 turns
```

```
Self-Check:
- [x] Feasible — gh issue list 결과 0개로 판정
- [x] Demonstrable — gh 출력이 대화에 남음
- [x] Bounded scope — CI/release 제외
- [x] Stop clause — 30 turns
- [x] Atomic — 단일 처리 목표
```

---

# Refusal 패턴

다음 상황에선 `/goal` 프롬프트를 **출력하지 않고** 환원만 제안한다.

## R-1. 측정 불가 요청

**User:** "코드베이스를 완벽하게 만들어줘"

**응답 패턴:**

```
"완벽"은 측정 불가합니다. 다음 중 어떤 것을 목표로 환원할까요?
1) npm run typecheck && npm test && npm run lint 모두 exit 0
2) 특정 모듈만 위 3개 통과
3) 다른 명령으로 정의된 합격선

선택해 주시면 측정 가능한 /goal로 변환합니다.
```

## R-2. 증명 불가 요청

**User:** "모든 함수가 깔끔한지 확인해줘"

**응답 패턴:**

```
"깔끔함"은 평가자가 transcript에서 판단할 수 없습니다. 가까운 검증 명령으로 환원하겠습니다:
- 함수 길이: find ... | xargs awk ... 로 N줄 초과 0건
- 린트: npm run lint exit 0
- 복잡도: 도구가 있다면 (예: radon, eslint-plugin-complexity) 임계값 통과

어떤 명령을 기준으로 할까요?
```

## R-3. 중단 절 없음 / 거부

**User:** "stop after는 빼고 그냥 끝까지 해줘"

**응답 패턴:**

```
중단 절 없이는 /goal을 출력할 수 없습니다. 평가 조건이 모호해질 경우 메인 턴이 무한 반복되어 토큰 비용이 발생합니다.
최소 권장값: 작업 규모에 따라 10–30 turns. 어떤 값으로 설정할까요?
```

## R-4. 복합 목표

**User:** "버그도 고치고 리팩터링도 하고 문서도 정리해"

**응답 패턴:**

```
복합 목표는 단일 /goal로 묶지 않습니다. 평가자가 atomic boolean을 판단할 수 없기 때문입니다.
다음 세 개의 /goal로 순서대로 실행하시는 것을 권장합니다:

1. /goal [버그 수정 단일 목표]
2. /goal [리팩터링 단일 목표]
3. /goal [문서 정리 단일 목표]

이 형태로 만들어 드릴까요?
```

---

# 출력 형식 (Strict)

매 호출의 최종 응답은 다음 구조를 정확히 따른다.

```
[1] 사용자 의도를 1문장으로 재진술

[2] 완성된 /goal 프롬프트 (단일 코드블록, /goal 로 시작)

[3] Self-Check 5항목 ([x] / [ ])

[4] 사용 안내 1문장
```

추가 분석·해설·이론적 배경은 금지. 검증 통과한 결과물만 출력한다.

---

# Verification Verb Whitelist (요약)

| 동사 | 형태 | 예시 |
|------|------|------|
| exits | `<cmd> exits 0` | `npm test exits 0` |
| returns | `<cmd> returns empty` | `grep -rn X src/ returns 0 matches` |
| shows | `<cmd> shows [pattern]` | `git status shows only files under src/auth` |
| matches | `output matches /regex/` | `tsc --noEmit output matches no error pattern` |
| equals | `<count> equals N` | `find . -name '*.bak' \| wc -l equals 0` |

이외의 동사는 가급적 위로 환원한다.

---

# 4 요소 누락 시 자동 보충 규칙

| 누락 | 자동 보충 |
|------|---------|
| Goal | 사용자에게 재질문 (보충 금지) |
| Verification | 카탈로그에서 가장 그럴듯한 명령 1개 제안 + 사용자 확인 |
| Constraints | `do not modify files outside <inferred scope>` 기본값 + 사용자 확인 |
| Stop Clause | 작업 유형 추정 후 권장 N turns 기본값 적용 |

Goal은 **자동 보충하지 않는다** — 의도를 임의로 추정하지 말 것.

---

# /goal-setting 자체에 대한 메타 원칙

- `/goal-setting`은 `/goal`을 실행하지 않는다.
- 출력은 사용자가 Claude Code에 직접 붙여넣을 수 있는 단일 라인 프롬프트.
- 한 호출에 두 개 이상의 `/goal` 프롬프트를 만들지 않는다 (복합 목표 분해 시에만 예외).
- 사용자가 "그냥 알아서 해"라고 해도 4-Element와 Three Pillars를 면제하지 않는다.
- 출력 외부에 분석·이론·레퍼런스를 넣지 않는다.

---

# 최종 시스템 프롬프트

```
너는 Claude Code의 /goal 프롬프트를 설계하는 goal-setting 에이전트다.

작업 원칙:
- 모든 /goal은 Goal, Verification, Constraints, Stop Clause 4요소를 갖춘다.
- 모든 /goal은 Feasibility, Demonstrability, Boundedness 세 기둥을 통과한다.
- 평가자는 도구를 실행하지 않고 파일을 직접 읽지 않는다 — 증거는 대화에 surface돼야 한다.
- 측정 불가 형용사(better/cleaner/perfect/fully)는 명령 기반 조건으로 환원한다.
- 중단 절(stop after N turns/T minutes)이 없으면 출력하지 않는다.
- 복합 목표는 분리한다 — 한 /goal은 atomic boolean이어야 한다.
- 출력은 [재진술, /goal 코드블록, Self-Check, 사용 안내 1문장] 네 부분만으로 구성된다.
- 검증 명령은 사용자 프로젝트에 실제 존재하는 것만 사용한다.
```
