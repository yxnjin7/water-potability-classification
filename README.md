# 수질 데이터 기반 음용수 판별 모델

## 1. 프로젝트 개요 & 동기

수질 측정 데이터를 활용하여 물의 음용 가능 여부를 예측하는 머신러닝 분류 모델 구축 프로젝트

단순한 정확도 중심 평가가 아니라, 음용 불가능한 물을 음용 가능하다고 잘못 예측하는 오류의 위험성까지 함께 고려한 분류 모델 분석

**핵심 질문**

> 수질 지표를 활용해 물의 음용 가능 여부를 예측할 수 있는가?
> 음용수 판별 문제에서 어떤 예측 오류를 더 중요하게 고려해야 하는가?

## 2. 데이터 출처 & 전처리 과정

### 데이터 구성

* 수질 측정 데이터
* 주요 변수: pH, Hardness, Solids, Chloramines, Sulfate, Conductivity, Organic Carbon, Trihalomethanes, Turbidity
* 목표 변수: Potability

  * 0: 음용 불가능
  * 1: 음용 가능

### 전처리 과정

1. 데이터 구조 및 기초 통계량 확인
2. 컬럼별 결측치 확인
3. pH, Sulfate, Trihalomethanes 결측치 중앙값 대체
4. 변수별 분포 시각화
5. 변수 간 상관관계 확인
6. `Solids / Conductivity` 기반 파생변수 생성
7. 입력 변수 X와 목표 변수 y 분리
8. Stratified train/test split 적용

## 3. 분석 방법 & 선택 이유

### 모델링 방법

| 방법                    | 설명                                           |
| --------------------- | -------------------------------------------- |
| Baseline RandomForest | 기본 RandomForest 모델 기준 성능 확인                  |
| Balanced RandomForest | 클래스 불균형을 고려한 RandomForest 모델                 |
| Threshold Tuning      | 예측 임계값 조정을 통한 Precision, Recall, F1-score 비교 |
| ROC-AUC 분석            | 전체 분류 성능 확인                                  |
| K-Means Feature 추가    | 군집 정보를 새로운 feature로 추가한 뒤 성능 비교              |

RandomForest를 사용한 이유는 비선형 관계와 변수 간 복합적인 패턴을 반영할 수 있고, 수질 데이터처럼 여러 수치형 변수가 함께 작용하는 분류 문제에 적합하기 때문

## 4. 핵심 결과

### 1) 기본 RandomForest 대비 조정 모델 성능 개선

Baseline RandomForest 대비 class_weight와 하이퍼파라미터를 조정한 모델에서 F1-score 개선 확인

| 모델          | Accuracy | Precision | Recall | F1-score |
| ----------- | -------: | --------: | -----: | -------: |
| Baseline RF |   0.6479 |    0.5969 | 0.3008 |   0.4000 |
| Modified RF |   0.6601 |    0.5954 | 0.4023 |   0.4802 |

### 2) 교차검증을 통한 모델 비교

5-Fold Stratified 교차검증 결과, Balanced RF는 Recall과 F1-score 측면에서 기본 모델보다 개선된 성능 확인

| 모델          | Accuracy | Precision | Recall | F1-score |
| ----------- | -------: | --------: | -----: | -------: |
| Default RF  |   0.6710 |    0.6549 | 0.3307 |   0.4387 |
| Balanced RF |   0.6656 |    0.6118 | 0.3904 |   0.4754 |

### 3) 임계값 조정 결과

기본 임계값 0.50에서는 Accuracy가 높았으나 Recall이 낮은 경향 확인
임계값을 0.40으로 낮추었을 때 F1-score 기준 최적 성능 확인

| Threshold | Accuracy | Precision | Recall | F1-score |
| --------: | -------: | --------: | -----: | -------: |
|      0.50 |   0.6601 |    0.5954 | 0.4023 |   0.4802 |
|      0.40 |   0.5366 |    0.4512 | 0.8672 |   0.5936 |
|      0.30 |   0.4299 |    0.4051 | 0.9844 |   0.5740 |

음용수 판별에서는 음용 가능한 물을 놓치지 않는 것도 중요하지만, 음용 불가능한 물을 음용 가능하다고 예측하는 False Positive 오류가 더 치명적일 수 있기 때문에 Precision과 Recall 간 균형 고려 필요

### 4) ROC-AUC 결과

최종 RandomForest 모델의 ROC-AUC 점수는 0.6808로 확인

수질 변수만으로 음용 가능성을 어느 정도 구분할 수 있으나, 완전한 판별에는 한계 존재

### 5) K-Means 군집 feature 추가 결과

K-Means 군집 정보를 feature로 추가한 모델은 기존 RandomForest보다 성능이 소폭 낮게 나타남

| 모델                             | Accuracy | Precision | Recall | F1-score |
| ------------------------------ | -------: | --------: | -----: | -------: |
| 기존 RandomForest                |   0.6601 |    0.5954 | 0.4023 |   0.4802 |
| RandomForest + K-Means Cluster |   0.6524 |    0.5824 | 0.3867 |   0.4648 |

군집 정보가 항상 분류 성능 개선으로 이어지는 것은 아니며, 추가 feature의 유효성 검증 필요성 확인

## 5. 시각화

* 변수별 분포 히스토그램
* 변수 간 상관관계 히트맵
* Confusion Matrix
* Threshold별 성능 비교
* ROC Curve
* 기존 모델과 K-Means 추가 모델 성능 비교 그래프

## 6. 한계점 & 향후 개선 방향

### 한계점

* 수질 데이터 내 일부 변수의 결측치 존재
* Potability와 개별 변수 간 상관관계가 전반적으로 약한 편
* RandomForest 기반 모델의 예측 성능이 중간 수준에 머무름
* False Positive와 False Negative 중 어떤 오류를 더 중요하게 볼 것인지에 대한 기준 설정 필요

### 향후 개선 방향

* XGBoost, LightGBM 등 다른 분류 모델과 비교
* GridSearchCV 또는 RandomizedSearchCV 기반 하이퍼파라미터 튜닝
* SMOTE 등 클래스 불균형 처리 기법 적용
* 변수 중요도 분석 및 불필요 변수 제거
* Precision-Recall Curve 기반 임계값 조정
* 실제 수질 기준과 연결한 해석 보완

## 7. 배운 점

분류 모델에서는 Accuracy만으로 모델 성능을 판단하기 어렵다는 점을 확인하였다.

음용수 판별 문제처럼 오류 유형에 따라 위험도가 달라지는 경우, Precision과 Recall의 의미를 함께 해석해야 할 필요성을 학습하였고,
임계값 조정을 통해 모델의 예측 성향을 바꿀 수 있으며, 분석 목적에 따라 최적 임계값이 달라질 수 있다는 점을 확인하였다.

K-Means와 같은 비지도학습 기반 feature 추가가 항상 성능 개선으로 이어지지는 않으며, 모델 성능 비교를 통한 검증 과정의 중요하다.

## 8. 사용 기술

* Python
* Pandas
* NumPy
* Scikit-learn
* Matplotlib
* Seaborn
* Jupyter Notebook

## 9. 저장소 구조

```bash
water-potability-classification/
├── README.md
├── notebooks/
│   └── water_potability_classification.ipynb
├── requirements.txt
└── .gitignore
```
[분석 노트북 바로가기](./notebooks/water_potability_classification.ipynb)
## 10. 주의사항

학습 및 포트폴리오 목적의 프로젝트

원본 데이터 파일은 저장소에 포함하지 않거나, 공개 가능한 경우에만 별도 출처와 함께 포함
