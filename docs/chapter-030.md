
## Chapter 30. Recap - Namespace

네임스페이스는 쿠버네티스 환경을 고유하게 구분할 수 있다. 쿠버네티스에 배포되는 모든 파드, 레플리카 세트, 디플로이먼트 같은 오브젝트들은 모두 네임스페이스 내부에 존재한다. 별도로 네임스페이스를 지정하지 않으면 디폴트(default) 네임스페이스를 사용한다.

네임스페이스는 같은 쿠버네티스 클러스터 내부에 있더라도 리소스들을 다르게 구분지을 수 있는 고유한 공간이다. 어떤 사용자가 실수나 의도적으로 다른 사용자의 파드, 서비스를 지울 수 없도록 리소스들을 각 네임스페이스에 고립시킨다. 각 네임스페이스에 속한 오브젝트들은 각자의 역할을 수행한다. 

쿠버네티스에 의해 자동적으로 생성되는 네임스페이스는 다음과 같다.

- kube-system
    - 네트워킹 솔루션, DNS 서비스 등을 수행하는 오브젝트들이 존재하는 공간
- default
    - 쿠버네티스에 의해 기본적으로 만들어진 공간이다. 
    - 일반 사용자들이 별도로 네임스페이스를 지정하지 않은 경우 오브젝트들이 이 곳에 배포된다.
- kube-public
    - 모든 사용자가 사용할 수 있어야 하는 리소스가 생성되는 공간이다. 

개발자나 운영자들은 네임스페이스 사용해 작업 환경을 구분지을 수 있다. 이를 통해 개발 환경과 프로덕션 환경을 다르게 구축하는 것도 가능하다. 

네임스페이스는 어떤 특징을 가질까?

- 네임스페이스는 각자 고유한 정책 모음을 가질 수 있다. 
    - 누가 어떤 작업을 수행할 것인지 정의할 수 있다. 
- 각 네임스페이스에 고유한 리소스를 할당할 수 있다. 
    - 각 네임스페이스는 일정량을 보장 받고 허용된 한도 이상을 사용하지 않는다. 
- 같은 네임스페이스에 있는 경우 이름만으로 호출할 수 있다.
    - 다른 네임스페이스에 있는 오브젝트에도 도달할 수 있지만, 상대방 네임스페이스 이름과 서비스 이름을 함께 불러야 한다.

다음과 같이 외부 네임스페이스에 존재하는 서비스를 호출할 때 사용한다. 외부 네임스페이스에 속한 서비스에도 접근 가능한 이유는 서비스가 생성될 때 DNS 엔트리 항목에 자동으로 이 포맷에 추가으로 때문이다. 

- db-service - 외부 서비스 이름
- dev - 개발 환경 네임스페이스
- svc - 서비스 오브젝트를 위한 서브 도메인 
- cluster.local - 쿠버네티스 도메인 이름

```
db-service.dev.svc.cluster.local
```

### Namespace Definition

다음과 같은 명령어를 통해 파드를 생성할 때 K8S 네임스페이스를 지정할 수 있다. 

```
$ kubectl create -f pod-definition.yml --namespace=dev
```

yml 파일을 통해 네임스페이스를 지정할 수 있다. 파일 방식은 같은 종류의 오브젝트가 항상 같은 네임스페이스에 배포될 수 있도록 만든다. 사람 실수를 줄인다.

```yml
apiVersion: v1
kind: Pod
metadata: 
  name: myapp-pod
  namespace: dev
  labels: 
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```

네임스페이스도 다음과 같은 방식으로 파일로 선언할 수 있다.

```yml
apiVersion: v1
kind: Namespace
metadata:
    name: dev
```

### Using Other Namespace

다른 네임스페이스 작업을 하면 다음과 같은 불편함이 있다.

```
$ kubectl get pods --namespace=dev

NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   1/1     Running   0          12s
```

매번 네임스페이스를 위한 옵션을 추가하는 것이 아니라 영구적으로 변경할 수 있다. 다음 명령어를 통해 쿠버네티스 기본 네임스페이스 공간을 `dev`로 변경한다.

```
$ kubectl config set-context $(kubectl config current-context) --namespace=dev 

Context "docker-desktop" modified.
```

네임스페이스 옵션 없이 파드를 조회하면 이전 단계에서 dev 네임스페이스에 생성한 파드가 조회된다.

```
$ kubectl get pods

NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   1/1     Running   0          48s
```

모든 네임스페이스에 존재하는 오브젝트를 보고 싶다면 --all-namespaces 옵션을 사용한다.

```
$ kubectl get pods --all-namespaces

NAMESPACE     NAME                                     READY   STATUS    RESTARTS      AGE
dev           myapp-pod                                1/1     Running   0             15m
kube-system   coredns-5dd5756b68-9qjxr                 1/1     Running   1 (91m ago)   2d23h
kube-system   coredns-5dd5756b68-dpv6f                 1/1     Running   1 (91m ago)   2d23h
kube-system   etcd-docker-desktop                      1/1     Running   1 (91m ago)   2d23h
kube-system   kube-apiserver-docker-desktop            1/1     Running   1 (91m ago)   2d23h
kube-system   kube-controller-manager-docker-desktop   1/1     Running   1 (91m ago)   2d23h
kube-system   kube-proxy-txt8g                         1/1     Running   1 (91m ago)   2d23h
kube-system   kube-scheduler-docker-desktop            1/1     Running   1 (91m ago)   2d23h
kube-system   storage-provisioner                      1/1     Running   2 (90m ago)   2d23h
kube-system   vpnkit-controller                        1/1     Running   1 (91m ago)   2d23h
```

### ResourceQuota

네임스페이스에서 리소스를 제한하려면 리소스 할당량을 생성해야 한다. 리소스쿼타(ResourceQuota) 오브젝트를 통해 리소스 할댱량을 정의할 수 있다. 파일 정의는 다음과 같이 할 수 있다.

```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```
