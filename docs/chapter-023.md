
## Chapter 23. Recap - ReplicaSets

파트가 하나만 있을 떄 파드에 문제가 생기면 서비스가 정지된다. 레플리케이션 컨트롤러(replication controller)를 사용하면 같은 파드를 여러 개 동시에 운영할 수 있다. 파드가 여러 개라면 하나가 문제되더라도 서비스에 지장이 없다. 이를 고가용성(high availability)라고 한다. 레플리케이션 컨트롤러는 지정한 파드 수를 유지해주기 때문에 한 개의 파드를 띄우더라도 사용하는 것이 좋다. 

레플리케이션 컨트롤러가 필요한 이유는 사용자가 접속이 많아졌을 때 발생하는 부하를 분산할 수 있다. 한 노드에서 여러 파드들이 서비스 중일 떄 리소스가 바닥나면 레플리케이션 컨트롤러는 다른 노드에 파드들을 배포한다. 레플리케이션 컨트롤러는 클러스터 내 여러 노드에 걸쳐 배포할 수 있다.

비슷한 용어가 있다.

- 레플리케이션 컨트롤러(replication controller)
- 레플리카 세트(replica set)

두 용도는 같지만, 완벽히 같지는 않다. 레플리케이션 컨트롤러는 레거시 기능이고, 레플리카 세트가 신규 기능으로 레플리카 세트 사용을 권장한다. 각 작동 방식에는 미세한 차이가 있다.

### Replication Controller

레플리케이션 컨트롤러 yml 파일을 생성한다. 레플리케이션 컨트롤러는 v1에서 동작한다. 스펙 영역에는 어떤 파드를 대상으로 복제본 제어를 수행할 것인지 정의한다. spec.template 속성을 사용해 파드 정보를 작성한다. 유지하기 위한 복제본 수에 대한 정보는 spec.replicas 속성을 사용한다.

```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  lables:
    app: myapp
    type: front-end
spec: 
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-pod
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
```

yml 파일을 보면 두 개의 metadata, spec 섹션이 있다. 

- 부모 metadata, spec 섹션 - 레플리케이션 컨트롤러 정보
- 자식 metadata, spec 섹션 - 파트 정보

생성된 레플리케이션 컨트롤러 정보를 볼 수 있다.

```
$ kubectl get replicationcontroller

NAME       DESIRED   CURRENT   READY   AGE
myapp-rc   3         3         3       24s
```

생성된 파드 정보를 볼 수 있다.

```
$ kubectl get pods                 

NAME             READY   STATUS    RESTARTS   AGE
myapp-pod        1/1     Running   0          23h
myapp-rc-5jrh8   1/1     Running   0          29s
myapp-rc-7c69k   1/1     Running   0          29s
myapp-rc-gddb2   1/1     Running   0          29s
```

### Replica Set

레플리카 세트를 생성하기 위한 yml 파일을 정의한다. 레플리케이션 컨트롤러와 유사하다. apiVersion 정보가 다르다. apps/v1 버전을 사용한다.

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: myapp-replicaset
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

레플리케이션 컨트롤러와 레플리카 세트의 큰 차이점은 selector 섹션의 존재 유무이다. 레플리카 세트는 selector 정보가 필수로 필요하다. selector 섹션을 통해 어떤 파드가 필요한지 식별한다. 파드 정보가 template 섹션에 있음에도 필요한 이유는 레플리카 세트는 파드 복제본을 관리할 때 selector 섹션에 정의된 기준을 만족하는 파드 수까지 고려해 리소스를 관리한다. 동일한 라벨을 가진 파드가 있다면 새로운 파드를 생성하지 않기 때문에 리소스를 효율적으로 다룰 수 있다. 레플리케이션 컨트롤러에서 selector 섹션은 필수가 아니지만, 사용은 가능하다.

레플리카 세트를 생성한다.

```
$ kubectl get replicaset

NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   3         3         3       9s
```

생성된 파드를 확인한다.

```
$ kubectl get pods

NAME                     READY   STATUS    RESTARTS   AGE
myapp-pod                1/1     Running   0          23h
myapp-rc-5jrh8           1/1     Running   0          19m
myapp-rc-7c69k           1/1     Running   0          19m
myapp-rc-gddb2           1/1     Running   0          19m
myapp-replicaset-2dbs6   1/1     Running   0          20s
myapp-replicaset-4tzww   1/1     Running   0          20s
myapp-replicaset-bv96w   1/1     Running   0          20s
```

### Labels and Selectors

라벨(label)을 통해 쿠버네티스 오브젝트들에게 이름을 붙힐 수 있다. 레플리케이션 컨트롤러나 레플리카 세트는 자신이 관리하는 파드들을 모니터링 할 수 있다. 복제본 파드들 중 하나라도 문제가 생기면 이를 복원한다. 

쿠버네티스 클러스터에 배포된 파드들은 굉장히 많을 것이다. 이들을 구분짓기 위해 파드를 생성할 때 라벨을 붙힌다. selector 섹션에 정의된 라벨 매칭을 통해 관심있는 파드들만 필터링할 수 있다. 파드 라벨링과 선택기(selector) 덕분에 레플리카 세트가 수 많은 파드들 중에서 필요한 파드들만 모니터링할 수 있다.
