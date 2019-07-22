# MYSQL 일기장

이번 노트는 MYSQL를 사용하면서 필요한 내용들을 기록하고, 만났던 error들을 놓치지 않기 위해 기록하는 MYSQL 일기장이다. 주로 ORM을 사용하여 쿼리문을 직접 작성하는 경우는 많이 없으나, DB 성능 측정을 위한 테스트 쿼리들을 작성하면서 생긴 이슈와 쿼리문들을 기록해 나갈 예정이다.



### Explain

- 쿼리 앞에 Explain을 붙이면, 쿼리가 어떤 테이블에서 어떻게 실행되는지에 대한 정보들을 확인 할 수 있다. 이 정보중 눈여겨 봐야할 정보는 type으로 조인이 어떤 방식으로 이루어지는 에 대한 정보이다. 보다 자세한 내용은 [공식문서](http://www.mysqlkorea.com/sub.html?mcode=manual&scode=01&m_no=21444&cat1=7&cat2=217&cat3=227&lang=k)를 참고하자



### Database 명령어

```sql
show databases; // 데이터베이스 확인
use ${데이터베이스 이름}; // 데이터베이스 선택
```



### Inner Join

```sql
select *
from user_relation_tb 
Inner join member_tb on board_tb.user_FK = member_tb.idx_PK
where board_tb.idx_PK = 125001
```



### Insert

```sql
INSERT INTO ${테이블이름}(${column A} , ${column B}...)
          VALUES(${A 컬럼에 넣고자 하는 값}, {B 컬럼에 넣고자 하는 값}...);
// VALUES에 mysql에서 제공하는 함수들을 이용해서 값을 넣을 수 있음
```



### 프로시저를 이용한 더미데이터 생성

```sql
DELIMITER $$
DROP PROCEDURE IF EXISTS loopInsertRelation$$
 
CREATE PROCEDURE loopInsertRelation(IN size INT) -- IN 키워드는 불변성을 보장한다. --
BEGIN
  DECLARE i INT DEFAULT 0;
	DECLARE  stateNum INT DEFAULT  0;
	DECLARE  stateValue varchar(20);
	DECLARE  master INT DEFAULT  RAND() * 10 % 5; -- RAND함수는 0~1사이의 값을 반환한다. --
	DECLARE  slave INT DEFAULT  RAND() * 10 % 5;	
    
	WHILE i < size  DO
		SET stateNum = RAND() * 10 % 5; -- 변수에 값을 할당 할 시에는 SET을 이용한다. --
    SET master =   RAND() * 310000;
		SET slave = RAND() * 310000;
        
    IF stateNum =  0 THEN SET stateValue = "FLLOW";
		ELSEIF stateNum = 1 THEN SET stateValue = "REQUEST";
		ELSEIF stateNum = 2 THEN SET stateValue = "BLOCK";
		ELSE  SET stateValue = "BLOCKED";
		END IF; -- IF문의 마지막에는 END IF로 닫아준다. --
		
    INSERT INTO user_relation_tb( master_FK, slave_FK,  state, created_DT, updated_DT)
			VALUES(master, slave, stateValue, now(), now());
    SET i = i + 1;    
    END WHILE;
END$$
DELIMITER $$

call loopInsertRelation(100000); --호출--
```



### 데이터베이스 용량확인

```sql
SELECT TABLE_NAME AS 'Tables',
                     round(((data_length + index_length) / 1024 / 1024), 2) 'Size in MB'
FROM information_schema.TABLES
WHERE table_schema = "${DB 이름}"
ORDER BY (data_length + index_length) DESC;
```



###  Auto Increment 재정렬

```mysql
ALTER TABLE '테이블네임' AUTO_INCREMENT=1; 
SET @COUNT = 0; 
UPDATE '테이블네임' SET '테이블네임'.'컬럼네임' = @COUNT:=@COUNT+1;
```



### 마주한 Error

`Error Code: 1066. Not unique table/alias: 'board_tb'`

+ 발생 쿼리

  ```sql
    select *
    from board_tb 
    Inner join board_tb on board_tb.user_FK = member_tb.idx_PK
    where board_tb.idx = 2;
  ```

+ 발생원인

  - board_tb를 from절에서 한번, join절에서 한번 총 두번 호출하고 있었다.

  

`Error Code: 1175. You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column To disable safe mode, toggle the option in Preferences -> SQL Editor and reconnect.`

- 발생원인 : safe update mode 사용 on 

- 해결 : `SET SQL_SAFE_UPDATES = 0;`

  

`Error Code: 1451. Cannot delete or update a parent row: a foreign key constraint fails (`summer_challenge`.`user_relation_tb`, CONSTRAINT `slave` FOREIGN KEY (`slave_FK`) REFERENCES `member_tb` (`idx_PK`) ON DELETE CASCADE ON UPDATE NO ACTION)`

-  발생원인 : member_tb를 참조하는 외래키를 가지는 테이블이 있고, 이 테이블에 CASCADE ON UPDATE가 NO Action으로 설정되어 있어, 제약사항으로 인해 기본키값을 바꿀 수 없는 경우이다.
- 해결 : 외래키를 가지는 테이블을 비워주고 수행한다. 실제 서비스시에는 이러한 해결방법이 불가하니, 초기에 신중하게 설계 할 필요가 있겠다.