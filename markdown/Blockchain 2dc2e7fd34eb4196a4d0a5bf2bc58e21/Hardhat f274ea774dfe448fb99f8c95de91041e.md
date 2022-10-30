# Hardhat

npx hardhat 에서 prompt가 안나오면 

npx hardhat —verbose 를 실행 config file 위치를 찾고 삭제 후 재실행

Hardhat unit test

npx hardhat test .\test\SimpleStorageUpgrad_test.js

hardhat 설정

hardhat-config.js

오픈제펠린 업그레이드 플러그인 추가

require(’@openzeppelin/hardhat-upgrades’);

배포 스크립트 작성

./scripts/SimpleStorageUpgrade.deploy.js

upgrades.deployProxy를 사용하여 배포

가상 노드 수행

npx hardhat node

포트에러시

****netstat -ano | findstr :포트번호****

혹은

```
net stop winnat
netsh int ipv4 set dynamic tcp start=49152 num=16384
netsh int ipv6 set dynamic tcp start=49152 num=16384
net start winnat
```

npx hardhat run —network [localhost](http://localhost) ./scripts/SimpleStorageUpgrade.deploy.js

npx hardhat run --network localhost ./scripts/SimpleStorageUpgrade.deploy.js
Compiled 3 Solidity files successfully
SimpleStorageUpgrade deployed to: 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0

업그레이드 proxy 주소

yarn add @nomiclabs/hardhat-etherscan --dev