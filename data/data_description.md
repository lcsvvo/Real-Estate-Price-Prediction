# 데이터 설명 — Ames Housing

데이터셋: Ames Housing (De Cock, 2011). 대회 원본 CSV는 라이선스상 저장소에 포함하지 않습니다(.gitignore).

학습 2,344행 · 31개 변수 (수치형 16 · 범주형 15) · 타깃 `sale_price`.

| 구분 | 예시 변수 | 비고 |
|---|---|---|
| 수치형 | gr_liv_area, total_bsmt_sf, lot_area, overall_qual | 중앙값 대치 → 표준화 |
| 순서형 | exter_qual, kitchen_qual, bsmt_qual | 등급 매핑(OrdinalEncoder) |
| 명목형 | neighborhood, bldg_type, ms_zoning | 원-핫(min_frequency로 희소범주 처리) |
| 타깃 | sale_price | 원본 스케일 |
