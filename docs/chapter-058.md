
## Chapter 58. Service Account

서비스 계정(service account)는 보안과 관련된 개념이다. 인증, 권한 부여, 역할 기반 액세스 컨트롤 등과 관련 있다. 쿠버네티스 계정에는 두 가지 유형이 있다. 

- 사용자 계정
  - 일반 사용자들이 사용한다.
  - 클러스터에 접근하는 관리자 혹은 응용 프로그램이 될 수 있다.
  - 배포를 위해 클러스터에 접근하는 개발자가 될 수 있다.
- 서비스 계정
  - 컴퓨터가 사용한다.
  - 쿠버네티스 클러스터와 상효 작용하기 위한 애플리케이션일 수 있다.
  - 프로메테우스 같은 모니터링 앱이 서비스 게정을 이용해 쿠버네티스 API 퍼포먼스를 메트릭으로 가져오기 위해 서비스 계정을 사용한다.
  - 젠킨스 같은 자동화 빌드 툴은 서비스 계정을 사용해 클러스터에 앱을 배포한다.

응용 프로그램을 구축해 쿠버네티스 API에게 질의(query)하려면 사용자 인증이 필요하다. 이때 서비스 계정을 사용한다. 다음과 같은 명령어를 통해 서비스 계정 오브젝트를 생성할 수 있다.

```
$ kubectl create serviceaccount dashboard-sa
```

쿠버네티스 클러스터에서 서비스 계정 정보를 확인하려면 다음 명령어를 실행한다.

```
$ kubectl get serviceaccount
```

서비스 계정이 생기면 자동으로 토큰을 생성할 수 있다. 서비스 계정 토큰은 쿠버네티스 API에 인증하는 동안 외부 앱이 사용해야 하는 토큰이다. 토큰은 시크릿 오브젝트로 저장된다. 다음과 같은 과정을 통해 토큰이 발급된다.

1. 서비스 계정이 생성되면 서비스 계정 오브젝트를 생성한다. 
2. 서비스 계정을 위한 토큰을 생성한다. 
3. 시크릿 오브젝트를 생성하고 토큰을 시크릿 오브젝트에 저장한다.
4. 시크릿 오브젝트가 서비스 계정 오브젝트에 링크된다.

cURL 명령어로 쿠버네티스 API에 데이터를 요청할 때 생성한 토큰을 사용할 수 있다. 생성한 토큰은 Authorization 헤더에 Bearer 토큰 타입과 함께 전달하면 원하는 데이터를 얻을 수 있다. 서비스 계정을 만들고 역활 기반 접근 제어 메커니즘을 이용해 올바른 권한을 할당할 수 있다.

쿠버네티스 클러스터에 서드-파티(3rd-party) 애플리케이션이 함께 배포되어 있는 경우 볼륨 마운트를 통해 이를 사용할 수 있다. 파드 안에 애플리케이션 컨테이너가 쉽게 토큰을 읽을 수 있도록 설계하였다. 수동으로 제공할 필요가 없다. 

서비스 계정을 살펴보면 기본(default) 서비스 계정 오브젝트가 있는 것을 확인할 수 있다. 모든 네임스페이스는 디폴트 값으로 명명된 서비스 계정이 자동으로 생성되어 있다. 네임스페이스마다 서비스 계정 오브젝트가 존재한다. 파드가 생성될 때마다 기본으로 서비스 계정과 토큰이 볼륨으로 마운트 된다.

예를 들어 다음과 같은 기본 파드를 정의하더라도 describe 정보를 보면 디폴트 토큰 정보가 볼륨으로 지정되어 있는 것을 볼 수 있다.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
```

쿠버네티스 시크릿 오브젝트에 서비스 어카운트 디렉토리를 확인하면 세 개의 파일이 마운트 된 것을 볼 수 있다.

```
$ kubectl exec -it my-kubernetes-dashboard --ls /var/run/secrets/kubernetes.io/serviceaccount
```

기본 서비스 계정은 제한되어 있다. 기본 쿠버네티스 API 쿼리를 실행하는 권한만 가지고 있다. 다른 서비스 계정을 사용하고 싶다면 파드에 새로운 서비스 계정 오브젝트를 지정한다. 기존 파드에 서비스 계정을 수정할 수 없다. 반드시 삭제하고 다시 만들어야 한다. 

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  serviceAccountName: dashboard-sa
```

디플로이먼트(deployment)의 경우 서비스 계정을 변경할 수 있다. 파드 정의 파일에 변화가 생기면 롤 아웃을 통해 새로운 파드를 배포하기 때문이다. 아무런 정보를 입력하지 않으면 디폴트 서비스 계정 오브젝트를 사용하기 때문에 다음과 같은 명령어를 통해 이를 금지할 수 있다.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  automountServiceAccountToken: false
```

