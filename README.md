![AI Skills for Everyone](author/wildmental-bjpark.png)

# goal-setting
> Skill for Cursor, Claude, Codex agents

**Claude Code의 `/goal` 명령에 바로 붙여넣을 수 있는, 완료 조건이 명확한 구조화된 마크다운 프롬프트를 함께 설계해 주는 Skill입니다. Cursor, Claude Code, Codex 모두 지원합니다.**

`/goal`은 "완료 조건을 만족할 때까지 여러 턴에 걸쳐 작업을 이어가는" 강력한 명령이지만, **조건을 모호하게 적으면 무한 반복·토큰 낭비·증명 불가 종료**가 발생합니다. 평가자는 도구를 실행하지 않고 파일도 직접 보지 않으며, 오직 대화에 드러난 내용만 보고 yes/no를 판단합니다. 이 스킬은 그 함정들을 **사전에 차단하는 행동 원칙**으로 고정하고, 결과를 **Required 4 Sections + 필요 시 추가 섹션**의 마크다운 블록으로 산출합니다.

---

## 이 스킬이 해결하는 것

### 1. `/goal` 프롬프트를 "측정 가능한 계약서"로 만든다

- 측정 불가 형용사(`better`, `perfect`, `cleaner`, `fully`) 자동 거부
- 평가자가 transcript에서 yes/no를 판정할 수 있는 **검증 동사**로 환원
- 종료 조건 절(turn cap, budget cap, queue empty 등) 누락 시 출력 차단

### 2. Required 4 Sections + 추가 섹션 구조를 강제한다

```
/goal

## 1) 작업 핵심 목표 및 범위
## 2) 작업 세부 규칙
## 3) 종료 조건 및 종료 방법
## 4) 기타 제약조건

## (선택) 5) ~ N) 추가 섹션 — 의존성·보고 포맷·외부 연동 등
```

4섹션 중 하나라도 비면 `/goal` 프롬프트를 출력하지 않습니다. 누락된 항목만 사용자에게 묻고 채웁니다. 추가 섹션은 사용자 요청에 따라 유연하게 편성합니다.

### 3. Three Pillars 검증을 자동화한다

```
Feasibility       → 측정 가능한 boolean 종료 상태인가?
Demonstrability   → 검증 결과가 대화 transcript에 남는가?
Boundedness       → Section 3 종료 조건(turn cap / budget)이 있는가?
```

세 기둥을 모두 통과한 결과만 사용자에게 전달됩니다.

### 4. 출력은 시각적 행 구분자로 명확히 분리한다

매 출력은 `---` 가로줄 3개로 4개 영역(의도 재진술 / `/goal` 프롬프트 / Self-Check / 사용 안내)을 분리합니다. 프롬프트 영역은 단일 마크다운 코드블록 안에 들어가 어디서부터 어디까지가 복사 대상인지 즉시 식별 가능합니다.

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
Boundedness       — Section 3 종료 조건(turn cap / budget) 필수
```

### Required 4 Sections 필수

| 섹션 | 핵심 항목 |
|------|----------|
| 1) 작업 핵심 목표 및 범위 | 한 문장 목표 · 시작 지점 · 작업 대상 · 자율성 수준 |
| 2) 작업 세부 규칙 | 워크플로 · 브랜치/PR · 의사결정 기록 · 도구 사용 규칙 |
| 3) 종료 조건 및 종료 방법 | 종료 조건(budget·turn cap·queue empty 등) + 종료 시 실행할 검증·상태 확인 명령 |
| 4) 기타 제약조건 | 금지 행동 · 수정 금지 파일 · 활성 범위 외 변경 금지 |

추가 섹션(`## 5)` ~ `## N)`)은 사용자 요청·작업 특성에 따라 유연하게 편성 (의존성·보고 포맷·외부 연동·보안 정책 등).

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

````markdown
/goal

