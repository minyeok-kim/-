# 🚗 송파구 불법주차 데이터 분석 (Songpa Illegal Parking Analysis)

> **"불법주차는 도시 공간 구조의 문제다"**
> 서울 송파구 27개 행정동의 5대 환경요인을 통합 분석해 핫스팟의 구조적 원인을 규명하고, 정량적 솔루션을 도출한 공간 데이터 분석 프로젝트

[![R](https://img.shields.io/badge/R-4.x-276DC3?style=flat&logo=r&logoColor=white)](https://www.r-project.org/)
[![Spatial Analysis](https://img.shields.io/badge/Spatial-Analysis-2E7D32?style=flat)]()
[![Status](https://img.shields.io/badge/Status-Completed-success?style=flat)]()

---

## 📌 프로젝트 한 줄 요약

서울시 불법주정차 단속 데이터 + 도로망 + POI + 야간조도 + 토지용도 엔트로피를 통합해, **송파구 핫스팟 3곳의 공통 구조를 PCA + K-means + ANOVA로 통계적으로 증명**하고 IoT 기반 솔루션을 제안했습니다.

---

## 🎯 핵심 결과

| 지표 | AS-IS (현재) | TO-BE (목표) | 변화 |
|------|--------------|--------------|------|
| 연간 불법주차 (Cluster 1) | 7,621건 | **4,496건** | **-41%** |
| 주차 탐색 시간 | 20분 | **10분** | **-50%** |
| 주차면 회전율 | 2.5회 | **3.0회** | **+20%** |

> **통계적 검증**: One-way ANOVA 결과 F = 5.93, p = 0.0038 → 클러스터 간 차이 유의

---

## 🏆 분석의 차별화 포인트

1. **2축 엔트로피 정의** — 기존 연구가 동 단위 수평 엔트로피만 측정하는데, **건물 내부 층별 용도 변화를 측정하는 수직 엔트로피를 면적 가중평균으로 추가 정의**
2. **정책 이전 가능성(Policy Transferability) 검토** — 솔루션 제안에서 그치지 않고 인용 사례(부천/은평/서울)의 공간 구조와 송파구 Cluster 1의 유사성을 검증
3. **방법론 비교 검증** — POI 분석 시 "행정동 직접 매핑" vs "격자 기반 집계"를 둘 다 시도하고 비교 후 최종 채택
4. **가설 반증을 분석 확장의 근거로 활용** — 엔트로피 가설이 부분적으로 안 맞는 것을 인지하고 다변량 분석으로 정당화

---

## 📁 프로젝트 구조

```
songpa-illegal-parking/
├── README.md
├── data/
│   ├── songpa_filtered.csv              # 송파구 불법주차 단속 데이터
│   ├── seoul_dong.geojson               # 서울 행정동 경계
│   ├── MOCT_LINK.shp / MOCT_NODE.shp    # 국가교통DB 도로망
│   ├── songpa_building.csv              # 건축물대장
│   ├── songpa_cctv.csv                  # CCTV 위치
│   └── songpa_night_light.csv           # 야간조도
├── src/
│   ├── 01_kde_hotspot.R                 # 4주차: KDE 핫스팟 도출
│   ├── 02_road_features.R               # 4주차: 도로 Feature Engineering
│   ├── 03_night_light_zscore.R          # 5주차: 야간조도 분석
│   ├── 04_cctv_density.R                # 5주차: CCTV 밀도 분석
│   ├── 05_entropy_analysis.R            # 6주차: 토지용도 엔트로피
│   ├── 06_poi_analysis.R                # 7주차: POI 밀도 + 클러스터링
│   ├── 07_pca_kmeans.R                  # 8주차: 통합 PCA + K-means
│   └── 08_anova_test.R                  # 8주차: ANOVA 가설검정
├── output/
│   ├── songpa_dong_summary.csv
│   ├── songpa_entropy_by_dong.csv
│   └── Songpa_Full_Clustering_Result.csv
└── docs/
    └── final_presentation.pdf
```

---

## 🔬 분석 파이프라인 (8주차 흐름)

```
[문제 정의]
   ↓
4주차: KDE로 핫스팟 도출 (잠실본동/방이2동/가락본동)
       + 도로 데이터 Feature Engineering (3개 파생변수)
   ↓
5주차: 야간조도 + CCTV 분석 → "핫스팟별 다른 처방 필요"
   ↓
6주차: 토지용도 엔트로피 (수평·수직 2축)
       + 주차장 부족 가설 기각 (확보율 138.9%)
   ↓
7주차: POI 밀도 + 격자 vs 직접매핑 방법론 비교
   ↓
8주차: 5대 환경요인 통합 (PCA + K-means, k=4)
       + ANOVA 검정 (F=5.93, p<0.004)
   ↓
[결론] 회전율 관리 솔루션 + 정책 이전 가능성 검토
```

---

## 📊 주차별 핵심 내용

### Week 4 — 핫스팟 도출 + 도로 데이터 분석

- **트러블슈팅**: 주소 문자열 필터링 시 "올림픽로"처럼 여러 구에 걸친 도로명 문제 → **위경도 기반 공간 필터링**으로 전환
- **KDE 핫스팟**: `spatstat::density.ppp()` (sigma=200, eps=20, edge=TRUE) → 잠실본동/방이2동/가락본동 도출
- **메모리 최적화**: 전국 도로 데이터(114,450건)를 SQL query로 서울권만 필터링 후 로드
- **파생변수 3종 정의**:
  - 도로 밀도 = TotalRoadLength / Area
  - 이면도로 비율 = BackRoadLength / TotalRoadLength × 100
  - 교차로 밀도 = NodeCount / Area

### Week 5 — 야간조도 + CCTV 분석

- **프록시 변수 활용**: 직접 측정 불가능한 '상업 밀집도'를 **야간조도**로 대체
- **공간 조인 + Z-score**: 송파구 27개 행정동 기준 정규화
- **결정적 인사이트**:

| 핫스팟 | 불법주차 | CCTV 밀도 Z | 진단 |
|--------|----------|-------------|------|
| 가락본동 | Hotspot | +1.49 | 🔴 **단속의 한계** |
| 방이2동 | Hotspot | +0.37 | 🟡 **인프라 부족** |
| 잠실본동 | Hotspot 1위 | -0.03 | ⚫ **단속 사각지대** |

> 같은 핫스팟이어도 처방이 달라야 한다는 정책적 함의 도출

### Week 6 — 토지용도 엔트로피 분석

- **2축 정의** (학부생 수준 차별화):
  - 수평 엔트로피: 동 단위 용도 다양성
  - 수직 엔트로피: 건물 내부 층별 용도 변화 (면적 가중평균)
- **수식**: $H = -\sum_{j=1}^{k} P_j \log(P_j)$
- **결과**: 4분면 분류 (고수평/고수직 = 문정/방이/석촌)
- **가설 반증**: 핫스팟 3곳 중 가락본동이 엔트로피 최하위 → "단일 지표 한계"
- **가설 기각**: 주차장 확보율 138.9% → "물리적 부족" 가설 기각

### Week 7 — POI 밀도 + 클러스터링

- **POI 정의**: 상가, 오피스텔, 식당, 카페, 병원 등 유동인구 발생 시설
- **방법론 비교 시도**:
  - 1차: 단속 좌표를 행정동에 직접 매핑 → **면적 정확** ✅
  - 2차: 100m 격자 기반 집계 → 시각적 깔끔하지만 면적 오차
  - **최종**: 면적 정확성 우선해 1차 채택
- **로그-로그 산점도**: POI ↑ → 단속 ↑ 우상향 추세 검증
- **K-means (k=4)**: Cluster 4(POI Z=+1.69, 단속 Z=+2.12) = 가설 부합

### Week 8 — 통합 PCA + K-means + ANOVA

#### PCA 결과 해석 (단순 차원축소를 넘어 비즈니스 의미 부여)

| 축 | 의미 | 양의 방향 | 음의 방향 |
|----|------|----------|-----------|
| **PC1** | 도시 복잡도 / 상권 중심성 | 잠실본동, 송파1동, 문정1동 | 잠실2/3/4/7동, 오륜동 |
| **PC2** | 공간 구조적 특성 | 장지동, 위례동 (계획도시) | 풍납1·2동, 마천동 (이면도로) |

#### K-means 4개 클러스터

| 클러스터 | 별칭 | 특징 |
|----------|------|------|
| 🔴 Cluster 1 | **상권 중심 불법주차 Hotspot** | 불법주차↑↑, POI↑↑, 조도↑↑ — 핫스팟 3곳 |
| 🟣 Cluster 4 | 오밀조밀 주거 밀집지역 | 교차로 밀도 ↑↑ |
| 🟢 Cluster 2 | 도로망 발달 지역 | 도로 밀도 ↑, 불법주차 ↓ |
| 🔵 Cluster 3 | 대단지 아파트 Safe Zone | 모든 수치 평균 이하 |

#### ANOVA 가설검정

```
H₀: 클러스터별 불법주차 건수 평균에 차이 없음
H₁: 적어도 한 클러스터의 평균은 다름

F_critical (α=0.05) = 3.028
F_statistic         = 5.9268
p-value             = 0.0037756

→ F_stat > F_crit, p < 0.05 → H₀ 기각
→ 클러스터 분류는 통계적으로 유의미함
```

---

## 💡 최종 결론 & 솔루션

### 핵심 인사이트

> **불법주차는 도로가 좁아서가 아니라, 도로 용량 대비 상권(POI) 수요가 폭발적이고 야간 활동(조도)이 많아 외부 유입 차량이 끊이지 않기 때문이다.**

→ 물리적 주차장 확충이 아닌 **회전율 관리**가 핵심

### 솔루션 (Cluster 1 타겟)

| 솔루션 | 효과 | 근거 |
|--------|------|------|
| AI 기반 스마트 단속 | 회전율 +20% | KOTI 한국교통연구원 |
| IoT 공유주차 | 불법주차 -41% | 서울시 공유주차 실증사업 |
| 스마트 로딩존 | 탐색시간 -50% | 국토부 IoT 센서 도입 결과 |

### 정책 이전 가능성(Policy Transferability) 검토

| 인용 사례 지역 | 공간 특성 | Cluster 1 유사성 |
|----------------|-----------|------------------|
| 부천시 | 상업 + 주거 복합형 | ✅ |
| 은평구 | 노후 주거 + 상권 혼재 | ✅ |
| 서울시 일반 | 도시 평균 환경 | ✅ |

> 신도시(평지)나 산악지역이 아닌 **도시 복합형 구조**라는 공통점 확인 → 유사 효과 기대 가능, 단 사전 시범사업 필요

---

## 🛠️ 사용 기술 / 라이브러리

### Spatial / GIS
- `sf` — Simple Features for spatial data
- `spatstat` — Kernel Density Estimation
- `stars` — Spatiotemporal arrays

### Data Analysis
- `dplyr` — Data manipulation
- `stringr` — String processing (행정동 정규식 추출)

### Visualization
- `ggplot2` + `viridis` — 정적 시각화
- 행정동 centroid 라벨 오버레이

### Statistical Methods
- Z-score 표준화 (`scale()`)
- PCA (Principal Component Analysis)
- K-means clustering (k=4, nstart=20)
- One-way ANOVA (`aov()`)

### 좌표계
- EPSG:4326 (WGS84) → EPSG:5179 / 5181 (Korea Central Belt)

---

## 📚 데이터 출처

- 서울시 불법주정차 단속 정보 (서울 열린데이터광장)
- 국가교통DB MOCT_LINK / MOCT_NODE
- 송파구 야간조도 (공공데이터포털)
- 송파구 건축물대장 (건축행정시스템 세움터)
- 서울시 상가/오피스텔 정보 (소상공인시장진흥공단)
- 송파구 통계연보

---

## 🎓 프로젝트 회고

이 프로젝트를 통해 단순한 데이터 분석을 넘어:

1. **"왜 이 변수인가"** — 직접 측정 불가능한 변수를 프록시(야간조도)로 대체하는 사고
2. **"왜 이 방법인가"** — 격자 vs 직접매핑 비교, K-means vs DBSCAN 검토 등 방법론적 정직성
3. **"왜 이 결론인가"** — 단순 EDA가 아닌 ANOVA로 통계적 정당성 확보
4. **"이 결론을 어디까지 신뢰할 수 있는가"** — 정책 이전 가능성 검토 + 한계 명시

라는 데이터 분석가의 핵심 사고를 적용했습니다.

---

## 👥 팀 구성

- **이원재** (홍익대학교 소프트웨어융합학과 / 산업데이터공학과 복수전공)
  - 도로 데이터 Feature Engineering (4주차)
  - 통합 PCA + K-means 클러스터링 (8주차)
  - ANOVA 가설검정 + 정책 이전 가능성 검토 (8주차)
  - 최종 시뮬레이션 시각화

- 팀원 3명: POI 분석, 야간조도 분석, 엔트로피 분석 등 분담

---

## 📬 Contact

- Portfolio: [https://wonjae1230.github.io/](https://wonjae1230.github.io/)
- GitHub: [@wonjae1230](https://github.com/wonjae1230)
