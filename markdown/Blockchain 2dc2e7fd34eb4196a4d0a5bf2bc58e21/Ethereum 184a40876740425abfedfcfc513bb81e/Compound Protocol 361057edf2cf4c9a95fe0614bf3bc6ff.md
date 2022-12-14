# Compound Protocol

[중개인 없는 단기대출시장의 등장: 컴파운드 프로토콜](https://medium.com/decon-lab/%EC%A4%91%EA%B0%9C%EC%9D%B8-%EC%97%86%EB%8A%94-%EB%8B%A8%EA%B8%B0%EB%8C%80%EC%B6%9C%EC%8B%9C%EC%9E%A5%EC%9D%98-%EB%93%B1%EC%9E%A5-%EC%BB%B4%ED%8C%8C%EC%9A%B4%EB%93%9C-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C-a2058305b436)

### 암호화폐 담보대출

: 차입자가 암호화폐를 담보로 제공. 법정화폐 혹은 다른 암호화폐를 차입하는 대출 방식.

- 수탁형(custody): 기존 금융기관과 같이 중앙화된 수탁자 혹은 제 3자로부터 대출 진행
- 비수탁형(non-custody): 분산화된 대출 프로토콜로부터 대출 진행. 중개인을 완전히 배제. 중개수수료를 낮추고 중개인 리스크 방지.

비수탁형 대출의 유동성 문제 ⇒ 단기금융시장(money market) 시스템으로 해결

## 1. P2P 대출 문제

: P2P: 온라인 플랫폼을 통해 대출자(lender)와 차입자(borrower)를 연결함으로서 제 1금융권 대출을 받기 어려운 개인 혹은 기업이 중금리(8~12%)로 대출을 받을 수 있게 하는 것.
국내 대다수 P2P대출 서비스는 ‘대출형 크라우드 펀딩'. 다수의 개인으로부터 모아진 대출 자금을 제공.

![Untitled](Compound%20Protocol%20361057edf2cf4c9a95fe0614bf3bc6ff/Untitled.png)

1) 대출심사 및 신용/담보평가 불투명성 (Opacity/ Lack of Transparency)

- P2P 대출 회사가 차입자와 짜고 허위 대출 시행. 신용 및 담보평가 부실화로 투자자가 원금 회수 불가능.

2) 자산 수탁자/ 업체에 대한 위험 (Counterparty Risk)

- P2P 대출 회사가 수탁 관리하던 투자금 및 담보 등이 중간에 횡령. P2P 대출 회사 자진 폐업 시, 투자자는 잔금 상환 받기 어려움.

## 2. 비수탁형 1:1 매칭 암호자산담보대출 과 유동성 문제

1) 수탁형 대출(Custodial Loan) vs 비수탁형 대출(Non-custodial Loan)

P2P는 수탁형 대출. 신뢰비용이 낮다는 장점.

> 신뢰비용: 서로 신뢰할 수 없는 대출자와 차입자간 거래 시, 정상적인 거래가 발생하지 않을 것을 우려하여 발생하는 비용. (e.g., 스트레스)
> 

블록체인은 비수탁형 대출 구현 가능. SC로 대출금, 담보자산의 관리 및 전달이 자동화. 거래 내역은 조작 불가능한 공동 장부에 기록.

![Untitled](Compound%20Protocol%20361057edf2cf4c9a95fe0614bf3bc6ff/Untitled%201.png)

Smart Contract를 통한 비수탁형 대출 예시

30일 만기 담보 대출 계약

대출자는 대출금을 예치, 차입자가 계약 조건에 부합하는 담보자산을 예치 시, 대출금은 차입자에게 자동으로 이전. 차입자가 30일(만기) 이내에 대출금을 갚기 위해 원금 및 이자를 예치하면 담보를 되돌려 받을 수 있음. 이 모든 기능은 smart contract에 의해 동작.

2) 비수탁형 대출의 장단점

- 장점: 중개인 제거로 중개인 리스크 제거, 중개 수수료 줄일 수 있음
- 단점: 
대출자는 오더북에 올라온 대출 요청 건 중 자신의 조건에 맞는 것을 직접 찾아야 함.
유동성 문제. 유통되는 통화량이 부족한 경우. 평소보다 높은 이자를 지불해야 하는 문제점.

## 3. Compound Protocol(유동성 문제 솔루션)

: 컴파운드 프로토콜은 1:1 매칭이 아닌 money marktet 시스템.
기존 금융수탁기관처럼 유동성 풀을 가지지만 개인은 유동성풀의 자기 자산으 스스로 통제 가능! 사용자는 개인 키 및 퍼블릭 키를 통해 프로토콜에 자산 관리 권한 통제 가능(무슨 상관 ???)

