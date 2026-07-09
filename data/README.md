# Data

Ames Housing (De Cock, 2011). 대회 원본 CSV는 라이선스상 저장소에 미포함(`.gitignore`).

학습 2,344행 · 31개 변수(수치형 16 · 범주형 15), 타깃 `sale_price`. Private 평가셋은 586행(라벨 없음, 최종 제출 전용). 모델링 단계에서 파생변수 2종(`total_sf`·`house_age`)이 추가된다.

| 구분 | 변수 | 처리 |
|---|---|---|
| 수치형 (15) | `overall_qual`·`overall_cond`·`gr_liv_area`·`lot_area`·`lot_frontage`·`total_bsmt_sf`·`year_built`·`year_remod_add`·`yr_sold`·`full_bath`·`half_bath`·`bedroom_abv_gr`·`garage_cars`·`fireplaces`·`open_porch_sf` | 결측 중앙값 대치 → StandardScaler |
| 순서형 (8) | `exter_qual`·`kitchen_qual`·`heating_qc`·`fireplace_qu`·`bsmt_qual`·`garage_finish`·`bsmt_fin_type1`·`bsmt_exposure` | 등급 매핑(OrdinalEncoder), 결측 최빈값 대치 |
| 명목형 (7) | `neighborhood`·`ms_zoning`·`garage_type`·`central_air`·`paved_drive`·`bldg_type`·`sale_condition` | 원-핫(OneHotEncoder, `min_frequency=0.01`로 희소범주 병합) |
| 파생변수 (2) | `total_sf`(`total_bsmt_sf`+`gr_liv_area`) · `house_age`(`yr_sold`-`year_built`) | 원본 컬럼 유지한 채 파이프라인에서 추가 |
| 타깃 | `sale_price` | 원본 스케일 유지(로그 변환 실험 후 미채택) |

구조적 결측(시설 없음을 의미)은 통계적 대치 전에 상수로 먼저 채운다: `fireplace_qu`·`garage_type`·`garage_finish` → `"None"`, `bsmt_qual`·`bsmt_exposure`·`bsmt_fin_type1` → `"NoBasement"`, `garage_cars`·`total_bsmt_sf` → `0`.

> 재현 시 원본 CSV(`regression_train_public.csv`·`regression_test_private.csv`)를 `data/raw/`에 배치. 전체 파이프라인: 루트 [`README.md`](../README.md).
