# FabCar 소스코드 분석



### Launch The Network

네트워크를 실행하기 위해서 `startFabric.sh` 를 사용하다. 이 스크립트 파일을 분석해보자



**1번 인자 값에 따라, `CC_SRC_PATH` 경로를 잡아준다.**

```bash
# Exit on first error
set -e

# don't rewrite paths for Windows Git Bash users
export MSYS_NO_PATHCONV=1
starttime=$(date +%s)
CC_SRC_LANGUAGE=${1:-"go"}
CC_SRC_LANGUAGE=`echo "$CC_SRC_LANGUAGE" | tr [:upper:] [:lower:]`
if [ "$CC_SRC_LANGUAGE" = "go" -o "$CC_SRC_LANGUAGE" = "golang"  ]; then
	CC_RUNTIME_LANGUAGE=golang
	CC_SRC_PATH=github.com/chaincode/fabcar/go
elif [ "$CC_SRC_LANGUAGE" = "java" ]; then
...
else
	echo The chaincode language ${CC_SRC_LANGUAGE} is not supported by this script
	echo Supported chaincode languages are: go, javascript, and typescript
	exit 1
fi
```

- `set -e`  명령어는 에러가 발생하는 즉시 스크립트 실행을 멈춘다. 반대로 `set +e` 는 에러가 발생하더라도 계속해서 명령을 수행한다.



**키가 존재한다면 삭제한다.**

```bash
rm -rf ./hfc-key-store
```



**`byfn.sh`을 이용해 채널을 만들고, 피어를 채널에 가입시킨다.**

```bash
# launch network; create channel and join peer to channel
cd ../first-network
echo y | ./byfn.sh down
echo y | ./byfn.sh up -a -n -s couchdb
```



**`byfn.sh` 에서 어떻게 채널을 만들고 피어를 가입시키는지 살펴보자**

```bash
# Generate the needed certificates, the genesis block and start the network.
function networkUp() {
  checkPrereqs
  # generate artifacts if they don't exist
  if [ ! -d "crypto-config" ]; then
    generateCerts
    replacePrivateKey
    generateChannelArtifacts
  fi
  ....
```

`checkPrereqs`  함수를 호출하고, `crypto-config`  디렉토리가 없을 경우 해당 아티팩트를 만드는 함수들을 실행한다. `checkPrereqs`  함수는 도커가 설치되어 있는지, 설치 된 버전을 지원하는지에 대한 검증 부분이 작성되어 있다. 

`generateCerts` 에서는 아래 명령어를 수행한다.

```bash
cryptogen generate --config=./crypto-config.yaml
```

`generateChannelArtifacts` 에서는 `configtxgen` 명령어를 통해 네트워크에 필요한 요소들을 만든다. 

이후  networkUp() 에서 `docker-compose-cli.yaml` 를 도커에 올린다.



다시 `startFabric.sh` 로 돌아오자

**org1의 peer0 에 smart contract를 설치한다.**

```bash
echo "Installing smart contract on peer0.org1.example.com"
docker exec \
  -e CORE_PEER_LOCALMSPID=Org1MSP \
  -e CORE_PEER_ADDRESS=peer0.org1.example.com:7051 \
  -e CORE_PEER_MSPCONFIGPATH=${ORG1_MSPCONFIGPATH} \
  -e CORE_PEER_TLS_ROOTCERT_FILE=${ORG1_TLS_ROOTCERT_FILE} \
  cli \
  peer chaincode install \
    -n fabcar \
    -v 1.0 \
    -p "$CC_SRC_PATH" \
    -l "$CC_RUNTIME_LANGUAGE"
```

마찬가지로 다른 피어들에서도 smart contract를 설치한다.

**이후 내 채널에 smart contract를 인스턴스화 시킨다.**

```bash
echo "Instantiating smart contract on mychannel"
docker exec \
  -e CORE_PEER_LOCALMSPID=Org1MSP \
  -e CORE_PEER_MSPCONFIGPATH=${ORG1_MSPCONFIGPATH} \
  cli \
  peer chaincode instantiate \
    -o orderer.example.com:7050 \
    -C mychannel \
    -n fabcar \
    -l "$CC_RUNTIME_LANGUAGE" \
    -v 1.0 \
    -c '{"Args":[]}' \
    -P "AND('Org1MSP.member','Org2MSP.member')" \
    --tls \
    --cafile ${ORDERER_TLS_ROOTCERT_FILE} \
    --peerAddresses peer0.org1.example.com:7051 \
    --tlsRootCertFiles ${ORG1_TLS_ROOTCERT_FILE}
```



**내 채널에 smart contract에 initLedger 트랜잭션을 제출한다. 이 트랜잭션은 모든 피어에게 전달 된다.**

```bash
echo "Submitting initLedger transaction to smart contract on mychannel"
echo "The transaction is sent to all of the peers so that chaincode is built before receiving the following requests"
docker exec \
  -e CORE_PEER_LOCALMSPID=Org1MSP \
  -e CORE_PEER_MSPCONFIGPATH=${ORG1_MSPCONFIGPATH} \
  cli \
  peer chaincode invoke \
    -o orderer.example.com:7050 \
    -C mychannel \
    -n fabcar \
    -c '{"function":"initLedger","Args":[]}' \
    --waitForEvent \
    --tls \
    --cafile ${ORDERER_TLS_ROOTCERT_FILE} \
    --peerAddresses peer0.org1.example.com:7051 \
    --peerAddresses peer1.org1.example.com:8051 \
    --peerAddresses peer0.org2.example.com:9051 \
    --peerAddresses peer1.org2.example.com:10051 \
    --tlsRootCertFiles ${ORG1_TLS_ROOTCERT_FILE} \
    --tlsRootCertFiles ${ORG1_TLS_ROOTCERT_FILE} \
    --tlsRootCertFiles ${ORG2_TLS_ROOTCERT_FILE} \
    --tlsRootCertFiles ${ORG2_TLS_ROOTCERT_FILE}
```

