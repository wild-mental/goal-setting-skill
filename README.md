![AI Skills for Everyone](author/wildmental-bjpark.png)

# goal-setting
> Skill for Cursor, Claude, Codex agents

**Claude Code의 `/goal` 명령에 바로 붙여넣을 수 있는, 완료 조건이 명확한 한 줄 프롬프트를 함께 설계해 주는 Skill입니다. Cursor, Claude Code, Codex 모두 지원합니다.**

`/goal`은 "완료 조건을 만족할 때까지 여러 턴에 걸쳐 작업을 이어가는" 강력한 명령이지만, **조건을 모호하게 적으면 무한 반복·토큰 낭비·증명 불가 종료**가 발생합니다. 평가자는 도구를 실행하지 않고 파일도 직접 보지 않으며, 오직 대화에 드러난 내용만 보고 yes/no를 판단합니다. 이 스킬은 그 함정들을 **사전에 차단하는 행동 원칙**으로 고정합니다.

---

## 이 스킬이 해결하는 것

### 1. `/goal` 프롬프트를 "측정 가능한 계약서"로 만든다

- 측정 불가 형용사(`better`, `perfect`, `cleaner`, `fully`) 자동 거부
- 평가자가 transcript에서 yes/no를 판정할 수 있는 **검증 동사**로 환원
- 중단 절(`or stop after N turns / T minutes`) 누락 시 출력 차단

### 2. 4-Element Structure를 강제한다

```
/goal [GOAL],
prove it by [VERIFICATION] and show the result in the conversation,
do not [CONSTRAINTS],
or stop after [N turns / T minutes]
```

네 요소 중 하나라도 비면 `/goal` 프롬프트를 출력하지 않습니다. 누락된 것만 사용자에게 묻고 채웁니다.

### 3. Three Pillars 검증을 자동화한다

```
Feasibility       → 측정 가능한 boolean 종료 상태인가?
Demonstrability   → 검증 결과가 대화 transcript에 남는가?
Boundedness       → or stop after ... 절이 있는가?
```

세 기둥을 모두 통과한 결과만 사용자에게 전달됩니다.

---

## 왜 skill 이 필요한가

| 흔한 시도 | 기대 | 실제 |
|-----------|------|------|
| `/goal make the codebase perfect` | 알아서 정리 | "perfect" 정의 불가 → 평가자가 yes를 못 냄 → 메인 턴 반복 |
| `/goal fix the login bug` | 버그 수정 | 어떤 버그·어떤 테스트·어디까지 변경할지 없음 → 증명 불가 종료 |
| `/goal all files are correct` | 검증 통과 | 평가자는 파일을 직접 안 봄 → 영원히 no |
| `/goal complete the migration` | 마이그레이션 끝 | "어디까지가 끝인가" 정의 없음 |
| stop 절 없이 실행 | 잘 끝나겠지 | 메인 턴 비용 폭주 가능 |
| 복합 목표 한 줄에 묶기 | 한 번에 처리 | atomic boolean 판정 불가 → 평가자 혼란 |

이 스킬은 위 패턴을 **사전에 차단**하고, 평가자가 판단 가능한 형태로 자동 환원합니다.

---

## 빠른 시작

### 사전 요구사항

- Claude Code v2.1.139 이상 (`/goal` 사용 시)
- Cursor, Claude Code, 또는 Codex

> `/goal-setting` **자체**는 어떤 도구에서든 동작합니다. 산출물인 `/goal ...` 프롬프트는 Claude Code에서 실행하세요.

### 스킬 설치

스킬은 **개인** 또는 **프로젝트** 범위 중 하나를 골라 설치합니다. 두 범위 모두 `curl`로 `SKILL.md`만 받습니다. **이 저장소 전체를 작업 repo에 clone하지 마세요.**

