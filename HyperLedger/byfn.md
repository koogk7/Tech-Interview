# BYFN



### `byfn.sh`  스크립트 명령어 

```bash
Usage:
byfn.sh <mode> [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-l <language>] [-o <consensus-type>] [-i <imagetag>] [-v]"
  <mode> - one of 'up', 'down', 'restart', 'generate' or 'upgrade'"
    - 'up' - bring up the network with docker-compose up"
    - 'down' - clear the network with docker-compose down"
    - 'restart' - restart the network"
    - 'generate' - generate required certificates and genesis block"
    - 'upgrade'  - upgrade the network from version 1.3.x to 1.4.0"
  -c <channel name> - channel name to use (defaults to \"mychannel\")"
  -t <timeout> - CLI timeout duration in seconds (defaults to 10)"
  -d <delay> - delay duration in seconds (defaults to 3)"
  -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)"
  -s <dbtype> - the database backend to use: goleveldb (default) or couchdb"
  -l <language> - the chaincode language: golang (default), node, or java"
  -o <consensus-type> - the consensus-type of the ordering service: solo (default), kafka, or etcdraft"
  -i <imagetag> - the tag to be used to launch the network (defaults to \"latest\")"
  -v - verbose mode"
byfn.sh -h (print this message)"

Typically, one would first generate the required certificates and
genesis block, then bring up the network. e.g.:"

  byfn.sh generate -c mychannel"
  byfn.sh up -c mychannel -s couchdb"
  byfn.sh up -c mychannel -s couchdb -i 1.4.0"
  byfn.sh up -l node"
  byfn.sh down -c mychannel"
  byfn.sh upgrade -c mychannel"

Taking all defaults:"
      byfn.sh generate"
      byfn.sh up"
      byfn.sh down"
```



`docker-compose-cli.yaml`  에서  `FABRIC_LOGGING_SPEC` 를 `DEBUG`  로 수정

```bash
cli:
  container_name: cli
  image: hyperledger/fabric-tools:$IMAGE_TAG
  tty: true
  stdin_open: true
  environment:
    - GOPATH=/opt/gopath
    - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
    - FABRIC_LOGGING_SPEC=DEBUG
    #- FABRIC_LOGGING_SPEC=INFO

```



### Crpyto Generator

이 튜토리얼에서는 `cryptogen` tool을 이용하여 인증서를 생성한다. `cryptogen` 은  `crypto-config.yaml` 파일을 기준으로 동작한다. `crypto-config.yaml` 에는 오더링 노드, 각 조직에 대한 설정 내용들이 포함되어 있다.  각 조직들은 `ca-cert` 라는 유니크한 루트 인증서를 가진다. 이렇게 함으로서 CA를 사용하는 것과 같은 흉내를 낼 수 있다. 트랜잭션과 커뮤니케이션은 각 엔티티의 개인키(`keystore`)에 서명하고 공개키(`signcerts`) 에 의해 검증되어진다.

 `cryptogen` tool 이 실행되고 난 이후에, 생성된 인증서와 키는 `crypto-config` 라는 디렉토리에 저장되어진다. 



### Configuration Transaction Generator

`configtxgen` tool 은 아래의 artifacts를 만드는데 사용된다.

- orderer genesis block
- channel configuration transaction
  - Channel 생성시 orderer에게 broadcast 된다.
- 조직별  `anchor peer transactions` 
  - 채널에 각 조직의 Anchor Peer를 지정한다.

`configtxgen` 은 `configtx.yaml`  파일을 기준으로 동작한다.  `configtx.yaml` 는 sample network에 대한 정의를 포함하고 , 두개의 조직으로 이루어진 컨소시엄을 지정하고 있다. 또한 각 조직의  보증정책과 msp dir등을 설정할 수 있다.

이 파일에서 주목할만한 두가지가 있다. 첫번째 각 피어조직에 대해, 앵커 피어를 지정한다. 두번째는 각 멤버에 대해 MSP 디렉토리 위치를 지정한다. 차례대로 오더 제니시스 블록에 각 조직에 대해 루트 인증서를 저장할 수 있다. 이것은 중요한 개념이다. 이제 오더링 서비스와의 모든 엔티티는 검증된 디지털 서명을 가질 수 있다.



### **Run the tools**

`configxgen` 와 `cryptogen`  명령을 통해 인증서/키, 다양한 구성 아티팩트를 수동으로 생성 할 수 있다. 

**수동으로 아키팩트 생성하기**

`byfn.sh`  의 `generateCerts` 를 참조하여,  `crypto-config.yaml` 파일에 정의된 대로 인증서를 생성 할 수 있다.  그러나 이보다 편한 방법을 알아보자.

우선 `cryptogen` tool을 실행시키자. 바이너리 파일은 bin 디렉토리 안에 있음으로, tool이 있는 상대경로를 지정해야한다.

```bash
../bin/cryptogen generate --config=./crypto-config.yaml
```

인증서와 키(MSP 자료)는 fisrt-network 디렉토리 내부에 crypto-config 폴더에 보관된다.

이제 `configxgen` 에게 `configtx.yaml` 이 어디에 있는지 알려주자. 

```bash
export FABRIC_CFG_PATH=$PWD
```

그러고 난 후 `configtxgen` tool을 호출하여 오더 제니시스 블록을 생성하자

```bash
../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```



**Channel Configuration Transaction 생성**

`CHANNEL_NAME` 을 환경변수로 지정하고, configtxgen를 실행하여 채널 트랜잭션 아티팩트를 생성하자.

