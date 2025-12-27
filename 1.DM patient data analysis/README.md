# Diabetes Readmission Prediction (Kaggle)
당뇨 환자의 30일 이내 재입원 예측 및 인사이트 도출
## 1. 문제정의: 데이터를 통해 얻고자 하는 것은?
본 프로젝트는 Kaggle의 “Diabetes 130-US hospitals(1999–2008)” 데이터를 활용하여 당뇨 환자의 재입원 위험을 예측하고, 임상/운영 관점에서 활용 가능한 **위험군 선별 기준(Threshold)**을 제안하는 것을 목표로 한다.

핵심 질문
- 30일 이내 재입원(<30) 환자를 예측할 수 있는가?
- 재입원 위험에 영향을 주는 요인(입원 기간, 검사/처치량, 과거 의료 이용, 약물 변화 등)은 무엇인가?
- 모델을 실제 운영에 적용한다면, **Recall(놓치지 않기)**과 Precision(오탐 줄이기) 사이에서 어떤 임계값이 적절한가?
예측 타깃 정의
- 원본 타깃: readmitted ∈ {NO, >30, <30}
- 이진 분류 타깃:
  -Positive(1): <30 (30일 이내 재입원)
  Negative(0): NO 또는 >30

## 2. 데이터 수집
Dataset
- Dataset: diabetic_data.csv
- URI: https://www.kaggle.com/datasets/brandao/diabetes/data?select=diabetic_data.csv
분석 환경: Google Colab
데이터 특징(요약)
- 총 관측치: 101,766 encounters
- 환자 식별자: patient_nbr
- 방문(입원) 식별자: encounter_id
- 동일 환자가 여러 번 입원한 기록이 존재 → 데이터 분할 시 누수(leakage) 주의 필요

## 3. Data 전처리(EDA)
(분석 목적과 변수 확인 + 결측/이상치/관계 분석)
3.1 데이터 전체 구조 확인
- head(), tail(), shape, dtypes
- 수치형 기술통계: describe()
- 범주형 유니크 개수: nunique()

3.2 타깃 분포 확인
- readmitted 분포:
  - NO: 약 53.9%
  - >30: 약 34.9%
  - <30: 약 11.2% → 불균형 문제
따라서 ROC-AUC뿐 아니라 PR-AUC, Recall 기반 평가가 중요하다고 판단

3.3 결측치 및 결측 표기 탐색
- "?"가 결측을 의미하는 컬럼: race, weight, payer_code, medical_specialty, diag_1, diag_2, diag_3
- 결측률이 매우 높은 변수:
  - weight (약 96.9%)
  - max_glu_serum (약 94.8%)
  - A1Cresult (약 83.3%)
  - medical_specialty (약 49.1%)
  - payer_code (약 39.6%)

3.4 변수 간 관계(예: 카운트형/의료이용 패턴)
- 대표 카운트형 변수:
  - num_lab_procedures (검사 횟수)
  - num_procedures (시술/수술 횟수)
  - num_medications (투약 약물 수)
  - number_outpatient / number_emergency / number_inpatient (최근 1년 의료 이용)
  - 카운트형은 분포가 긴 꼬리를 가질 가능성이 높아 log 변환, 0 여부 파생 등을 FE 후보로 설정

## 4. Data Readiness Check (Gate 기반)
본 프로젝트는 모델링 전에 “게이트”를 통과해야만 다음 단계로 이동하는 구조를 사용했다.

Gate 1) 타깃 준비상태
- <30 비율이 약 11% → 불균형 존재
- 대응: PR-AUC, Recall 중심 평가 + class_weight 또는 threshold tuning 고려

Gate 2) 결측 및 표기 통일
- "?", "None", "Unknown/Invalid" 등 결측 표기를 통일(NaN)
- 결측률이 과도한 변수(weight 등)는 drop 후보로 관리
- A1Cresult, max_glu_serum은 “검사 안함(None)”이 정보일 수 있어 토큰 확인 후 처리