## 1) 작업 핵심 목표 및 범위
- [한 문장 단일 목표]
- 시작 지점: [브랜치/이슈/커밋]
- 작업 대상: [범위·이슈 목록·디렉터리]
- 작업 자율성: [어디까지 승인 없이 진행 가능]

## 2) 작업 세부 규칙
- [워크플로 / 사이클]
- [브랜치·PR 전략]
- [의사결정 기록 / 카운터 / 로그 파일]
- [도구·명령 사용 규칙]

## 3) 종료 조건 및 종료 방법
- 종료 조건 (아래 중 하나라도 충족되는 순간 루프를 즉시 멈춘다):
  - [budget A → STOP REASON: <CODE_A>]
  - [budget B → STOP REASON: <CODE_B>]
  - 평가-진행 라운드(turn) 누적 N회 도달 → STOP REASON: TURN_CAP (= or stop after N turns)
- 종료 방법:
  1) [기록 작업]
  2) [검증 명령 실행 → exit 0 / 0 matches / 카운트 일치 등 출력을 대화에 surface]
  3) [상태 확인 명령 실행 → 카운터·PR 목록·이슈 목록을 대화에 surface]

## 4) 기타 제약조건
- [금지 행동: main 머지, 자동배포 유발 등]
- [수정 금지 파일·디렉터리]
- [활성 범위 외 변경 금지, 단 예외: ...]
````

### 작업 유형별 권장 turn cap

| 작업 유형 | 권장 N |
|----------|--------|
| 단순 버그 수정 | 10–15 turns |
| 리팩터링 | 15–25 turns |
| 마이그레이션 | 20–30 turns |
| 문서 작업 | 8–12 turns |
| 멀티 이슈 자동화 루프 | 30–60 turns |

---

## 출력 검증 체크리스트

스킬이 `/goal` 프롬프트를 출력하기 전에 자동으로 확인합니다.

- [ ] Required Sections — 1) 목표·범위 2) 세부규칙 3) 종료조건+방법 4) 제약 모두 채움
- [ ] Feasible — Section 1 목표가 boolean으로 측정 가능
- [ ] Demonstrable — Section 3 종료 방법의 검증 명령 출력이 대화에 surface됨
- [ ] Bounded scope — Section 4 제약이 변경 범위를 명시
- [ ] Stop clause — Section 3 종료 조건에 turn cap 또는 명시적 budget 포함
- [ ] Atomic — 단일 목표 (Section 1이 하나의 완료 기준)

여섯 항목 중 하나라도 fail이면 사용자에게 환원 제안만 주고 프롬프트는 출력하지 않습니다.

---

## 출력 예시

매 호출의 최종 응답은 `---` 가로줄 3개로 4영역이 분리됩니다. 사용자는 `/goal` 코드블록 한 덩어리만 Claude Code에 그대로 붙여넣으면 됩니다.

**입력:** "lecture-hub 후속 이슈를 ISSUE_LIST.md 순서대로 자동 구현. main 머지·자동배포는 금지."

**출력 (요약):**

````
**[의도 재진술]** ISSUE_LIST.md 순서대로 해소 가능한 이슈를 TDD 사이클과 stack형 draft PR로 자동 구현하면서, 의사결정 budget과 turn cap으로 안전하게 멈추는 멀티 이슈 루프 /goal을 만든다.

---

**[/goal 프롬프트 — 아래 코드블록 전체를 그대로 복사해 Claude Code에 붙여넣으세요]**