1) Money market Fund

![Untitled](Compound%20Protocol%20361057edf2cf4c9a95fe0614bf3bc6ff/Untitled%202.png)

: 상환 만기를 1년 이내로 자금을 운용하는 단기 금융 시장.
은행, 증권사와 같은 금융 기관 혹은 우량 기업이 일시적으로 자금 부족시 단기 채권을 발행하여 신속하게 유동성을 공급 받기 위한 곳.

펀드는 투자를 위해 모인 자금.

2) MMF of Compound Protocol

![Untitled](Compound%20Protocol%20361057edf2cf4c9a95fe0614bf3bc6ff/Untitled%203.png)

암호자산을 누구나 원할 때 바로 단기 대출 받을 수 있는 머니마켓.

암호화폐별 머니마켓 존재

특정 암호화폐에 대한 머니마켓 별 유동성 풀 구축

3) 작동 방식

![Untitled](Compound%20Protocol%20361057edf2cf4c9a95fe0614bf3bc6ff/Untitled%204.png)

차입자는 대출자산 총 가치의 1.5배 이상이 되는 암호자산을 컴파운드 머니마켓에 맡기면 해당 암호자산으로 담보 대출 시행.

대출을 통해 발생한 이자는 암호자산을 맡긴 사람에게 분배. 담보에도 이자가 쌓임.

장점

- 상시 이자 제공: 15초 간격(erc20 기준) 이자 쌓임(연 이자율의 1/2102400)
- 대출계약 시 마찰 제거: 알고리즘으로 이자율은 자동 결정. 대출자-투자자 간 협의 없음.
- 최소 지급 금액 없음: 차입자가 원하는 금액에 맞춰 빌려 줄 필요 없이 원하는 만큼 자유롭게!
- 수시 입출금 가능: 투자자는 원래 차입자가 대출금 상황하기 전까지 투자금 회수가 어려우나 이건 언제나 가능!

4) Compound Protocol 유동성 조절 메커니즘

잠재적 유동성 부족 문제

- 머니마켓내 대출 가능 자금 대비 대출 수요가 월등히 높을 때
- 이용자가 차입금을 갚지 않거나 담보 부실로 담보를 통한 원금회수가 어려운 경우 빈번히 발생할 때.

해결

- 알고리즘을 통한 자율 조절
- 담보 자산 청산

알고리즘

![Untitled](Compound%20Protocol%20361057edf2cf4c9a95fe0614bf3bc6ff/Untitled%205.png)

Utilization Rratio: 대출 여력 수치

S(spread): 컴파운드 프로토콜의 수수로율(10~15%). 수수료 축적 후 차입자가 제대로 대출금을 갚지 못해 유동성 위기가 올때 활용.

담보 자산 청산을 통한 담보 부실화 방지

최소 담보대출 비율(collateral ratio)를 맞추지 못하면 이용자 계정의 담보 청산.

담보대출 비율: 담보물의 가격과 대출금액의 비율.

컴파운드의 최소 담보대출 비율은 150%.

가격 오라클이 상위 10개의 거래소의 가격 정보를 가져와 자산 가치 결정.

담보자산의 가치가 하락하거나

차입한 자산의 가치가 상승하면

담보부실화 발생으로 청산 발생 및 원금 회수가 어려워짐.

계정 유동성(전체 공급 자산 - 전체 차입액 * 담보대출비율)을 이용자에게 알림. 0이면 최소담보대출비율에 맞춘다는 것. 마이너스이면 추가로 담보를 제공하거나 담보를 청산하여 차입액을 갚아야 함. 유동성 적자 전환 시, 해당 계정의 담보 일부를 5% 할인된 가격에 처분하여 차입금 일부를 상환.

### 단점

1. 위원회에 높은 의존성
: 담보자산과 차입자산간 교환 비율 결정 및 담보 청산 행위를 모두 컴파운드 프로토콜 위원회에서 이루어짐.
: 특정 머니마켓을 중단 시킬 수 있음(탈중앙화 할 것이라고 함)
2. 모든 종류의 암호자산 대출 제공하지 않음.
3. 예금이자율이 0일 수 있음.
: 공급 대비 대출 수요가 지극히 없는 경우.

[로직 분석 - 예금. 출금](Compound%20Protocol%20361057edf2cf4c9a95fe0614bf3bc6ff/%E1%84%85%E1%85%A9%E1%84%8C%E1%85%B5%E1%86%A8%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20-%20%E1%84%8B%E1%85%A8%E1%84%80%E1%85%B3%E1%86%B7%20%E1%84%8E%E1%85%AE%E1%86%AF%E1%84%80%E1%85%B3%E1%86%B7%200b349050104945bb8af63c674fb17729.md)