| | 개인 스킬 | 프로젝트 스킬 |
|---|----------|--------------|
| **적용 범위** | 내가 여는 모든 프로젝트 | 현재 repo에서만 |
| **경로** | `~/…/skills/goal-setting/` | `<repo-root>/.cursor/skills/goal-setting/` 등 |
| **Git 영향** | 작업 repo에 파일 추가 없음 | repo에 스킬 파일 commit 가능 (팀 공유) |
| **언제 쓰나** | 혼자 모든 프로젝트에서 쓸 때 | 팀 repo에 스킬을 고정·공유할 때 |

| 도구 | 개인 경로 | 프로젝트 경로 |
|------|-----------|---------------|
| Cursor | `~/.cursor/skills/goal-setting/` | `.cursor/skills/goal-setting/` |
| Claude Code | `~/.claude/skills/goal-setting/` | `.claude/skills/goal-setting/` |
| Codex | `~/.agents/skills/goal-setting/` | `.agents/skills/goal-setting/` |

#### 개인 스킬 (권장 — Git repo 무변경)

```bash
# Cursor
mkdir -p ~/.cursor/skills/goal-setting
curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.cursor/skills/goal-setting/SKILL.md \
  -o ~/.cursor/skills/goal-setting/SKILL.md

# Claude Code
mkdir -p ~/.claude/skills/goal-setting
curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.claude/skills/goal-setting/SKILL.md \
  -o ~/.claude/skills/goal-setting/SKILL.md

# Codex
mkdir -p ~/.agents/skills/goal-setting
curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.agents/skills/goal-setting/SKILL.md \
  -o ~/.agents/skills/goal-setting/SKILL.md
```

#### 프로젝트 스킬 (프로젝트 경로에 설치)

AI 스킬 설정을 repo에 포함·공유하려는 경우에만 사용하세요.

```bash
# Cursor
mkdir -p .cursor/skills/goal-setting
curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.cursor/skills/goal-setting/SKILL.md \
  -o .cursor/skills/goal-setting/SKILL.md

# Claude Code
mkdir -p .claude/skills/goal-setting
curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.claude/skills/goal-setting/SKILL.md \
  -o .claude/skills/goal-setting/SKILL.md

# Codex
mkdir -p .agents/skills/goal-setting
curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.agents/skills/goal-setting/SKILL.md \
  -o .agents/skills/goal-setting/SKILL.md
```

필요한 도구(Cursor / Claude Code / Codex)만 골라 실행하면 됩니다.

#### 설치 후

- **Cursor**: **Reload Window** 한 번
- **Claude Code**: 스킬 수정은 세션 중 live 반영; 세션 시작 후 새 top-level `.claude/skills/`는 재시작 필요할 수 있음
- **Codex**: 스킬이 안 보이면 Codex 재시작

### 사용 방법

`/goal` 프롬프트가 필요한 작업을 설명하면 Agent가 스킬을 자동으로 적용합니다.

| 도구 | 수동 호출 |
|------|-----------|
| Cursor | `/goal-setting` |
| Claude Code | `/goal-setting` |
| Codex | `/skills` 또는 `$goal-setting` |

**적용되는 요청 예시:**

- "로그인 버그 수정을 `/goal`로 만들고 싶어"
- "payment 모듈 리팩터링을 `/goal`로 어떻게 적어?"
- "legacyAuth → newAuth 마이그레이션 `/goal` 프롬프트 짜줘"
- "이 요청을 `/goal`로 바꿔줘: 코드 더 깔끔하게 만들어줘"
- "테스트가 다 통과할 때까지 돌리는 `/goal` 프롬프트"

---

## 핵심 행동 원칙 요약

스킬 본문에 상세 규칙이 있으며, 아래는 가장 자주 막히는 지점입니다.

### Three Pillars 동시 통과

```
Feasibility       — exit code, count, regex 같은 boolean 종료 상태로 환원
Demonstrability   — 검증 결과를 대화 transcript에 surface
Boundedness       — or stop after N turns / T minutes 필수
```

### 4-Element Structure 필수

