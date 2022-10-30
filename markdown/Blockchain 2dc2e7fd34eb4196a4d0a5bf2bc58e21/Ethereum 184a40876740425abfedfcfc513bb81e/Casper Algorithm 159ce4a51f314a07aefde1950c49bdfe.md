# Casper Algorithm

# [Casper Algorithm]

문수복, 이종수

# CasperFFG

**[Casper the Friendly Fianlity Gadget](https://arxiv.org/pdf/1710.09437.pdf)**

[https://medium.com/onther-tech/casper-ffg-overview-e09fbe4f7d2c](https://medium.com/onther-tech/casper-ffg-overview-e09fbe4f7d2c)

[https://github.com/ethereum/casper/blob/75639081fc26a251cfc1f191f8426fc31dc84795/casper/contracts/simple_casper.v.py#L561:63](https://github.com/ethereum/casper/blob/75639081fc26a251cfc1f191f8426fc31dc84795/casper/contracts/simple_casper.v.py#L561:63)

[https://github.com/ethereum/casper/blob/master/VALIDATOR_GUIDE.md](https://github.com/ethereum/casper/blob/master/VALIDATOR_GUIDE.md)

### 1. Chain-based PoS

지분에 따른 작업증명 난이도 차별. 지분이 많을 수록 작업증명 난이도 감소. 지분이 작을 수록 난이도 상대적 증가.

### 2. BFT-based PoS

Bizantine Fault Tolerance. 투표에 의해 한가지 값을 전체 노드가 합의하는 방안에 관한 연구. PBFT는 비동기 네트워크에서 전체 노드의 2/3 이상이 정직하면 합의에 이를 수 있다는 수학적 증명. 하이퍼렛저, 테더민트 등 여러 블록체인에 활용. 캐스퍼도 같음.

### BFT에 4가지 개념 추가

1. Accountability
: 검증자는 캐스퍼 프로토콜이 정의한 책임과 의무를 따를 필요가 있음. 의무에 위배 시, 스테이킹 보증금 전액 패널티로 삭감.
2. Dynamic Validators
: EOS의 위임지분증명(DPoS)는 대표자 21명 끼리 블록체인을 운영하며 블록을 검증.
테더민트는 PBFT기반 지분증명으로 100명의 검증인만 참여.
이더리움은 누구나 검증인이 될 수 있으며 언제든 참여하거나 이탈할 수 있음. 따라서 이를 동적으로 관리할 필요가 있음.
3. Defenses
: 지분증명은 작업증명과 다른 공격의 가능성에 노출되어 있으며 이에 대한 방어책이 있음
4. Modular Overlay
: 작업증명을 사용하는 어떤 블록체인도 캐스퍼 알고리즘을 도입하면 작업증명과 지분증명을 하이브리드 방식으로 사용 가능.

### 분기선택규칙(Fork Choice Rule)

블록체인에서는 분기가 필연적으로 발생. (네트워크 지연과 많은 노드 수로 인해)

작업 증명의 경우 Longest Chain을 실제 블록체인으로 기록. 어려운 작업 증명 과정을 거쳐 형성된 체인 중 가장 높이가 높기 때문에 블록이 번복될 가능성이 적음.

### 캐스퍼 프로토콜 용어

1. 체크포인트
:  100번째 마다 생성되는 블록(블록높이 % 100 = 0). 제네시스 블록 포함. 검증인들은 생성된 체크 포인트에 투표. 생성되는 모든 블록마다 투표하면 자원 소모가 크고 합의 과정이 느려지기 때문. (—> 블라드 짐라크가 [CBC Casper](https://github.com/cbc-casper/cbc-casper-paper/blob/master/cbc-casper-paper-draft.pdf)에서 DAG을 이용한 다른 방안 제시)
2. 투표
: 검증인이 보내는 투표 트랜잭션은 투표하고자 하는 체크포인트 + 투표하려는 블록 이전에 생성된 블록 정보를 포함 검증인 끼리 투표 메시지를 공유. (이전 블록 + 현재 블록 정보). 검증인이 투표하는 것은 블록이 아닌 블록과 블록 사이의 연결성.
이전 체크포인트 블록은 반드시 Justified or Finalized 상태야 함. 2/3넘는 검증인이 같은 블록에 투표하면 절대적 다수 링크 생성하고 블록은 안정 or 마감됨.
3. 절대적 다수 링크(s→t)
: 2/3이 넘는 검증인이 같은 블록 연결성에 투표한 결과. 두 개의 분기가 발생 시, 각각 절반의 검증인이 1/2씩 투표하면 절대적 다수 링크 생성하지 않음. 다음 체크포인트 블록 생성까지 기다림. A→C
4. 안정된 블록(Justified Block)
: 제네시스 블록 혹은 절대적 다수 링크 이전 블록 → 이후 블록이 존재. 이전 블록이 안정된 블록이면 이후 블록 또한 안정적. 새로운 다수 링크를 생성할 수 없음.
5. 마감된 블록(Finalized Block)
: 절대적 다수 링크가 존재. 이후 블록이 바로 이전 안정화 된 체크포인트 블록의 자식이면 이전 체크 포인트 블록은 마감된 블록.
6. 충돌(Conflict)
: 두 개의 체크포인트가 같은 높이에 서로 다른 분기에 존재하면 두 체크포인트는 충돌 상태.

### 검증인의 의무와 처벌

1. 같은 블록 높이에 중복 투표하지 말 것
: 지분증명은 블록 생성에 소모하는 컴퓨팅 자원이 매우 작아 수많은 블록을 생성할 수 있음. 이를 억제하기 위해 패널티로 보증금 전액 차감을 채택. h(x) 함수 이용. x는 체크포인트이며 함수는 블록 높이를 반환.
2. Nested Voting Prohibition
: 서로다른 분기에 마감된 블록 생성을 방지. 검증인의 33% 미만이 악의적익 검증인이라도 네트워크 문제 방생을 방지. 즉, 전체 검증인의 33%미만이 위 두 규칙을 어겨도 다른 분기에 동시에 마감 블록이 생성되지 않음. (보증금 전액 차감 패널티)

![https://miro.medium.com/max/1200/1*HoLCvx9V2mRKxlPt8eVH4Q.png](https://miro.medium.com/max/1200/1*HoLCvx9V2mRKxlPt8eVH4Q.png)

1. 분기 선택 규칙
: 검증자 그룹의 투표가 2/3를 도달하지 못하는 상황이 계속 발생하여 블록이 생기지 않을수도 있음. 이를 위해 가장 높이가 큰 안정화 블록이 존재하는 분기를 선택.
2. Dynasty
: 검증인 합류와 이탈 발생 시, 투표권의 생성과 상실 기준을 정의하는 수치.
특정 블록의 dynasty란 제네시스 블록부터 특정 블록까지의 이전 블록까지 존재하는 마감 블록의 수. 모든 블록은 dynasty 값을 가짐.
- 새로운 검증자는 보증금 예치 메시지가 포함된 블록 x’로 부터 D(x’) + 2 가 되는 시점부터 투표권을 가진다. (D(x) 함수는 특정 블록 x의 dynasty를 반환한다.)
- 기존 검증자는 보증금 출금 메시지가 포함된 블록 y’로 부터 D(y’) + 2가 되는 시점부터 투표권을 상실한다.
- 기존 검증자는 투표권을 상실한 이후 약 4개월 이후에 예치한 보증금을 돌려받는다. 이를 출금 지연(Withdrawal Delay)이라 부른다. 검증가들이 출금을 신청하면 바로 받지 못하고 4개월 이후 예치한 보증금을 돌려받는 정책. 검증인들이 이탈한 직후 과거 블록 내용을 바꾸는 것을 방지하기 위함.

### 장거리 수정 공격(Long Range Revision Attack)

악의적인 검증자가 검증자 그룹 이탈 직후, 과거 블록으로 돌아가 투표하는 행위. 

![https://miro.medium.com/max/1400/1*AS94xxozwjde2Fc-Kp81ag.png](https://miro.medium.com/max/1400/1*AS94xxozwjde2Fc-Kp81ag.png)

모든 지분증명 알고리즘이 겪는 공통적 문제이며 각각의 PoS채택 블록체인은 각각의 해결법을 가지고 있음.

- 보증금을 돌려받지 않은 상태에서 과거 블록에 투표하면 보증금 전액 차감
- 4개월 후 보증금을 돌려받고 4개월 이전의 블록 내용을 수정하면 출금 지연 정책에 따라서 그 변경 내용은 수상한 것으로 여겨 그 블록은 받아들이지 않음.
- 4개월 후 보증금을 돌려받고 4개월 이전이 되기 전까지 블록(e.g., 1개월 전 블록)에 투표 하려 하나 투표권이 없어 투표 불가.

### 파괴적 충돌(Castastrophic Crashes)

검증인의 1/3 이상이 오프라인 상태가 되는 경우. 2/3 이상의 합의를 이루어 낼 수 없기 때문에 전체 거래기록은 무한정 연기하는 상황 발생 (?? 앞서 1/2 무한 합의에 따른 길이 기준 선택이 있지 않았던가?!) 캐스퍼 프로토콜에서 2/3 이상의 합의란 검증인 수가 2/3 이상이  아닌 검증인이 예치한 보증금의 숫자가 2/3이상인 경우를 의미. 오프라인 상태의 검증인(투표하지 않는 검증인)은 보증금을 조금씩 삭감하는 방식 제안. (삭감 방식은 도입되지 않을 가능성이 높음.(현재는??) 

# CBC(Correct by construction) Casper

## 1. Estimate Safety Consensus Protocol

Estimate Safety Consensus Protocols는 (𝐶,𝐿_𝐶,Σ,E) 로 이루어져있습니다.

- Consensus Value: 합의의 대상
- Estimate: consensus value가 올바른지 아닌지에 대한 명제. {True, False}로 가는 map.
- Logic: estimate의 집합. 어느 estimates를 이용하여 합의를 이루었는가.
- Protocol states: 합의를 이루는 노드들의 상태. 검증자가 보낸 프로토콜 메시지에 따라 결정. Protocol message의 집합으로 정의
- Estimator: protocol states에서 estimate로 가는 map.
- Protocol message: consensus values, validators, protocol states의 tuple.

### 1.1 Estimate Safety****(𝑆(𝑝,𝜎)S(p,σ))****

proposition 𝑝가 프로토콜 상태 𝜎에서 safe하다는 의미이며, 𝑆(𝑝,𝜎)로 나타냄.

### 1.2 Consensus Safety

각 노드들이 estimate safety를 가지는 common future protocol state를 개별적으로 판단할 수 있음. 두 노드가 각각 protocol state에 있을 때, 각 노드가 개별적으로 내린 결정들은 서로 일관성이 존재.

- Lemma 1: Forward Safety: estimate safety가 모든 future state를 포함
- Lemma 2: Current Consisstency: 상반되는 두 proposition가 동시에 safe할 수 없음.
- Lemma 3: Backward Consistency: protocol state o’에서 safe한 estimate p의 반대 -p가 previous state o에서 safe하지 않음.
- Lemma 4: Distributed Consistency: 노드가 safe estimate를 기반으로 내린 개별적인 결정(decision)는 다른 노드가 내린 결정과 일관성을 가짐.

Non-triviality

Common future state가 존재하지 않는 경우. Blockchain fork 발생 혹은 byzantin 노드가 내린 결정에 따르지 않는 상황 등.

이 경우에도 두 protocol states에 대해 safety를 보장해야 함.

- Lemma 5: Maintaing a shared future is non-trivial: non-triviality 때문에 common future state가 항상 존재하지 않음.
- Lemma 6: Consensus Failure: non-triviality로 인해 common future state를 가지지 않는 상황. 두 개의 분기가 존재. 각 분기별로 두 개의 future states로 나뉨.

![Untitled](Casper%20Algorithm%20159ce4a51f314a07aefde1950c49bdfe/Untitled.png)

# Casper the Friendly Binary Consensus

프로토콜이 어떻게 byzantine fault tolerant 할 수 있는지. Binary Consensus와 Blockchain Consensus 예시로 확인.

- Protocol states: t 이하의 byzantine fault를 보이는 protocol message의 집합.
- Protocol messages: estimate, sender, justification 의 집합.
- Estimate(p): {0, 1} 중 하나
- Estimator는 protocol messages를 0, 1 혹은 0/ 중 하나의 값으로 보내는 map.
- Sender: validator(v∈V)
- Justification: Protocol messages

Protocol message의 estimator를 justification에 적용한 결과가 estimate와 동일할 때 유효.

Protocol message의 justification은 특정 검증자의 protocol state를 확인하는 용도로 사용.

Protocol message를 받은 노드는 해당 검증자의 protocol state를 justification으로 파악한 이후 common future state를 찾을 수 있음.

Consensus safety 정리를 적용할 수 있는지 각 노드가 개별적으로 파악 가능.

- equivocation: 한 검증자가 생성한 두 메시지가 dependency를 가지지 않는 메시지 쌍의 성질.
non-triviality 관점에서 보면 common future state를 가지지 않는 o1, o2를 justification으로 사용하는 protocol message는 항상 equivocation 함.
equivocation은 합의할 수 없는 두 protocol states를 판별하고 최종적으로 byzantine validator의 메시지를 검열하는데 사용.
- Equivocate한 검증자를 byzantine하다고 부르거나 집합 M에서 byzantine fault를 보였다고 함.
- Protocol executions: protocol message 집합 o, o’( E 간 transition. 
protocol execution은 o, o’ 가 improper subset 관계를 가짐. 
= protocol execution은 composition에 대해 닫혀있다.