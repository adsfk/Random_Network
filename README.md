# Random Graph Analysis

다양한 무작위 그래프 생성 모델(ER, Configuration, Chung-Lu, BA)을 기반 그래프의 속성을 바탕으로 생성하고, 
각 그래프 모델의 차수 분포 등을 비교·분석할 수 있도록 설계된 Python 클래스.

`networkx`, `numpy`, `itertools`, `random` 라이브러리를 기반으로 작동함.
## 사용 방법 (Usage Example)
클래스 가져오기
```python
!git clone https://github.com/adsfk/Random_Network.git
%cd ./Random_Network/
```
```python
import networkx as nx
from random_graph_analysis import *
```
정치도서구매그래프(polbooks.gml)분석 예시
```python
G = nx.read_gml('polbooks.gml')
analyser = RandomGraphAnalysis(G) # 클래스 초기화
```
```python
chunglu_ensemble = analyser.create_random_graph_ensemble("chunglu", 100) # Chung-Lu Model 앙상블 생성(100개)

original_dist = analyser.degree_distribution(G) # 원본 그래프의 차수 분포
chunglu_degree_dists = analyser.ensemble_degree_distributions(chunglu_ensemble) # 앙상블의 차수분포
```
## 주요 기능
### 무작위 그래프 생성 모델
- ER, Configuration, Chung-Lu, BA
### Ensemble 생성
선택한 무작위 그래프 모델로 여러 개의 랜덤 그래프 한 번에 생성 가능
'num_simulations'만큼 생성해 통계 분석에 활용 가능
### 차수 분포 계산
차수 분포 계산, Ensemble 그래프 차수 분포 전체 반환
### 구현 함수 및 모델 설명
#### 1. ER
ER 모델은 각 노드 쌍이 확률 p로 독립적으로 연결
선택된 확률에 따라 엣지가 랜덤하게 생성됨
set_init_ER(): 입력 그래프의 평균 차수 기반으로 p 계산
create_ERnp_graph(): 확률적 방식으로 모든 노드 쌍 연결 여부 판단
예외 처리: p가 0 < p < 1 범위인지 확인, 노드/엣지 수가 0인 경우 오류 발생
#### 2. Configuration
Configuration 모델은 입력 그래프의 차수 시퀀스를 그대로 유지하면서 랜덤한 그래프를 생성
각 노드는 자신의 차수만큼 stub를 가지고, 무작위 연결을 통해 엣지를 형성
create_config_graph(): stub 리스트 생성 → 섞기 → 랜덤 매칭, self-loop 제거, multi-edge 제거
예외 처리: 차수 합이 짝수인지 확인, 차수가 음수인지 확인
#### 3. Chung-Lu
Chung-Lu 모델은 노드별 기대 차수를 유지하면서 그래프를 생성 
두 노드 i와 j가 연결될 확률 P_ij는 두 노드의 차수의 곱에 비례
create_chunglu_graph(): itertools.combinations를 사용하여 모든 노드 쌍을 순회하며 연결 확률을 계산
1. 예외 처리: 차수 음수 여부 확인, 차수 합이 0이면 오류 처리, 최대 차수 곱으로 확률이 1 이상이 되는 경우 → Z 재조정
2. 노드 쌍에 대해 확률적 연결 시도: 모든 노드 쌍 (i, j)에 대해 P_ij 계산, 난수 비교 후 엣지 생성
3. 정규화 상수 Z: Z = total degree, 확률이 1 이상이 되는 경우 발생 → 이를 방지하기 위해 Z를 (최대차수 × 2번째 최대차수) + 0.1로 재설정하여 안정화
#### 4. BA
BA 모델은 우선 연결 기반 성장 모델로 새로운 노드는 기존 노드의 차수에 비례하여 연결 → Scale-free 네트워크 생성
set_init_BA(): 초기 fully-connected m0 설정
choose_target_node(): 차수 비례 확률로 노드 선택
create_BA_graph(): 성장 기반 네트워크 생성