Gate 3) 카디널리티(범주 폭발) 대응
- diag_1/2/3가 700~800개 유니크 → 원핫 폭발 위험
- 해결: ICD-9 대분류 그룹화로 범주 축소

Gate 4) 코드형 numeric 처리
- numeric이지만 코드 의미를 가진 변수: admission_type_id (8), admission_source_id (17), discharge_disposition_id (26)
- 해결: categorical로 취급하도록 string 변환

Gate 5) 누수 방지 분할 전략
- patient_nbr 기준으로 동일 환자가 train/test에 섞이지 않도록 Group split 적용
결과:
- Train: (81,613, 44)
- Test: (20,153, 44)
- train/test 양성률도 유사하여 분포 안정적

Gate 5 통과 후 모델링용 산출물
- X_train, X_test, y_train, y_test

5. Feature Engineering (FE)
- Gate 기반 전처리 완료 후, 카운트형/이용패턴/약물변화 요약을 중심으로 파생변수를 생성했다.
- 생성된 주요 파생변수(22개, 총 44→66 columns)
- log1p 카운트 변환
  - log1p_num_lab_procedures, log1p_num_procedures, log1p_num_medications,
  - log1p_number_outpatient, log1p_number_emergency, log1p_number_inpatient
- 0 vs >0 이용 여부
  - has_number_outpatient, has_number_emergency, has_number_inpatient
-최근 1년 의료 이용 요약
  - prior_visits_total, emergency_share
- 입원기간 대비 강도(per day)
  - labs_per_day, meds_per_day, procedures_per_day
- 처치 총량
  - total_procedures
- 연령 ordinal
  - age_ord
- 약물 변화 요약(Up/Down/Steady/No 기반 자동 탐지)
  - meds_any, meds_changed_count, meds_up_count, meds_down_count, meds_steady_count, meds_intensity_score

## 6. Modeling
모델 1) Logistic Regression (Baseline)
처리: 결측 대체 + OneHot + scaling + class_weight="balanced"
결과(테스트):
ROC-AUC ≈ 0.6690
PR-AUC ≈ 0.2036

모델 2) HistGradientBoostingClassifier (Baseline+)
OneHot 결과가 sparse → dense 변환 후 학습

결과(테스트):
ROC-AUC ≈ 0.6798
PR-AUC ≈ 0.2254
PR-AUC 기준으로 Logistic 대비 개선 확인

## 7. Model Evaluation & Summary
7.1 평가 지표
불균형 데이터 특성상 ROC-AUC 외에 PR-AUC, Recall, Precision, F1을 함께 사용
운영 적용을 고려하여 Recall 목표 기반 threshold tuning 수행

7.2 Threshold Tuning (HGB+FE)
Recall 목표를 높일수록 재입원 환자를 더 많이 잡지만(False Negative↓), 오탐(False Positive↑)이 급증한다.
예시:
Recall ≥ 0.50 → threshold≈0.1300
Recall ≥ 0.70 → threshold≈0.0991 (FP 크게 증가)
Recall ≥ 0.80 → threshold≈0.0823 (경보 대상 급증)

해석

“놓치지 않기(Recall)”를 최우선으로 하면 낮은 threshold가 유리하지만,
실제 운영에서는 인력/리소스에 따라 top-k(상위 위험군만 관리) 방식이 더 현실적일 수 있다.

## 8. Insight 도출 (현재 단계에서의 시사점)

- 데이터 누수 방지(환자 단위 split)가 성능보다 선행되어야 함
동일 환자가 train/test에 섞이면 성능이 과대평가될 수 있다.
- <30 양성 비율이 약 11%로 낮아
단순 accuracy보다 PR-AUC/Recall 중심 의사결정이 더 적절하다.
- 임계값 선택은 “모델 성능”이 아니라 “운영 목적”의 문제
- 높은 Recall을 선택하면 FP가 급증하여 추가 스크리닝 절차가 필요
- 리소스 제한이 있다면 top-k 방식(상위 N% 관리)이 현실적
