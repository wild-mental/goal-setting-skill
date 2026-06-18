![AI Skills for Everyone](author/wildmental-bjpark.png)

# goal-setting
> Skill for Cursor, Claude, Codex agents

**Language / 언어:** [한국어](README.md) · [English](README.en.md)

**A Skill that helps you co-design a structured markdown prompt with clear completion conditions, ready to paste straight into Claude Code's `/goal` command. Works with Cursor, Claude Code, and Codex.**

`/goal` is a powerful command that "keeps working across multiple turns until the completion conditions are met," but **writing those conditions vaguely leads to infinite loops, wasted tokens, and unprovable termination**. The evaluator does not run any tools and does not look at files directly; it judges yes/no based solely on what is surfaced in the conversation. This skill locks down those traps with **behavioral principles that block them up front**, and produces the result as a markdown block of **Required 4 Sections + optional additional sections as needed**.

---

## What this skill solves

### 1. Turns the `/goal` prompt into a "measurable contract"

- Automatically rejects unmeasurable adjectives (`better`, `perfect`, `cleaner`, `fully`)
- Reduces them to **verification verbs** that let the evaluator judge yes/no from the transcript
- Blocks output if a termination-condition clause (turn cap, budget cap, queue empty, etc.) is missing

### 2. Enforces the Required 4 Sections + additional-section structure

```
/goal

## 1) 작업 핵심 목표 및 범위 (Core Goal & Scope)
## 2) 작업 세부 규칙 (Detailed Rules)
## 3) 종료 조건 및 종료 방법 (Termination Conditions & Method)
## 4) 기타 제약조건 (Other Constraints)

## (Optional) 5) ~ N) additional sections — dependencies, report format, external integration, etc.
```

If any one of the four sections is empty, the `/goal` prompt is not produced. It asks the user only about the missing items and fills them in. Additional sections are organized flexibly per the user's request.

### 3. Automates the Three Pillars verification

```
Feasibility       → Is it a measurable boolean termination state?
Demonstrability   → Do the verification results remain in the conversation transcript?
Boundedness       → Is there a Section 3 termination condition (turn cap / budget)?
```

Only results that pass all three pillars are delivered to the user.

### 4. Clearly separates output with visual row dividers

Every output uses three `---` horizontal rules to separate four areas (restated intent / `/goal` prompt / Self-Check / usage guide). The prompt area sits inside a single markdown code block, so you can instantly see exactly what to copy from start to end.

---

## Why a skill is needed

| Common attempt | Expectation | Reality |
|-----------|------|------|
| `/goal make the codebase perfect` | Cleans up on its own | "perfect" cannot be defined → evaluator can't return yes → main turn repeats |
| `/goal fix the login bug` | Bug fixed | No which bug · which test · how far to change → unprovable termination |
| `/goal all files are correct` | Verification passes | Evaluator doesn't look at files directly → forever no |
| `/goal complete the migration` | Migration done | No definition of "where is done" |
| Running without a stop clause | It'll probably end fine | Main-turn cost can explode |
| Bundling compound goals into one line | Handle it all at once | Atomic boolean judgment impossible → evaluator confusion |

This skill **blocks the patterns above up front** and automatically reduces them into a form the evaluator can judge.

---

## Quick start

### Prerequisites

- Claude Code v2.1.139 or later (when using `/goal`)
- Cursor, Claude Code, or Codex

> `/goal-setting` **itself** works in any tool. Run its output, the `/goal ...` prompt, in Claude Code.

### Installing the skill

Install the skill at either **personal** or **project** scope. Both scopes fetch only `SKILL.md` via `curl`. **Do not clone this entire repository into your working repo.**

| | Personal skill | Project skill |
|---|----------|--------------|
| **Scope** | Every project I open | The current repo only |
| **Path** | `~/…/skills/goal-setting/` | `<repo-root>/.cursor/skills/goal-setting/`, etc. |
| **Git impact** | No files added to the working repo | Skill files can be committed to the repo (team sharing) |
| **When to use** | When using it solo across all projects | When pinning/sharing the skill in a team repo |

| Tool | Personal path | Project path |
|------|-----------|---------------|
| Cursor | `~/.cursor/skills/goal-setting/` | `.cursor/skills/goal-setting/` |
| Claude Code | `~/.claude/skills/goal-setting/` | `.claude/skills/goal-setting/` |
| Codex | `~/.agents/skills/goal-setting/` | `.agents/skills/goal-setting/` |

