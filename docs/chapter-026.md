
## Chapter 26. Recap - Deployment

쿠버네티스 환경에서 애플리케이션 컨테이너 파드가 여러 개 실행된다. 실행되는 애플리케이션을 동시에 모두 교체하는 것은 좋지 않다. 하나씩 실행 중인 파드를 교체해 나간다. 이를 롤링 업그레이드 혹은 롤링 배포(rolling deploy)라고 한다. 

배포된 파드를 하나씩 업그레이드하는 중 예기치 못한 오류가 발생한 경우에는 어떻게 해야할까? 파드의 일부는 구버전, 나머지는 새로운 버전인 상태로 서비스를 수행해야할까? 배포되었던 신규 버전들은 모두 롤백되어야 한다. 

배포된 컨테이너의 웹 서버 버전 업그레이드, 환경 스케일링, 리소스 할당 수정 등의 작업을 수행할 때 모든 변화를 한번에 롤-아웃(roll-out)될 수 있도록 배포 환경을 일시적으로 중단, 실행할 필요도 있다. 

디플로이먼트(deployment) 오브젝트는 위처럼 배포와 환경 변화에 어려움을 해결해준다. 디플로이먼트 구조는 다음과 같다. 

- 단일 파드들이 레플리카 세트(replica set)에 의해 관리된다.
- 레플리카 세트는 디플로이먼트에 의해 관리된다. 디플로이먼트의 계층 구조가 더 높다.

디플로이먼트는 애플리케이션 인스턴스를 매끄럽게 업그레이드할 수 있도록 돕는다. 

### Deployment Definition

레플리카 세트라 거의 유사하며 종류(kind)만 `Deployment`로 정의한다.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels: 
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

다음 명령어를 통해 디플로이먼트를 생성한다.

```
$ kubectl create -f deployment-definition.yml

deployment.apps/myapp-deployment created
```

실행된 디플로이먼트를 확인한다.

```
$ kubectl get deployments

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
myapp-deployment   3/3     3            3           14s
```

실행된 레플리카 세트를 확인한다.

```
$ kubectl get replicaset 

NAME                         DESIRED   CURRENT   READY   AGE
myapp-deployment-84ccc5558   3         3         3       19s
```

실행된 파드를 확인한다.

```
$ kubectl get pods

NAME                               READY   STATUS    RESTARTS   AGE
myapp-deployment-84ccc5558-2wzdb   1/1     Running   0          23s
myapp-deployment-84ccc5558-4gpqk   1/1     Running   0          23s
myapp-deployment-84ccc5558-zwkmt   1/1     Running   0          23s
```

모든 오브젝트를 보고 싶은 경우 다음과 같은 명령어를 사용한다.

```
$ kubectl get all
```