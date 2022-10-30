# Solidity 정리

## ABI(Application Binary Interface)

[Ethereum:: ABI와 관련된 Q&A 정리](https://medium.com/pocs/ethereum-abi%EC%99%80-%EA%B4%80%EB%A0%A8%EB%90%9C-q-a-%EC%A0%95%EB%A6%AC-40e639ee1a03)

ABI: 컨트랙트의 함수와 매개변수들을 JSON형식으로 나타낸 리스트. 스마트 컨트랙트 함수를 이ㅛㅇㅇ하기 위해서는 ABI를 사용하여 함수를 해시할 수 있어야 함. 이렇게 하면 해당 함수를 호출하기 위해 필요한 EVM바이트 코드 생성 가능. (Think of ABI as an API at a low level.)

The API consists of a set of routines, objects, etc. that you can use in your code to access the functionality of that external component. An ABI is very similar. Think of it as the compiled version of an API (or as an API on the low level). as you know The contract are stored as bytecode in a binary form into the blockchain under a specific address. so you can accesses the binary data in the contract through the ABI which indicates to the caller the needed information (functions signatures and variables declarations) to encode a meaningful(understood by the VM) call to the bytecode(contract). The ABI defines the structures and methods that you will use to interact with binary contract (just like the API did), only on a lower level.

ABI는 어떻게 바이트 코드로 저장되는가?

Method ID를 가지고 컨트랙트의 바이트코드 안에 주소와 정확하게 연결시켜주는 작업을 컴파일러가 진행. 컴파일러는 정확하게 동작하는 EVM 바이트코드를 생성함.

Method ID는 4byte이기 떄문에, 해시 충돌(2개의 서로 다른 함수에서) 가능. 솔리디티 컴파일러는 해시 충돌의 경우 에러를 반환. 어떠한 바이트코드도 더 이상 생성하지 않음.

```solidity
method_id = first 4 bytes of msg.data
if method_id == 0x25d8dcf2 jump to 0x11
if method_id == 0xaabbccdd jump to 0x22
if method_id == 0xffaaccee jump to 0x33
other code
0x11:
code for function with method id 0x25d8dcf2
0x22:
code for another function
0x33:
code for another function
```

두 프로그램 모듈의 인터페이스 역할을 함. 데이터를 머신코드로 인코딩/디코딩 하기 위함. 이더리움에서 기본적으로 EVM에 솔리디티 컨트랙트 호출을 할 때 인코딩을 하거나 트랜잭션을 읽는 방법.

컨트랙트 내의 함수를 호출하거나 컨트랙트로부터 데이터를 얻는 방법. 컨트랙트 내에 여러 개의 함수가 있을 것. ABI는 컨트랙트 내의 어떤 함수를 호출할지를 지정하는데 필요. 우리가 생가했던 데로 함수가 데이터를 리턴한다는 것을 보장하기 위해 필요!

```solidity
contract Foo {
  function bar(real[2] xy) {}
  function baz(uint32 x, bool y) returns (bool r) { r = x > 32 || y; }
  function sam(bytes name, bool z, uint[] data) {}
}
```

baz 함수를 69와 true를 매개변수로 호출. 총 68바이트의 데이터를 사용.

메서드 식별자 4바이트 + 32바이트로 감싸진 69 + 32바이트로 감싸진 true

```solidity
0xcdcd77c0: the Method ID. This is derived as the first 4 bytes of the Keccak-256 hash of the ASCII form of the signature baz(uint32,bool). 0x0000000000000000000000000000000000000000000000000000000000000045: the first parameter, a uint32 value 69 padded to 32 bytes 0x0000000000000000000000000000000000000000000000000000000000000001: the second parameter - boolean true, padded to 32 bytes
```

위 68바이트 데이터는 트랜잭션의 data필드에 기술 됨. 관련 보안 이슈로 데이터 필드에 값을 넣을 때 조심. 컨트랙트 호출 시 데이터를 함께 보내는 과정에서 부작용이 발생할 수 있음.

Method식별자를 추출할 때, 일반적으로 알려진 위험을 예방하려면 canonical types가 반드시 사용. uint대신 uint256을 사용하는 것과 같이!

솔리디티가 메서드 식별자를 계산할 때의 과정을 위 코드의 sam메서드를 예시로 들면.

```solidity
bytes4(sha3("sam(bytes, bool, uint256[])")
```

web3.js와 같은 고수준 라이브러리 사용 시, 디테일을 놓칠 수 있음. JSON형식의 ABI 제공 필요.

ABI는 코어 이더리움 프로토콜에는 포함되지 않음. 누구나 자신의 컨트랙트에 대응되는 ABI를 만들 수 있음.

Function Selector?

함수 호출시 발생하는 4바이트의 콜 데이터는 호출할 함수를 결정. 4바이트의 데이터는 Keccak256 함수로 서명정보를 해시하여 빅 엔디안 방식으로 기술 시, 첫 번째 네 바이트를 일컬음.

서명 정보는 기본적인 프로토타입에 대한 표현법으로 정의. 예로, 함수 이름과 매개변수들의 자료형이 있음. 매개변수의 자료형은 콤마로 구분. 공백구분자는 인정 안함. 리턴 타입의 경우 함수 서명정보에 포함 안됨.

## Abstract Contract

Solidity의 contract는 객체지향의 class와 유사. 영구 데이터를 포함하는 상태 변수와 상태 변수내 데이터를 변경할 수 있는 함수가 있음. Contract는 적어도 하나의 function이 implementation이 없으면 abstract contract가 된다. 결과적으로 컴파일 불가. 하지만 다른 contract에서 상속받아 사용 가능.

```solidity
pragma solidity ^0.4.24;

contract Person {
    function gender() public returns (bytes32);
}

contract Employee is Person {
    function gender() public returns (bytes32) { return "female"; }
}
```

향상된 확장성과 함께 abstract contract는 더 나은 self-documentation을 제공하고 패턴(template)을 주입. 코드 중복을 제거.

## Constuctor

컨트랙트가 처음 생성 시, 혹은 인스턴스화 될 때, 변수의 값을 원하는 값으로 초기화하기 위해 사용. 컨트랙트 생성 시 단 한 번 동작. 생성자를 payable하게 작성하면 컨트랙트 생성 시 컨트랙트 계정에 이더를 보낼 수 있음.

## Event

Emit으로 TX로그에 이벤트 데이터 넣기

```solidity
function withdraw(uint withdraw_amount) public {
   msg.sender.transfer(withdraw_amount);
   emit withdrawal(msg.sender, withdraw_amount);
}
```

트랜잭션 로그 확인

```solidity
truffle(develop)
> 
Faucet.deployed().then(i => {FaucetDeployed = i})
FaucetDeployed.send(web3.toWei(1, "ether")).then(res=>
    {console.log(res.logs[0].event, res.logs[0].args})
```

프론트 앱에서 특정 액션에 대한 이벤트를 listening함. 이벤트 실행 시 앱에 함수가 실행 됨을 알림.

## Interface

abstract contract와 유사하지만 contract ABI가 나타낼 수 있는 것으로 제한. 즉 ABI를 인터페이스로 변환하거나 그 반대로 변환할 수 있으며 정보가 손실되지 않음.

서로 다른 컨트랙트 간의 상호작용을 위함.

어떤 외부 컨트랙트의 인터페이스를 코드에 포함하면, 해당 컨트랙트에 정의된 함수의 특성, 호출, 응당 내용을 알 수 있음. 외부 컨트랙트의 주소만 가져오지 않고 해당 컨트랙트의 상호작용하고자 하는 내용이 포함되어 있는 인터페이스를 기재해줘야 함. 외부 컨트랙트 주소에 접근하고 인터페이스를 씌워 솔리티티 코드에서 이용 가능한 형태로 사용.

1. 함수의 기능을 정의하지 않음.
2. 다른 인터페이스에서 상속 가능.
3. 함수는 무조건 external 타입이어야 한다.
4. 생성자를 선언할 수 없다.
5. 상태 변수를 선언할 수 없다.

```solidity
pragma solidity ^0.4.24;
//이건 external이 아닌데??
interface token {
	function totalSupply() public view returns (uint256);
	function balanceOf(address who) public view returns (uint256);
	function transfer(address to, uint256 value) public returns (bool);
}
```

```solidity
pragma solidity ^0.5.0;

interface ICalculator {
  function getResult() external view returns (uint);
}

contract Test is ICalculator {
  constructor() public {}
  function getResult() external view returns (uint) {
 ₩   uint a = 1;
    uint b = 2;
    uint result = a + b;
    return result;
  }
}
```

## Payable

가상화폐에 접근하기 위한 키워드. Solidity언어에서 payable키워드는 이더리움 위에서 이더를 전송하는 스마트 컨트랙트 작성시 반드시 필요.

- address payable
    - address타입의 확장. 이더를 전송할 수 있는 send(), transfer()함수를 내장.
    - address 에 사용: ethereum storage/memory 사용 시.
    - address(): address 타입을 address payable로 형변환하는 경우.
    - msg.sender에 사용: smart contract 함수를 실행한 사용자의 주소(address payable)를 사용 시.
- function payable
    - Smart Contract 계정에 초기 ether값 송금하고 싶으면 constructor function에 payable을 사용.
    - msg.value(smart contract를 실행한 사람이 전송하는 ether)를 사용할 때.
    - Smart Contract 내에 직접 정의한 함수들을 호출하지 않고, ether만 전송할 때, fallback function에 사용.
    - constuctor function, 일반 function, fallback function 옆에 사용