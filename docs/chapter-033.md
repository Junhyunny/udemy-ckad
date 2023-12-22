
## Chapter 33. Certification Tip: Imperative Commands

쿠버네티스 오브젝트가 정의된 yml 파일을 효과적으로 작성하는 방법이 있다. `--dry-run` 옵션을 사용하면 된다. 기본 옵션은 리소스가 생성되기 때문에 `--dry-run=client` 옵션을 사용한다. 리소스가 생성되지 않는다. `-o yaml` 옵션을 사용하면 결과물을 yml 형태로 볼 수 있다. 두 옵션을 조합한 명령어와 리눅스의 결과물 리다이렉션을 사용하면 정의 파일을 생성할 수 있다.

```
$ kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

파드를 생성할 때 다음과 같다.

```
$ kubectl run nginx --image=nginx --dry-run=client -o yaml
```

디플로이먼트(deployment)를 생성할 때 다음과 같다.

```
$ kubectl create deployment --image=nginx nginx --dry-run -o yaml
```

서비스를 생성할 때 다음과 같다.  

- 이름이 redis-service인 서비스를 생성한다.
- 6379 포트를 통해 redis 파드를 노출한다.

```
$ kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

- 이름이 redis인 ClusterIP 타입의 서비스를 생성한다.
- 6379 포트를 통해 파드의 6379 포트를 노출한다.

```
$ kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

레디스가 아닌 nginx를 사용한 예제이다. NodePort 서비스 타입을 사용한다.

- 이름이 nginx-service인 NodePort 타입의 서비스를 생성한다.
- 80 포트를 통해 nginx 파드를 노출한다.

```
$ kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
```

- 이름이 nginx-service인 NodePort 타입의 서비스를 생성한다.
- 80 포트를 통해 nginx 파드를 노출한다.
- 노드 포트는 30080 포트 번호를 사용한다.

```
$ kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```