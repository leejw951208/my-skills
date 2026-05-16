# project-patch

검증 후 보강 단계 스킬. `review.md` 의 OPEN 항목을 수정하고 `/project-verify` 재실행을 준비한다.

## 설정 방법

Claude Code 글로벌 또는 프로젝트 단위로 배치한다. 심볼릭 링크가 권장된다(원본 갱신이 자동 반영됨).

**글로벌 설치.** 모든 프로젝트에서 사용.

```bash
mkdir -p ~/.claude/skills
ln -s "$(pwd)/project-patch" ~/.claude/skills/project-patch
# 또는: cp -r project-patch ~/.claude/skills/project-patch
```

**프로젝트 단위 설치.** 해당 레포에서만 사용.

```bash
mkdir -p .claude/skills
ln -s "$(pwd)/project-patch" .claude/skills/project-patch
```

배치 후 Claude Code 세션에서 `/project-patch` 으로 호출한다.

## 의존 스킬

| 스킬 | 호출 시점 | 필수 여부 |
|------|----------|-----------|
| `superpowers:subagent-driven-development` | OPEN 항목 보강 태스크 실행 | 필수 |
| `superpowers:test-driven-development` | 수정 부분 RED-GREEN-REFACTOR 사이클 | 필수 |

`review.md` 의 OPEN 항목이 사전 입력으로 필요하므로 `/project-verify` 의 실행 결과에 의존한다.

## 사용법

```
/project-patch <feature-slug>
```

- `review.md` 의 OPEN 항목이 0건이면 중단하고 `/project-verify` 결과 확인을 안내한다.
- slug가 kebab-case가 아니면 자동으로 후보를 제안한다.

## 동작 흐름

1. **대상 확인.** slug 검증 후 OPEN 항목 존재 여부를 확인한다.
2. **준비.** `review.md` 에서 `| OPEN |` 항목만 추출해 심각도 순(`HIGH` → `MEDIUM` → `LOW`)으로 정렬한다.
3. **수정 실행.** `superpowers:subagent-driven-development` 로 OPEN 항목을 보강 태스크로 분해해 실행한다.
4. **테스트.** `superpowers:test-driven-development` 로 RED-GREEN-REFACTOR 사이클을 적용한다.
5. **완료 보고.** `phase.md` 를 `implemented` 로 되돌리고 검증 재실행을 안내한다.

## 코딩 제약

**변경 범위 최소화.**

- OPEN 항목에 명시된 부분만 수정한다.
- 인접 코드, 주석, 포맷팅을 함께 "개선" 하지 않는다.
- 깨지지 않은 코드를 리팩터링하지 않는다. 기존 스타일을 따른다.
- 자신의 수정으로 미사용 상태가 된 import, 변수, 함수는 제거한다.

**단순함 우선.**

- 수정에 필요한 최소한의 변경만 한다.
- 한 번만 사용되는 코드에 추상화 계층을 만들지 않는다.
- 발생할 수 없는 시나리오에 에러 처리를 추가하지 않는다.

## 산출물

| 파일 | 역할 |
|------|------|
| 소스 코드 | OPEN 항목을 해소한 수정. |
| `docs/features/<slug>/phase.md` | 단계 표시. `verified` → `implemented`. |

`review.md` 의 OPEN 상태는 본 스킬에서 직접 갱신하지 않는다. 다음 `/project-verify` 가 새 OPEN 목록을 생성하면서 반영된다.

## 다음 단계

`/clear` 로 세션을 초기화하고 `/project-verify <slug>` 를 다시 실행한다.
