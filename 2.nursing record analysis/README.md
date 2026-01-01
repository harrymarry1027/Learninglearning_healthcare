### 프로젝트 개요: MIMIC-IV 간호 기록 텍스트 표준화 및 CUI 매핑

### 1. 프로젝트 배경 및 목표

- 배경: MIMIC-IV 데이터셋의 간호 기록(Chart Events, 특히 Problem List)은 비정형 자유 텍스트(Free-text)로 작성되어 있어, 동일한 질병도 다양하게 표현됨(예: *Heart Failure*, *CHF*, *Heart failure*). 이를 그대로 분석이나 챗봇에 활용하기에는 데이터의 일관성이 부족함.
- 목표: 비정형 텍스트를 전처리하고, SciSpaCy와 UMLS(Unified Medical Language System)를 활용하여 표준 의학 용어 코드(CUI)로 매핑함으로써 데이터를 구조화함. 또한, NegSpaCy를 통해 임상적 의미가 없는 '부정(Negation)' 표현을 필터링하여 데이터의 정확도를 높임.

### 2. 기술 스택 (Tech Stack)

- Language: Python 3.12
- Data Analysis: Pandas, NumPy, Matplotlib, Seaborn
- NLP & AI: SpaCy (v3.x), SciSpaCy (`en_core_sci_sm`), NegSpaCy (부정어 탐지)
- UMLS : https://www.nlm.nih.gov/research/umls/index.html
- Dataset: MIMIC-IV Clinical Database (`chartevents.csv`)
- https://mimic.mit.edu/fhir/CodeSystem-mimic-chartevents-d-items.html(item 설명)

---

### 3. 주요 수행 과정 (Work Process)

### STEP 1: 데이터 로드 및 전처리 (Data Preprocessing)

- 데이터 필터링: `chartevents` 테이블에서 'Problem List'(ItemID: 220001)에 해당하는 텍스트 데이터 추출.
- 텍스트 정제(Normalization): 소문자 변환, 특수문자 제거, 다중 공백 제거를 수행하는 `clean_text` 함수 구현.
- 데이터 다이어트: 전처리를 통해 중복된 표현을 통합하여 처리해야 할 고유 문장(Unique Vocabulary)의 수를 줄임 (전/후 비교 시각화 수행).

![image.png](attachment:5514577f-d8ba-4668-89c3-9b38cdc6e01c:image.png)

### **STEP 2: 엔티티 링킹 (Entity Linking to UMLS)**

- 모델 구축: 생의학 도메인에 특화된 `en_core_sci_sm` 모델 로드.
- CUI 매핑: `scispacy_linker`를 파이프라인에 추가하여 텍스트 내 엔티티를 UMLS의 고유 개념 식별자(CUI, Concept Unique Identifier)로 변환.
- 성과: 서로 다른 텍스트(예: *Aortic dissection* vs *aortic dissection*)가 동일한 CUI(`C0340643`)로 매핑됨을 확인하여 데이터 표준화 달성.

### **STEP 3: 매핑 검증 (Validation)**

- 매핑 성공률 분석: 전체 데이터 중 약 **83.09%**가 유효한 CUI로 매핑됨을 확인.
- 동의어 그룹핑 검증: 하나의 CUI에 2개 이상의 텍스트 표현이 매핑된 케이스를 분석하여 표준화 성능 검증.
    - *예시:* `C0018801` (Heart Failure) → "heart failure chf diastolic chronic", "heart failure chf systolic acute" 등 다양한 표현이 하나의 코드로 통합됨.
    
    ![image.png](attachment:87e23b3e-8931-443c-8c58-63e2741e9fa4:image.png)
    

### **STEP 4: 부정어 탐지 (Negation Detection)**

