
## Chapter 49. Secretes

컨피그 맵(config map)은 암호화가 필요한 설정을 저장하기에 적합하지 않다. 애플리케이션을 개발할 떄 암호화가 필요한 설정은 다음과 같다.

- 데이터베이스 호스트 이름
- 데이터베이스 비밀번호
- 레디스 호스트 이름
- 레디스 비밀번호
- 외부 서비스와 통신할 때 사용하는 시크릿 키(secret key)

시크릿(secret) 오브젝트는 민감한 암호 같은 정보를 저장힐 때 사용한다. 인코딩 된 값이 저장된다는 것을 제외하면 컨피그 맵과 동일하다. 시크릿 오브젝트는 명령형 방법과 선언형 방법으로 생성할 수 있다.

- 명령형 방법
    - 리터럴하게 설정 값을 선언할 수 있다.
    - .properties 파일을 통해 설정 값을 선언할 수 있다.

```
$ kubectl create secret generic\
    app-secret --from-literal=DB_HOST=mysql\
               --from-literal=DB_USER=root\
               --from-literal=DB_PASSWORD=password

$ kubectl create secret generic\
    app-secret --from-file=<path-to-file>
```

- 선언형 방법
    - 시크릿 오브젝트에 대해 정의된 yml 파일을 사용한다.

```
$ kubeclt create -f secret-defintion.yml
```

파일 형식은 다음과 같다.

```yml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_HOST: mysql
  DB_USER: root
  DB_PASSWORD: password
```

데이터의 키 부분이 플레인 텍스트(plain text)라면 시크릿 오브젝트를 사용할 의미가 없다. 이 부분을 Base64 형식으로 인코딩할 필요가 있다. 에코(echo) 명령어를 통해 인코딩한다. 

```
$ echo -n 'mysql' | base64
bXlzcWw=

$ echo -n 'root' | base64
cm9vdA==

$ echo -n 'password' | base64
cGFzc3dvcmQ=
```

인코딩 된 시크릿 정의 파일은 다음과 같다.

```yml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_HOST: bXlzcWw=
  DB_USER: cm9vdA==
  DB_PASSWORD: cGFzc3dvcmQ=
```

명령어를 통해 시크릿을 배포한다.

```
$ kubectl create -f ./examples/chapter-049/secret-definition.yml

secret/app-secret created
```

시크릿 정보를 확인하면 데이터가 숨겨진 것을 볼 수 있다.

```
$ kubectl get secret

NAME         TYPE     DATA   AGE
app-secret   Opaque   3      6s
```

```
$ kubectl describe secret app-secret

Name:         app-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
DB_HOST:      5 bytes
DB_PASSWORD:  8 bytes
DB_USER:      4 bytes
```

`-o yaml` 옵션으로 숨겨진 시크릿 정보를 확인할 수 있다.

```
$ kubectl get secret app-secret -o yaml

apiVersion: v1
data:
  DB_HOST: bXlzcWw=
  DB_PASSWORD: cGFzc3dvcmQ=
  DB_USER: cm9vdA==
kind: Secret
metadata:
  creationTimestamp: "2023-12-28T04:23:53Z"
  name: app-secret
  namespace: default
  resourceVersion: "507679"
  uid: 8109d6ab-d112-4c3b-b3ed-ae7962e5fb25
type: Opaque
```

파드에서 다음과 같은 방법으로 시크릿을 사용할 수 있다.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-app
  labels:
    name: simple-app
spec:
 ccontainers: 
   - name: simple-web-app
     image: simple-web-app
     ports:
       - containerPort: 8080
     envFrom:
       - secretRef:
           name: app-secret
```

시크릿을 주입 받는 방법은 다양하다. 시크릿을 통째로 주입 받을 수 있다.

```yml
envFrom:
  - secretRef:
      name: app-config
```

시크릿 중 필요한 값만 주입 받을 수 있다.

```yml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_PASSWORD
```

볼륨으로부터 주입 받을 수 있다.

```yml
volumes:
  - name: app-secret-volume
    secret:
      secretName: app-sec
```

볼륨을 사용하는 경우 각 키-값 엔트리는 각자 파일로 정의된다. 컨테이너 내부 `/opt/<volume-name>` 경로에는 시크릿들이 파일로 선언된 것을 볼 수 있다. 시크릿 데이터가 3개인 경우 파일이 3개 생성된다.

### 주의사항

시크릿 오브젝트는 암호화(encryption)를 수행하는게 아니다. Base64 인코딩을 수행하기 때문에 누구나 다 값을 디코드해서 볼 수 있다. 시크릿 정의 파일은 Git, SCM 같은 저장소에 올리지 않도록 주의해야 한다. 시크릿은 ETCD 내부에 암호화되지 않고 평문으로 저장된다. ETCD에 접근할 수 있는 사용자는 모두 비밀번호를 알 수 있다. ETCD에 암호화하여 저장할 수 있는 방법이 있다. 아래 링크를 참조한다.

- <https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/>

동일한 네임스페이스에 파드나 디플로이먼트(deployment)를 생성하면 시크릿 오브젝트에 접근할 수 있다. AWS, Azure, GCP 같은 클라우드 제공자가 지원하는 서드-파티(3rd-party) 시크릿 저장소를 사용하거나 Vault 같은 암호화 플랫폼을 사용하는 것도 좋은 방법이다.
