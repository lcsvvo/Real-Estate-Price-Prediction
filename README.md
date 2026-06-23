# 🔍 Ames Housing — An Audit & Reproducibility Case Study

> 회귀 모델을 만든 뒤, **내 결과를 스스로 감사(Audit)하여 코드–보고서 불일치·dead code·해석 오류를 발견하고 재현 가능한 형태로 수정한** 기록입니다. 이 저장소의 핵심은 점수가 아니라 **신뢰할 수 있는 분석 과정**입니다.

태그 규칙: `[R]` 코드·데이터로 입증된 결과 · `[I]` 데이터가 시사하는 해석 · `[L]` 배운 점. (입증 불가한 Speculation은 수록하지 않음)

---

## 1. 프로젝트 개요

`[R]` Ames Housing(아이오와) 데이터로 주택 가격을 예측하는 규제 선형회귀 모델을 구축했습니다.
`[R]` 초기 목표는 예측 성능이었으나, 완성 후 재검증 과정에서 구조적 결함을 발견하여 **목표가 "결과 재검증·수정"으로 전환**되었습니다.

## 2. 문제 정의

`[R]` 다양한 주택 속성(품질·면적·입지 등)으로 `sale_price`를 예측. 평가지표는 대회 기준 M(R² 계열) 및 RMSE.
`[I]` 주택 가격은 비선형·이상치가 많아, 삭제보다 **규제로 영향을 흡수**하는 접근이 적절하다고 판단했습니다.

## 3. 데이터 설명

`[R]` 학습 2,344행 · 31개 변수(수치형 16 · 범주형 15).

| 구분 | 예시 | 처리 |
|---|---|---|
| 수치형 | `gr_liv_area`, `total_bsmt_sf`, `overall_qual` | 중앙값 대치 → 표준화 |
| 순서형 | `exter_qual`, `kitchen_qual`, `bsmt_qual` | 등급 매핑 (`OrdinalEncoder`) |
| 명목형 | `neighborhood`, `bldg_type`, `ms_zoning` | 원-핫 (`min_frequency`로 희소범주 처리) |

`[R]` 결측은 구조적(시설 없음 → `None`/`0`)과 비구조적(동네별·Train 기준 중앙값/최빈값)으로 구분 처리. 모든 통계는 **Train 기준으로만** 산출.

## 4. EDA

`[R]` 타깃 `sale_price`는 양의 왜도(약 1.75)로 우측 꼬리가 김.
`[R]` 수치형 중 `overall_qual`, `gr_liv_area`가 타깃과 높은 상관.
`[I]` 일부 고가 주택은 이상치가 아니라 예측해야 할 실제 패턴으로 보여, 삭제 대신 규제로 대응했습니다.

## 5. 모델링

`[R]` Ridge·Lasso·ElasticNet을 5-Fold CV(`shuffle=True, random_state=42`)로 `GridSearchCV` 튜닝.

| 모델 | CV RMSE | fold 표준편차 |
|---|:---:|:---:|
| ElasticNet (α=0.01, l1=0.5) | 30,077 | ± 7,384 |
| Ridge (α=10) | 30,078 | ± 7,397 |
| Lasso (α=1.0) | 30,259 | ± 7,013 |

`[R]` 모델 간 평균 차이(약 180) ≪ fold 표준편차(약 7,300) → 세 모델은 **통계적으로 동등**.
`[I]` 최종 모델은 성능 우위가 아니라 **단순성·계수 안정성**을 근거로 ElasticNet을 선택. ("최고/가장 견고"라는 주장은 데이터가 뒷받침하지 않아 하지 않습니다.)

## 6. Audit 과정

`[R]` 완성된 노트북을 데이터 누수·교차검증 설계·코드 정합성 관점에서 재검토했습니다. 검토 대상: 전처리, 변수 생성, 모델링, 교차검증, 리더보드 결과, 보고서 서술.

## 7. 발견된 문제

