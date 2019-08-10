# Tutorial - Writing Your First Applications

> 이 자료는 [HyperLedger Tutorial](https://hyperledger-fabric.readthedocs.io/en/release-1.4/tutorials.html) 를 정리한 자료입니다.



HyperLedger Tutorial에서는 **FabCar** 를 이용하여 app과 smart contracts를 구성하고 HyperLedger Fabric를 이해하기 위한  Starting point를 제공한다. 이 과정을 통해 ledger에 데이터를 호출하고, 업데이트하는 과정을 배우고, CA를 사용하는 방법을 학습한다.

아래 3 과정을 진행한다.

 1. 개발환경 세팅

    ![image-20190809161204441](https://hyperledger-fabric.readthedocs.io/en/release-1.4/_images/AppConceptsOverview.png)

2. Smart contract, FabCar에 대한 학습

3. FabCar를 이용한 Sample App 개발





### 시작하기전에

#### **Samples 설치**

1. fabric-smaples을 받을 디렉토리 생성 이동

   ```bash
   mkdir fabric-tutorial
   cd fabric-tutorial
   ```

2. fabric-smaples clone 및 check out

   + 사용할 버전에 알맞는 branch로 check out 한다. 
   + 필자는 release 1.4를 이용할 것임으로 별도의 branch 이동이 필요없다.

   ```bash
   git clone https://github.com/hyperledger/fabric-samples.git
   ```

3. Binary 파일을 설치한다.

   ```bash
   curl -sSL http://bit.ly/2ysbOFE | bash -s
   ```

   - curl에 의해 설치된 파일이 `|` 파이프라인 연산자를 통해, bash의 입력 인자로 들어가서, 설치 후 스크립트 파일이 바로 실행된다.



#### 도커 컨테이너 정리

```bash
docker rm -f $(docker ps -aq)
docker rmi -f $(docker images | grep fabcar | awk '{print $3}')
```



### Launch the network

> 이번 section은 fabric-samples 안에 있는 fabcar 디렉토리를 사용한다.

네트워크를 실행하기 위해서 `startFabric.sh` 를 사용하다. 이 스크립트 파일은 블록체인 네트워크에 comporising peers, orderers, ca 등등을 등록한다.  또한 ledger 접근을 위한 smart contract 버전의 javascript를 설치하고 인스턴스화 시킨다.

```bash
cd fabcar
./startFabric.sh javascript
```



### Install the application

App 실행을 위한 javascript 의존성들을 설치한다.

```bash
cd javascript
npm install
```



### Enrolling the admin user

네트워크를 만들 때, admin user는 CA  담당자로 생성되어진다. 첫번째 단계는 `enroll.js` 를 이용하여 admin을 위한 공개키, 개인키, X.509 인증서를 만드는 것이다.이 과정에서 [Cerificate Signing Request(CSR)](https://en.wikipedia.org/wiki/Certificate_signing_request)을 사용하다.

공개키와 개인키는 local에서 생성되고, 이후 공개키는 App 사용을 위한 인증서를 발급하는 CA에 보낸다. 인증서들은 wallet에 보관된다.



`admin` 을 등록시켜보자

```bash
node enrollAdmin.js
```

위 명령은 CA 관리자 인증서를 wallet 디렉토리에 저장한다.



ledger를 사용할 `user1` 을 등록시켜보자

```bash
node registerUser.js
```

`registerUser.js` 는 CSR을 이용하여 `user` 을 등록시키고, 마찬가지로 wallet에 인증서를 보관한다.



### **Querying the ledger**

블록체인 네트워크에  각 피어들은 ledger 를 복사한다. App user는 smart contract를 호출함으로서 ledger에 접근 할 수 있다.

![image-20190810164634210](https://hyperledger-fabric.readthedocs.io/en/release-1.4/_images/write_first_app.diagram.1.png)

App은 쿼리를 사용함으로서 ledger의 데이터를 읽을 수 있다. 대부분의 쿼리는 world state를 포함하고 있다. world state는 key-value set으로 App은 single key 또는 multiple keys를 요청할 수 있다. 게다가 world state는 CouchDB와 같은 데이터베이스를 사용하여 구성되어 있어, '특정한 주인을 가지는 모든 차'와 같은 복잡한 쿼리를 요청 할 수 있다.

먼저 ledger에 모든 차를 조회하기 위해  `qurey.js` 를 실행시켜보자. (이 코드는 앞에서 등록했던 user1의 권한으로 ledger에 접근한다.)

```bash
node query.js
```



이제  프로그램 내부를 살펴보자. App은 `fabric-network ` 모듈로부터 `FileSystemWallet` 과 `Gateway` 를 가져오는 것으로 부터 시작된다. 이 클래스들은 `user1` 의 identity를 wallet으로 이동시키고, 네트워크에 연결시키는데 사용된다.

```javascript
const { FileSystemWallet, Gateway } = require('fabric-network')
```

```javascript
const gateway = new Gateway();
        await gateway.connect(ccpPath, { wallet, identity: 'user1', discovery: { enabled: true, asLocalhost: true } });
```

위 코드는 새로운 게이트웨이를 만들고, 이를 이용해 네트워크에 연결한다.  ccpPath는 게이트웨이가 접근 할 네트워크를 나타낸다.

```javascript
const ccpPath = path.resolve(__dirname, '..', '..', 'first-network', 'connection-org1.json');
```

ccpPath는 ../../first-network/connection-org1.json으로 잡혀있다.



네트워크는 다수의 채널로 나눠질 수 있다.

```javascript
const network = await gateway.getNetwork('mychannel');
```

위 코드는 게이트웨이로부터 파라미터로 넘겨준 'mychannel'를 가져와서 network 변수에 할당한다. 가져온 채널에서 ledger에 접근하기 위한 smart contract를 가져와보자

```java
const contract = network.getContract('fabcar');
const result = await contract.evaluateTransaction('queryAllCars');
```

가져온 smart contract안에는 많은 transactions들이 존재하고, 우리는 위에서 ledger의 world state data 접근을 위해 `queryAllCars` transaction을 사용했다.

`evaluateTransaction` 메소도는 smart contract의 상호작용을 위한 가장 간단한 메소드이다. connection profile에 정의 되어 있는 peer를 선택하고 파라미터로 넘겨진 트랜잭션을 그곳에 보낸다.



만약 CAR4 자동차를 찾고 싶으면 reuslt부분을 아래와 같이 고쳐주자

```javascript
const result = await contract.evaluateTransaction('queryCar', 'CAR4');
```

`queryCar`  트랜잭션을 이용해 CAR4에 해당하는 정보를 ledger에서 가져온다.



### The FabCar smart contract

fabric-samples 내부에 /chaincode/fabcar/javascript/lib 로 이동해서 `fabcar,js` 를 열어보자

```bash
cd /chaincode/fabcar/javascript/lib
vim fabcar.js
```

`FabCar`  smart contract가 어떻게 정의되어 살펴보면, 클래스 내부에 `initLedger`, `queryCar`, `queryAllCars`, `createCar`, and `changeCarOwner` 가 정의되어 있다. 그중 `queryAllCars` 가 어떻게 ledger와 통신하는지 살펴보자

```javascript
async queryAllCars(ctx) {
        const startKey = 'CAR0';
        const endKey = 'CAR999';

        const iterator = await ctx.stub.getStateByRange(startKey, endKey);
```

위 코드는 ledger에서  `queryAllCars`  가 가져올 수 있는 범위를 정의한다.



![image-20190810174918158](https://hyperledger-fabric.readthedocs.io/en/release-1.4/_images/RunningtheSample.png)

각 트랜잭션들은 `getStateByRange` 와 같은 API를 통해 ledger와 통신한다. 더 많은 API를 읽고 싶으면 [여기](https://fabric-shim.github.io/master/index.html?redirect=true) 를 참고하자.



### Updating the ledger

ledger에 새로운 차를 추가해보자

App 관점에서 ledger update는 매우 간단하다. App은 블록체인 네트워크에 트랜잭션을 보낸다. 이후 네트워크에서 vaildated 와 committed가 수행 된후 트랜잭션이 성공적으로 수행 됬다는 알림을 받는다. 내부적으로는 블록체인의 서로 다른 컴포넌트들의 의한 합의과정을 거친다.

![_images / write_first_app.diagram.2.png](https://hyperledger-fabric.readthedocs.io/en/release-1.4/_images/write_first_app.diagram.2.png)

새로운 차를 추가하기 위해 `invoke.js` 를 열어보자.

```javascript
await contract.submitTransaction('createCar', 'CAR12', 'Honda', 'Accord', 'Black', 'Tom');
```

위 코드는 createCar 트랜잭션을 네트워크에 제출한다.  `submitTransaction` 은 `evaluateTransaction` 보다 정교하다. 하나의 피어와 통신하기보다는 SDK는 `submitTransaction` 를 조직의 모든 피어들에게 보낸다. 각 피어들은 이 proposal를 사용하여 요청된 smart contract를 수행하고 트랜잭션 응답을 생성하여 SDK에게 보낸다. SDK는 응답을 수집하여 ordere로 보낸다 ordere는 연속된 트랜잭션들을 수집하고, 그것을 각 validated 와 committed를 수행할 피어들에게 분산 시킨다. 최종적으로 SDK는 알림을 받고 App은 다시 제어권을 찾는다.



