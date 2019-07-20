데이터베이스 기술면접 대비 문제

 

### Question

​	**a. 트랜직션이 무엇인지, 트랜직션이 가지는 4가지 성질에 대해서 설명하시오**

​	**b. 트랜잭션을 병행으로 처리할려고 할 때 발생 할 수 있는 문제에 대해 설명**

​	**c. 위와 같은 문제를 방지하기 위한 방법을 설명**



### Solution

---

 **a.** DB의 상태를 변화시키는 하나의 논리적인 작업단위를 구성하는 연살들의 집합으로 트랜잭션은 아래와 같은 네가지 성질을 만족해야합니다. 

첫번째 원자성을 만족해야합니다. 즉 트랜잭션의 모든 연산들은 정상적으로 수행완료되거나 어떠한 연산도 수행되지 않은 상태를 보장해야합니다. 

두번째 일관성을 만족해야합니다. 트랜잭션이 완료 된 이후에도 데이터베이스는 일관된 상태로 유지되어야합니다. 예를 들어 계좌이체를 성공적으로 실행됬다면, A계좌 잔액과 B계좌 잔액의 합이 트랜잭션 실행 전의 합과 동일해야 합니다.

세번째 독립성을 만족해야합니다. DB는 클라이언트들이 같은 데이터를 공유하는 것이 목적임으로 여러 트랜잭션이 동시에 수행됩니다. 이 때 트랜잭션은 상호 간의 존재를 모르고 독립적으로 수행되어야 합니다. 또한 트랜잭션이 실행중인 데이터는 다른 트랜잭션이 동시에 참조하지 못하도록 보장되어야 합니다.

마지막으로 지속성을 만족해야합니다. 성공적으로 수행된 트랜잭션은 영원히 반영되어야합니다. 완료된 결과는 디스크와 같은 보조기억장치에 저장되거나 시스템 장애가 회복되고 난 후에 어떠한 형태로든지 그 데이터를 복구 할 수 있어야 합니다.

 

**b.** 아래와 같은 네가지 문제가 발생 할 수 있습니다. 

첫번째는 Lost Update로 하나의 값에 계속해서 Overwritie하여 데이터의 갱신이 무효화 되는 현상입니다. 가령 2개 이상의 트랜잭션이 같은 자료를 공유하고 있다면 갱신 할 때 이 둘 중하나의 결과가 없어지는 경우입니다.

두번째는 비완료 의존성으로, 하나의 트랜잭션이 수행이 실패 한후 회복되기 전에 다른 트랜잭션이 이 실패한 갱신 결과를 참조하는 현상입니다.

세번째는 모순성으로 두개의 트랜잭션이 병행 수행 될 때 어떤 값은 갱신 전의 값을, 다른 값은 갱신 후의 값을 읽어 데이터가 불이치하는 현상입니다.. 가령 T1이 x와 y에 +3연산을 수행하고, T2가 x,y에 3배를 곱하는 연산을 수행 할 때, T1이 x에  3을 더하고, 이후 T2 트랜잭션이 수행되면 T1은 3y에다가 3을 더하는 현상이 발생합니다.

네번째는 연쇄복귀로, 복수의 트랜잭션이 데이터 공유 시 특정 트랜잭션이 처리를 취소할 경우, 다른 트랜잭션이 처리한 부분에 대해 취소가 불가능한 현상입니다.

 

**c.** 위와 같은 문제를 해결하기 위해 사용하는 주요 데이터 엑세스를 상호 배타적으로 하는 로킹, 트랜잭션 간의 순서를 미리 정하여 제어하는 타임스탬프 오더링, 데이터의 원본이 아닌 사본에 대해서만 갱신이 이루어짐, 트랜잭션 종료시 검증이 되면 DB에 반영된다.