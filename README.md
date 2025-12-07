# Random Graph Analysis

다양한 무작위 그래프 생성 모델(ER, Configuration, Chung-Lu, BA)을 기반(원본) 네트워크의 속성을 바탕으로 생성하고, 각 그래프 모델의 차수 분포 등을 비교·분석할 수 있도록 설계된 Python 클래스.

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
정치도서구매 네트워크(polbooks.gml) 사용 예시
```python
G = nx.read_gml('polbooks.gml')
analyser = RandomGraphAnalysis(G) # 클래스 초기화
```
앙상블 생성 및 차수 분포 분석
```python
chunglu_ensemble = analyser.create_random_graph_ensemble("chunglu", 100) # Chung-Lu Model 앙상블 생성(100개)

original_dist = analyser.degree_distribution(G) # 원본 그래프의 차수 분포
chunglu_degree_dists = analyser.ensemble_degree_distributions(chunglu_ensemble) # 앙상블의 차수분포
```

## 주요 기능
### 무작위 그래프 생성 모델
- ER, Configuration, Chung-Lu, BA
### Ensemble 생성
- 선택한 무작위 그래프 모델로 여러 개의 랜덤 그래프 한 번에 생성 가능
- 'num_simulations'만큼 생성해 통계 분석에 활용 가능
### 차수 분포 계산
- 차수 분포 계산, Ensemble 그래프 차수 분포 전체 반환
## 클래스 구조
```python
__init__(self, G: nx.Graph) # 클래스가 초기화될 때 실행되며, 입력으로 받은 networkx.Graph 객체 G의 핵심 속성들을 클래스 속성(Attribute)으로 저장

self.random_graph_list() # 클래스를 이용해 생성할 수 있는 무작위 그래프 종류를 출력
```

## Random graph 생성 모델 설명
- 4가지 무작위 그래프 모델을 구현하여 생성하며, 각각 원본 그래프의 다른 특성(노드 수, 엣지 수, 차수 시퀀스)을 보존
### 1. ER
- 노드 간의 확률 $p$로 엣지를 연결하여 그래프를 생성
- set_init_ER(self): ER 그래프 생성에 필요한 엣지 연결 확률 $p$를 계산
- create_ERnp_graph(self): 노드 $N$개와 $p$ 확률을 사용하여, 모든 가능한 노드 쌍에 대해 $p$ 확률로 엣지를 연결
- 예외 처리: p가 0 < p < 1 범위인지 확인
### 2. Configuration
- 각 노드의 차수(입력그래프의 차수 시퀀스)를 그대로 유지하면서 엣지를 무작위 연결하여 랜덤한 그래프를 생성
- create_config_graph(self): 원본 그래프의 degree sequence $k$를 보존하고 각 노드의 차수만큼 Stub를 생성한 후, Stub들을 무작위로 짝지어 엣지를 만듦, self-loop 제거, multi-edge 제거
- 예외 처리: 차수 합이 짝수인지 확인, 차수가 0 이상인지 확인
### 3. Chung-Lu
- 노드별 기대 차수를 유지하면서 차수에 따른 확룰로 엣지를 연결하여 그래프를 생성 
- 두 노드 i와 j가 연결될 확률 P_ij는 두 노드의 차수의 곱에 비례함
- create_chunglu_graph(self): itertools.combinations를 사용하여 모든 노드 쌍을 순회하며 연결 확률을 계산(모든 노드 쌍 (i, j)에 대해 P_ij 계산, 난수 비교 후 엣지 생성)
- 예외 처리: 차수 0 이상인지 확인, 차수 총합이 0 아님을 확인, 연결 확률 1 미만임을 확인/최대 차수 곱으로 확률이 1 이상이 되는 경우 → Z 재조정
- 정규화 상수 $Z$: Z = total_degree, 확률이 1 이상이 되는 경우 발생 → 이를 방지하기 위해 Z를 (최대차수 × 2번째 최대차수) + 0.1로 재설정하여 안정화
### 4. BA
- 선호적 연결 기반 성장 모델로 새로운 노드는 기존 노드의 차수에 비례하여 연결 → Scale-free 네트워크 생성
- set_init_BA(self): BA 그래프 생성에 사용될 초기 노드수와 새로 추가하는 노드가 가지고 올 미연결 엣지수를 계산
- choose_target_node(self, existing_nodes:list, G:nx.Graph): 새로 들어온 노드가 엣지를 연결할 노드를 고름(그래프의 현재 노드 리스트와 그래프 자체를 매개변수로 받음)
- create_BA_graph(self): BA 그래프 생성, 노드 특성은 유지되지 않음 
- 예외 처리: 노드/엣지 수가 0이 아님을 확인
## 앙상블 생성 및 분석 함수 설명
```python
create_random_graph_ensemble(self, random_graph, num_simulations)
```
선택한 무작위 그래프 모델("ER", "configuration", "chunglu", "BA")을 지정된 횟수(num_simulations)만큼 반복 생성하여, 무작위 그래프들의 리스트(앙상블)를 반환
```python
degree_distribution(self, graph=None)
```
그래프를 입력 받아 해당 그래프의 차수 분포 array를 반환
```python
ensemble_degree_distributions(self, ensemble_graphs:list)
```
입력받은 앙상블 그래프 리스트에 대해 각각 degree_distribution을 호출하고, 그 결과인 차수 분포 array의 리스트를 반환
