# DOCKER



### container 시작/재시작/정지

```bash
docker start ${container name}
docker restart ${container name}
docker stop ${container name}
```



### container 확인

```bash
# 실행중인 컨테이너 확인
docker ps 

# 전체 컨테이너 확인
docker ps -a
```

+ `CONTAINER ID` : 컨테이너에게 자동으로 할당되는 `고유한 ID`

+ `IMAGE` : 컨테이너를 생성할 때 사용된 이미지 이름

+ `COMMAND` : 컨맨드는 컨테이너가 시작될 때 실행될 명렁어, 기본은 `/bin/bash`명령어라 명령을 쓸 수 있습니다.

+ `CREATED` : 컨테이너가 생성되고 난 뒤 흐른 시간

+ `STATUS` : 컨테이너의 상태 ex) Up(실행 중), Exited(종료), Pause(일시 중지)

+ `PORTS` : 컨테이너가 개방한 포트와 호스트에 연결한 포트

+ `NAMES` : 컨테이너의 고유한 이름, `--name 옵션`으로 이름을 설정하지 않으면 도커 엔진이 임의의로 설정

  

### container에 접속

```bash
docker exec (옵션) (컨테이너 이름 또는 아이디의 앞부분 일부) (커맨드)
//example
docker exec -it 175 bash
```

- 실행중인 container에 명령어를 전달
- -i  : STDIN 표준 입출력을 연다
- -t : 가상 tty(pseudo-TTY) 를 통해 접속한다.
- -e: 환경변수 설정

 `exit`  명령을 통해 접속을 종료 할 수 있다.



```bash
docker run (옵션들) (이미지) (컨테이너 시작 시 실행할 커맨드)
```

- 이미지를 기반으로(없다면 다운로드) **컨테이너를 생성하고 시작**
- -p: 컨테이너와 호스트의 포트를 연결
- —name : 컨테이너 이름 설정

`exit`  명령을 통해 종료시, 컨테이너도 함께 종료, 쉘만 빠져나오기 위해서는 `ctrl +p`  -> `ctrl +q`  순서대로 입력



### container 삭제

```bash
docker rm -f `docker ps -a -q`
```



### 외부접속

- 컨테이너는 가상 머신과 마찬가지로 가상 ip를 할당받음, 기본적으로 도커는 172.17.0.x의 ip 부터 순차적으로 할당

- 아무 설정을 하지 않으면 기본적으로 외부에서 접근할 수 없고, **도커가 설치된 호스트**에서만 접근이 가능

- 외부에 노출하기 위해서는 eto0의 ip와 포트를 호스트의 ip와 포트에 바인딩해야 한다.

- `-p` 옵션을 이용해 컨테이너 포트를 호스트의 포트와 바인딩 할 수 있음

  ```bash
  -p [호스트의 포트]:[컨테이너의 포트]
  docker run -i -t --name mywebserver -p 80:80 ubuntu:14.04
  ```

  여러개의 포트 설정이 필요하면 `-p` 옵션을 여러번 사용 할 수  있다.

- apache를 이용한 외부접속 테스트

  ```bash
  apt-get update # apt-get 업데이트
  apt-get install apache2 -y # 아파치2 설치, -y 옵션은 중간에 "설치하시겠습니까?"에 대해서 Yes라는 의미
  service apache2 restart # 설치한 아파치2 실행
  service --status-all # 서비스 실행 확인
  ```

