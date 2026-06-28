# Ames Housing Price Prediction
### 내가 만든 모델을 스스로 감사하다 — 코드–보고서 불일치를 찾아 고친 기록

**The model in the report was not the model I submitted. I found it by auditing my own work.**

Ames Housing 주택 가격 회귀 프로젝트. 모델을 완성한 뒤 직접 감사(audit)해 코드–보고서 불일치·dead code·데이터 누수를 찾아내고, 재현 가능한 형태로 수정한 기록

![Data](https://img.shields.io/badge/data-Ames%20Housing-blue)
![Task](https://img.shields.io/badge/task-regression-blueviolet)
![Metric](https://img.shields.io/badge/metric-RMSE%20·%20R²-informational)
![Result](https://img.shields.io/badge/private-2위%20·%20gap%201.0%25-success)
![Python](https://img.shields.io/badge/python-3.x-3776AB?logo=python&logoColor=white)

<p align="center">
  <img src="results/leaderboard_result.png" width="760" alt="Private 리더보드 2위, Train-Test Gap 1.0%"><br>
  <sub><b>Figure 1.</b> Private Test M <b>0.8708 · 전체 2위 · Train–Test Gap 1.0%</b>. 단, 감사 결과 이 2위는 보고서가 "핵심"이라 설명한 파생변수 <b>없이</b> 달성된 것 — 그 정보는 이미 원천 변수에 담겨 있었다.</sub>
</p>

> 대회 원본 CSV는 라이선스상 미포함(`.gitignore`). 코드·노트북·감사 보고서·결과표만 공개. → [`data/data_description.md`](data/data_description.md)

---

## 30초 요약

**질문.** 좋은 리더보드 점수가 곧 신뢰할 수 있는 분석인가. 내가 보고한 모델은 실제로 내가 제출한 모델과 같은가.

**핵심 발견.**
- Private 2위(M 0.8708, Gap 1.0%)는 보고서가 "핵심"이라던 파생변수 **없이** 달성 — 제출 모델은 74 feature, 파생변수 부재.
- 희귀범주 병합은 **dead code**였고, 변수중요도 해석은 산출 코드가 **없었다**.
- 전처리를 CV 밖에서 fit한 **스케일링 누수** 존재(영향은 소, 약 50 RMSE).
- 5개 지적 중 3개 CONTRADICTED · 1개 UNSUPPORTED — 모두 원본 CSV로 재현해 확인.

**방법.** 규제 선형회귀(Ridge·Lasso·ElasticNet) 5-Fold CV 튜닝 → 자기 감사(누수·CV 설계·코드 정합성) → `Pipeline`/`ColumnTransformer`로 구조적 수정.

**의미.** 이 저장소의 핵심은 점수가 아니라 **코드가 보고서대로 동작하는지 검증하고 고치는 과정**이다.

## Role

- **Individual Project** — 모델링·감사·수정 전 과정 단독 수행
- **Goal Shift** — 초기 목표(예측 성능) → 결과 재검증·수정으로 전환
- **End-to-End** — 모델링 → Self-Audit → 구조적 수정

---

## 왜 중요한가

대부분의 포트폴리오는 가장 좋은 결과만 보여준다. 이 프로젝트는 반대로, 이미 2위를 한 모델을 **스스로 가장 보수적인 기준으로 다시 검토**한 기록이다.

감사 기준은 "교수가 데이터 누수와 교차검증 설계를 가장 엄격하게 본다"는 가정이었다. 기존 분석을 방어하지 않고, 모든 지적을 원본 CSV로 재현해(원본 CV RMSE 30,129.947을 소수점까지 복원) 경험적으로 확인했다.

결과적으로 헤드라인 모델이 보고서가 설명하는 모델과 달랐다. 좋은 점수가 정직한 보고서를 보장하지 않는다는 것, 그리고 그 간극을 어떻게 찾아 고치는지가 이 프로젝트가 보여주려는 역량이다.

---

<details>
<summary><b>데이터</b></summary>

<br>

| 항목 | 내용 |
|---|---|
| 데이터셋 | Ames Housing (De Cock, 2011) — 아이오와 주택 거래 |
| 규모 | 학습 2,344행 · 31개 변수 (수치형 16 · 범주형 15) · 타깃 `sale_price` |
| 타깃 분포 | 양의 왜도(약 1.75), 우측 꼬리 — 고가 주택은 이상치가 아닌 실제 패턴으로 보고 삭제 대신 규제로 대응 |
| 결측 처리 | 구조적(시설 없음 → `None`/`0`)과 비구조적(Train 기준 중앙값/최빈값) 구분, 모든 통계는 **Train 기준** |

> 변수 처리 상세: [`data/data_description.md`](data/data_description.md).

</details>

---

## 기술 스택

| 구분 | 도구 |
|---|---|
| 언어 | Python 3.x |
| 데이터 처리 | pandas · numpy |
| 모델·검증 | scikit-learn — Ridge · Lasso · ElasticNet · `Pipeline` · `ColumnTransformer` · `GridSearchCV` |
| 직렬화 | joblib (`model.pkl`) |
| 시각화 | matplotlib · seaborn |

---

<details>
<summary><b>모델링 — 세 모델은 통계적으로 동등</b></summary>

<br>

Ridge·Lasso·ElasticNet을 5-Fold CV(`shuffle=True, random_state=42`) `GridSearchCV`로 튜닝.

| 모델 | CV RMSE | fold 표준편차 |
|---|:---:|:---:|
| ElasticNet (α=0.01, l1=0.5) | 30,077 | ± 7,384 |
| Ridge (α=10) | 30,078 | ± 7,397 |
| Lasso (α=1.0) | 30,259 | ± 7,013 |

모델 간 평균 차이(약 180) ≪ fold 표준편차(약 7,300) → 세 모델은 통계적으로 동등. 최종 모델은 성능 우위가 아니라 **단순성·계수 안정성**을 근거로 ElasticNet 선택. ("최고/가장 견고"라는 주장은 데이터가 뒷받침하지 않아 하지 않음.)

</details>

---

## 자기 감사 (Self-Audit)

이 프로젝트의 핵심. 완성된 노트북을 데이터 누수·교차검증 설계·코드 정합성 관점에서 재검토하고, 모든 지적을 원본 CSV로 재현해 판정했다.

### 발견된 문제

| # | 문제 | 검증 | 판정 |
|---|---|---|---|
| 1 | 파생변수(`total_sf`,`house_age`)가 변수 덮어쓰기로 제출 모델에서 누락 | 제출 파일 74 feature(파생 부재) 확인 | CONTRADICTED |
| 2 | 희귀범주(Rare) 병합이 `df_train`만 수정 → dead code | `X_train`에 `'Rare'` 없음 | CONTRADICTED |
| 3 | 변수중요도 해석에 산출 코드 없음 | 전 셀 검토 | UNSUPPORTED |
| 4 | 전처리가 CV 밖에서 fit → 스케일링 누수 | 누수 전후 CV 비교(영향 약 50 RMSE) | 사실, 영향 소 |
| 5 | 버전 불일치: `과정기록`(파생 포함) vs 제출 모델(파생 미포함) | 두 파일 비교 | CONTRADICTED |

> 즉, Private 2위 성과는 파생변수 없이 달성된 것이며 그 정보는 원천 변수(`gr_liv_area` 등)에 이미 담겨 있었다. 전체 감사 보고서: [`audit/audit_report.md`](audit/audit_report.md).

### 수정 내용

| 항목 | Before | After |
|---|---|---|
| 파생변수 | 모델에서 누