# Voting DApp Project

Cloud Fund 예제를 바탕으로 Voting DApp 개발

### Cloud Fund 예제

```solidity
// SPDX-License-Identifier: MIT
// https://solidity-by-example.org/app/crowd-fund
pragma solidity ^0.8.13;

//Interface 다른 컨트랙트와 interact 위해 필요 순수한 틀
//모든 함수가 추상 함수
//다른 컨트랙트나 인터페이스 사용 불가
//IERC20은 오픈제플린(표준프레임워크)에서 제공하는 라이브러리
interface IERC20 {		
		// 이더 전송(주소와 금액)
    function transfer(address, uint) external returns (bool);
		// 이더 전송: 토큰을 누군가에게 보내는 기능. approve 함수로 내 토큰 사용할 수 있는 사람 지정
		// function approve(address spender, uint256 _value) public returns(bool)
    function transferFrom(
        address,
        address,
        uint
    ) external returns (bool);
		// 추가
		function approve(address _spender, uint256 _value) external returns (bool);
}

contract CrowdFund {
		// 트랜잭션 로그 생성, 특정의 이벤트가 일어나는지 감시하고 사용자 인터페이스에 반영하거나 
    // 컨트랙상 이벤트에 대응하는 변화를 어플리케이션 상태에도 반영. 프론트엔드에서 listen을 위한 코드
    event Launch(
        uint id,
        address indexed creator,
        uint goal,
        uint32 startAt,
        uint32 endAt
    );
    event Cancel(uint id);
    event Pledge(uint indexed id, address indexed caller, uint amount);
    event Unpledge(uint indexed id, address indexed caller, uint amount);
		// 받기: 목표 달성 성공
    event Claim(uint id);
		// 환불: 목표 달성 실패
    event Refund(uint id, address indexed caller, uint amount);
		
		// 펀딩을 받는 캠페인
    struct Campaign {
        // 캠페인 등록자: Creator of campaign
        address creator;
        // 목표금액: Amount of tokens to raise
        uint goal;
        // 총 펀딩 금액: Total amount pledged
        uint pledged;
        // 시작일: Timestamp of start of campaign
        uint32 startAt;
        // 종료일: Timestamp of end of campaign
        uint32 endAt;
        // 금액 달성 시, 등록자가 토큰 가짐: True if goal was reached and creator has claimed the tokens.
        bool claimed;
    }

    IERC20 public immutable token;
    // Total count of campaigns created.
    // It is also used to generate id for new campaigns.
    uint public count;
    // Mapping from id to Campaign
    mapping(uint => Campaign) public campaigns;
    // Mapping from campaign id => pledger => amount pledged
    mapping(uint => mapping(address => uint)) public pledgedAmount;

    constructor(address _token) {
				// ERC20 표준 토큰
        token = IERC20(_token);
    }

		// 캠페인 등록
    function launch(
        uint _goal,
        uint32 _startAt,
        uint32 _endAt
    ) external {
				// 입력 값 오류 확인
        require(_startAt >= block.timestamp, "start at < now");
        require(_endAt >= _startAt, "end at < start at");
				// 최대 기간 90일
        require(_endAt <= block.timestamp + 90 days, "end at > max duration");
				// 캠페인 생성에 따른 count를 id로 하여 하나 증가
        count += 1;
        campaigns[count] = Campaign({
            creator: msg.sender, // 컨트랙 호출자가 creator
            goal: _goal,
            pledged: 0,
            startAt: _startAt,
            endAt: _endAt,
            claimed: false
        });

        emit Launch(count, msg.sender, _goal, _startAt, _endAt);
    }
		// 캠페인 취소
    function cancel(uint _id) external {
        Campaign memory campaign = campaigns[_id];
				// creator 여부
        require(campaign.creator == msg.sender, "not creator");
				// 시작 전 여부
        require(block.timestamp < campaign.startAt, "started");
				// 삭제
        delete campaigns[_id];
        emit Cancel(_id);
    }
		// 모금
    function pledge(uint _id, uint _amount) external {
        Campaign storage campaign = campaigns[_id];
        require(block.timestamp >= campaign.startAt, "not started");
        require(block.timestamp <= campaign.endAt, "ended");
				// 모금액 증가
        campaign.pledged += _amount;
				// 모금자 기록
        pledgedAmount[_id][msg.sender] += _amount;
				// 토큰 전송(approve는 어디에서?)
        token.transferFrom(msg.sender, address(this), _amount);

        emit Pledge(_id, msg.sender, _amount);
    }
		// 모금 취소
    function unpledge(uint _id, uint _amount) external {
        Campaign storage campaign = campaigns[_id];
				// 취소 전
        require(block.timestamp <= campaign.endAt, "ended");
				// TODO: 모금 금액 여부 확인 먼저 체크해야지
				// 모금액 감소
        campaign.pledged -= _amount;
				// 모금자 기록
        pledgedAmount[_id][msg.sender] -= _amount;
				// 토큰 전송
        token.transfer(msg.sender, _amount);

        emit Unpledge(_id, msg.sender, _amount);
    }
		// 캠페인 성공 시, 등록자가 가짐
    function claim(uint _id) external {
        Campaign storage campaign = campaigns[_id];
				// 등록자 여부 확인
        require(campaign.creator == msg.sender, "not creator");
				// 캠페인 종료일 확인
        require(block.timestamp > campaign.endAt, "not ended");
				// 목표 금액 달성 여부 확인
        require(campaign.pledged >= campaign.goal, "pledged < goal");
				// 캠페인 이미 claimed 인지 확인
        require(!campaign.claimed, "claimed");
				// claim 상태 변경
        campaign.claimed = true;
				// token 전송
        token.transfer(campaign.creator, campaign.pledged);
				// 로그 기록
        emit Claim(_id);
    }
		// 펀딩한 사람이 호출하여 환불 받기
    function refund(uint _id) external {
        // 캠페인 정보 불러오기
				Campaign memory campaign = campaigns[_id];
				// 캠페인 종료일 확인
        require(block.timestamp > campaign.endAt, "not ended");
				// 목표 금액 달성 실패 확인
        require(campaign.pledged < campaign.goal, "pledged >= goal");
				// refund 호출 한 캠페인 참여자 참여 금액
        uint bal = pledgedAmount[_id][msg.sender];
				// 금액 0으로 바꿈
        pledgedAmount[_id][msg.sender] = 0;
				// 환불 토큰 전송
        token.transfer(msg.sender, bal);
				// 로그 기록
        emit Refund(_id, msg.sender, bal);
    }
		// 추가
		function approve(address _spender, uint256 _value) public returns (bool) {
		    allowed[msg.sender][_spender] = _value;
		    Approval(msg.sender, _spender, _value);
		    return true;
		}
}
```

확장성은 더 크고 복잡한 DApp을 구축할 때 중요. Solidity는 abstract contract와 interface라는 두 가지 방법을 제공!

테스트를 위한 js코드 작성 필요