- 문제 해결: 단순히 질병명을 추출할 경우, "No pneumonia"(폐렴 없음)라는 문장에서도 "pneumonia"가 추출되어 환자가 질병을 가진 것으로 오인될 수 있음.
- NegEx 알고리즘 적용: `negspacy` 라이브러리를 NLP 파이프라인(`ner`와 `linker` 사이)에 추가.
- 로직 구현: `ent._.negex` 속성이 `True`인 경우(부정된 경우) 해당 엔티티를 결과에서 제외하는 필터링 로직 구현.
    - *테스트 결과:* "Patient has diabetes but no evidence of pneumonia" 문장에서 Diabetes(당뇨)만 추출하고 Pneumonia(폐렴)는 제외하는 데 성공.

---

### 4. 주요 성과 및 결과 (Key Results)

1. **데**이터 구조화: 1,159건의 비정형 간호 기록 텍스트를 분석 가능한 형태의 CUI 코드 및 표준명(Standard Name)으로 변환.
2. 높은 매핑 정확도: SciSpaCy 링커를 최적화하여 83.09%의 매핑 성공률 달성.
3. 임상 정확도 확보: Negation Detection을 통해 '질병이 없음'을 나타내는 기록을 데이터셋에서 배제

---

### 5. 향후 개선 사항: RAG 기반의 문맥 인식 및 고차원 표준화

(Future Improvements: Context-Aware Standardization via RAG)

본 프로젝트는 Rule-based 및 머신러닝 모델(SciSpaCy)을 활용하여 텍스트를 구조화하는 데 성공했으나, 복잡한 임상 문맥(Context)을 해석하는 데에는 한계가 있었습니다. 이를 극복하기 위해 다음과 같은 고도화 전략을 제안합니다.

### 1. 다차원적 속성 추출 (Multi-dimensional Attribute Extraction)

단순한 질병 유무 확인을 넘어, 간호 중재(Intervention) 결정에 필수적인 세부 속성을 식별해야 합니다. 

- 심각도 (Severity): *Mild(경미), Moderate(중등도), Severe(심각)* 구분을 통해 응급도를 판단.
    - *(예: "Severe chest pain"은 즉각적인 EKG 모니터링이 필요)*
- 부위 및 방향 (Laterality & Body Site): *Left(좌), Right(우), Bilateral(양측), Distal(말초)* 등 정확한 환부 식별.
    - *(예: "Right leg swelling"을 식별하여 DVT 의심 시 해당 부위만 집중 관찰)*
- 확실성 (Certainty/Assertion): *Suspected(의심), Rule out(배제 진단), Confirmed(확진)* 구분.
    - *(예: "Rule out pneumonia"는 확진이 아니므로 예방적 간호와 모니터링 수행)*

### 2. RAG(Retrieval-Augmented Generation) 기반의 지능형 표준화

기존의 정적인 매핑 방식(Linker)은 문맥에 따른 의미 변화를 포착하기 어렵습니다. 이를 해결하기 위해 LLM(Large Language Model)에 외부 지식 베이스를 연동하는 RAG 기술을 도입합니다.

- Vector Database 구축: UMLS, SNOMED-CT 등 표준 의학 용어집과 최신 간호 가이드라인을 벡터화하여 저장합니다.
- Retrieval & Reasoning (검색 및 추론):
    - 사용자가 입력한 비정형 기록(예: *"환자가 어제보다 숨쉬기 힘들어함"*)에 대해, Vector DB에서 관련된 표준 증상 정의(Dyspnea)와 평가 가이드라인을 검색합니다.
    - LLM은 검색된 근거 자료를 바탕으로 이것이 'Acute Dyspnea(급성 호흡곤란)'인지, 'Worsening(악화)' 상태인지를 판단합니다.
- Explainable Standardization (설명 가능한 표준화):
    - 단순히 코드로 변환하는 것을 넘어, "왜 이 증상을 심각(Severe)으로 분류했는지"에 대한 근거(Reference)를 함께 제시하여 의료진의 신뢰도를 확보합니다.
