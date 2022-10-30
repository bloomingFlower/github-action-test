# 3. CoinFlip

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v3.3/contracts/math/SafeMath.sol';

contract PsychicFlip {

  using SafeMath for uint256;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
  bool public checkvar;
  
  function callCoinFlip(address add) public returns (bool) {
      bool flipVar = getflip();
      (bool check, bytes memory data) = address(add).call(abi.encodeWithSignature("flip(bool)",flipVar));
      checkvar = check;
      return check;
  }

  function getflip() public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number.sub(1)));

    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;

    return side;
  }
}
```

contract 코드를 직접 call 하는 방식

interface를 통해 call하는 방식

후자가 깔끔해보이긴 함.

무엇이 더 옳을까.

[Blocks | ethereum.org](https://ethereum.org/en/developers/docs/blocks/)

[Solidity by Example](https://solidity-by-example.org/interface/)

[https://www.youtube.com/watch?v=YWtT0MNHYhQ](https://www.youtube.com/watch?v=YWtT0MNHYhQ)