
## Chpater 46. ConfigMap

 쿠버네티스 파드가 환경 변수를 사용할 수 있도록 yml 파일에 정의하지만, 이를 컨피그 맵(ConfigMap) 오브젝트로 구성해 주입 받을 수 있다. 컨피그 맵 오브젝트를 사용하는 방법은 단순하다. 

 1. 컨피그 맵 만든다.
 1. 파드(Pod) 내부으로 값을 주입한다.

 컨피그 맵을 생성하는 방법도 두 가지 존재한다.

 1. 파일을 정의하지 않고 명령적(imperitive)으로 사용하는 방법 (imperative way)
 1. 파일로 선언한 컨피그 맵 설정을 사용하는 방법 (declarative way)

### Imperative Way

명령적으로 선언하는 방법을 먼저 알아본다. 환경 변수 값을 리터럴(literal)하게 나열하는 방법이다.

```
$ kubectl create configmap\
    <config-name> --from-literal=<key>=<value>
```

예를 들어 다음과 같이 사용할 수 있다. 항목 값이 많아질수록 복잡해지는 경향이 있다. 

```
$ kubectl create configmap\
    app-config --from-literal=APP_COLOR=blue\
               --from-literal=APP_MODE=prod
```

다음으로 파일로 선언하는 방법이다. 환경 변수를 `.properties` 파일에 작성하고 파일 경로를 지정한다.

```
$ kubectl create configmap\
    <config-name> --from-file=<path-to-file>
```

예를 들어 다음과 같이 사용할 수 있다. 

```
$ kubectl create configmap\
    app-config --from-file=app_config.properties
```

### Declarative Way

다음과 같은 yml 정의 파일을 만든다.

```yml
apiVersion: v1
kind: ConfigMap
metadata:
    name: app-config
data:
    APP_COLOR: blue
    APP_MODE: prod
```

### View ConfigMaps

생성한 컨피그 맵 정보를 볼 수 있는 방법은 다음과 같다.

```
$ kubectl get configmaps

NAME               DATA   AGE
app-config         2      7s
kube-root-ca.crt   1      7d15h
```

```
$ kubectl describe configmaps app-config
Name:         app-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
APP_COLOR:
----
blue
APP_MODE:
----
prod

BinaryData
====

Events:  <none>
```

### Inject Values from ConfigMap

컨피그 맵으로부터 값을 주입 받는 방법은 다음과 같다.

- 컨피그 맵에 선언된 모든 변수를 사용한다.
- 컨피그 맵에 선언된 특정 값만 사용한다.

컨피그 맵에 선언된 모든 변수를 사용할 땐 `envFrom` 항목을 사용한다.

```yml
# ...
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```

특정 값만 사용하고 싶은 경우 `valueFrom` 항목을 사용한다.

```yml
# ...
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          valueFrom: 
            configMapKeyRef:
              name: app-config
              key: APP_COLOR
```

볼륨을 사용해 컨피그 맵을 마운트 하는 방법도 있다.

```yml
# ...
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```