| # | 문제 | 검증 | 판정 |
|---|---|---|---|
| 1 | 파생변수(`total_sf`,`house_age`)가 변수 덮어쓰기로 **제출 모델에서 누락** | 제출 파일 74 feature(파생 부재) 확인 | `[R]` CONTRADICTED |
| 2 | 희귀범주(Rare) 병합이 `df_train`만 수정 → **dead code** | `X_train`에 `'Rare'` 없음 | `[R]` CONTRADICTED |
| 3 | 변수중요도 해석에 **산출 코드 없음** | 전 셀 검토 | `[R]` UNSUPPORTED |
| 4 | 전처리가 CV 밖에서 fit → **스케일링 누수** | 누수 전후 CV 비교(영향 약 50 RMSE) | `[R]` 사실, 영향 소 |
| 5 | **버전 불일치:** `과정기록`(파생 포함) vs 제출 모델(파생 미포함) | 두 파일 비교 | `[R]` CONTRADICTED |

`[R]` 즉, Private 2위 성과는 **파생변수 없이** 달성된 것이며, 그 정보는 원천 변수(`gr_liv_area` 등)에 이미 담겨 있었습니다.

## 8. 수정 내용

| 항목 | Before | After |
|---|---|---|
| 파생변수 | 모델에서 누락 | `FunctionTransformer`로 Pipeline에 반영 |
| 희귀범주 | dead code | `OneHotEncoder(min_frequency=0.01)` |
| 전처리 | CV 밖 fit (누수) | `Pipeline`/`ColumnTransformer` 내부 fit |
| 성능 보고 | 그리드 최솟값(낙관) | Nested CV + held-out |
| 변수중요도 | 근거 코드 없음 | 계수 직접 산출 |
| 재현성 | 비멱등 | 멱등(Run-All 안정) |

## 9. 최종 결과

`[R]` Private Test M **0.8708 / 전체 2위 / Train–Test Gap 1.0%** (대회 리더보드).
`[R]` 누수 제거 후 CV RMSE ≈ 30,100 (±7,300).
`[R]` 실제 가격 결정 요인(계수 기준): **입지(Neighborhood) > 건물 유형 > 지상 면적 > 전반 품질**.
`[I]` 동일 과제에서 한 팀이 Gap 86%로 붕괴한 것과 달리 Gap 1.0% → 안정적 일반화의 근거.

## 10. 배운 점

`[L]` CV 점수 차이는 fold 표준편차와 함께 해석해야 한다 — 작은 차이로 우열을 주장하면 안 된다.
`[L]` 좋은 결과 ≠ 정직한 보고서. 코드가 보고서대로 동작하는지는 별도 검증이 필요하다.
`[L]` `Pipeline`/`ColumnTransformer`로 데이터 누수를 *구조적으로* 차단하는 법.
`[L]` Private Test의 작은 Gap이 일반화의 진짜 증거다.

## 11. 재현성

`[R]` 원본 CV RMSE 30,129.947을 소수점까지 재현, 저장 모델(`model.pkl`) reload 예측 일치 확인.
`[R]` 누수 없는 대표 노트북은 Run-All로 동일 결과를 산출(멱등).
환경: Python · pandas · scikit-learn. `requirements.txt` 참조.

## 12. 디렉토리 구조

```
Housing-Price-Audit/
├── README.md
├── requirements.txt
├── notebooks/
│   ├── 01_final_model_corrected.ipynb   # 대표: 누수 없는 재구현
│   └── 02_report_annotated.ipynb        # 원본 보고서 + Audit 정정
├── audit/
│   └── audit_report.md
├── reports/
│   └── process_log.ipynb                # 탐색 기록(참고)
├── results/
│   ├── submission_test.csv
│   └── model.pkl
└── data/
    └── data_description.md              # 원본 데이터는 비공개(.gitignore)
```

---
*모든 수치는 실제 데이터로 재현·검증됨. 데이터셋: Ames Housing (De Cock, 2011). 대회 원본 데이터는 라이선스상 저장소에 포함하지 않습니다.*
