# my-skills

Codex CLI 와 Claude Code 에서 기능 개발 워크플로(계획 → 구현 → 검증 → 보강)를 단계별 슬래시 스킬로 강제하는 개인 스킬 모음.

각 단계는 직전 단계 산출물(`docs/features/<slug>/`)을 입력으로 받아 다음 단계 산출물을 만든다. 산출물이 없으면 스킬이 중단되므로, 단계를 건너뛰는 실수를 구조적으로 방지한다.

---

## 구조

```
my-skills/
├── codex/                       # Codex CLI 용 스킬
│   ├── AGENTS.md                # Codex 공통 행동 규칙
│   ├── project-plan/
│   ├── project-implement/
│   ├── project-verify/
│   └── project-patch/
└── claude-code/                 # Claude Code 용 스킬 (내용 동일)
    ├── CLAUDE.md                # Claude Code 공통 행동 규칙
    ├── project-plan/
    ├── project-implement/
    ├── project-verify/
    └── project-patch/
```

`codex/` 와 `claude-code/` 의 스킬 정의(`SKILL.md`)와 템플릿은 동일하다. 디렉터리가 분리된 이유는 각 에이전트의 설정 디렉터리(`~/.codex/`, `~/.claude/`)에 별도 배치하기 위함이다.

---

## 워크플로

```
/project-plan ──► /project-implement ──► /project-verify ─┐
                                              ▲           │
                                              └ /project-patch ◄ OPEN ≥ 1
```

| 단계 | 슬래시 | 입력 | 산출물 | phase.md |
|------|--------|------|--------|----------|
| 계획 | `/project-plan <기능 설명>` | 자연어 기능 설명 | `spec.md`, `plan.md` | `planning` → `planned` |
| 구현 | `/project-implement <slug>` | `spec.md`, `plan.md` | `progress.md`, 소스 코드 | `implementing` → `implemented` |
| 검증 | `/project-verify <slug>` | 구현 결과 + 명세/계획 | `review.md` (OPEN 항목 포함) | `verifying` → `verified` |
| 보강 | `/project-patch <slug>` | `review.md` 의 OPEN 항목 | 수정된 소스 코드 | → `implemented` (재검증 필요) |

각 단계 완료 후 `/clear` 로 세션을 초기화하고 다음 단계를 실행하도록 안내한다. 컨텍스트가 누적되지 않으므로 큰 기능을 다룰 때도 안정적이다.

---

## 스킬 목록

| 스킬 | 설명 | 상세 |
|------|------|------|
| `project-plan` | 자연어 기능 설명을 명세·계획 문서로 정리 | [codex](codex/project-plan/README.md) · [claude-code](claude-code/project-plan/README.md) |
| `project-implement` | 계획 문서를 입력받아 TDD 사이클로 구현 | [codex](codex/project-implement/README.md) · [claude-code](claude-code/project-implement/README.md) |
| `project-verify` | 코드 리뷰·기능 검증·보안 감사를 통합 수행 | [codex](codex/project-verify/README.md) · [claude-code](claude-code/project-verify/README.md) |
| `project-patch` | `review.md` 의 OPEN 항목을 보강 후 재검증 안내 | [codex](codex/project-patch/README.md) · [claude-code](claude-code/project-patch/README.md) |

---

## 설정 방법

각 스킬 디렉터리를 에이전트의 스킬 경로에 글로벌 또는 프로젝트 단위로 배치한다. 심볼릭 링크가 권장된다(원본 갱신이 자동 반영됨).

### Claude Code

```bash
# 글로벌 설치 (모든 프로젝트에서 사용)
mkdir -p ~/.claude/skills
for s in project-plan project-implement project-verify project-patch; do
  ln -s "$(pwd)/claude-code/$s" ~/.claude/skills/$s
done

# 프로젝트 단위 설치 (해당 레포에서만 사용)
mkdir -p .claude/skills
for s in project-plan project-implement project-verify project-patch; do
  ln -s "$(pwd)/claude-code/$s" .claude/skills/$s
done
```

### Codex CLI

```bash
# 글로벌 설치
mkdir -p ~/.codex/skills
for s in project-plan project-implement project-verify project-patch; do
  ln -s "$(pwd)/codex/$s" ~/.codex/skills/$s
done

# 프로젝트 단위 설치
mkdir -p .codex/skills
for s in project-plan project-implement project-verify project-patch; do
  ln -s "$(pwd)/codex/$s" .codex/skills/$s
done
```

복사 방식이 필요하면 `ln -s` 를 `cp -r` 로 교체한다.

스킬별 개별 설치 명령은 각 스킬 README 의 "설정 방법" 섹션을 참고한다.

---

## 의존 스킬

본 레포의 스킬은 외부 스킬 패키지를 조합해 동작한다. 사전 설치 필요.

### gstack

| 스킬 | 사용처 |
|------|--------|
| `office-hours` | `/project-plan` (아이디어 단계 게이트, 조건부) |
| `autoplan` | `/project-plan` (CEO/Eng/Design/DX 리뷰 통합) |
| `review` | `/project-verify` (코드 리뷰) |
| `qa-only` | `/project-verify` (기능 검증) |
| `cso` | `/project-verify` (보안 감사) |

### superpowers

| 스킬 | 사용처 |
|------|--------|
| `writing-plans` | `/project-implement` (구현 태스크 구체화, 조건부) |
| `subagent-driven-development` | `/project-implement`, `/project-patch` (태스크 단위 실행) |
| `test-driven-development` | `/project-implement`, `/project-patch` (RED-GREEN-REFACTOR) |

---

## 공통 행동 규칙

모든 스킬은 다음 규칙을 따른다. 상세는 [codex/AGENTS.md](codex/AGENTS.md) · [claude-code/CLAUDE.md](claude-code/CLAUDE.md) 참조.

| # | 규칙 | 핵심 원칙 |
|---|------|-----------|
| 1 | 불명확하면 중단하고 묻는다 | 요청이 모호하면 임의로 진행하지 않는다. |
| 2 | 한국어 출력에서 콜론으로 문장을 끝내지 않는다 | 문장 종결 부호는 마침표를 사용한다. |
| 3 | 한국어 파일 헤더 주석을 작성한다 | 새 소스 파일 첫 줄에 역할 설명 주석을 추가한다. |
| 4 | 의미 단위로 커밋한다 | 하나의 논리적 변경마다 커밋한다. |

> 트레이드오프. 이 규칙은 속도보다 안정성을 우선한다.

---

## 설계 원칙

- **단계 분리.** 각 단계는 독립된 세션에서 실행한다. 산출물 파일이 단계 간 유일한 인터페이스이므로 컨텍스트 누수가 없다.
- **산출물 입력 전용.** 후행 단계는 선행 단계 산출물을 수정하지 않는다. 요구사항 변경이 필요하면 `/project-plan` 으로 되돌아간다.
- **메인 세션 직접 수행.** 본 레포의 스킬 자체는 서브에이전트를 디스패치하지 않는다. 의존 스킬(`subagent-driven-development` 등)이 필요로 할 때만 위임한다.
- **템플릿 고정.** 산출물 템플릿의 헤더·표 컬럼·순서를 임의로 바꾸지 않는다. 빈 셀과 플레이스홀더만 채워 일관된 형식을 유지한다.
