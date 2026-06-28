# CONTRIBUTING

개인 저장소이지만 협업하듯 관리한다. 사람과 AI 에이전트(Claude · Codex 등) 모두 동일하게 따른다.

## 브랜치 흐름

`main` pull → 작업 브랜치 생성 → commit → push → PR → 리뷰 → merge.

작업 시작 전 항상 `main`을 pull 한다. 워크트리 기반 도구는 지시만으로 로컬 `main`에 반영되지 않을 수 있으므로 반영 여부를 확인한다.

## 브랜치 네이밍

`<type>/<short-topic>` 형식. 예: `docs/readme-polish`, `fix/cv-leakage`, `refactor/pipeline`.

`claude/*`·`codex/*` 같은 임의 브랜치는 만들지 않는다.

## 커밋 메시지

[Conventional Commits](https://www.conventionalcommits.org) 사용.

| type | 용도 |
|---|---|
| `feat` | 기능·모델 추가 |
| `fix` | 버그·누수·오류 수정 |
| `docs` | 문서·README 변경 |
| `refactor` | 동작 변화 없는 구조 개선 |
| `chore` | 설정·의존성·정리 |

## 분석 산출물 원칙

- 수치·실험 결과는 임의로 변경하지 않는다.
- 모든 지적·수정은 원본 데이터로 재현해 확인한다.
- 재현 불가능한 표·해석은 추가하지 않는다.
