
## Chapter 45. Environment Variables

도커 컨테이너를 실행할 때 환경 변수를 지정할 수 있듯이 쿠버네티스 파드를 실행할 때도 환경 변수를 설정할 수 있다. 도커 컨테이너를 실행할 떄 다음과 같은 명령어를 수행한다고 가정해보자.

```
$ docker run -e APP_COLOR=pink simple-webapp-color
```

쿠버네티스 파드 오브젝트를 통해 배포할 때 다음과 같이 선언할 수 있다.

- env 속성은 키-값 쌍으로 리스트로 표현된다.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink
```

키-값 쌍으로 환경 변수를 선언하는 방법도 있지만, ConfigMap나 Secrets 오브젝트를 사용하는 방법이 있다. 

ConfigMap 오브젝트를 사용하면 다음과 같다.

```yml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef: 
```

Secrets 오브젝트를 사용하면 다음과 같다.

```yml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef: 
```
