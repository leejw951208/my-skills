# project-plan

기능 계획 단계 스킬. 자연어 기능 설명을 입력받아 비즈니스/엔지니어링 리뷰를 거친 뒤 `docs/features/<slug>/` 경로에 명세와 계획 문서를 생성한다.

## 설정 방법

Codex CLI 글로벌 또는 프로젝트 단위로 배치한다. 심볼릭 링크가 권장된다(원본 갱신이 자동 반영됨).

**글로벌 설치.** 모든 프로젝트에서 사용.

```bash
mkdir -p ~/.codex/skills
ln -s "$(pwd)/project-plan" ~/.codex/skills/project-plan
# 또는: cp -r project-plan ~/.codex/skills/project-plan
```

**프로젝트 단위 설치.** 해당 레포에서만 사용.

```bash
mkdir -p .codex/skills
ln -s "$(pwd)/project-plan" .codex/skills/project-plan
```

배치 후 Codex CLI 세션에서 `/project-plan` 으로 호출한다.

## 의존 스킬

| 스킬 | 호출 시점 | 필수 여부 |
|------|----------|-----------|
| `office-hours` (gstack) | 아이디어 구체화 게이트에서 "아이디어 단계" 선택 시 | 조건부 |
| `autoplan` (gstack) | 계획 리뷰 단계에서 항상 호출 | 필수 |

`autoplan` 은 내부적으로 CEO / Eng / Design / DX 리뷰 스킬을 순차 실행한다. 모든 게이트는 메인 세션의 `AskUserQuestion` 으로 그대로 노출된다.

## 사용법

```
/project-plan <기능 설명>
```

- 인수 없이 실행하면 어떤 기능을 계획할지 묻는다.
- 한국어 설명은 의미 기반 kebab-case slug로 변환되며, 사용자 확인을 거친다.
- 예. `/project-plan 사용자 로그인` → `user-login`.

## 동작 흐름

1. **기능 설명 수집.** `$ARGUMENTS` 를 받아 slug로 변환한다.
2. **준비.** `docs/features/<slug>/` 디렉터리를 만들고 `phase.md` 를 `planning` 으로 표시한다.
3. **아이디어 구체화 게이트.** 아이디어 단계라면 gstack `office-hours` 스킬로 디자인 문서를 만든다. 요구사항이 명확하면 건너뛴다.
4. **계획 리뷰.** gstack `autoplan` 스킬로 CEO/Eng/Design/DX 리뷰를 통과한다. 모든 게이트는 메인 세션의 `AskUserQuestion` 으로 사용자에게 노출된다.
5. **산출물 작성.** `templates/spec.md`, `templates/plan.md` 구조 그대로 채워 작성한다.
6. **완료 보고.** `phase.md` 를 `planned` 로 업데이트하고 다음 단계 안내를 출력한다.

## 산출물

| 파일 | 역할 |
|------|------|
| `docs/features/<slug>/spec.md` | 기능 명세. 배경, 동작 방식, 입출력, 제약, 예외, 채택 근거, 비기능 요건. |
| `docs/features/<slug>/plan.md` | 구현 계획. 단계 구성, 태스크, 아키텍처 다이어그램, 테스트 매트릭스. |
| `docs/features/<slug>/phase.md` | 단계 표시. `planning` → `planned`. |

## 제약

- 본 스킬은 메인 세션에서 직접 수행한다. 서브에이전트 디스패치 없음.
- 산출물 템플릿의 헤더/표 컬럼/순서를 임의로 바꾸지 않는다. 빈 셀과 플레이스홀더만 채운다.
- 산출물 작성으로 종료한다. 코드는 작성하지 않는다.

## 다음 단계

문서 검토 후 `/clear` 로 세션을 초기화하고 `/project-implement <slug>` 를 실행한다.
