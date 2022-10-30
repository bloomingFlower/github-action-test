# CryptoZombie

Chapter 6. 배열

구조체 배열 생성. getter 메소드 자동 생성

Chapter 10. 함수 제어

view: only read

pure: no read or manipulate

Chanpter 11. Keccak256

입력 스트링을 random 256 bit 16진수

Chapter 2. Mapping and Address

mapping(address => uint) public zombieToOwner;

address가 key, uint가 value!

Chapter 3. Msg.sender

Solidity 함수 실행은 항상 외부 호출자가 시작. msg.sender

Chapter 7. Storage vs Memory

Storage: 온체인 상에 영구적으로 저장

Memory: 임시적으로 저장되는 변수, 컨트렉트 함수 벗어나면 삭제

Chapter 9. 함수 접근 제어자

internal: private + 상속하는 contract도 접근 가능

external: public - contract바깥에선 호출 가능 contract내 다른 함수 호출 불가

—> 정확하게 모르겠음..

Chapter 10. Interface

소유하지 않는 다른 컨트랙트와 상호작용 하려면 인터페이스 정의가 필요.

인터페이스는 컨트랙트 정의와 유사하나 함수 선언만 할 뿐. 몸체 정의를 하지 않음.

Chapter 3. onlyOwner 함수 제어자

modifier 를 함수 정의부 끝에 넣어 함수 동작 방식을 변경.

함수 실행 전 제어자 먼저 실행

함수를 누가 호출할 수 있는지 잘 구분~!

`Ownable` 컨트랙트는 기본적으로 다음과 같은 것들을 하네:

1. 컨트랙트가 생성되면 컨트랙트의 생성자가 `owner`에 `msg.sender`(컨트랙트를 배포한 사람)를 대입한다.
2. 특정한 함수들에 대해서 오직 `소유자`만 접근할 수 있도록 제한 가능한 `onlyOwner` 제어자를 추가한다.
3. 새로운 `소유자`에게 해당 컨트랙트의 소유권을 옮길 수 있도록 한다.

function likeABoss() external onlyOwner {
LaughManiacally("Muahahahaha");
}

*이더리움에서 돌아가는 DApp이라고 해서 그것만으로 분산화되어 있다고 할 수는 없네. 반드시 전체 소스 코드를 읽어보고, 자네가 잠재적으로 걱정할 만한, 소유자에 의한 특별한 제어가 불가능한 상태인지 확인하게. 개발자로서는 자네가 잠재적인 버그를 수정하고 DApp을 안정적으로 유지하도록 하는 것과, 사용자들이 그들의 데이터를 믿고 저장할 수 있는 소유자가 없는 플랫폼을 만드는 것 사이에서 균형을 잘 잡는 것이 중요.*

*이더리움에서 돌아가는 DApp이라고 해서 그것만으로 분산화되어 있다고 할 수는 없네. 반드시 전체 소스 코드를 읽어보고, 자네가 잠재적으로 걱정할 만한, 소유자에 의한 특별한 제어가 불가능한 상태인지 확인하게. 개발자로서는 자네가 잠재적인 버그를 수정하고 DApp을 안정적으로 유지하도록 하는 것과, 사용자들이 그들의 데이터를 믿고 저장할 수 있는 소유자가 없는 플랫폼을 만드는 것 사이에서 균형을 잘 잡는 것이 중요하네.*

모든 `public`과 `external`함수를 검사하고, 사용자들이 그 함수들을 남용할 수 있는 방법을 생각

****View 함수는 가스를 소모하지 않네****

지금으로서는 DApp의 가스 사용을 최적화하는 비결은 가능한 모든 곳에 읽기 전용의

`external view` 함수를 쓰는 것이라는 것만 명심해두게.

> 참고: 만약 view 함수가 동일 컨트랙트 내에 있는, view 함수가 아닌 다른 함수에서 내부적으로 호출될 경우, 여전히 가스를 소모할 것이네. 이것은 다른 함수가 이더리움에 트랜잭션을 생성하고, 이는 모든 개별 노드에서 검증되어야 하기 때문이네. 그러니 view 함수는 외부에서 호출됐을 때에만 무료라네.
> 

비용을 최소화하기 위해서, 진짜 필요한 경우가 아니면 storage에 데이터를 쓰지 않는 것이 좋네. 이를 위해 때때로는 겉보기에 비효율적으로 보이는 프로그래밍 구성을 할 필요가 있네 - 어떤 배열에서 내용을 빠르게 찾기 위해, 단순히 변수에 저장하는 것 대신 함수가 호출될 때마다 배열을 `memory`
에 다시 만드는 것처럼 말이지.

*메모리 배열은 현재로서는 storage 배열처럼 `array.push()`로 크기가 조절되지는 않네.*

극단적으로 가스 소모가 많을 것이네. 왜냐하면 위치를 바꾼 모든 좀비에 대해 쓰기 연산을 해야 하기 때문이지. 소유자가 20마리의 좀비를 가지고 있고 첫 번째 좀비를 거래한다면, 배열의 순서를 유지하기 위해 우린 19번의 쓰기를 해야 할 것이네.

난수 생성 or Oracle 사용

[https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract)

keccak256() return 은 uint형이 아님

함수에 modifier 연속으로 가능