#### Personal skill (recommended — no Git repo changes)

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

#### Project skill (install at the project path)

Use this only when you want to include and share the AI skill configuration in the repo.

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

Just pick and run the tool(s) you need (Cursor / Claude Code / Codex).

#### After installation

- **Cursor**: **Reload Window** once
- **Claude Code**: skill edits apply live during the session; a new top-level `.claude/skills/` created after the session started may require a restart
- **Codex**: if the skill doesn't show up, restart Codex

### How to use

Describe a task that needs a `/goal` prompt and the Agent applies the skill automatically.

| Tool | Manual invocation |
|------|-----------|
| Cursor | `/goal-setting` |
| Claude Code | `/goal-setting` |
| Codex | `/skills` or `$goal-setting` |

**Example requests it applies to:**

- "I want to turn fixing the login bug into a `/goal`"
- "How do I write the payment-module refactor as a `/goal`?"
- "Write me a `/goal` prompt for the legacyAuth → newAuth migration"
- "Convert this request into a `/goal`: make the code cleaner"
- "A `/goal` prompt that runs until all tests pass"

---

## Summary of core behavioral principles

Detailed rules live in the skill body; below are the points where people most often get stuck.

### Passing the Three Pillars simultaneously

```
Feasibility       — Reduce to a boolean termination state like exit code, count, or regex
Demonstrability   — Surface the verification result in the conversation transcript
Boundedness       — A Section 3 termination condition (turn cap / budget) is required
```

### Required 4 Sections are mandatory

| Section | Key items |
|------|----------|
| 1) 작업 핵심 목표 및 범위 (Core Goal & Scope) | One-sentence goal · starting point · work target · autonomy level |
| 2) 작업 세부 규칙 (Detailed Rules) | Workflow · branch/PR · decision logging · tool-usage rules |
| 3) 종료 조건 및 종료 방법 (Termination Conditions & Method) | Termination conditions (budget · turn cap · queue empty, etc.) + verification/status-check commands to run at termination |
| 4) 기타 제약조건 (Other Constraints) | Forbidden actions · files not to modify · no changes outside the active scope |

Additional sections (`## 5)` ~ `## N)`) are organized flexibly per the user's request and the nature of the task (dependencies, report format, external integration, security policy, etc.).

### Automatic rejection of unmeasurable adjectives

Words like `better` / `cleaner` / `perfect` / `fully` / `correct` are not output unless they can be reduced to command-based conditions.

### The evaluator does not run tools

The single most important line:

> "The evaluator does not read files directly and does not run commands directly. It only sees results that Claude has surfaced in the conversation."

Therefore every verification is written in the `<command> exits 0 and shows result in the conversation` pattern.

---

## Recommended verification verbs (Whitelist)

| Verb | Form | Example |
|------|------|------|
| exits | `<cmd> exits 0` | `npm test exits 0` |
| returns | `<cmd> returns empty` | `grep -rn X src/ returns 0 matches` |
| shows | `<cmd> shows [pattern]` | `git status shows only files under src/auth` |
| matches | `output matches /regex/` | `tsc --noEmit output matches no error pattern` |
| equals | `<count> equals N` | `find . -name '*.bak' \| wc -l equals 0` |

---

## Standard template

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

### Recommended turn cap by task type

| Task type | Recommended N |
|----------|--------|
| Simple bug fix | 10–15 turns |
| Refactoring | 15–25 turns |
| Migration | 20–30 turns |
| Documentation work | 8–12 turns |
| Multi-issue automation loop | 30–60 turns |

---

## Output verification checklist

The skill checks this automatically before producing a `/goal` prompt.

- [ ] Required Sections — 1) goal·scope 2) detailed rules 3) termination conditions+method 4) constraints all filled in
- [ ] Feasible — Section 1 goal is measurable as a boolean
- [ ] Demonstrable — the verification-command output of Section 3's termination method is surfaced in the conversation
- [ ] Bounded scope — Section 4 constraints specify the change scope
- [ ] Stop clause — Section 3 termination conditions include a turn cap or an explicit budget
- [ ] Atomic — single goal (Section 1 is one completion criterion)

If any one of the six items fails, it only gives the user a reduction suggestion and does not output the prompt.

---

## Output example

The final response of every invocation is separated into four areas by three `---` horizontal rules. The user only needs to paste the single `/goal` code block as-is into Claude Code.

