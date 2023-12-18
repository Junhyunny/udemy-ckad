
## Chapter 14. Recap - Pods with YAML

yml 파일을 사용해 파드(pod)를 정의할 때 최상위 레벨에 필드가 4가지 존재한다.

```yml
apiVersion:
kind:
metadata:
  # something for metadata
  name: myapp
  labels: 
    app: myapp
    type: frontend
spec:
  # something for object spec
  containers:
    - name: nginx-container
      image: nginx
```

루트 레벨 속성이라고하며 반드시 설정에 포함되어야 한다.

apiVersion 항목은 이 개체를 생성할 때 사용하는 쿠버네티스 버전이다. 만들려는 것이 무엇인지에 따라 올바른 API 버전이 필요하다.

kind 항목은 개체 유형을 정의한다. 

metadata 항목은 개체에 대한 데이터다. 이름, 라벨 같은 것들이 필요하다. 위 항목들과 다르게 사전 자료구조 형태이다. 메타 데이터에 사용하는 name, labels은 형제이자 metadata의 자식 값으로 위치해야 한다. name 정보는 문자열, labels 정보는 사전 자료구조를 사용한다. 

메타데이터 라벨에는 어떤 값도 명시할 수 있다. 레이블에 명시된 값을 사용해 쿠버네티스는 파드를 그룹핑할 수 있다.

spec 항목도 사전 자료구조 형태이다. spec 항목 자식으로 containers 항목은 리스트 형태로 필요한 이미지를 모두 정의할 수 있다. 필요한 값을 모두 정의한 후 다음 명령어를 수행하면 파드가 실행된다.

```
$ kubectl create -f pod-definition.yml 
```

파드가 리스트를 보기 위해선 다음 명령어를 수행한다.

```
$ kubectl get pods
```

파드의 상세 정보를 보기 위해선 다음 명령어를 수행한다. 다음과 같은 정보들을 알 수 있다. 

- 파드가 생성된 시점
- 파드에 배정된 라벨
- 기타 등등

```
$ kubectl describe pod myapp
```
