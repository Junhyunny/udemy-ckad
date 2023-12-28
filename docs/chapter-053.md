
## Chapter 53. Demo: Encrypting Secret Data at Rest

시크릿 오브젝트를 생성한다.

```
$ kubectl create secret generic my-secret --from-literal=key1=supersecret

secret/my-secret created
```

시크릿 오브젝트를 살펴보면 Base64 인코딩 된 데이터를 확인할 수 있다. 

```
$ kubectl get secret my-secret -o yaml 

apiVersion: v1
data:
  key1: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  creationTimestamp: "2023-12-28T22:31:45Z"
  name: my-secret
  namespace: default
  resourceVersion: "510260"
  uid: c5fda064-7468-4edb-aef1-124a9959e6cd
type: Opaque
```

Base64 인코딩 된 데이터는 복호화가 쉽게 가능하다. ETCD에는 평문(plain text)가 저장된다. ETCD에 암호화하여 저장하는 방법은 아래 레퍼런스를 참조한다.

#### REFERENCE

- <https://www.udemy.com/course/certified-kubernetes-application-developer/learn/lecture/34549240#overview>
- <https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/>
