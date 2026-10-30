# Notebooks

**최종 예측 모델**과 보고서 해석용 Notebook 안내.

| Notebook | Content |
|---|---|
| [Final Model](01_final_model_corrected.ipynb) | 전처리, 모델 비교, Nested CV, held-out 평가, 계수 추출과 최종 예측 |
| [Report Notes](02_report_annotated.ipynb) | 보고서 주석과 분석 해석 |

## Setup

- 저장소 루트에서 의존성 설치

```bash
pip install -r requirements.txt
```

- `data/raw/`에 `regression_train_public.csv`, `regression_test_private.csv` 배치
- `notebooks/`를 작업 폴더로 설정한 뒤 **Final Model을 위에서 아래로 실행**
- `RANDOM_STATE=42` 사용. 입력 형식은 [Data 안내](../data/README.md) 참조

## Outputs

- 실행한 작업 폴더에 `submission_train_corrected.csv`, `submission_test_corrected.csv`, `model_corrected.pkl` 생성
- **Private Test는 예측 전용**, 성능 평가는 학습 자료의 CV와 held-out으로 수행
- 저장된 모델 비교 수치는 [Results](../results/README.md) 참조

