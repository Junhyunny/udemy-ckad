
## Chapter 41. Commands and Arguments in Kubernetes

우분투 이미지는 다음과 같은 도커 파일을 기반으로 생성되었다.

```dockerfile
FROM ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

다음과 같은 명령어를 통해 생성되었다.

```
$ docker build -t ubuntu-sleeper .
```

도커 명령어처럼 쿠버네티스도 명령어와 인수를 전달할 수 있다. 다음과 같은 yml 파일을 정의한다. 

- 위에서 생성한 우분투 이미지를 사용한다.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["ls"] # replace entry point for docker container
      args: ["."] # replace argument for docker container
```

쿠버네티스 yml 파일의 command 선언은 도커 파일의 ENTRYPOINT를 대체한다. args 선언은 CMD를 대체한다.