**Input:** "Automatically implement the follow-up issues of lecture-hub in the order of ISSUE_LIST.md. No main merge or auto-deploy."

**Output (summary):**

````
**[Restated intent]** Build a multi-issue loop /goal that automatically implements resolvable issues in the order of ISSUE_LIST.md using a TDD cycle and stacked draft PRs, while stopping safely with a decision budget and a turn cap.

---

**[/goal prompt — copy the entire code block below as-is and paste it into Claude Code]**

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

- [x] Required Sections — all 4 sections filled in
- [x] Feasible — boolean judgment possible per STOP REASON code
- [x] Demonstrable — 4 verification commands + DECISION_LOG + gh pr list results surfaced in the conversation
- [x] Bounded scope — files not to modify · no auto-deploy · no out-of-scope changes specified
- [x] Stop clause — CORE/MINOR budget + NO_UNBLOCKED_ISSUES + 40 turns
- [x] Atomic — single automation-loop goal

---

**[Usage guide]** Paste the code block above as-is into Claude Code and run it. Just type `/goal` to check the progress status and the evaluator's reasoning.
````

> The output example above is a demo wrapped in a 4-backtick fence within the README. In the actual skill output, the user sees it rendered as normal markdown, and only the `/goal` prompt body sits inside a 3-backtick `` ```markdown ... ``` `` code block so it can be copied as a single chunk.

---

## Automatic externalization of long inputs and long outputs

Detailed requirements for long-running work tend to get too long to exchange in a single conversation. The skill automatically splits two cases into files (the document path `<docs>` is the existing docs directory, or `docs/` if none; `<slug>` is a kebab-case slug derived from the one-sentence goal).

### A. When a long structured prompt already exists

If the user already has a **long requirements prompt** with its own sections and structure, the skill does not force it into the skill template. It **preserves the original structure as much as possible**, makes only minimal edits so the intent becomes clear, saves it to `<docs>/goals/original-requirements/<slug>.md`, and instructs `/goal`'s `## 2) 작업 세부 규칙` to reference that file to apply the detailed rules.

### B. When the `/goal` body exceeds 4,000 characters

If the generated `/goal` body exceeds 4,000 characters, the full text is saved to `<docs>/goals/<slug>.md`, and only a one-line call prompt is provided in the conversation.

```
/goal 지금부터 <docs>/goals/<slug>.md 에 명시된 목표를 달성하기 위한 작업을 시작하라
```

Externalizing the detailed rules with A can shorten the body so it doesn't cross the B threshold; if it still crosses, B turns the `/goal` itself into a file as well.

---

## Skill structure

```
.cursor/skills/goal-setting/SKILL.md   # Cursor용
.claude/skills/goal-setting/SKILL.md   # Claude Code용
.agents/skills/goal-setting/SKILL.md   # Codex용
```

| Section | Contents |
|------|------|
| Basic principles | Measurement · proof · boundary · singularity · termination condition · visual row divider required |
| Three Pillars | Definition/verification table for Feasibility / Demonstrability / Boundedness |
| Required Sections | 4-section definition (goal·scope / detailed rules / termination conditions+method / constraints) + Optional additional sections |
| Workflow | 5 stages: Intake → Audit → Mapping → Anti-Pattern Block → Output |
| Verification Catalog | Collection of verification commands: test · lint · typecheck · grep · gh, etc. |
| Scenario examples | Single bug / multi-issue automation loop / migration (additional sections) |
| Refusal patterns | Handling unmeasurable · unprovable · missing termination condition · compound goal · missing section |
| Output format | Restated intent + /goal code block + Self-Check + usage guide (separated by three `---`) |
| Output externalization | Long input preserves original structure in `goals/original-requirements/` and references it; a `/goal` over 4,000 characters is saved to `goals/` and a call prompt is provided |

---

## Recommended for

- Those who have used `/goal` in Claude Code but found it **ambiguous when to judge it as done**, so it wouldn't stop, or were frustrated that the evaluator wouldn't return yes
- Those who want to upgrade their "ability to get AI to do work well" into the **ability to design termination conditions**
- Those who want to adopt `/goal` as a shared team workflow and need **common authoring rules**
- Those who want to control the **token cost and safety limits** of long-running agent work up front

---

## References

- [Claude Code `/goal` official docs](https://code.claude.com/docs/en/goal)
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

## License

[MIT License](LICENSE)
