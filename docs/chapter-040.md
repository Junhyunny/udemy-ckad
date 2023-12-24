
## Chapter 40. Commands and Arguments in Docker

컨테이너는 운영체제를 호스팅하도록 설계된 것이 아니다. 컨테이너는 특정 작업이나 프로세스를 실행하도록 만들어진다. 컨테이너 내부에 애플리케이션이 충돌 등으로 인해 종료되면 함께 종료된다. 예를 들어 nginx, mysql 등 이미지를 보면 도커 파일 마지막이 CMD 지시문를 통해 해당 프로그램을 실행하는 방식이다.

- nginx 이미지 도커 파일

```dockerfile
...
CMD ['nginx']
```

- mysql 이미지 도커 파일

```dockerfile
...
CMD ['mysql']
```

지시문를 통해 실행된 프로세스가 종료되면 컨테이너는 함께 종료된다. 우분투(ubuntu)의 도커 파일을 보면 다음과 같이 정의되어 있다.

```dockerfile
...
CMD ['bash']
```

bash는 nginx, mysql 프로세스와 다르게 일회성 쉘(shell) 명령어이다. 전달 받은 인수(argument)를 전달받아 명령을 실행하고 즉시 종료한다. 이런 이유로 우분투 컨테이너는 실행되지마자 종료된다. 

### ENTRYPOINT and CMD

docker 명령어로 컨테이너를 실행할 때 뒤에 명령어를 붙일 수 있다. 

- 10초 동안 잠든다.

```
$ docker run ubuntu sleep 10 
```

도커 파일 마지막에 명시된 CMD 지시문는 선택적으로 수행된다. 도커 컨테이너를 실행할 때 명령어 인수가 추가된 경우 실행되지 않는다. 컨테이너를 실행할 때 명령어가 주어지면 완전히 대체된다.

반대로 ENTRYPOINT 지시문는 반드시 실행된다. 컨테이너를 실행할 때 추가되는 인수는 ENTRYPOINT에 지정된 지시문의 인수로 추가된다.

둘을 조합하여 사용할 수 있다. 다음과 같은 도커 파일을 예시로 들 수 있다.

- ENTRYPOINT 지시문으로 지정한 sleep 커맨드는 반드시 실행된다.
- CMD 지시문으로 지정한 5는 선택적으로 실행된다.
    - 컨테이너가 실행될 때 인수가 추가되면 외부에서 전달받은 인수를 사용한다.
    - 컨테이너가 실행될 때 인수가 추가되지 않으면 기본 값으로 지정된 5를 사용한다.

```dockerfile
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

명령어 옵션을 통해 도커 파일에 지정된 엔트리 포인트(entry point)를 변경할 수 있다. 

```
$ docker run --entrypoint ls ubuntu .
```
