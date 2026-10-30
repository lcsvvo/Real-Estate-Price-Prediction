# Results

**저장된 모델 비교 결과**와 예측 산출물.

| File | Content |
|---|---|
| [cv_results.csv](cv_results.csv) | 기존 5-fold 모델별 평균 RMSE와 fold 표준편차 |
| [model_comparison.svg](model_comparison.svg) | CV 결과표를 시각화한 대표 그림 |
| [submission_test.csv](submission_test.csv) | 기존 제출 예측 |
| [model.pkl](model.pkl) | 기존 저장 모델 |
| [leaderboard_result.png](leaderboard_result.png) | 기존 제출 평가 화면 |
| [audit_flow.png](audit_flow.png) | 모델 검토 흐름 |

## CV Scores

| Model | RMSE | Fold SD |
|---|---:|---:|
| ElasticNet | **30,077** | 7,384 |
| Ridge | 30,078 | 7,397 |
| Lasso | 30,259 | 7,013 |

- **Fold SD는 fold별 점수의 표준편차**, 신뢰구간이 아님
- 이 표는 **Nested CV 결과와 구분** 필요
- 최종 Notebook 재실행 산출물은 `_corrected` 파일명으로 생성. 기존 저장 파일과 구분 필요
- 실행 안내는 [Notebooks](../notebooks/README.md) 참조

