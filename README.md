# Ames Housing Price Prediction
### 규제 선형모형으로 주택 가격을 예측하고, 모델 간 성능 차이가 유의미한지까지 검증

**31개 변수로 Ames 주택 가격을 예측하고, 후보 모델 간 차이가 실제로 의미 있는지 fold 변동성까지 함께 본 회귀 프로젝트.**

개인 프로젝트. 문제 정의부터 feature engineering · 모델 비교 · 비편향 성능 추정 · 해석까지 전 과정 수행.

![Data](https://img.shields.io/badge/data-Ames%20Housing%202.3K-blue)
![Task](https://img.shields.io/badge/task-price%20regression-blueviolet)
![Metric](https://img.shields.io/badge/metric-RMSE-informational)
![Method](https://img.shields.io/badge/method-Regularized%20Linear%20%2B%20FE-orange)
![Python](https://img.shields.io/badge/python-3776AB?logo=python&logoColor=white)

> 대회 원본 데이터는 라이선스상 미포함(`.gitignore`). 코드·설명·결과표만 공개. → [`data/README.md`](data/README.md)

---

## 요약

**질문.** 어떤 규제 선형모형이 Ames 주택 가격을 가장 잘 예측하며, 후보 간 차이는 유의한가.
**데이터.** Ames Housing 2,344행 × 31변수(수치형 16 · 범주형 15), 타깃 `sale_price`.
**방법.** 결측 대치 · 순서형/명목형 인코딩 · 표준화 · 파생변수(`total_sf`·`house_age`) → Ridge/Lasso/ElasticNet inner-CV 튜닝 → Nested CV 비편향 추정 → held-out 확인.
**결과.** ElasticNet CV RMSE 30,077(fold std 7,384) 선택. 단, 모델 간 차이는 fold 표준편차의 수% 이내로 노이즈 수준.
**역할.** 개인 프로젝트, 전 과정.

---

## 문제 정의

Ames Housing 데이터로 주택 판매가(`sale_price`)를 예측한다. 수치·순서·명목형이 섞인 31개 변수를 다루며, 목표는 최고 점수 자체가 아니라 재현 가능한 파이프라인에서 모델을 비교·선택하고 그 차이를 정직하게 해석하는 것.

---

## 데이터 이해

| 항목 | 내용 |
|---|---|
| 자료원 | Ames Housing (De Cock, 2011) |
| 규모 | train 2,344행 · 31변수(수치형 16 · 범주형 15) |
| 타깃 | `sale_price` — 우편향 분포(로그 변환 실험 → 미채택) |
| 변수 예 | 수치 `gr_liv_area`·`total_bsmt_sf`·`overall_qual` / 순서 `exter_qual`·`kitchen_qual` / 명목 `neighborhood`·`ms_zoning` |

> 변수 정의·전처리 매핑 상세: [`data/README.md`](data/README.md).

---

## 기술 스택

| 구분 | 도구 |
|---|---|
| 언어 | Python |
| 데이터 처리 | pandas, numpy |
| 모델·검증 | scikit-learn (Ridge, Lasso, ElasticNet, Pipeline, ColumnTransformer, GridSearchCV, Nested CV) |
| 시각화 | matplotlib, seaborn |

---

## Feature Engineering

| 단계 | 처리 |
|---|---|
| 결측 대치 | 수치형 중앙값 · 범주형 최빈값 (CV 내부 fit — 누수 차단) |
| 순서형 인코딩 | `exter_qual`·`kitchen_qual`·`bsmt_qual` 등급 매핑(OrdinalEncoder) |
| 명목형 인코딩 | `neighborhood`·`bldg_type`·`ms_zoning` 원-핫(`min_frequency`로 희소범주 병합) |
| 표준화 | StandardScaler |
| 파생변수 | `total_sf`(면적 합) · `house_age`(주택 연령) |

> 로그 변환·이상치 윈저라이징은 실험 후 미채택 — 근거는 Appendix.

---

## 모델 비교 및 선택

Ridge · Lasso · ElasticNet을 inner CV(GridSearchCV)로 튜닝하고, fold 표준편차와 함께 비교했다.

| 모델 | CV RMSE | fold std |
|---|---:|---:|
| **ElasticNet (선택)** | **30,077** | 7,384 |
| Ridge | 30,078 | 7,397 |
| Lasso | 30,259 | 7,013 |

> 모델 간 최대 차이(182)는 fold 표준편차(약 7,400)의 2.5% 수준 → 순위는 통계적 노이즈에 가깝다. ElasticNet은 성능 우위가 아니라 이중 규제(L1+L2)의 견고성을 근거로 선택.

---

## 성능 평가

- Nested CV — 하이퍼파라미터 선택과 성능 추정을 분리해 비편향 일반화 성능을 추정.
- Held-out 테스트 — 선택과 독립된 최종 확인.

---

## 주요 결과

- 세 규제 모형의 CV RMSE는 30,077–30,259로, 차이가 fold 변동성 안에 들어온다 → 모델 선택보다 feature engineering·검증 설계가 성능을 좌우.
- 변수 중요도는 전체 데이터로 재적합한 계수 기반으로 산출(노트북 §6, 결론은 §8). 입지(`neighborhood`)·건물 유형(`bldg_type`)·지상 거주면적(`gr_liv_area`)·전반적 품질(`overall_qual`) 순으로 가격에 영향을 미쳤다.

---

## 인사이트

- 후보 모델 간 성능차가 노이즈 수준이라는 것 자체가 결과 — "어떤 규제모형이냐"에 과의미 부여하지 않는다.
- 파생변수(`total_sf`)를 실제 학습에 반영했는지, 전처리를 CV 안에서 했는지가 성능·해석의 신뢰도를 결정.

---

## 역할

| 영역 | 기여 |
|---|---|
| 문제 정의 | 점수 경쟁이 아닌 "모델 비교·선택의 정직한 해석"으로 목표 설정 |
| Feature Engineering | 결측·인코딩·표준화·파생변수 설계 |
| 모델링 | Ridge/Lasso/ElasticNet inner-CV 튜닝, fold std 병기 |
| 검증 | Nested CV 비편향 추정 · held-out 확인 |

**정직한 경계.** 모델 간 성능차는 노이즈 수준으로 SOTA 주장 아님 · 관측 데이터 예측이며 인과 아님 · 원본 노트북의 결함을 수정한 재현본이 정본.

---

## Lessons Learned

모델을 완성한 뒤 직접 감사하여, 보고서가 설명하는 모델과 실제 학습 모델이 다르다는 점을 발견하고 누수 없는 재현 파이프라인으로 재구축했다.

| 발견 | 수정 |
|---|---|
| 파생변수(`total_sf`·`house_age`)가 최종 모델에 미반영 | 파이프라인에 실제 반영 |
| 결측 대치·스케일링을 CV 밖에서 fit(누수) | CV 내부로 이동 |
| 모델 선택 CV로 성능까지 보고(낙관 편향) | Nested CV로 비편향 추정 분리 |
| 변수 중요도 해석이 코드로 산출된 적 없음 | 실제 coefficient 산출 |

> 감사 전 과정: [`audit/audit_report.md`](audit/audit_report.md).

---

## Appendix

<details>
<summary><b>의사결정 기록</b></summary>

<br>

- 로그 변환 미채택 — `sale_price` 우편향에 log1p를 실험했으나 변별력·해석(금액 기준) 측면에서 원 스케일 유지.
- 윈저라이징 미적용 — 고가 주택을 이상치로 잘라내기보다 원본성 유지, 규제로 대응.
- 명시적 단계 vs Pipeline — 정정본은 누수 차단을 위해 전처리를 CV 파이프라인 안으로 통합.

원본·정정 보고서: [`reports/`](reports/).

</details>

<details>
<summary><b>재현</b></summary>

<br>

```bash
git clone https://github.com/jiwooo411/Real-Estate-Price-Prediction.git
cd Real-Estate-Price-Prediction
pip install -r requirements.txt
# data/README.md 안내대로 data/raw/ 에 원본 CSV 배치
jupyter notebook notebooks/01_final_model_corrected.ipynb
```

</details>

---

## 저장소 구조

```
.
├── notebooks/   # 01_final_model_corrected(정본) · 02_report_annotated(보고서 주석본)
├── data/        # 데이터 설명 (원본 미포함) → data/README.md
├── audit/       # 감사 보고서 (Lessons 근거)
├── reports/     # 보고서 원본·정정본 (PDF)
├── results/     # cv_results.csv · 예측 산출·그림
└── archive/     # 초기 과정·구버전 노트북 (비정본 보존)
```
