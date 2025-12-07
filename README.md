# Random Graph Analysis

다양한 무작위 그래프 생성 모델(ER, Configuration, Chung-Lu, BA)을 기반 그래프의 속성을 바탕으로 생성하고, 
각 그래프 모델의 차수 분포 등을 비교·분석할 수 있도록 설계된 Python 클래스.

`networkx`, `numpy`, `itertools`, `random` 라이브러리를 기반으로 작동함.
RandomGraphAnalysis 클래스로 구성되어 있으며
## 4가지 랜덤 그래프 생성 모델 지원
1. ER, 2. Configuration, 3. Chung-Lu, 4. BA
## Ensemble 기능
동일 모델로 여러 랜덤 그래프 반복 생성, 시뮬레이션 기반 통계 분석 가능
## 차수 분포 계산 기능
차수 분포 계산, Ensemble 그래프 차수 분포 전체 반환
## 구현 함수 및 모델 설명
### 1. ER Graph
