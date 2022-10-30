# Avalanche(AVAX)

## What is Avanlanche?

Open-source Platform

4500+ TPS

Instantly confirm(1초 미만)

Solidity

Fast, Flexible, Secure

Plug and Play

### Permissionless

Permissionless, Permissioned 둘 다 지원

AVAX로 새 서브넷을 만들거나 permissionless 서브넷 검증이 가능

private 서브넷 가능

## Ava Labs?

Founded by Cornell Computer scientists

The company has received funding from Andreessen Horowitz, Initialized Capital, and Polychain Capital, with angel investments from Balaji Srinivasan and Naval Ravikant.

## Chains

X-Chain: trade

P-Chain: stake, validating the Primary Network

C-Chain: smart contract, pay for gas

## Consensus Protocol(X-Chain)

([https://medium.com/avalancheavax/avalanche-consensus-101-99c68a3e3159](https://medium.com/avalancheavax/avalanche-consensus-101-99c68a3e3159))

([https://docs.avax.network/overview/getting-started/avalanche-consensus/](https://docs.avax.network/overview/getting-started/avalanche-consensus/))

([https://www.reddit.com/r/Avax/comments/n2k7jq/want_a_full_description_of_the_snowman_consensus/](https://www.reddit.com/r/Avax/comments/n2k7jq/want_a_full_description_of_the_snowman_consensus/))

Voting Protocol

DAG-optimized consensus protocol(a Directed Acyclic Graph)

high throughput(CPU-bound), paralleizable, simple to prune

![Untitled](Avalanche(AVAX)%20dfaff77ca5c049358bc9b18340f993a4/Untitled.png)

*[https://avalabs.org/whitepapers](https://avalabs.org/whitepapers)*

probabilistic protocol(make the chance of error just as microscopic) smaller than chance of SHA-256 hash collision

No tx to vote on → nodes in the network don’t do anything except listen.

투표를 하기 위해서 일정량의 토큰을 가져야함(PoS)

messages per node

Ava(O(1))

Classical(O(n^2))

Validator: listen for tx

vote: accepted/ rejected

1. A validator is presented with a set of transactions that have been issued and asked to decide which transactions to “Accept.”
    
    ![Untitled](Avalanche(AVAX)%20dfaff77ca5c049358bc9b18340f993a4/Untitled%201.png)
    
2. The node client presents the virtual machines (“VMs”) with their transactions, and the VMs will provide information to help the client determine which transactions are acceptable.
3. The validator selects a subset of these transactions that do not conflict, marks them as preferred, and attempts to accept them over the network.
4. Any nodes that query this validator receives its latest preferred choice for this decision.
5. This validator selects K nodes from the entire validator list (probability of selection is weighted by stake amount) to query for their preferred decision.
6. Each queried validator responds with their preferred decisions, the validator’s votes are updated accordingly, and confidence is built.
7. Meanwhile, other nodes are also selecting random validators from the validator set to be queried for their preferred response and updating their answers accordingly.
8. This continues for at least M rounds or until a sufficient confidence threshold is reached, whatever comes last, with K validators selected randomly each round (without replacement).
9. Once a confidence threshold is reached, the decision on the transaction is locked in as final.
10. If “Accepted”, the transaction is sent to the VM to be handled; if “Rejected”, the transaction is removed from consensus.

## Avalanche-Ethereum Bridge

bridge.avax.network

![Untitled](Avalanche(AVAX)%20dfaff77ca5c049358bc9b18340f993a4/Untitled%202.png)

WETH.e로 C-Chain내 전송. 바로 Eth. Network로 전송 시 소실, bridge를 꼭 통해서 가야함.

CoinbasePro 가 더 간편.

.e는 WETH에서 WETH로 오전송을 막기 위한 브릿지에서 사용하는 지정자

### Bridge-SGX application

### Bridge-Wardens

third-party indexers and verifiers

ERC20 지원 

## Wallet

Avalanche Wallet: [https://wallet.avax.network](https://wallet.avax.network/)

MetaMask(C-Chain): [https://support.avax.network/en/articles/4626956-how-do-i-set-up-metamask-on-avalanche](https://support.avax.network/en/articles/4626956-how-do-i-set-up-metamask-on-avalanche)

![Untitled](Avalanche(AVAX)%20dfaff77ca5c049358bc9b18340f993a4/Untitled%203.png)

X-Chain ↔ C-Chain

P-Chain ↔ C-Chain

X-Chain ↔ P-Chain

by Avanlanche Wallet

![Untitled](Avalanche(AVAX)%20dfaff77ca5c049358bc9b18340f993a4/Untitled%204.png)

![Untitled](Avalanche(AVAX)%20dfaff77ca5c049358bc9b18340f993a4/Untitled%205.png)

![Untitled](Avalanche(AVAX)%20dfaff77ca5c049358bc9b18340f993a4/Untitled%206.png)

## Stake

About 9% returns

P-Chain 만 가능(최소 스테이킹 25 AVAX)

![Untitled](Avalanche(AVAX)%20dfaff77ca5c049358bc9b18340f993a4/Untitled%207.png)

## Fee

Ethereum to Avalanche

$3 ERC20 asset (Avalanche bridge only allows transfers to the same address

Avalanche to Ethereum

expected Ethereum transaction fee.($20)

## DEX

Avalanche 자체 DEX는 없음.

타 DEX 서비스 이용

## NFT

AVA는 X-Chain과 C-Chain에 NFT를 가질 수 있음. 다만 X와 C-Chain 간 NFT 이동은 불가

- [Joepegs](https://joepegs.com/home) is a new NFT marketplace created by TraderJoe.
- [NFTrade](https://nftrade.com/) is one of the larger NFT marketplaces on the Avalanche network and can get you started.
- [Kalao](https://marketplace.kalao.io/) is also one of the larger NFT marketplaces on the Avalanche network.

## Bug Report

[https://hackenproof.com/avalanche](https://hackenproof.com/avalanche)

![Untitled](Avalanche(AVAX)%20dfaff77ca5c049358bc9b18340f993a4/Untitled%208.png)

## Communitiy

**Ava Labs Official Links:**

- [Ava Labs Website](https://avalabs.org/)
- [Ava Labs Team](https://www.avalabs.org/team)
- [Ava Labs Blog](https://medium.com/avalancheavax)
- [Whitepapers](https://www.avalabs.org/whitepapers)

**Ava Labs Social:**

- [GitHub](https://github.com/ava-labs/)
- [Twitter](https://twitter.com/avalabsofficial)
- [LinkedIn](https://www.linkedin.com/company/avalabsofficial/)
- [Facebook](https://www.facebook.com/avalabsofficial/)
- [Instagram](https://instagram.com/avalabsofficial/)

**Avalanche Official Links:**

- [Avalanche Website](https://avax.network/)
- [Avalanche Hub](https://community.ava.network/)
- [Avalanche Documentation](https://docs.avax.network/)

**Avalanche Social:**

- [Twitter](https://twitter.com/intent/follow?screen_name=avalancheavax)
- [Discord](https://chat.avalabs.org/)
- [Announcements Telegram](https://t.me/avalanche_announcements)
- [Community Telegram](https://t.me/avalancheavax)
- [Facebook](https://facebook.com/https://twitter.com/avalancheavax)
- [LinkedIn](https://linkedin.com/company/avalancheavax/)
- [Reddit](https://reddit.com/r/avax)
- [YouTube](https://youtube.com/c/avalancheavax)

**International Social:**

- [Spanish Facebook](https://www.facebook.com/AvalancheEsp)
- [Dutch Twitter](https://twitter.com/Avalanche_NL)
- [Filipino Twitter](https://twitter.com/Avalanche_ph)
- [French Twitter](https://twitter.com/avalanche_fr)
- [German Twitter](https://twitter.com/avalanche_dach)
- [Indonesia Twitter](https://twitter.com/AvalancheID)
- [Italian Twitter](https://twitter.com/avalanche_it)
- [Japanese Twitter](https://twitter.com/avalanchejp)
- [Korean Twitter](https://twitter.com/avalanche_ko)
- [Nigerian Twitter](https://twitter.com/avalanche_ng)
- [Russian Twitter](https://twitter.com/avalanche_ru)
- [Spanish Twitter](https://twitter.com/avalanche_esp)
- [Turkish Twitter](https://twitter.com/avalanche_tr)

**International Community:**

- [Chinese Telegram](https://t.me/avalanche_zh)
- [Dutch Telegram](https://t.me/avalanche_nl)
- [Filipino Telegram](https://t.me/avalanche_fil)
- [French Telegram](https://t.me/avalanche_fr)
- [German Telegram](https://t.me/avalanche_dach)
- [Indonesia Telegram](https://t.me/avalanche_id)
- [Italian Telegram](https://t.me/avalanche_it)
- [Japanese Telegram](https://t.me/avalanche_jp)
- [Korean Telegram](https://t.me/avalanche_ko)
- [Nigeria Telegram](https://t.me/avalanche_ng)
- [Persian Telegram](https://t.me/Avalanche_fa)
- [Portuguese Telegram](https://t.me/avalanche_pt)
- [Russian Telegram](https://t.me/avalanche_ru)
- [Spanish Telegram](https://t.me/avalanche_es)
- [Turkish Telegram](https://t.me/avalanche_tr)
- [Vietnamese Telegram](https://t.me/avalanche_vn)

### 참고

[https://www.mk.co.kr/news/economy/view/2019/08/612050/](https://www.mk.co.kr/news/economy/view/2019/08/612050/)