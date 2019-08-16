# DOCKER



### container 확인

```bash
# 실행중인 컨테이너 확인
docker ps 

# 전체 컨테이너 확인
docker ps -a
```



### container에 접속

```bash
docker exec -it ${img} bash
```

- -i  : STDIN 표준 입출력을 연다
- -t : 가상 tty(pseudo-TTY) 를 통해 접속한다.
- -e: 환경변수 설정

 `exit`  명령을 통해 접속을 종료 할 수 있다.

