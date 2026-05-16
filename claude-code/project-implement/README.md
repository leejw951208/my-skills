# project-implement

기능 구현 단계 스킬. `/project-plan` 이 만든 명세와 계획을 입력으로 받아 코드를 작성한다.

## 설정 방법

Claude Code 글로벌 또는 프로젝트 단위로 배치한다. 심볼릭 링크가 권장된다(원본 갱신이 자동 반영됨).

**글로벌 설치.** 모든 프로젝트에서 사용.

```bash
mkdir -p ~/.claude/skills
ln -s "$(pwd)/project-implement" ~/.claude/skills/project-implement
# 또는: cp -r project-implement ~/.claude/skills/project-implement
```

**프로젝트 단위 설치.** 해당 레포에서만 사용.

```bash
mkdir -p .claude/skills
ln -s "$(pwd)/project-implement" .claude/skills/project-implement
```

배치 후 Claude Code 세션에서 `/project-implement` 으로 호출한다.

## 의존 스킬

| 스킬 | 호출 시점 | 필수 여부 |
|------|----------|-----------|
| `superpowers:writing-plans` | 구현 태스크 구체화 단계 | 조건부 (plan.md 가 충분히 구체적이면 건너뜀) |
| `superpowers:subagent-driven-development` | 태스크 단위 실행 | 필수 |
| `superpowers:test-driven-development` | RED-GREEN-REFACTOR 사이클 | 필수 |

`spec.md`, `plan.md` 가 사전 산출물로 필요하므로 `/project-plan` 의 실행 결과에 의존한다.

## 사용법

```
/project-implement <feature-slug>
```

- 인수 없이 실행하면 어떤 기능을 구현할지 묻는다.
- slug가 kebab-case가 아니면 자동으로 후보를 제안하고 확인을 받는다.
- `docs/features/<slug>/spec.md`, `plan.md` 가 없으면 중단하고 `/project-plan` 실행을 안내한다.

## 동작 흐름

1. **대상 확인.** slug 검증 후 명세/계획 문서를 입력으로 로드한다.
2. **준비.** `phase.md` 를 `implementing` 으로 표시하고 `templates/progress.md` 로 진행 추적 파일을 만든다.
3. **구현 파이프라인.**
   - `superpowers:writing-plans` 로 구현 태스크를 구체화한다. `plan.md` 가 충분히 구체적이면 건너뛴다.
   - `superpowers:subagent-driven-development` 로 태스크 단위 실행을 수행한다.
   - `superpowers:test-driven-development` 로 RED-GREEN-REFACTOR 사이클을 적용한다.
4. **진행 추적.** 각 태스크 완료 시 `progress.md` 상태를 갱신한다.
5. **완료 보고.** `phase.md` 를 `implemented` 로 업데이트하고 다음 단계 안내를 출력한다.

## 코딩 제약

**단순함 우선.**

- 한 번만 사용되는 코드에 추상화 계층을 만들지 않는다.
- 요청되지 않은 유연성·확장성·설정 가능성을 추가하지 않는다.
- 발생할 수 없는 시나리오에 에러 처리를 작성하지 않는다.
- 200줄로 쓴 코드가 50줄로 표현 가능하면 다시 작성한다.

**변경 범위 최소화.**

- 태스크와 직접 연관된 파일만 수정한다.
- 깨지지 않은 코드를 리팩터링하지 않는다. 기존 스타일을 따른다.
- `spec.md`, `plan.md` 는 입력 전용이다. 요구사항 변경이 필요하면 중단하고 `/project-plan` 재실행을 안내한다.

## 산출물

| 파일 | 역할 |
|------|------|
| `docs/features/<slug>/progress.md` | 태스크 진행 상태. 단계마다 갱신. |
| `docs/features/<slug>/phase.md` | 단계 표시. `implementing` → `implemented`. |
| 소스 코드 | 명세와 계획에 따라 추가/수정된 실제 구현. |

## 다음 단계

`/clear` 로 세션을 초기화하고 `/project-verify <slug>` 를 실행한다.
