# Uniswap v2

[](https://uniswap.org/whitepaper.pdf)

[https://github.com/Uniswap/v2-periphery](https://github.com/Uniswap/v2-periphery)

[https://github.com/Uniswap/v2-core](https://github.com/Uniswap/v2-core)

[Uniswap-v2 Contract Walk-Through | ethereum.org](https://ethereum.org/en/developers/tutorials/uniswap-v2-annotated-code/)

# Introduction

---

## User Types

- Liquidity Provider: provide the pool with two tokens that can be exchanged. In return, receive a third token that represents partial ownership of the pool called a liquidity token
- Trader: send one type of token to the pool and receive the other

## Core Contracts vs Periphery Contracts

- Core: hold the assets and therefore have to be secure, to be simpler and easier to audit
- Periphery: all the extra functionality required by traders

## Data and Control Flows

1. Swap between different tokens
2. Add liquidity to the market and get reward with pair exchange ERC-20 liquidity tokens
3. Burn ERC-20 liquidity tokens and get back the ERC-20 tokens that the pair exchange allows traders to exchange

---

## Swap

used by traders

### Caller

1. Provide the periphery account with an allowance in the amount to be swapped
2. Call one of the periphery contract’s many swap functions. Every swap function accepts a path, an array of exchanges to go through

### In the periphery contract (UniswapV2Router02.sol)

1. Identify the amounts that need to be traded on each exchange along the path.
2. Iterates over the path. For every exchange along the way it sends the input token and then calls the exchange’s swap function.

### In the core contract (UniswapV2Pair.sol)

1. Verify that the core contract is not being cheated and can maintain sufficient liquidity after the swap.
2. See how many extra tokens we have in addition to the known reserves. That amount is the number of input tokens we received to exchange.
3. Send the output tokens to the destination
4. Call _update to update the reserve amounts

### Back in the periphery contract (UniswapV2Router02.sol)

1. Perform any necessary cleanup (e.g., burn WETH tokens to get back ETH to send the trader)

---

## Add Liquidity

### Caller

1. Provide the periphery account with and allowance in the amounts to be added to the liquidity pool.
2. Call one of the periphery contract’s addLiquidity functions.

### In the periphery contract (UniswapV2Router02.sol)

1. Create a new pair exchange if necessary
2. If there is an existing pair exchange, calculate the amount of tokens to add. This is supposed to be identical value for both tokens, so the same ratio of new tokens to existing tokens.
3. Check if the amounts are acceptable (callers can specify a minimum amount below which they’d rather not add liquidity)
4. Call the core contract

### In the core contract (UniswapV2Pair.sol)

1. Mint liquidity tokens and send them to the caller
2. Call _update to update the reserve amounts

---

## Remove Liquidity

### Caller

1. Provide the periphery account with and allowance of liquidity tokens to be burned in exchange for the underlying tokens
2. Call one of the periphery contract’s removeLiquidity functions.

### In the periphery contract (UniswapV2Router02.sol)

1. Send the liquidity tokens to the pair exchange

### In the contract (UniswapV2Pair.sol)

1. Send the destination address the underlying tokens in proportion to the burned tokens. For example if there are 1000 A tokens in the pool, 500 B tokens, and 90 liquidity tokens, and we receive 9 tokens to burn, we’re burning 10% of the liquidity tokens so we send back the user 100 A tokens and 50 B tokens.
2. Burn the liquidity tokens
3. Call _update to update the reserve amounts

---

# The Core Contracts

These are the secure contracts which hold the liquidity.

---

## UniswapV2Pair.sol

This contract implements the actual pool that exchanges tokens. It is the core Uniswap functionality

```solidity
pragma solidity =0.5.16;

import './interfaces/IUniswapV2Pair.sol';
import './UniswapV2ERC20.sol';
import './libraries/Math.sol';
import './libraries/UQ112x112.sol';
import './interfaces/IERC20.sol';
import './interfaces/IUniswapV2Factory.sol';
import './interfaces/IUniswapV2Callee.sol';
```

These are all the interfaces that the contract needs to know about, either because the contract implements them(*IUniswapV2Pair* and *UniswapV2ERC20*) or because it calls contracts that implement them.

```solidity
contract UniswapV2Pair is IUniswapV2Pair, UniswapV2ERC20 {
```

This contact inherits from UniswapV2ERC20, which provides the ERC-20 functions for the liquidity tokens.

```solidity
using SafeMath  for uint;
```

The SafeMath library is used to avoid overflows and underflows. This is important because otherwise we might end up with a situation where a value should be -1, but is instead 2^256-1.

```solidity
using UQ112x112 for uint224;
```

A lot of calculations in the pool contract require fractions. However, fractions are not supported by the EVM. The solution that Uniswap found is the use 224 bit values, with 112 bits for the integer part, and 112 bits for the fraction. So 1.0 is represented as 2^112, 1.5 is represented as 2^112 + 2^111, etc.

---

### Variables

```solidity
uint public constant MINIMUM_LIQUIDITY = 10**3;
```

To avoid cases of division by zero, there is a minimum number of liquidity tokens that always exist (but are owned by account zero). That number is MINUMUM_LIQUIDITY, a thousand.

```solidity
bytes4 private constant SELECTOR = bytes4(keccak256(bytes('transfer(address,uint256)')));
```

This is the ABI selector for the ERC-20 transfer function. It is used to transfer ERC-20 tokens in the two token accounts.

```solidity
address public factory;
```

This is the factory contract that created this pool. Every pool is an exchange between two ERC-20 tokens, the factory is a central point that connects all of these pools.

```solidity
address public token0;
address public token1;
```

There are the addresses of the contracts for the two types of ERC-20 tokens that can be exchanged by this pool.

```solidity
uint112 private reserve0;           // uses single storage slot, accessible via getReserves
uint112 private reserve1;           // uses single storage slot, accessible via getReserves
```

The reserves the pool has for each token type. We assume that the two represent the same amount of value, and therefore each token0 is worth reserve1/reserve0 token1’s.

```solidity
uint32  private blockTimestampLast; // uses single storage slot, accessible via getReserves
```

The timestamp for the last block in which an exchange occurred, used to track exchange rates across time.

One of the biggest gas expenses of Ethereum contracts is storage, which persists from one call of the contract to the next. Each storage cell is 256 bits long. So three variables, reserve0, reserve1, and blockTimestampLast, are allocated in such a way a single storage value can include all three of them (112+112+32=256).

```solidity
uint public price0CumulativeLast;
uint public price1CumulativeLast;
```

These variables hold the cumulative costs for each token (each in term of the other). They can be used to calculate the average exchange rate over a period of time.

```solidity
uint public kLast; // reserve0 * reserve1, as of immediately after the most recent liquidity event
```

The way the pair exchange decides on the exchange rate between token0 and token1 is to keep the multiple of the two reserves constant during trades. kLast is this value. It changes when a liquidity provider deposits or withdraws tokens, and it increases slightly because of the 0.3% market fee.

Here is a simple example. Note that for the sake of simplicity the table only has three digits after the decimal point, and we ignore the 0.3% trading fdee so the numbers are not accurate.

| Event | reserve0 | reserve1 | reserve0 * reserve1 | Average exchange rate (token1 / token0) |
| --- | --- | --- | --- | --- |
| Initial setup | 1,000.000 | 1,000.000 | 1,000,000 |  |
| Trader A swaps 50 token0 for 47.619 token1 | 1,050.000 | 952.381 | 1,000,000 | 0.952 |
| Trader B swaps 10 token0 for 8.984 token1 | 1,060.000 | 943.396 | 1,000,000 | 0.898 |
| Trader C swaps 40 token0 for 34.305 token1 | 1,100.000 | 909.090 | 1,000,000 | 0.858 |
| Trader D swaps 100 token1 for 109.01 token0 | 990.990 | 1,009.090 | 1,000,000 | 0.917 |
| Trader E swaps 10 token0 for 10.079 token1 | 1,000.990 | 999.010 | 1,000,000 | 1.008 |

As traders provide more of token0, the relative value of token1 increases, and vice versa, based on supply and demand.

---

### Lock

```solidity
uint private unlocked = 1;
```

There is a class of security vulnerabilities that are based on reentrancy abuse. Uniswap needs to transfer arbitrary ERC-20 tokens, which means calling ERC-20 contracts that may attempt to abuse the Uniswap market that calls them. By having an unlocked variable as part of the contract, we can prevent functions from being called while they are running(within the same transaction).

```solidity
modifier lock() {
```

This function is a modifier, a function that wraps around a normal function to change its behavior is some way.

```solidity
require(unlocked == 1, 'UniswapV2: LOCKED');
unlocked = 0;
```

If unlocked is equal to one, set it to zero. If it is already zero revert the call, make it fail.

```solidity
_;
```

In a modifier _; is the original function call (with all the parameters). Here it means that the function call only happens if unlocked was one when it was called, and while it is running the value of unlocked is zero.

```solidity
	unlocked = 1;
}
```

After the main function returns, release the lock.

---

### Misc. functions

```solidity
function getReserves() public view returns (uint112 _reserve0, uint112 _reserve1, uint32 _blockTimestampLast) {
    _reserve0 = reserve0;
    _reserve1 = reserve1;
    _blockTimestampLast = blockTimestampLast;
}
```

This function provides callers with the current state of the exchange. Notice that Solidity functions can return multiple values.

```solidity
function _safeTransfer(address token, address to, uint value) private {
    (bool success, bytes memory data) = token.call(abi.encodeWithSelector(SELECTOR, to, value));
```

This internal function transfers an amount of ERC20 tokens from the exchange to somebody else. SELECTOR specifies that the function we are calling is transfer(address, uint)

To avoid having to import an interface for the token function, we “manually” create the call using one of the ABI functions.

```solidity
	require(success && (data.length == 0 || abi.decode(data, (bool))), 'UniswapV2: TRANSFER_FAILED');
}
```

There are two ways in which an ERC-20 transfer call can report failure:

1. Revert. If a call to an external contract reverts, then the boolean return value is false.
2. End normally but report a failure. In that case the return value buffer has a non-zero length, and when decoded as a boolean value it is false.

If either of these conditions happen, revert.

---

### Events

```solidity
event Mint(address indexed sender, uint amount0, uint amount1);
event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);
```

These two events are emitted when a liquidity provider either deposits liquidity (Mint) or withdraws it (Burn). In either case, the amounts of token0 and token1 that are deposited or withdrawn are part of the event, as well as the identity of the account that called us (sender). In the case of a withdrawal, the event also includes the target that received the tokens (to), which may note be the same as the sender.

```solidity
event Swap(
    address indexed sender,
    uint amount0In,
    uint amount1In,
    uint amount0Out,
    uint amount1Out,
    address indexed to
);
```

This event is emitted when a trader swaps one token for the other. Again, the sender and the destination may not be the same. Each token may be either sent to the exchange, or received from it.

```solidity
event Sync(uint112 reserve0, uint112 reserve1);
```

Finally, Sync is emitted every time tokens are added or withdrawn, regardless of the reason, to provide the latest reserve information (and therefore the exchange rate).

---

### Setup Functions

These functions are supposed to be called once when the new pair exchange is et up.

```solidity
constructor() public {
    factory = msg.sender;
}
```

The constructor makes sure we’ll keep track of the address of the factory that created the pair. This information is required for initialize and for the factory fee (if one exists)

```solidity
// called once by the factory at time of deployment
function initialize(address _token0, address _token1) external {
    require(msg.sender == factory, 'UniswapV2: FORBIDDEN'); // sufficient check
    token0 = _token0;
    token1 = _token1;
}
```

This function allows the factory (and only the factory) to specify the ERC-20 tokens that this pair will exchange.

---

### Internal Update Functions

_update

```solidity
// update reserves and, on the first call per block, price accumulators
function _update(uint balance0, uint balance1, uint112 _reserve0, uint112 _reserve1) private {
```

This function is called every time tokens are deposited or withdrawn.

```solidity
	require(balance0 <= uint112(-1) && balance1 <= uint112(-1), 'UniswapV2: OVERFLOW');
```

If either balance0 or balance1 (uint256) is higher than uint112(-1)(=2^112-1) (so it overflows & wraps back to 0 when converted to uint112) refuse to continue the _update to prevent overflows. With a normal token that can be subdivided into 10^18 units, this means each exchange is limited to about 5.1*10^15 of each tokens. So far that has not been a problem.

```solidity
	uint32 blockTimestamp = uint32(block.timestamp % 2**32);
	uint32 timeElapsed = blockTimestamp - blockTimestampLast; // overflow is desired
	if (timeElapsed > 0 && _reserve0 != 0 && _reserve1 != 0) {
```

If the time elapsed is not zero, it means we are the first exchange transaction on this block. In that case, we need to update the cost accumulators.

> ****4. False Positives****
Even if we can identify that an overflow has occurred, we can understand what operation caused it and whether the 
operands are signed or unsigned integers, sometimes it still doesn’t enough. Sometimes an overflow is desirable behavior. Some compilers create an overflow intentionally to run some functionality, and sometimes even the smart contract’s developers base their coding logic on desirable overflows. Therefore, even when we detect an overflow, we can’t be sure whether it is an unexpected behavior that can be a potential vulnerability or a desirable functionality. Thus, the FP (False Positive) rate of integer overflow detection is high.
> 
> 
> [Valid Network | Integer Overflow in Ethereum](https://valid.network/post/integer-overflow-in-ethereum)
> 

```solidity
// * never overflows, and + overflow is desired
            price0CumulativeLast += uint(UQ112x112.encode(_reserve1).uqdiv(_reserve0)) * timeElapsed;
            price1CumulativeLast += uint(UQ112x112.encode(_reserve0).uqdiv(_reserve1)) * timeElapsed;
        }
```

Each cost accumulator is updated with the latest cost (reserve of the other token/reserve of this token) times the elapsed time in seconds. To get an average price you read the cumulative price is two points in time, and divide by the time difference between them. For example, assume this sequence of events:

| Event | reserve0 | reserve1 | timestamp | Marginal exchange rate (reserve1 / reserve0) | price0CumulativeLast |
| --- | --- | --- | --- | --- | --- |
| Initial setup | 1,000.000 | 1,000.000 | 5,000 | 1.000 | 0 |
| Trader A deposits 50 token0 and gets 47.619 token1 back | 1,050.000 | 952.381 | 5,020 | 0.907 | 20 |
| Trader B deposits 10 token0 and gets 8.984 token1 back | 1,060.000 | 943.396 | 5,030 | 0.890 | 20+10*0.907 = 29.07 |
| Trader C deposits 40 token0 and gets 34.305 token1 back | 1,100.000 | 909.090 | 5,100 | 0.826 | 29.07+70*0.890 = 91.37 |
| Trader D deposits 100 token1 and gets 109.01 token0 back | 990.990 | 1,009.090 | 5,110 | 1.018 | 91.37+10*0.826 = 99.63 |
| Trader E deposits 10 token0 and gets 10.079 token1 back | 1,000.990 | 999.010 | 5,150 | 0.998 | 99.63+40*1.1018 = 143.702 |

Let’s say we want to calculate the average price of Token0 between the timestamps 5,030 and 5,150. The difference in the value of price0Cumulative is 143.702-29.07=114.632. This is the average across two minutes (120 seconds). So the average price is 114.632/120 = 0.955.

This price calculation is the reason we need to know the old reserve sizes.

```solidity
		reserve0 = uint112(balance0);
    reserve1 = uint112(balance1);
    blockTimestampLast = blockTimestamp;
    emit Sync(reserve0, reserve1);
}
```

Finally, update the global variables and emit a Sync event.

---

_mintFee

```solidity
// if fee is on, mint liquidity equivalent to 1/6th of the growth in sqrt(k)
function _mintFee(uint112 _reserve0, uint112 _reserve1) private returns (bool feeOn) {
```

In Uniswap 2.0 traders pay a 0.30% fee to use the market. Most of that fee (0.25% of the trade) always goes to the liquidity providers. The remaining 0.05% can go either to the liquidity providers or to an address specified by the factory as a protocol fee, which pays Uniswap for their development effort.

To reduce calculations (and therefore gas costs), this fee is only calculated when liquidity is added or removed from the pool, rather than at each transaction.

```solidity
	address feeTo = IUniswapV2Factory(factory).feeTo();
  feeOn = feeTo != address(0);
```

Read the fee destination of the factory. If it is zero then there is no protocol fee and no need to calculate it that fee.

```solidity
	uint _kLast = kLast; // gas savings
```

The kLast state variable is located in storage, so it will have a value between different calls to the contract. Access to storage is a lot more expensive than access to the volatile memory that is released when the function call to the contract ends, so we use an internal variable to save on gas.

```solidity
	if (feeOn) {
    if (_kLast != 0) {
```

The liquidity providers get their cut simply by the appreciation of their liquidity tokens. But the protocol fee requires new liquidity tokens to be minted and provided to the feeTo address.

```solidity
	uint rootK = Math.sqrt(uint(_reserve0).mul(_reserve1));
  uint rootKLast = Math.sqrt(_kLast);
  if (rootK > rootKLast) {
```

If there is new liquidity on which to collect a protocol fee. You can see the square root function.

```solidity
		uint numerator = totalSupply.mul(rootK.sub(rootKLast));
	  uint denominator = rootK.mul(5).add(rootKLast);
	  uint liquidity = numerator / denominator;
```

This complicated calculation of fees is explained in the whitepaper on page 5. We know that between the time kLast was calculated and the present no liquidity was added or removed (because we run this calculation every time liquidity is added or removed, before it actually changes), so any change in reserve0 * reserve1 has to come from transaction fees(without them we’d keep reserve0 * reserve1 constant).

```solidity
				if (liquidity > 0) _mint(feeTo, liquidity);
      }
    }
```

Use the UniswapV2ERC20._mint function to actually create the additional liquidity tokens and assign them to feeTo.

```solidity
				} else if (_kLast != 0) {
            kLast = 0;
        }
    }
```

If there is no fee set `kLast` to zero (if it isn’t that already). When this contract was written there was a [gas refund feature](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-3298.md) that encouraged contracts to reduce the overall size of the Ethereum state by zeroing out storage they did not need. This code gets that refund when possible.

---

### Externally Accessible Functions

Note that while any tractions or contract can call these functions, they are designed to be called from the periphery contract. If you call them directly you won’t be able to cheat the pair exchange, but you might lose value through a mistake.

---

mint

```solidity
// this low-level function should be called from a contract which performs important safety checks
function mint(address to) external lock returns (uint liquidity) {
```

This function is called when a liquidity provider adds liquidity to the pool. It mints additional liquidity tokens as a reward. It should be called from a periphery contact that calls it after adding the liquidity in the same transaction (so nobody else would be able to submit a transaction that claims the new liquidity before the legitimate owner).

```solidity
		(uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
```

This is the way to read the results of a Solidity function that returns multiple values. We discard the last returned values, the block timestamp, because we don’t need it.

```solidity
		uint balance0 = IERC20(token0).balanceOf(address(this));
    uint balance1 = IERC20(token1).balanceOf(address(this));
    uint amount0 = balance0.sub(_reserve0);
    uint amount1 = balance1.sub(_reserve1);
```

Get the current balances and see how much was added of each token type.

```solidity
		bool feeOn = _mintFee(_reserve0, _reserve1);
```

Calculate the protocol fees to collect, if any, and mint liquidity tokens accordingly. Because the parameters to `_mintFee` are the old reserve values, the fee is calculated accurately based only on pool changes due to fees.

```solidity
	uint _totalSupply = totalSupply; // gas savings, must be defined here since totalSupply can update in _mintFee
  if (_totalSupply == 0) {
      liquidity = Math.sqrt(amount0.mul(amount1)).sub(MINIMUM_LIQUIDITY);
     _mint(address(0), MINIMUM_LIQUIDITY); // permanently lock the first MINIMUM_LIQUIDITY tokens
```

If this is the first deposit, create `MINIMUM_LIQUIDITY` tokens and send them to address zero to lock them. They can never be redeemed, which means the pool will never be emptied completely (this saves us from division by zero in some places). The value of `MINUMUM_LIQUIDITY` is a thousand, which considering most ERC-20 are subdivided in into units of 10^-18’th of a token, as ETH is divided into wei, is 10^-15 to the value of a single token. Not a high cost.

In the time of the first deposit we don’t know the relative value of the two tokens, so we just multiply the amounts and take a square root, assuming that the deposit provides us with equal value in both tokens.

We can trust this because it is the depositor’s interest to provide equal value, to avoid losing value to arbitrage. Let’s say that the value of the two tokens is identical, but our depository deposited four times as many of Token1 as of Token0. A trader can use the fact the pair exchange thinks that Token0 is more valuable to extract value out of it.

| Event | reserve0 | reserve1 | reserve0 * reserve1 | Value of the pool (reserve0 + reserve1) |
| --- | --- | --- | --- | --- |
| Initial setup | 8 | 32 | 256 | 40 |
| Trader deposits 8 Token0 tokens, gets back 16 Token1 | 16 | 16 | 256 | 32 |

As you can see, the trader earned an extra 8 tokens, which come from a reduction in the value of the pool, hurting the depositor that owns it.

```solidity
} else {
            liquidity = Math.min(amount0.mul(_totalSupply) / _reserve0, amount1.mul(_totalSupply) / _reserve1);
```

With every subsequent deposit we already know the exchange rate between the two assets, and we expect liquidity providers to provide equal value in both. If they don’t, we give them liquidity tokens based on the lesser value they provided as a punishment.

Whether it is the initial deposit or a subsequent one, the number of liquidity tokens we provide is equal to the square root of the change in reserve0*reserve1 and the value of the liquidity token doesn’t change (unless we get a deposit that doesn’t have equal values of both types, in which case the “fine” gets distributed). Here is another example with two tokens that have the same value, with three good deposits and one bad one (deposit of only one token type, so it doesn’t produce any liquidity tokens).

| Event | reserve0 | reserve1 | reserve0 * reserve1 | Pool value (reserve0 + reserve1) | Liquidity tokens minted for this deposit | Total liquidity tokens | value of each liquidity token |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Initial setup | 8.000 | 8.000 | 64 | 16.000 | 8 | 8 | 2.000 |
| Deposit four of each type | 12.000 | 12.000 | 144 | 24.000 | 4 | 12 | 2.000 |
| Deposit two of each type | 14.000 | 14.000 | 196 | 28.000 | 2 | 14 | 2.000 |
| Unequal value deposit | 18.000 | 14.000 | 252 | 32.000 | 0 | 14 | ~2.286 |
| After arbitrage | ~15.874 | ~15.874 | 252 | ~31.748 | 0 | 14 | ~2.267 |

```solidity
		}
    require(liquidity > 0, 'UniswapV2: INSUFFICIENT_LIQUIDITY_MINTED');
    _mint(to, liquidity);
```

Use the UniswapV2ERC20. `_mint` function to actually create the additional liquidity tokens and give them to the correct account.

```solidity
				_update(balance0, balance1, _reserve0, _reserve1);
        if (feeOn) kLast = uint(reserve0).mul(reserve1); // reserve0 and reserve1 are up-to-date
        emit Mint(msg.sender, amount0, amount1);
    }
```

Update the state variables (reserve0, reserve1, and if needed kLast) and emit the appropriate event.

---

burn

```solidity
// this low-level function should be called from a contract which performs important safety checks
function burn(address to) external lock returns (uint amount0, uint amount1) {
```

This function is called when liquidity is withdrawn and the appropriate liquidity tokens need to be burned. It should also be called from a periphery account.

```solidity
	(uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
  address _token0 = token0;                                // gas savings
  address _token1 = token1;                                // gas savings
  uint balance0 = IERC20(_token0).balanceOf(address(this));
  uint balance1 = IERC20(_token1).balanceOf(address(this));
  uint liquidity = balanceOf[address(this)];
```

The periphery contract transferred the liquidity to be burned to this contract before the call. That way we know how much liquidity to burn, and we can make sure that it gets burned.

```solidity
	bool feeOn = _mintFee(_reserve0, _reserve1);
  uint _totalSupply = totalSupply; // gas savings, must be defined here since totalSupply can update in _mintFee
  amount0 = liquidity.mul(balance0) / _totalSupply; // using balances ensures pro-rata distribution
  amount1 = liquidity.mul(balance1) / _totalSupply; // using balances ensures pro-rata distribution
  require(amount0 > 0 && amount1 > 0, 'UniswapV2: INSUFFICIENT_LIQUIDITY_BURNED');
```

The liquidity provider receives equal value of both tokens. This way we don’t change the exchange rate.

```solidity
	_burn(address(this), liquidity);
  _safeTransfer(_token0, to, amount0);
  _safeTransfer(_token1, to, amount1);
  balance0 = IERC20(_token0).balanceOf(address(this));
  balance1 = IERC20(_token1).balanceOf(address(this));

  _update(balance0, balance1, _reserve0, _reserve1);
  if (feeOn) kLast = uint(reserve0).mul(reserve1); // reserve0 and reserve1 are up-to-date
  emit Burn(msg.sender, amount0, amount1, to);
}
```

The rest of the `burn` function is the minor image of the `mint` function above.

---

swap

```solidity
// this low-level function should be called from a contract which performs important safety checks
function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external lock {
```

This function is also supposed to be called from a periphery contract.

```solidity
	require(amount0Out > 0 || amount1Out > 0, 'UniswapV2: INSUFFICIENT_OUTPUT_AMOUNT');
  (uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
  require(amount0Out < _reserve0 && amount1Out < _reserve1, 'UniswapV2: INSUFFICIENT_LIQUIDITY');

  uint balance0;
  uint balance1;
  { // scope for _token{0,1}, avoids stack too deep errors
```

Local variables can be stored either in memory or, if there aren’t too many of them, directly on the stack. If we can limit the number so we’ll use the stack we use less gas. For more details see the yellow paper, the formal Ethereum specifications, p. 26, equation 298.

```solidity
	address _token0 = token0;
  address _token1 = token1;
  require(to != _token0 && to != _token1, 'UniswapV2: INVALID_TO');
  if (amount0Out > 0) _safeTransfer(_token0, to, amount0Out); // optimistically transfer tokens
  if (amount1Out > 0) _safeTransfer(_token1, to, amount1Out); // optimistically transfer tokens
```

This transfer is optimistic, because we transfer before we are sure all the conditions are met. This is OK in Ethereum because if the conditions aren’t met later in the call we revert out of it and any changes it created.

```solidity
	if (data.length > 0) IUniswapV2Callee(to).uniswapV2Call(msg.sender, amount0Out, amount1Out, data);
```

Inform the receiver about the swap if requested.

```solidity
	balance0 = IERC20(_token0).balanceOf(address(this));
	balance1 = IERC20(_token1).balanceOf(address(this));
}
```

Get the current balances. The periphery contract sends us the tokens before calling us for the swap. This makes it easy for the contract to check that it is not being cheated, a check that has to happen in the core contract (because we can be called by other entries than our periphery contract).

```solidity
	uint amount0In = balance0 > _reserve0 - amount0Out ? balance0 - (_reserve0 - amount0Out) : 0;
  uint amount1In = balance1 > _reserve1 - amount1Out ? balance1 - (_reserve1 - amount1Out) : 0;
  require(amount0In > 0 || amount1In > 0, 'UniswapV2: INSUFFICIENT_INPUT_AMOUNT');
  { // scope for reserve{0,1}Adjusted, avoids stack too deep errors
      uint balance0Adjusted = balance0.mul(1000).sub(amount0In.mul(3));
      uint balance1Adjusted = balance1.mul(1000).sub(amount1In.mul(3));
      require(balance0Adjusted.mul(balance1Adjusted) >= uint(_reserve0).mul(_reserve1).mul(1000**2), 'UniswapV2: K');
```

This is a sanity check to make sure we don’t lose from the swap. There is no circumstance in which a swap should reduce `reserve0*reserve1`. This is also where we ensure a fee of 0.3% is being sent on the swap; before sanity checking the value of K, we multiply both balances by 1000 subtracted by the amounts multiplied by 3, this means 0.3% is being deducted from the balance before comparing its K value with the current reserves K value.

```solidity
	}

    _update(balance0, balance1, _reserve0, _reserve1);
    emit Swap(msg.sender, amount0In, amount1In, amount0Out, amount1Out, to);
}
```

Update `reserve0` and `reserve1`, and if necessary the price accumulators and the timestamp and emit an event.

---

Sync or Skim {#sync-or-skim}

It is possible for the real balances to get out of sync with the reserves that the pair exchange thinks it has. There is no way to withdraw tokens without the contract’s consent, but deposits are a different matter. An account can transfer tokens to the exchange without calling either `mint` or `swap`.

In that case there are two solutions:

- `sync`, update the reserves to the current balances
- `skim`, withdraw the extra amount. Note that any account is allowed to call `skim` because we don’t know who deposited the tokens. This information is emitted in and event, but events are not accessible from the blockchain.

```solidity
		// force balances to match reserves
    function skim(address to) external lock {
        address _token0 = token0; // gas savings
        address _token1 = token1; // gas savings
        _safeTransfer(_token0, to, IERC20(_token0).balanceOf(address(this)).sub(reserve0));
        _safeTransfer(_token1, to, IERC20(_token1).balanceOf(address(this)).sub(reserve1));
    }

    // force reserves to match balances
    function sync() external lock {
        _update(IERC20(token0).balanceOf(address(this)), IERC20(token1).balanceOf(address(this)), reserve0, reserve1);
    }
}
```

---

## UniswapV2Factory.sol

This contract creates the pair exchanges.

```solidity
pragma solidity =0.5.16;

import './interfaces/IUniswapV2Factory.sol';
import './UniswapV2Pair.sol';

contract UniswapV2Factory is IUniswapV2Factory {
    address public feeTo;
    address public feeToSetter;
```

These state variables are necessary to implement the protocol fee (see the whitepaper, p. 5). The `feeTo` address accumulates the liquidity tokens for the protocol fee, and `feeToSetter` is the address allowed to change `feeTo` to a different address.

```solidity
	mapping(address => mapping(address => address)) public getPair;
  address[] public allPairs;
```

These variables keep track of the pairs, the exchanges between two token types.

The first one, `getPair`, is a mapping that identifies a pair exchange contract based on the two ERC-20 tokens it exchanges. ERC-20 tokens are identified by the addresses of the contracts that implement them, so the keys and the value are all addresses. To get the address of the pair exchange that lets you convert from `tokenA` to `tokenB`, you use `getPair[<tokenA address>][<tokenB address>]` (or the other way around).

The second variable, `allPair`, is an array that includes all the addresses of pair exchanges created by this factory. In Ethereum you cannot iterate over the content of a mapping, or get a list of all the keys, so this variable is the only way to know which exchanges this factory manages.

Note: The reason you cannot iterate over all the keys of a mapping is that contract data storage is expensive, so the less of it we use the better, and the less often we change it the better. [You can create mappings that support iteration](https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol), but they require extra storage for a list of keys. In most applications you do not need that.

```solidity
event PairCreated(address indexed token0, address indexed token1, address pair, uint);
```

This event is emitted when a new pair exchange is created. It includes the tokens’ addresses, the pair exchange’s address, and the total number of exchanges managed by the factory.

```solidity
constructor(address _feeToSetter) public {
    feeToSetter = _feeToSetter;
}
```

The only thing the constructor does is specify the `feeToSetter`. Factories start without a fee, and only `feeSetter` can change that.

```solidity
function allPairsLength() external view returns (uint) {
    return allPairs.length;
}
```

This function returns the number of exchange pairs.

```solidity
function createPair(address tokenA, address tokenB) external returns (address pair) {
```

This is the main function of the factory, to create a pair exchange between two ERC-20 tokens. Note that anybody can call this function. You do not need permission from Uniswap to create a new pair exchange.

```solidity
	require(tokenA != tokenB, 'UniswapV2: IDENTICAL_ADDRESSES');
  (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
```

We want the address of the new exchange to be deterministic, so it can be calculated in advance off chain (this can be useful for layer 2 transactions). To do this we need to have a consistent order of the token addresses, regardless of the order in which we have received them, so we sort them here.

```solidity
	require(token0 != address(0), 'UniswapV2: ZERO_ADDRESS');
  require(getPair[token0][token1] == address(0), 'UniswapV2: PAIR_EXISTS'); // single check is sufficient
```

Large liquidity pools are better than small ones, because they have more stable prices. We don’t want to have more than a single liquidity pool per pair of tokens. If there is already an exchange, there’s no need to create another one for the same pair.

```solidity
	bytes memory bytecode = type(UniswapV2Pair).creationCode;
```

To create a new contract we need the code that creates it (both the constructor function and code that writes to memory the EVM bytecode of the actual contract). Normally in Solidity we just use `addr = new <name of contract>(<constructor parameter>)` and the compiler takes care of everything for us, but to have a deterministic contract address we need to use the CREATE2 opcode. When his code was written that opcode was not yet supported by Solidity, so it was necessary to manually get the code. This is no longer an issue, because Solidity now supports CREATE2.

```solidity
	bytes32 salt = keccak256(abi.encodePacked(token0, token1));
  assembly {
      pair := create2(0, add(bytecode, 32), mload(bytecode), salt)
  }
```

When an opcode is note supported by Solidity yet we can call it using inline assembly.

```solidity
	IUniswapV2Pair(pair).initialize(token0, token1);
```

Call the `initialize` function to tell the new exchange what two tokens it exchanges.

```solidity
		getPair[token0][token1] = pair;
    getPair[token1][token0] = pair; // populate mapping in the reverse direction
    allPairs.push(pair);
    emit PairCreated(token0, token1, pair, allPairs.length);
}
```

Save the new pair information in the state variables and emit an event to inform the world of the new pair exchange.

```solidity
	function setFeeTo(address _feeTo) external {
      require(msg.sender == feeToSetter, 'UniswapV2: FORBIDDEN');
      feeTo = _feeTo;
  }

  function setFeeToSetter(address _feeToSetter) external {
      require(msg.sender == feeToSetter, 'UniswapV2: FORBIDDEN');
      feeToSetter = _feeToSetter;
  }
}
```

These two functions allow `feeSetter` to control the fee recipient (if any), and to change `feeSetter` to a new address.

---

## UniswapV2ERC20.sol

This contract implements the ERC-20 liquidity token. It is similar to the OpenZeppelin ERC-20 contract, so I will only explain the part that is different, the `permit` functionality.

Transaction on Ethereum cost ether (ETH), which is equivalent to real money. If you have ERC-20 tokens but not ETH, you can’t send transactions, so you can’t do anything with them. One solution to avoid this problem is [meta-transactions](https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/supporting-meta-transactions/). The owner of the tokens signs a transaction that allows somebody else to withdraw tokens off chain and sends it using the Internet to the recipient. The recipient, which does have ETH, then submits the permit on behalf of the owner.

```solidity
		bytes32 public DOMAIN_SEPARATOR;
    // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
    bytes32 public constant PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;
```

This hash is the [identifier for the transaction type](https://eips.ethereum.org/EIPS/eip-712#rationale-for-typehash). The only one we support here is `Permit` with these parameters.

```solidity
	mapping(address => uint) public nonces;
```

It is not feasible for a recipient to fake a digital signature. However, it is trivial to send the same transaction twice (this is a form of replay attack). To prevent this, we use a nonce. If the nonce of a new `Permit` is not one more than the last one used, we assume it is invalid.

```solidity
	constructor() public {
      uint chainId;
      assembly {
          chainId := chainid
      }
```

This is the code to retrieve the chain identifier. It uses an EVM assembly dialect called Yul. Note that in the current version of Yul you have to use `chainid()`, not `chainid`.

```solidity
	DOMAIN_SEPARATOR = keccak256(
      abi.encode(
          keccak256('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)'),
          keccak256(bytes(name)),
          keccak256(bytes('1')),
          chainId,
          address(this)
      )
  );
}
```

Calculate the domain separator for EIP-712.

```solidity
	function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external {
```

This is the function that implements the permissions. It receives as parameters the relevant fields, and the three scalar values for the signature (v, r, and s).

```solidity
	require(deadline >= block.timestamp, 'UniswapV2: EXPIRED');
```

Don’t accept transactions after the deadline.

```solidity
	bytes32 digest = keccak256(
      abi.encodePacked(
          '\x19\x01',
          DOMAIN_SEPARATOR,
          keccak256(abi.encode(PERMIT_TYPEHASH, owner, spender, value, nonces[owner]++, deadline))
      )
  );
```

`abi.encodePacked(...)` is the message we expect to get. We know what the nonce should be, so there is no need for us to get it as a parameter.

The Ethereum signature algorithm expects to get 256 bits to sign, so we use the `keccak256` hash function.

```solidity
	address recoveredAddress = ecrecover(digest, v, r, s);
```

From the digest and the signature we can get the address that signed it using ecrecover.

```solidity
	require(recoveredAddress != address(0) && recoveredAddress == owner, 'UniswapV2: INVALID_SIGNATURE');
        _approve(owner, spender, value);
  }
```

If everything is OK, treat this as an ERC-20 approve.

---

# THE PERIPHERY CONTRACTS

The periphery contracts are the API (application program interface) for Uniswap. They are available for external calls, either from other contracts or decentralized applications. You could call the core contracts directly, but that’s more complicated and you might lose value if you make a mistake. The core contracts only contain tests to make sure they aren’t cheated, not sanity checks for anybody else. Those are in the periphery so they can be updated as needed.l

## UniswapV2Router01.sol

This contract has problems, and should no longer be used. Luckily, the periphery contracts are stateless and don’t hold any assets, so it is easy to deprecate it and suggest people use the replacement, `UniswapV2Router02`, instead.

---

## UniswapV2Router02.sol

In most cases you would use Uniswap through this contract. You can see how to use it here.

```solidity
pragma solidity =0.6.6;

import '@uniswap/v2-core/contracts/interfaces/IUniswapV2Factory.sol';
import '@uniswap/lib/contracts/libraries/TransferHelper.sol';

import './interfaces/IUniswapV2Router02.sol';
import './libraries/UniswapV2Library.sol';
import './libraries/SafeMath.sol';
import './interfaces/IERC20.sol';
import './interfaces/IWETH.sol';
```

Most of these we either encountered before, or are fairly obvious. The one exception is `IWETH.sol`. Uniswap v2 allows exchanges for any pair ERC-20 tokens, but ether (ETH) itself isn’t an ERC-20 token. It predates the standard and is transfered by unique mechanisms. To enable the use of ETH in contracts that apply to ERC-20 tokens people came up with the wrapped ether (WETH) contract. You send this contract ETH, and it mints you an equivalent amount of WETH. Or you can burn WETH, and get ETH back.

```solidity
contract UniswapV2Router02 is IUniswapV2Router02 {
    using SafeMath for uint;

    address public immutable override factory;
    address public immutable override WETH;
```

The router needs to know what factory to use, and for transactions that