```bash
export CHANNEL_NAME=mychannel
 ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```

Raft 또는 Kafka 오더링 서비스를 이용하기 위해 특별한 명령을 만들 필요가 없다. `TwoOrgsChannel` profile 은 네트워크를 위한 제니시스 블록이 생성 될 때 `configtx.yaml` 에서 지정한 오더링 서비스를 사용한다.

다음으로 우리가 구성할 채널에서 Org1의 앵커피어를 정의하자.

```bash
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
```

마찬가지로 Org2의 앵커피어도 정의하자

```bash
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
```



### Start the network

네트워크를 시작해보자

```bash
docker-compose -f docker-compose-cli.yaml up -d
```

`-d`  옵션은 Background로 실행한다는 뜻으로, 네트워크의 실시간 로그를 보고싶다면 제거하고 실행



**Create & Join Channel**

앞에서 Channel Configuration Transaction 생성의 과정과 같이 채널을 만든다. 

CLI 컨테이너에 접속하기 전에 CLI에서 사용될 환경변수들을 알맞게 설정 해준다. Defult는 `peer0.org1.example.com` 에 대한 설정으로 되어있으며, 다른 peer 또는 orderer에게 보내고자 한다면 해당값들을 적절하게 수정해야한다. `docker-compose-base.yaml` 에서 해당 설정들을 확인 할 수 있다.

```bash
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

이후 docker exec 명령어를 통해 CLI 컨테이너에 접속한다.

```
docker exec -it cli bash
```

다음, 만든 channel configuration transaction을 채널 생성요청의 일환으로 ordere에게 보낸다. 



채널이름은 `-c` flag를 통해 지정할 수 있으며, channel configuration transactions는 `-f` 를 통해 지정 할 수 있다.  이 경우에는 `channel.tx` 이지만 다다른 이름으로 설정 할 수 있다. 파라미터로 넘겨줄 필요가 없도록 컨테이너 내부에서 `CHANNEL_NAME` 을 환경변수로 선언해주자. 채널 이름은 반드시 소문자여야하며, 250글자보다 작아야하며 정규식 `[a-z][a-z0-0.-]*` 만족해야한다. 

```bash
export CHANNEL_NAME=mychannel

peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

위 명령어는 채널 가입에 사용할 genesis block `CHANNEL_NAME.block` 을 반환한다. 해당 블록에는 `channel.tx` 에서 지정한 구성정보들을 포함하고 있다. `ls` 명령어를 통해 blcok이 반환되었음을 확인 할 수 있다.

>  **connection error: desc = "transport: authentication handshake failed: x509: certificate signed by unknown authority". Reconnecting…** 와 같은 에러가 발생한다면 network를껐다가 다시 실행시키고 위 과정 반복

```bash
./byfn.sh down
```



이제 `peero0.org1.example.com` 을 채널에 가입시켜보자 

```bash
peer channel join -b mychannel.block
```

위 4가지 환경변수들을 적절하게 설정함으로서 다른 peer들 역시 채널에 가입시킬 수 있다. 이 튜토리얼에서는 채널의  Anchor peer 정의를 업데이트하기 위해  간단하게 `peer0.org2.example.com` 를 가입시켜보자

```bash
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ADDRESS=peer0.org2.example.com:9051
CORE_PEER_LOCALMSPID="Org2MSP" 
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
peer channel join -b mychannel.block
```



### Update the anchor peers

다음 명령은 채널 업데이트로 채널의 정의로 전파한다. 필수적으로 채널의 제네시스 블록 위에 추가적인 구성정보를 추가한다. 중요한 것은 제네시스 블록을 수정하는 것이 아니라 단순히 앵커피어를 정의하는 체인을 더하는 것이다.

Org1의 앵커피어를 `peer0.org1.example.com` 로 정의하기 위해 채널 정의를 업데이트 하자

> 채널정의를 업데이트 하기 전에 4가지 환경변수들을 다시 peer0.org1.example.com으로 맞추어놓자

```bash
peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

이제 Org2의 앵커피어를 `peer0.org2.example.com` 로 정의하자.

```bash
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp 
CORE_PEER_ADDRESS=peer0.org2.example.com:7051 
CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt 
peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```



### Install and define a chaincode

Appdms chaincode를 이용해서 ledger와 통신한다. 때문에 트랜잭션을 보장하고 실행하는 모든 피어는 chaincode를 설치할 필요가 있다. 그리고 난 후 채널에서 체인코드를 인스턴스화 시킨다.

> 채널정의를 업데이트 하기 전에 4가지 환경변수들을 다시 peer0.org1.example.com으로 맞추어놓자

```bash
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/
```

위 명령어는 지정된 소스코드를 피어의 파일 시스템에 배치한다.

채널에서 인스턴스화 시킬 때,  Org1, Org2 양쪽 피어들에게 요구하기 위한 보증정책이 설정된다. 때문에 Org2에도 chaincode를 설치해야 한다. 환경변수를 수정하고 설치하자.

```bash
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/
```

이제 채널에서 체인코드를 추상화 시켜보자. 채널에서 체인코드를 초기화하고, 각 체인코드에 대해 보증정책을 설정한다. 그리고 모든 타켓 피어를 위한 체인코드 컨테이너를 실행하자.  `-P`  옵션을 통해 보증정책을 설정할 수 있다.

> Node.js에 경우 몇분 정도 소요될 수 있다.

```bash
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l node -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

