# Bug Hunting Smart Contract for Fun & Profit 사전 자료

## 공지사항

- 세미나 장소 : 서울특별시 강남구 테헤란로 126 지하1층, 수호아이오
- 주차 : 세미나 장소 내 주차는 어려우므로, 가급적 대중교통 혹은 근처 유료 주차장을 이용해주시기 바랍니다. (그레이스타워, 한국과학기술회관, 한국지식재산센터 등)
- Session 03 시간 : 2022년 8월 12일 19:30 ~ 21:00
- 노트북 지참을 권장드리며, 기기 사양이나 종류는 상관 없습니다. (인원이 많은 관계로 카페 내 배터리 충전이 어려울 수 있습니다.)
- [사전/사후 질문 및 보안 연구팀 참여 관련 디스코드](https://discord.gg/v4XksasYtr)
- [MATZIP or 수호 채용 관련 커피챗 신청](https://calendly.com/henleykim/coffeechat)

## 사전 자료

- **Session 01 : Introduction to Web3 and Smart Contract**
    - Session 01의 목표는 **Ethereum과 Solidity에 대해 알고, Hardhat 사용법을 익히는 것**입니다.
    - Ethereum과 Solidity에 대해 잘 알고, Hardhat을 사용할 줄 아시는 분들께는 본 세션이 다소 지루할 수 있습니다.
    - **Javascript / Typescript**을 알고 계셔야 실습과 시연을 정확히 이해하실 수 있습니다. (모르셔도 테스트를 돌려 볼 수 있도록 코드는 제공해 드릴 예정입니다.)
    - **Node.js를 실습환경에 사전에 세팅해오셔야 실습이 가능합니다.**
        - [Node.js와 NPM 설치 (Windows)](https://hello-bryan.tistory.com/95)
        - [Node.js와 NPM 설치 (MacOS)](https://memostack.tistory.com/274)
    - [실습/시연 코드 Github 저장소](https://github.com/astean1001/2022_seminar_tutorial)
        - 실습 내용 : Hardhat으로 ERC20 코드 테스트 해보기 + SimpleSwap 코드 테스트 해보기 ([Hardhat 세팅하기](https://hardhat.org/hardhat-runner/docs/guides/project-setup))
        - [실습 영상 1 (Compile)](https://drive.google.com/file/d/1bwowoUX9Ego8crL61OdMQchDrmqdwdPV/view?usp=sharing), [실습 영상 2 (Test)](https://drive.google.com/file/d/1DSK-kkzF8k_qWt7UGc6XTWD8C4iEnO1y/view?usp=sharing)
    - **사전 참고 자료**
        - [Ethereum이 무엇인가요?](https://academy.binance.com/ko/articles/what-is-ethereum)
        - [Smart Contract란 무엇인가요?](https://brunch.co.kr/@c6086b58f6724c5/8)
        - [Solidity 언어 공식 문서](https://docs.soliditylang.org/en/v0.8.15/)
        - [Solidity by Example](https://solidity-by-example.org/)
        - [Hardhat 공식 문서](https://hardhat.org/docs)
        - [The Complete Hands-On Hardhat Tutorial](https://betterprogramming.pub/the-complete-hands-on-hardhat-tutorial-9e23728fc8a4)
        - [Ethers.js 공식 문서](https://docs.ethers.io/v5/)
        - [Chai 공식 문서](https://www.chaijs.com/)
        - [Damn Vulnerable DeFi](https://github.com/tinchoabbate/damn-vulnerable-defi/tree/master)
- **Session 02 : Common Vulnerabilities and Case Study**
    - Session 02의 목표는 **EVM-호환 Smart Contract 환경에서 발생할 수 있는 취약점들을 이해**하고 실사례 코드로 확인하는 것입니다.
    - **Solidity를 읽고 이해하는데 어려움이 없으셔야** 세션을 원활히 이해하실 수 있습니다.
    - **사전 참고 자료**
        - [Uniswap이 무엇인가요?](https://academy.binance.com/ko/articles/what-is-uniswap-and-how-does-it-work)
        - [Uniswap 코드 분석](https://velog.io/@wrjang96/Uniswap-Core)
        - [Smart Contract Weakness Classification](https://swcregistry.io/)
        - [Security Pitfalls & Best Practices 101](https://secureum.substack.com/p/security-pitfalls-and-best-practices-101?s=r)
        - [Rekt.news](https://rekt.news/)
        - [Slowmist Hacked](https://hacked.slowmist.io/)
        - [Punk.Finance Code](https://github.com/PunkFinance/punk.protocol/blob/1654cc66d47ce234d339d4437cf1420a4bbbc4b8/contracts/models/CompoundEthModel.sol)
        - [Reaper Farm Exploit Analysis](https://twitter.com/PeckShieldAlert/status/1554423041232629761)
        - [Reaper Farm Contract](https://github.com/Byte-Masons/reaper-core/tree/master/contracts)
        - [EIP-4626](https://eips.ethereum.org/EIPS/eip-4626)
        - [Elephant.Money](https://elephant.money/)
        - [Elephant Finance Incident Analysis](https://twitter.com/BlockSecTeam/status/1513966074357698563)
        - [Fei Protocol Incident Analysis](https://certik.medium.com/fei-protocol-incident-analysis-8527440696cc)
        - [bZx 소개](https://medium.com/hashed-kr/investment-in-bzx-kr-3a739e157e29)
        - [EIP-777 Reentrancy](https://blog.openzeppelin.com/exploiting-uniswap-from-reentrancy-to-actual-profit/)
        
- **Session 03 : How to find real-world bugs?**
    - Session 03의 목표는 실제 Bug Bounty 상황을 가정하고 주어진 **임의의 프로젝트를 분석하여 취약점을 찾는 방법**을 익히는 것입니다.
    - **Solidity를 읽고 이해하는 것과, Javascript / Typescript 코드를 작성하시는 것에 어려움이 없으셔야** 세션을 원활히 이해하실 수 있습니다.
    - **Node.js를 실습환경에 사전에 세팅해오셔야 실습이 가능합니다.**
    - [실습/시연 코드 Github 저장소](https://github.com/astean1001/2022_seminar_tutorial)
    - **사전 참고 자료**
        - [code4rena](https://code4rena.com/)
        - [immunefi](https://immunefi.com/)

## 발표 자료

*좌석 위치에 따라 화면 내 코드가 잘 안 보일 수 있으므로, 발표 자료를 미리 다운 받아 노트북을 지참해주시면 감사드리겠습니다. (발표 자료는 세미나 당일 업로드 될 예정입니다.)

**Session 01**

[SESSION01_Intro.pdf](Bug%20Hunting%20Smart%20Contract%20for%20Fun%20&%20Profit%20%E1%84%89%E1%85%A1%E1%84%8C%E1%85%A5%E1%86%AB%20%20fd278603e78c48a69b94ac455c68039b/SESSION01_Intro.pdf)

[SESSION01_01_COMPILE.mp4](Bug%20Hunting%20Smart%20Contract%20for%20Fun%20&%20Profit%20%E1%84%89%E1%85%A1%E1%84%8C%E1%85%A5%E1%86%AB%20%20fd278603e78c48a69b94ac455c68039b/SESSION01_01_COMPILE.mp4)

[SESSION01_02_TEST.mp4](Bug%20Hunting%20Smart%20Contract%20for%20Fun%20&%20Profit%20%E1%84%89%E1%85%A1%E1%84%8C%E1%85%A5%E1%86%AB%20%20fd278603e78c48a69b94ac455c68039b/SESSION01_02_TEST.mp4)

**Session 02** 

[SESSION02_Vulns.pdf](Bug%20Hunting%20Smart%20Contract%20for%20Fun%20&%20Profit%20%E1%84%89%E1%85%A1%E1%84%8C%E1%85%A5%E1%86%AB%20%20fd278603e78c48a69b94ac455c68039b/SESSION02_Vulns.pdf)

**Session 03**

[SESSION03_Real.pdf](Bug%20Hunting%20Smart%20Contract%20for%20Fun%20&%20Profit%20%E1%84%89%E1%85%A1%E1%84%8C%E1%85%A5%E1%86%AB%20%20fd278603e78c48a69b94ac455c68039b/SESSION03_Real.pdf)

safetransfer 주석 해제하고 한 번 테스트해보시길..!!

takeaway framework를 따라가면 왠만한 취약점은 다 잡을 수 있을 것이라 봄. 

바이트 단위 취약점 프록시 덮어씌우기 같은 취약점은 버그헌팅하면서 다른 사람이 찾은거보고 경험과 짬으로 채워나가면 될 거라 봄.