| 요소 | 절 형태 |
|------|--------|
| Goal | `[목표 동사 + 구체적 대상]` |
| Verification | `prove it by [command(s)] and show [result] in the conversation` |
| Constraints | `do not [금지 범위]` |
| Stop Clause | `or stop after [N turns / T minutes]` |

### 측정 불가 형용사 자동 거부

`better` / `cleaner` / `perfect` / `fully` / `correct` 같은 단어는 명령 기반 조건으로 환원하지 못하면 출력하지 않습니다.

### 평가자는 도구를 실행하지 않는다

가장 중요한 한 줄:

> "평가자는 파일을 직접 읽지 않고, 명령도 직접 실행하지 않는다. Claude가 대화에 surface한 결과만 본다."

따라서 모든 검증은 `<command> exits 0 and shows result in the conversation` 패턴으로 작성됩니다.

---

## 권장 검증 동사 (Whitelist)

| 동사 | 형태 | 예시 |
|------|------|------|
| exits | `<cmd> exits 0` | `npm test exits 0` |
| returns | `<cmd> returns empty` | `grep -rn X src/ returns 0 matches` |
| shows | `<cmd> shows [pattern]` | `git status shows only files under src/auth` |
| matches | `output matches /regex/` | `tsc --noEmit output matches no error pattern` |
| equals | `<count> equals N` | `find . -name '*.bak' \| wc -l equals 0` |

---

## 표준 템플릿

```
/goal [GOAL],
prove it by [VERIFICATION COMMANDS] and show the result in the conversation,
do not [CONSTRAINTS],
or stop after [N turns / T minutes]
```

### 작업 유형별 권장 stop 값

| 작업 유형 | 권장 N |
|----------|--------|
| 단순 버그 수정 | 10–15 turns |
| 리팩터링 | 15–25 turns |
| 마이그레이션 | 20–30 turns |
| 문서 작업 | 8–12 turns |

---

## 출력 검증 체크리스트

스킬이 `/goal` 프롬프트를 출력하기 전에 자동으로 확인합니다.

- [ ] Feasible — 측정 가능한 boolean 종료 상태인가?
- [ ] Demonstrable — 검증 결과가 대화 transcript에 남는가?
- [ ] Bounded scope — 변경 범위가 제한되었는가?
- [ ] Stop clause — `or stop after N turns / T minutes`가 포함됐는가?
- [ ] Atomic — 한 줄 안에 단일 목표인가?

다섯 항목 중 하나라도 fail이면 사용자에게 환원 제안만 주고 프롬프트는 출력하지 않습니다.

---

## 출력 예시

**입력:** "legacyAuth을 newAuth로 전부 마이그레이션"

**출력:**

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

---

## 스킬 구성

```
.cursor/skills/goal-setting/SKILL.md   # Cursor용
.claude/skills/goal-setting/SKILL.md   # Claude Code용
.agents/skills/goal-setting/SKILL.md   # Codex용
```

| 섹션 | 내용 |
|------|------|
| 기본 원칙 | 측정·증명·경계·단일성·중단 절 필수 |
| Three Pillars | Feasibility / Demonstrability / Boundedness 정의·검증 표 |
| 4-Element Structure | Goal / Verification / Constraints / Stop Clause 필수 4요소 |
| Workflow | Intake → Audit → Mapping → Anti-Pattern Block → Output 5단계 |
| Verification Catalog | 테스트·린트·타입체크·grep·gh 등 검증 명령 모음 |
| 시나리오 예시 | 버그·리팩터링·마이그레이션·문서·이슈 5종 |
| Refusal 패턴 | 측정 불가·증명 불가·중단 절 누락·복합 목표 처리 |
| 출력 형식 | 재진술 + 코드블록 + Self-Check + 사용 안내 1줄 |

---

## 이런 분께 추천합니다