```markdown
/goal

## 1) 작업 핵심 목표 및 범위
- 목표: lecture-hub 후속 구현을 자동화 루프로 진행한다.
- 시작 지점: 선행 이슈가 모두 해소된 다음 착수 지점.
- 작업 대상: docs/issues/ISSUE_LIST.md 순서의 열린 이슈 중 선행 이슈가 해소되어 있고 아직 PR이 없는 것.
- 작업 자율성: 종료 조건 도달 또는 모든 구현 완료 시까지 자율 진행.

## 2) 작업 세부 규칙
- 브랜치 전략: 각 이슈는 TDD(Red→Green→Refactor→Report→PR)를 따르고 선행 feat/* 위에 stack형 draft PR로 분기.
- 의사결정 로그: docs/loop/DECISION_LOG.md 에 CORE/MINOR 분류로 기록, grep 가능한 카운터 `CORE: N` · `MINOR: M` 유지.

## 3) 종료 조건 및 종료 방법
- 종료 조건 (아래 중 하나라도 충족되는 순간 루프를 즉시 멈춘다):
  - CORE 카운터가 3에 도달 → STOP REASON: CORE_BUDGET
  - MINOR 카운터가 10에 도달 → STOP REASON: MINOR_BUDGET
  - 선행 이슈가 해소된 열린 이슈가 더 없음 → STOP REASON: NO_UNBLOCKED_ISSUES
  - 평가-진행 라운드(turn) 누적 40회 도달 → STOP REASON: TURN_CAP (= or stop after 40 turns)
- 종료 방법:
  1) docs/loop/DECISION_LOG.md 마지막 줄에 `STOP REASON: <원인>` 한 줄 덧붙임.
  2) `pnpm typecheck && pnpm test && pnpm lint && pnpm build` 네 명령 exit 0 출력을 대화에 남김.
  3) `cat docs/loop/DECISION_LOG.md` 로 카운터와 STOP REASON 줄을 대화에 남김.
  4) `gh pr list` 로 draft PR 목록을 대화에 남김.

## 4) 기타 제약조건
- 어떤 PR도 main에 merge하지 않으며, Vercel 자동배포를 유발하지 않는다.
- docs/PRD_v1.0.md, docs/issues/ISSUE_LIST.md, docs/PRD_OPEN_DECISIONS.md 는 수정하지 않는다.
- 활성 이슈의 구현 범위 밖 파일은 수정하지 않는다 (단 docs/loop/DECISION_LOG.md 와 reports/ 는 예외).
```

---

**[Self-Check]**

- [x] Required Sections — 4섹션 모두 채움
- [x] Feasible — STOP REASON 코드별로 boolean 판정 가능
- [x] Demonstrable — 4개 검증 명령 + DECISION_LOG + gh pr list 결과가 대화에 surface
- [x] Bounded scope — 수정 금지 파일·자동배포 금지·범위 외 변경 금지 명시
- [x] Stop clause — CORE/MINOR budget + NO_UNBLOCKED_ISSUES + 40 turns
- [x] Atomic — 단일 자동화 루프 목표

---

**[사용 안내]** 위 코드블록을 그대로 Claude Code에 붙여넣어 실행하세요. `/goal`만 입력하면 진행 상태와 평가자 판단 이유를 확인할 수 있습니다.
````

> 위 출력 예시는 README에서 4-backtick fence로 감싼 데모입니다. 실제 스킬 출력 시 사용자에게는 일반 마크다운으로 렌더링되며, `/goal` 프롬프트 본문만 3-backtick `` ```markdown ... ``` `` 코드블록 안에 들어가 한 덩어리로 복사할 수 있습니다.

---

## 긴 입력·긴 출력 자동 외부화

장시간 작업용 상세 요구사항은 한 번에 대화로 주고받기엔 길어지기 쉽습니다. 스킬은 두 경우를 자동으로 파일로 분리합니다(문서 경로 `<docs>`는 기존 문서 디렉터리, 없으면 `docs/`; `<slug>`는 목표 한 문장에서 뽑은 kebab-case).

### A. 이미 장문 구조 프롬프트가 있을 때

