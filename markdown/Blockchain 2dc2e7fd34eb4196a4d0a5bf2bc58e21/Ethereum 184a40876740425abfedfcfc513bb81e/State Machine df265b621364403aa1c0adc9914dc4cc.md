# State Machine

# [Stateless Client]

- References
    
    [[이더리움2.0 깊이보기 시리즈] 스태이트리스 클라이언트(Stateless client)[7편]](https://medium.com/onther-tech/%EC%9D%B4%EB%8D%94%EB%A6%AC%EC%9B%802-0-%EA%B9%8A%EC%9D%B4%EB%B3%B4%EA%B8%B0-%EC%8B%9C%EB%A6%AC%EC%A6%88-%EC%8A%A4%ED%83%9C%EC%9D%B4%ED%8A%B8%EB%A6%AC%EC%8A%A4-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8-stateless-client-7%ED%8E%B8-b2e96d9f071b)
    
    [The Stateless Client Concept](https://ethresear.ch/t/the-stateless-client-concept/172)
    
    [Data from the Ethereum stateless prototype](https://medium.com/@akhounov/data-from-the-ethereum-stateless-prototype-8c69479c8abc)
    

Double batched log Accumulator and State-minimised execution model

이더리움 2.0에서 validator들은 shuffling되어 각각의 shard chain에 배정.

Validator가 shard chain에 배정되면, validator는 shard chain의 전체 상태 트리(entire state trie)를 새로 다운로드하여 상태를 sync.

하지만 validator가 shard chain에 배정될 때마다 entire state trie를 새로 다운받는 일은 validator에게 부담.

validator node가 stateless하다면 부담을 크게 줄일 수 있음.

stateless client는 entire state trie를 가지지 않고 상태 루트(state root) 값 만을 가지기 때문.

## Stateless Client Concept

### Definition

이더리움 1.0: 트랜잭션을 실행하면서 state에 접근하기 위해서는 entire state trie가 필요.

Stateless client에서 entire state trie에 대한 접근 없이 tx를 성공적으로 처리하려면 tx와 함께 witness 데이터가 필요. witness 데이터란 tx가 실행하면서 접근하게 되는 account 또는 storage key 접근에 필요한 데이터를 의미.

(Merkle trie의 예로 들면, 머클 트리의 리프 노드와 해당 리프 노드가 머클 트리에 존재함을 입증해 줄 수 있는 merkle path가 witness)

Stateless client에서 tx이 witness에 포함되지 않은 account나 storage key에 접근하면 error를 발생. tx가 성공적으로 처리되면 stateless client는 새로운 state root를 가짐.

State client는 state root만을 가짐. witness data는 miner가 제공. Miner는 항상 entire state trie를 가짐.

대안, tx sender가 전체 상태 트리를 가지게 해 witness 제공에 대한 책임을 tx sender에게 넘기는 방식이 있음. (miner를stateless 하게 함.)

→ 이게 무슨 의미가 있지??

Stateless client 는 이더리움 1.0에 적용이 어려움.

account에 대한 접근이 동적. 누구나 임의의 account에 접근이 가능.

tx sender가 tx와 함께 witness를 보내더라도 해당 witness를 포함하지 않은 account에 접근하게 되면 에러 발생.

대안. Tx 외부에 account list를 두는 방식

Tx를 미리 실행해보고 tx가 실행되면서 접근하는 account list를 tx와 함께 보냄. + witness 도 함께 보냄.

문제. tx와 account list를 보낼 때 시점과 실제 tx이 블록에 포함되어 처리되는 시점에 latency가 존재. 그 사이 해당 account의 state 변경이 이뤄지면 witness correctness가 깨짐.

대안. miner가 블록에 tx를 포함하기 전 miner가 다시 witness를 조정. miner가 갖고 있는 state trie로 부터 witness를 다시 생성.

### Advantage

1. Stateless miner와 node들은 전체 상태 트리를 가지지 않아도 됨. → 노드간 동기화가 빠름
2. Storage 접근에 필요한 SLOAD, SSTORE opcode에 대한 scheme이 필요 없음. 상태에 접근하지 않기 때문.
3. Stateless client는 전체 상태 트리 접근에 필요한 disk I/O가 필요하지 않음. disk I/O는 상태 값을 읽거나 쓸 때 발생. 이더리움 네트워크의 상태 트리가 커지면 입출력 비용이 비싸짐. 이는 Denial-of-Service 공격 벡터로 작용.
4. tx와 account list를 명시하는 것 → tx가 접근하는 account가 겹치지 않으면 tx처리에 대한 병렬화 가능.
5. tx와 account list를 명시하면 client에서 해당 account list storage data를 미리 가져 올 수 있음.
6. Sharding에서 validator는 각 shard에 shuffling 되어 배정. Shard chain의 state를 새로 다운로드하여 sync를 맞춤. Stateless한 validator가 shard chain 배정 시, state root 만 다운로드 하면 되어 validator가 shuffling 되어 배정되는 과정이 간소화.

이더리움 PoS전환에 Stateless client가 필수가 아닌 이유

No longer a prerequisite for the move to PoS.

[Can someone explain '"Stateless Ethereum" is no longer a prerequisite for the move to proof of stake.'](https://www.reddit.com/r/ethfinance/comments/pqsfpm/can_someone_explain_stateless_ethereum_is_no/)

# Stateful vs Stateless

[Stateful vs Stateless: Full Difference](https://www.interviewbit.com/blog/stateful-vs-stateless/)

- Network Protocol: a set of rulese that govern how data is formatted, sent and received by computer network devices, ranging from servers and routers to endpoints, regardless of their underlying infrastructures, designs, or standards.
- State: the condition or the quility of being at a given point of time.
- Stateful/Stateless의 구분은  the length of interaction a client has with it and how much of the information is stored.

## Stateful 하다?

전화를 생각해보자. 대화의 시작부터 끝까지 지속적으로 연결 중이다.

연결은 처음만 validate함. request를 보내고 응답이 없으면 request를 다시 보냄.

Session을 저장한다.

이전 tx가 현재 tx에 어떻게 영향을 주었는지? 여부. 은행 거래와 같은 것.

stateless는 그저 message를 send하기만 하면 됨.

### Advantages of Stateful

- keep track of the connection. continually keeping track of information
- more intuitive. can maintain data on the server between two requests.
- improve performance when data retrieval is required only once.

### Disadvantages of Stateful

- stateful protocol requires memory allocation in order to store data.
- can be decrease in the performance. requires continuous management of the service’s full lifecycle.
- require backing storage(usually)
- state is maintained —> stateful is not very secure

## Stateless란?

### Advantages of Stateless

- easy to recover from partial faliures like crashes.
- the server does not have to store session state between requesets. scalability can be easily enhanced
- does not need a large number of resources.
- each individual communication is unconnected and distinct.
- no need to refer to another packet in these packets.

### Disadvantages of Stateless

- essential to include additional information in each request. the server will need to interpret this new information.
- degrade network performance by increasing the amount of repetitive data delivered in a series of requests, which cannot be saved and reused.
- does not store information about a particular user session.

Save Data or Not

[Key Difference (1)](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Key%20Difference%20(1)%201a5f33dbf2054079abd267259ac6efbb.csv)

# Stateless Ethereum

[](https://ethresear.ch/t/the-stateless-client-concept/172)

## State

### Account State

- nonce, balance, storageRoot, codeHash(contract)

### World State

- mapping of account addresses between account states.(Merkle Patricia tree)

Statelessness allows the creation of light nodes.

A light node contains only the chain of headers without the execution of any txs or associated states.

When a node comes online it will be fully stateless.(zero information regarding state.

### Ethereum 1.X

[The 1.x Files: The State of Stateless Ethereum](https://blog.ethereum.org/2019/12/30/eth1x-files-state-of-stateless-ethereum/)

implements stateless client but not stateless miners.

launch-block: only records the input and output state of this block locally.

witness data is called by the node to construct blocks. 

witness data + input/output date ⇒ stateless client

### Ethereum 2.0

stateless clients and statelesss miners in Sharding.

All nodes are stateless so that faster processing with less data increases scalability greatly.

Eth1 is a sequential chain of blocks where one is complete on top of another in a linear fashion. This leads to traffic jams.

Sharding is a design where the Eth network is split into groups referrede to a shards. Each shard has its own independant state. Txs are delegated to different shards for processing. parallel computing increases efficienct by allowing the work to be split up and executed concurrently.

A truly stateless client would never keep a copy of state. It would only grab the latest txs together with the witness, and everything it needs to execute the next block.

If the entire network were stateless, this could actually hold up forever witnesses for new blocks can be produced from the previous block.

varying degrees of statefullness. and have a network in which some nodes keep a full copy of state and can serve everyone else fresh witnesses.

- Full-state nodes would operate as before, but would additionally compute a witness and either attach it to a new block, or propagate it through a secondary network sub-protocol.
- Partial-state nodes can keep a full state for just a short number of blocks, or perhaps just watch the piece of state that they are interested in and get the rest of the data that they need to verify blocks from witnesses.
- Zero-state nodes: want to keep their clients running as light as possible could rely entirely on witnesses to verify new blocks.

[State Provider Models in Ethereum 2.0](https://ethresear.ch/t/state-provider-models-in-ethereum-2-0/6750?u=benjaminion)

PoS ⇒ consensus comes very quickly.

### Aim

make ethereum scale, by mitigating unbounded state growth.

### Witness

When client receive validated blocks from miners, they will also receive its corresponding witness.

The block witness consists of all the data required to execute the transactions contained in that block.

### Modeling

the approach we typically use when we want to predict the future, or explore unintentional know-on effects that may occur when changes are made.

현재의 이더리움 네트워크를 stateless로 바꿀 때 시스템이 어떻게 적용되고 동작하는지를 잘 이해할 필요가 있음.

가장 최악과 최선의 시나리오는 어떨까?

### Baysian Network

a probabilistic graphical and visual modeling tool that represents the interaction of stakeholders whithin the model.

explicitly represent uncertainty using joint probability distributions across all the factors and interactions in the model.

be suitede to modeling complex system like ethereum.

be used regardless of whether ehre is a lot or little data.

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled.png)

**Figure 1: Stateless Ethereum model showing four sub-models**

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled%201.png)

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled%202.png)

full node, archive node: semi-stateless node

### Block Producer

Miner

### Node bandwidth

Enables fast propagation of blocks through the Ethereum network.

High bandwidth: businesses, hosting nodes

Medium bandwidth: colleges, residential

Low bandwitdh: others..

![스크린샷 2022-06-24 오후 7.31.10.png](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7.31.10.png)

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled%203.png)

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled%204.png)

## Accumulator

Stateless Client에서 사용

1. Persist the blockchain data, specifically the tx and block that have been agreed by validators via consensus protocol.
2. Provide a response with Merkle proofs to any query that asks for a part of the blockchain data.

Merkle tree가 strong accumulator!

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled%205.png)

[](https://eprint.iacr.org/2015/718.pdf)

- Merkle root value: Accumulator value
- Merkle path: Membership witness
- Strong: accumulator를 생성하고 새로운 element를 accumulator에 추가하기 위해 누군가를 trust하지 않아도 된다. Trustless한 accumulator를 strong accumulator라 함.(secret을 require하지 않음.)
—> 누군가를 trust하지 않는다의 누군가가 누구??
- Accumulator는 accumulator value를 가짐.
새로운 element를 추가할 때 마다 accumulator value가 변경. 특정 element가 accumulator에 존재하고 있음을 membership witness로 증명

Polynomial-time algorithm: 다항 시간 안에 해결할 수 있는 알고리즘

## Asynchronous Accumulator

Accumulator는 element 추가 횟수에 1:1 비율로 membership witness 업데이트를 새로 추가.

- Low update frequency: Asynchronous accumulator는 새로 추가되는 element 개수에 완전히 비례하지 않음.(Sub-linear)

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled%206.png)

- 여러 개의 merkle tree의 merkle root 값으로 이뤄짐. merkle root 값의 리스트
- Membership witness는 merkle tree의 leaf node에서부터 해당 leaf node를 가지고 있는 merkle root까지의 merkle path. (위 그림에서 점섬으로 그려진 노드)
- Membership witness 업데이트는 merkle tree가 merge 될 때만 발생.
- Accumulator에 저장되는 merkle root 값들은 업데이트가 아닌 append-only.
(merkle_root = H(H(h1 || h2) || (H3 || h4))

[Introduction to Accumulator | Starcoin](https://starcoin.org/en/developer/tech/accumulator/)

- Old-Accumulator Compatibility: 최신의 membership witness를 가지고 이전 상태의 accumulator 상태에서의 특정 데이터 membership 검증이 가능한 특성.
(3.2 참고)
- Merkle Mountain Ranges 자료 구조와 같다는 비탈릭의 피드백. → 새로운 accumulator를 제안

## History in the stateless model

Asynchronous accumulator, accumulator는 history data만을 다룸.

History는 변경이 불가능하며 append-only 한 data.

Transaction History, Block History, Receipt History, Contract

Stateless client는 witness가 tx와 함께 제공되기 때문에 state(데이터를 찾는 행위)를 필요로 하지 않음. Stateless client는 동적(데이터 수정 가능)인 상태를 다루는 것보다 정적(데이터 수정 불가능, 추가는 가능)인 히스토리를 다루는 것이 더욱 쉬움.

## Merkle Mountain Ranges(MMR)

[opentimestamps-server/merkle-mountain-range.md at master · opentimestamps/opentimestamps-server](https://github.com/opentimestamps/opentimestamps-server/blob/master/doc/merkle-mountain-range.md)

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled%207.png)

- MMR 데이터는 모두 digest하며 append-only.
- Digeset: 어떤 값을 해시 함수에 넣어서 나온 결과 값.
- MMR 각 노드위 숫자: 노드가 생성된 순서 (15개면 새로운 tree 생성??)
- 새로운 node 추가는 다른 node 변경을 야기하지 않음.
Merkle trie는 새로운 node가 생김에 따라 intermediate node의 변경이 일어남.
- mountain의 꼭대기를 peak이라 함.

1. Leaf node의 개수를 통해 MMR 높이를 구할 수 있음. 높이는 log2(n)의 내림 값.
2. MMR의 Peak node는 항상 왼쪽에서 먼저 생성. 가장 높은 peak 역시 왼쪽 mountain에서 생성. Peak node들은 모두 2^n-1 포지션에 존재.

    
    2^0 - 1 = 0, and 0 < 11
    
    2^1 - 1 = 1, and 1 < 11
    
    2^2 - 1 = 3, and 3 < 11
    
    2^3 - 1 = 7, and 7 < 11
    
    2^4 - 1 = 15, and 15 is not < 11
    
3. Peak node를 찾은 후, 다음 peak 찾기 위해서 right-sibling node로 점프(+2^(h+1)-1) 혹은 left-child(-2^h)를 취함. h는 높이. 

### Merkle root(Accumulator value)

MMR에는 대표하는 단 하나의 Merkle root가 존재하지 않음.

MMR에서 Merkle root를 만들기 위해서는 bagging이라는 작업 필요. bagging은 MMR에 존재하는 꼭대기(Peak) 노드들을 이어 붙인 다음 이를 해싱. 이 값이 Merkle root 값.

### Merkle path(Memebership witness)

위 그림 예시에서 position 10 leaf node의 merkle path는 다음의 data를 포함.

(1) Merkle Root(bagging)

(2) 꼭대기 노드 데이터: 17, 18

(3) 이외의 데이터: 11번(merge(10,11) = 12), 9번(merge(9, 12)= 13), 6번(merge(6,13)=14).

## Log Accumulator

[Batching and cyclic partitioning of logs](https://ethresear.ch/t/batching-and-cyclic-partitioning-of-logs/536)

[Multi-tries vs partial statelessness](https://ethresear.ch/t/multi-tries-vs-partial-statelessness/391)

MMR을 이용한 새로운 Accumulator.

히스토리를 저장. 히스토리는 log.

Log accumulator는 3MR를 사용. 3MR은 multi-MMR의 약자. MMR의 집합.

Stateless client에서 각각의 샤드는 2^n개로 구성된 3MR을 가짐.

각각의 3MR은 0, 1, …, (2^n)-1로 라벨링. 

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled%208.png)

Shard chain에서 Block을 Collation이라 함.

Collation을 생성하는 주체가 Collator.

높이가 i인 Collation을 받으면(누가??) collation에 포함된 log들을 가지고 merkle tree 생성.

이렇게 생성된 merkle tree의 root hash 값이 log batch root 라고 함.

log batch root 만드는 과정이 batching이라 함.

Batching 이후, cyclic partitioning 작업 수행. 

Log batch root는 샤드가 갖고 있는 2^n개의 MMR 중 하나에 삽입. 

높이 i인 collation에서의 log batch root는 i mod 2^n 연산을 통해 나온 숫자로 labeling 된 MMR에 삽입.

(e.g.,  n 이 2이고 높이가 10인 collation으로 부터 생긴 log batch root는 4개의 MMR 중 2로 라벨링된 MMR에 삽입.)

특정 log가 Log accumulator의 멤버임을 입증해야 함.

witness 가 필요.

1. 로그 l을 가진 leaf node에서 log batch root까지의 merkle path. (batching)
2. MMR에 삽입된 log batch root를 가지는 leaf node에서 MMR의 merkle root까지의 merkle path(MMR의 merkle path - cyclic partitioning)

Log Accumulator가 MMR에서 개선된 점.

1. Deterministic witness update events
: Batching 작업으로 인해, 높이 i인 collation에 포함된 log들을 포함하는 log batch root는 2^n개의 MMR 중 하나의 MMR m1(예시)에 추가. 여기서 결정적(deterministic)이라는 의미는 m1의 witness 업데이트가 collation 높이에만 의존. MMR, m1의 witness 업데이트가 언제 이뤄질지 예측 가능.
2. Reduced witness update events
: “partitioning”하기 때문에, 각각의 MMR에서의 witness 업데이트는 1/2^n(2^n == MMR의 개수) 만큼 발생. Log accumulator가 2^n개의 MMR로 구성되어 있음.
3. Cheap historical data availability
: 로그가 주어질 때, witness를 새로 업데이터하기 위해서는 genesis이후부터 만들어진 모든 collation의 log batch root만을 가지고 있으면 됨.  이 log batch root를 collation headere애 기록하면, 특정 샤드에 대한 모든 SPV(Simplified Payment Verification)노드 역시 log witness 업데이트에 필요한 데이터들을 제공할 수 있게 됨. SPV node는 collaation header만을 저장. 이 header에 log batch root가 기록 됨.

## Double-batched Merkle Log Accumulator

![Untitled](State%20Machine%20df265b621364403aa1c0adc9914dc4cc/Untitled%209.png)

Log accumulator에 log가 추가되면서 생기는 witness 의 업데이트를 최소화하도록 발전시킨 accumulator.

Log accumulator의 batching 부분을 two layer로 나눔.

모든 샤드에 32byte hash를 저장하는 두 개의 버퍼 제공.

1. Bottom buffer
: 2^n 개의 entry를 가지는 고정된 사이즈. 각각의 entry가 0, 1,.. 2^n-1로 라벨링되는 버퍼.
2. Top buffer
: Collation 높이에 따라 사이즈가 증가하는 버퍼.

높이가 i인 collation log들을 batching해서 log batch root 생성.

Log batch root를 bottom buffer의 i mod 2^n 연산으로 나온 숫자 값으로 labeling된 entry에 삽입. 

만약 bottom buffer의 마지막 entry(2^n-1)에 log batch root가 삽입되면 bottom buffer의 모든 entry들을 가지고 merkle tree를 만들어서 merkle root 값인 batch hash를 만듬.

만들어진 batch hash를 top buffer에 append.

즉 bottom buffer의 모든 log batch root로 merkle tree를 만들고, 이 merkle tree의 merkle root 값을 top buffer에 추가.

Bottom buffer 마지막 entry 값이 채워질 때만 witness update가 발생.

### Membership witness

- pre-witness: 로그 l에서부터 Bottom buffer의 entry에 포함된 log batch root 까지의 merkle path.
- permanent witness: log batch root 값들로 이루어진 merkle tree에서 merkle path와 pre-witness를 묶어서(concatenaating) 만든 것.

Double-batched Merkle Log Accumulator의 사이즈는 32 * (2^n + h/2^n) bytes가 됨. h는 collation의 높이. 

Collation 인터벌 8초 n=13 이라 가정.

Bottom buffer는 250 kB 고정.

Top buffer는 51년이 지나면 750 kB.

## State-minised executions

각각의 샤드 체인은 Double-batched Merkle Log Accumulator를 가짐. 유저로 부터 log를 받으면 해당 log를 accumulator에 저장.

이러한 샤드 체인을 log shard라 함.

State-minised contract를 사용. 

기존 stateful 한 contract는 nonce, balance, storageRoot, codeHash를 가짐.

State-minimised contract는 storageRoot만 가짐.

State-minimised contract가 저장하는 state root를 virtual state root라 함.

State-minimised의 state transition을 virtual state transition이라 함.

1. 유저는 Log T 형대의 tx를 log shard에 보냄. 이 tx를 virtual transaction이라 부름. 보냄 virtual tx는 log shard 의 double-batched merkle log accumulator에 push되어 저장.
2. State-minimised contract에 대한 virtual tx이 log shard에 제출되면 excutor는 state-minimiseed contract의 컨펌되지 않은 새로운 virtual state root를 제안.
executor는 새로운 virtual state root를 제안하는 자. executor가 virtual state root를 제안하려면 일정량의 담보를 deposit해야 함.
3. executor가 제안한 virtual state root 가 invalid한 값일 경우. 누구나 challenge 신청 가능. challenge가 성공하면 executor는 예치한 담보물을 잃음.
새로 제안한 virtual state root가 challenge 없이 성공적으로 confirm되면 executor 는 일정 금액을 보상으로 얻음. State-minimised contract의 virtual state root 값은 변경 됨.

Virtual tx는 witness를 함께 보내지 않음.

Virtual tx는 기존의 stateful한 node에서 실행되는 tx를 기반으로 만들어지기 때문.

Executor는 가장 tx에 대한 tx를 자신의 node, 즉 stateful한 node에서 실행하여 witness 없이도 상태에 접근 가능하게 함.

유저는 단지 virtual tx를 보내기만 하면 됨.

본래 제안된 Stateless client 는 stateless client에 tx를 보낼 때, 항상 witness가 필요했음.

State-minimised execution 모델에서 log shard가 저장하고 있는 log로부터 data availability를 확보하고 있으며, 담보물을 걸고 virtual state root를 제안하는 것과 같은 방식의 cryptoeconomic인 execution 모델로 데이터 유효성(validity)를 확보하는 것을 확인 할 수 있음. 그리고 tx를 보낼 때, witness가 필요하지 않음.

Executor는 validator와 달리 각각의 shard chain에 shuffling되어 배정되지 않음. Executor가 하는 일은 state-minimised contract에 대한 컨펌되지 않은 virtual  state root를 제공하는 것. Executor는 자신이 관리하는 state-minised contract가 속해 있는 특정 shard에서만 활동. (Eth 2.0 에서의 각각 shard chain들은 완전히 별개의 account들로 관리). Executor들은 log shard chain에서 발생하는 virtual tx에 대해 특정 state-minimised contract와 관련이 있는 것만 처리하여 virtual state root에 제공.

> Double-batched Merkle Log Accumulator와 State-minised 실행 모델이 shard protocol 레벨에서 구현될 수 있다고 함.
이를 기반으로 stateless client가 만들어진다면 validator shuffling이 더욱 빨라질 것이고, stateless client 기반 light node 대중화를 이끌 수 있음.
- Justin Drake
>