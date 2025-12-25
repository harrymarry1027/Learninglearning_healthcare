# Learninglearning_healthcare
healthcare, nursing, medication et al

1. 캐글 임상 데이터 사용 해서 뷴석
2. 간호 정보 데이터 사용 해서 분석
3. 위데이터로 어떤 앱을 만들지 기획



## 1. 캐글 임상 데이터 사용해서 분석

### 1. 문제정의 : 데이터를 통해 얻고자 하는 것은?
“만성질환 노인 환자의 응급입원 비율과 검사 결과는 어떠한가?”

### 2. 데이터 수집  
- DateSet : Healthcare Dataset
- Dataset Information:
Each column provides specific information about the patient, their admission, and the healthcare services provided, making this dataset suitable for various data analysis and modeling tasks in the healthcare domain. Here's a brief explanation of each column in the dataset -
Name: This column represents the name of the patient associated with the healthcare record.
Age: The age of the patient at the time of admission, expressed in years.
Gender: Indicates the gender of the patient, either "Male" or "Female."
Blood Type: The patient's blood type, which can be one of the common blood types (e.g., "A+", "O-", etc.).
Medical Condition: This column specifies the primary medical condition or diagnosis associated with the patient, such as "Diabetes," "Hypertension," "Asthma," and more.
Date of Admission: The date on which the patient was admitted to the healthcare facility.
Doctor: The name of the doctor responsible for the patient's care during their admission.
Hospital: Identifies the healthcare facility or hospital where the patient was admitted.
Insurance Provider: This column indicates the patient's insurance provider, which can be one of several options, including "Aetna," "Blue Cross," "Cigna," "UnitedHealthcare," and "Medicare."
Billing Amount: The amount of money billed for the patient's healthcare services during their admission. This is expressed as a floating-point number.
Room Number: The room number where the patient was accommodated during their admission.
Admission Type: Specifies the type of admission, which can be "Emergency," "Elective," or "Urgent," reflecting the circumstances of the admission.
Discharge Date: The date on which the patient was discharged from the healthcare facility, based on the admission date and a random number of days within a realistic range.
Medication: Identifies a medication prescribed or administered to the patient during their admission. Examples include "Aspirin," "Ibuprofen," "Penicillin," "Paracetamol," and "Lipitor."
Test Results: Describes the results of a medical test conducted during the patient's admission. Possible values include "Normal," "Abnormal," or "Inconclusive," indicating the

- URI : https://www.kaggle.com/datasets/prasad22/healthcare-dataset/data

### 3. data 전처리(EDA) 
1. 분석의 목적과 변수가 무엇이 있는지 확인
2. 데이터 전체적으로 살펴보기 : head, tail, 이상치, 결측치
3. 이상값 찾아내기
4. 속성간의 관계 분석하기 
URI :  https://eda-ai-lab.tistory.com/13

### 4. data analysis



### 5. insigth 도출


## 2. 간호 정보 데이터 사용 해서 분석

### 1. 문제정의 : 데이터를 통해 얻고자 하는 것은?
“간호기록을 어떻게 접근하고, 사용할것인지?"
https://www.saan2mari.com/s-projects-side-by-side-1

### 2. 데이터 수집  
- DateSet : MIMIC-IV (https://physionet.org/)
- Dataset Information:
The icu module contains data sourced from the clinical information system at the BIDMC: MetaVision (iMDSoft). MetaVision tables were denormalized to create a star schema where the icustays and d_items tables link to a set of data tables all suffixed with "events". Data documented in the icu module includes intravenous and fluid inputs (inputevents), ingredients for the aforementioned inputs (ingredientevents), patient outputs (outputevents), procedures (procedureevents), information documented as a date or time (datetimeevents), and other charted information (chartevents). All events tables contain a stay_id column allowing identification of the associated ICU patient in icustays, and an itemid column allowing identification of the concept documented in d_items. Additionally, the caregiver table contains caregiver_id, a deidentified integer representing the care provider who documented data into the system. All events tables (chartevents, datetimeevents, ingredientevents, inputevents, outputevents, procedureevents) have a caregiver_id column which links to the caregiver table.

- URI : https://physionet.org/
- 자료 현재 신청중 (확인하고 있는 상황임)

### 3. data 전처리(EDA) 


### 4. data analysis



### 5. insigth 도출