사용자가 자체 섹션·체계를 갖춘 **긴 요구사항 프롬프트**를 이미 가지고 있으면 스킬 템플릿에 억지로 끼워 맞추지 않습니다. **원래 구조를 최대한 보존**하고 의도가 또렷해지도록 최소 수정만 한 뒤 `<docs>/goals/original-requirements/<slug>.md`에 저장하고, `/goal`의 `## 2) 작업 세부 규칙`에서 그 파일을 참조해 세부 규칙을 적용하도록 지시합니다.

### B. `/goal` 본문이 4,000자를 넘을 때

생성된 `/goal` 본문이 4,000자를 초과하면 `<docs>/goals/<slug>.md`에 전문을 저장하고, 대화에는 한 줄짜리 호출 프롬프트만 제공합니다.

```
/goal 지금부터 <docs>/goals/<slug>.md 에 명시된 목표를 달성하기 위한 작업을 시작하라
```

A로 세부 규칙을 외부화하면 본문이 짧아져 B 임계값을 넘지 않을 수 있고, 그래도 넘으면 B로 `/goal` 자체도 파일화합니다.

---

## 스킬 구성

```
.cursor/skills/goal-setting/SKILL.md   # Cursor용
.claude/skills/goal-setting/SKILL.md   # Claude Code용
.agents/skills/goal-setting/SKILL.md   # Codex용
```

| 섹션 | 내용 |
|------|------|
| 기본 원칙 | 측정·증명·경계·단일성·종료 조건·시각적 행 구분자 필수 |
| Three Pillars | Feasibility / Demonstrability / Boundedness 정의·검증 표 |
| Required Sections | 4섹션 정의(목표·범위 / 세부규칙 / 종료조건+방법 / 제약) + Optional 추가 섹션 |
| Workflow | Intake → Audit → Mapping → Anti-Pattern Block → Output 5단계 |
| Verification Catalog | 테스트·린트·타입체크·grep·gh 등 검증 명령 모음 |
| 시나리오 예시 | 단일 버그 / 멀티 이슈 자동화 루프 / 마이그레이션(추가 섹션) |
| Refusal 패턴 | 측정 불가·증명 불가·종료 조건 누락·복합 목표·섹션 누락 처리 |
| 출력 형식 | 의도 재진술 + /goal 코드블록 + Self-Check + 사용 안내 (3개의 `---`로 분리) |
| 산출물 외부화 | 장문 입력은 `goals/original-requirements/`에 원 구조 보존·참조, 4,000자 초과 `/goal`은 `goals/`에 저장 후 호출 프롬프트 제공 |

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
  output_must_contain=[restated_intent, single /goal markdown codeblock with Required 4 Sections, 6-item Self-Check, 1-line usage hint]
  output_must_separate_areas_with=3 horizontal rules (---) between [intent | /goal block | Self-Check | usage hint]
  output_must_not_execute=/goal  # this skill designs the prompt only
  required_sections=[1) 작업 핵심 목표 및 범위, 2) 작업 세부 규칙, 3) 종료 조건 및 종료 방법, 4) 기타 제약조건]
  optional_sections=5)+ as needed (dependencies, report format, external integration, security, etc.)
  must_reject_if_missing_any_required_section=true
  must_reject_adjectives=[better, cleaner, perfect, fully, correct, nice, good]
  must_include_in_section_3=[stop conditions with STOP REASON codes, turn cap or budget, verification commands that surface output in conversation]
  prompt_block_fence=```markdown ... ```  # single fenced code block, copy-friendly
  docs_root=<docs> = existing docs dir, else docs/ ; <slug> = kebab-case from Section 1 goal
  externalize_existing_long_input=preserve user's structure, lightly edit for clarity, save to <docs>/goals/original-requirements/<slug>.md, reference it from /goal Section 2
  externalize_long_output=if /goal body > 4000 chars, save full prompt to <docs>/goals/<slug>.md and return only call prompt "/goal 지금부터 <docs>/goals/<slug>.md 에 명시된 목표를 달성하기 위한 작업을 시작하라"
```

---

## 라이선스

[MIT License](LICENSE)