- Claude Code에서 `/goal`을 써 봤지만 **언제 끝났다고 판단할지가 모호**해 멈추지 않거나, 평가자가 yes를 안 줘서 답답했던 분
- "AI에게 일을 잘 시키는 능력"을 **종료 조건 설계 능력**으로 업그레이드하고 싶은 분
- 팀에서 `/goal`을 공용 워크플로로 도입하려는데 **공통 작성 규칙**이 필요한 분
- 장기 실행 에이전트 작업의 **토큰 비용과 안전 한도**를 사전에 제어하고 싶은 분

---

## 참고

- [Claude Code `/goal` 공식 문서](https://code.claude.com/docs/en/goal)
- Cursor Agent Skills: [Creating Skills](https://docs.cursor.com)
- Claude Code Skills: [Extend Claude with skills](https://docs.anthropic.com/en/docs/claude-code/skills)
- Codex Agent Skills: [Agent Skills](https://developers.openai.com/codex/skills/)

---

## For LLMs

<!-- install-only; operational rules live in SKILL.md — load skill after install -->

```
skill=goal-setting
repo=https://github.com/wild-mental/goal-setting-skill
install.method=curl-only  # never clone this repo into user's project

flow:
  1. ask scope: "개인 스킬(~/, 모든 프로젝트) vs 프로젝트 스킬(./, 현재 repo만) 중 어디에 설치할까요?"
  2. ask tools: "Cursor / Claude Code / Codex 중 어떤 도구에 설치할까요?" (복수 선택 가능)
  3. run matching curl commands below; do not install both scopes unless user asks
  4. post_install steps; then load SKILL.md — do not infer rules from README

scope.user.paths:
  cursor=~/.cursor/skills/goal-setting/SKILL.md
  claude=~/.claude/skills/goal-setting/SKILL.md
  codex=~/.agents/skills/goal-setting/SKILL.md

scope.project.paths:
  cursor=.cursor/skills/goal-setting/SKILL.md
  claude=.claude/skills/goal-setting/SKILL.md
  codex=.agents/skills/goal-setting/SKILL.md

install.user.cursor=mkdir -p ~/.cursor/skills/goal-setting && curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.cursor/skills/goal-setting/SKILL.md -o ~/.cursor/skills/goal-setting/SKILL.md
install.user.claude=mkdir -p ~/.claude/skills/goal-setting && curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.claude/skills/goal-setting/SKILL.md -o ~/.claude/skills/goal-setting/SKILL.md
install.user.codex=mkdir -p ~/.agents/skills/goal-setting && curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.agents/skills/goal-setting/SKILL.md -o ~/.agents/skills/goal-setting/SKILL.md

install.project.cursor=mkdir -p .cursor/skills/goal-setting && curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.cursor/skills/goal-setting/SKILL.md -o .cursor/skills/goal-setting/SKILL.md
install.project.claude=mkdir -p .claude/skills/goal-setting && curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.claude/skills/goal-setting/SKILL.md -o .claude/skills/goal-setting/SKILL.md
install.project.codex=mkdir -p .agents/skills/goal-setting && curl -fsSL https://raw.githubusercontent.com/wild-mental/goal-setting-skill/main/.agents/skills/goal-setting/SKILL.md -o .agents/skills/goal-setting/SKILL.md

install.project.note=run from repo root; adds tracked files — confirm user chose project scope

invoke.cursor=/goal-setting
invoke.claude=/goal-setting
invoke.codex=/skills|$goal-setting

post_install.cursor=Reload Window
post_install.claude=live reload; restart if new top-level .claude/skills/ after session start
post_install.codex=restart if skill not detected

contract:
  output_must_contain=[restated_intent, single /goal codeblock, 5-item Self-Check, 1-line usage hint]
  output_must_not_execute=/goal  # this skill designs the prompt only
  must_reject_if_missing=[Goal, Verification, Constraints, StopClause]
  must_reject_adjectives=[better, cleaner, perfect, fully, correct, nice, good]
  must_include_clause=or stop after N turns | or stop after T minutes
```

---

## 라이선스

[MIT License](LICENSE)
