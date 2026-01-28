# TSUI 지수 산출 패키지

트럼프 지수(TSUI, Trump Statement Uncertainty Index) 산출을 위한 모듈화된 Python 패키지입니다.

# 의존성 설치
pip install -r requirements.txt
```

## 요구사항

- Python 3.8+
- numpy >= 1.24.0
- pandas >= 2.0.0

## 패키지 구조

```
index_generator/
├── __init__.py              # 패키지 초기화 및 export (run_analysis)
├── main.py                  # 메인 실행 스크립트
├── pipeline.py              # 전체 공정 오케스트레이션
├── utils.py                 # 공통 유틸리티 함수 (parse_datetime)
├── data/                    # [Data Layer]
│   ├── __init__.py
│   ├── constants.py         # 상수 정의
│   ├── data_loader.py       # 데이터 로딩 및 캐싱
│   └── src_data/            # 원본 데이터 파일 (CSV, JSON)
│       ├── track_01/        # Track 1 데이터
│       ├── track_02/        # Track 2 데이터
│       ├── etf_sector/       # ETF 가격 데이터
│       ├── DXY.csv          # 달러 인덱스
│       └── ...
├── engine/                  # [Engine Layer]
│   ├── __init__.py
│   ├── math_engines.py      # 순수 수학 계산 함수
│   └── index_calculator.py  # TSUI 지수 계산 함수
├── logic/                   # [Logic Layer]
│   ├── __init__.py
│   └── logic_helpers.py     # 데이터 가공 및 그룹화 함수
└── docs/                    # 문서 폴더
```

## 모듈 설명

### `data/` 패키지 (Data Layer)

#### `data/constants.py`
- 트럼프 재임 기간 정의
- 잔상 상수 (α) - 강도별
- 시간 가중치 설정
- 키워드 가중치 및 키워드 사전 (1기/2기 분리)
- Track 2 섹터명 매핑
- TSUI 지수 계산 상수 (Vol_Proxy, 로그 스케일링, EMA 등)

#### `data/data_loader.py`
- `load_track1_data()`: Track 1 데이터 로드
- `load_track2_data()`: Track 2 데이터 로드
- `load_etf_metadata()`: ETF 메타데이터 로드
- `load_all_etf_data()`: 모든 ETF 데이터 로드
- `load_dxy_data()`: DXY 데이터 로드
- `get_volatility_data()`: Volatility 데이터 조회
- `extract_sector_from_label_short()`: 섹터명 추출
- `extract_intensity_from_label_short()`: 강도 레벨 추출
- `get_daily_sector()`: 일별 섹터 정보 추출
- `initialize_data()`: 모든 데이터 로드 및 캐싱
- `get_track1_data()`, `get_track2_data()` 등: 캐시된 데이터 조회

### `utils.py` (루트)
- `parse_datetime()`: 날짜 문자열 파싱 (공통 유틸리티)

**참고**: TSUI 도메인 특화 함수들은 `logic/logic_helpers.py`로 이동했습니다.

### `engine/` 패키지 (Engine Layer)

#### `engine/math_engines.py`
- `calculate_cumulative_energy()`: 누적 에너지 계산
- `calculate_log_scaled_values()`: 로그 스케일링 적용
- `normalize_to_baseline()`: Baseline 기준 정규화
- `apply_ema_smoothing()`: EMA 적용
- `renormalize_after_ema()`: EMA 후 재정규화

#### `engine/index_calculator.py`
- `calculate_tsui_index()`: 최종 TSUI 지수 계산

### `logic/` 패키지 (Logic Layer)

#### `logic/logic_helpers.py`
**데이터 가공 및 그룹화:**
- `calculate_daily_raw_energy()`: 일일 원천 에너지 계산
- `calculate_daily_energies()`: 전체 기간 일일 에너지 계산
- `group_posts_by_date()`: 일별 포스트 그룹화
- `generate_all_dates()`: 전체 기간 날짜 생성
- `extract_daily_sectors()`: 일별 섹터 정보 추출
- `calculate_all_dates_vol_proxy()`: 모든 날짜 Vol_Proxy 계산
- `determine_baseline_scaled()`: Dynamic Baseline 설정

**TSUI 도메인 특화 계산 함수:**
- `calculate_time_weight()`: 시간 가중치 계산 (장중/장외)
- `calculate_keyword_weight()`: 키워드 가중치 계산 (Volfefe 키워드)
- `calculate_parkinson_volatility()`: Parkinson Volatility 계산
- `get_residue_alpha()`: 잔상 상수 반환 (강도별)

### `pipeline.py` (루트)
- `run_analysis()`: 전체 지수 산출 프로세스 조율 (구 `process_index_calculation`)


**참고**: 
- 현재 지수 계산에는 이미 전처리된 데이터(`trump_posts_36categori_final.csv`)를 사용하므로, 새 데이터를 추가하거나 재분류할 때만 실행하면 됩니다.
- 이 스크립트들은 PyTorch와 transformers 라이브러리가 필요합니다.

## 사용법

### 스크립트로 직접 실행

```bash
# 방법 1: 모듈로 실행
cd application
python -m index_generator.main

# 방법 2: 직접 실행
cd application/index_generator
python main.py
```

## 데이터 구조

지수 계산을 위해서는 다음 데이터가 필요합니다:

```
data/src_data/
├── track_01/                    # Track 1 데이터 (감성 분석 결과)
│   ├── trump_posts_term1_2017_2021_with_sentiment.csv
│   └── trump_posts_term2_2025_with_sentiment.csv
├── track_02/                    # Track 2 데이터 (섹터 분류 결과)
│   └── trump_posts_36categori_final.csv
├── etf_sector/                  # ETF 가격 데이터
│   ├── etf_metadata.json
│   ├── etf_XLY_2017-2021.csv
│   └── ...
└── DXY.csv                      # 달러 인덱스 데이터
```

**참고**: 
- 실제 데이터 파일은 저장소에 포함되지 않습니다. `.gitignore`에 의해 제외됩니다.
- 데이터 파일은 `data/src_data/` 폴더에 위치하며, Python 모듈(`data/constants.py`, `data/data_loader.py`)과 분리되어 있습니다.
