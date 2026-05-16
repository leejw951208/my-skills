# project-verify

기능 검증 단계 스킬. 구현이 명세와 계획을 만족하는지 코드 리뷰, 기능 검증, 보안 감사를 통합 수행하고 결과를 `review.md` 로 정리한다.

## 설정 방법

Codex CLI 글로벌 또는 프로젝트 단위로 배치한다. 심볼릭 링크가 권장된다(원본 갱신이 자동 반영됨).

**글로벌 설치.** 모든 프로젝트에서 사용.

```bash
mkdir -p ~/.codex/skills
ln -s "$(pwd)/project-verify" ~/.codex/skills/project-verify
# 또는: cp -r project-verify ~/.codex/skills/project-verify
```

**프로젝트 단위 설치.** 해당 레포에서만 사용.

```bash
mkdir -p .codex/skills
ln -s "$(pwd)/project-verify" .codex/skills/project-verify
```

배치 후 Codex CLI 세션에서 `/project-verify` 으로 호출한다.

## 의존 스킬

| 스킬 | 호출 시점 | 필수 여부 |
|------|----------|-----------|
| `review` (gstack) | 코드 리뷰 단계 | 필수 |
| `qa-only` (gstack) | 기능 검증 단계 | 필수 |
| `cso` (gstack) | 보안 감사 단계 | 필수 |

`spec.md`, `plan.md`, 구현 완료된 소스 코드가 사전 입력으로 필요하므로 `/project-implement` (또는 `/project-patch`) 의 실행 결과에 의존한다.

## 사용법

```
/project-verify <feature-slug>
```

- `phase.md` 가 `implemented` 가 아니면 그대로 진행할지 묻는다.
- 기존 `review.md` 의 OPEN 항목이 0건이면 재실행 여부를 묻는다.
- `spec.md` 또는 `plan.md` 가 없으면 중단하고 `/project-plan` 실행을 안내한다.

## 동작 흐름

1. **대상 확인.** slug 검증 후 `spec.md`, `plan.md`, `phase.md` 를 로드한다.
2. **준비.** `phase.md` 를 `verifying` 으로 표시한다.
3. **코드 리뷰.** gstack `review` 스킬 결과에 더해 요구사항별 구현 상태를 판정한다. (`DONE` / `PARTIAL` / `NOT DONE` / `CHANGED` / `SCOPE_CREEP` / `UNTESTED`).
4. **기능 검증.** gstack `qa-only` 스킬로 실제 동작을 확인한다.
5. **보안 감사.** gstack `cso` 스킬로 위협 모델을 점검한다.
6. **산출물 저장.** `templates/review.md` 구조 그대로 결과를 정리한다.
7. **완료 보고.** OPEN 항목 수에 따라 다음 단계(보강 또는 신규 계획)를 안내한다.

## 검증 규칙

- 본 단계에서는 신규 기능 코드를 작성하지 않는다.
- `spec.md`, `plan.md` 는 입력 전용으로 처리한다.
- 결함은 `review.md` 의 OPEN 항목으로 기록만 한다. 수정은 `/project-patch` 에서 처리한다.
- 파일:라인 근거 없이 `DONE` 판정하지 않는다.

## 산출물

| 파일 | 역할 |
|------|------|
| `docs/features/<slug>/review.md` | Spec 일치, Plan 일치, 테스트 커버리지, 발견 항목, 기능 검증, 보안 감사. |
| `docs/features/<slug>/phase.md` | 단계 표시. `verifying` → `verified`. |

## 다음 단계

- OPEN 항목 ≥ 1. `/clear` 후 `/project-patch <slug>` 로 보강 → 본 스킬 재실행.
- OPEN 항목 = 0. 검증 완료. 새 기능은 `/project-plan` 으로 시작.
