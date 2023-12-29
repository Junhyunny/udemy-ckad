 
 ## Chapter 55. Security Contexts

다음은 컨테이너 보안에 관련된 사용자 변경, 권한 추가, 삭제 방법들이다.

- 호스트 변경

```
$ docker run --user=1001 ubuntu sleep 3600
```

- 권한 추가 

```
$ docker run --cap-add MAC_ADMIN ubuntu
```

- 권한 삭제

```
$ docker run --cap-drop KILL ubuntu
```

쿠버네티스도 동일한 작업이 가능하다. 쿠버네티스는 컨테이너를 파드에 보관하기 때문에 컨테이너 레벨이나 파드 레벨에서 보안 설정을 선택할 수 있다. 파드 수준으로 설정하면 파드 내 모든 컨테이너에 설정이 적용된다. 파드와 컨테이너 둘 다 같은 설정을 하면 컨테이너의 설정이 파드의 설정을 무시한다. 즉 컨테이너 설정의 우선순위가 높다.

파드 레벨의 시큐리티 컨텍스트 설정은 다음과 같다. 컨테이너 설정과 동일한 레벨에 정의한다.

```yml
apiVersion: v1
kind: Pod
metadata: 
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
    capabilities:
      add: ["MAC_ADMIN"]
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
```

컨테이너 레벨의 시큐리티 컨텍스트 설정은 다음과 같다. 컨테이너 설정 하위에 정의한다.

```yml
apiVersion: v1
kind: Pod
metadata: 
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```
