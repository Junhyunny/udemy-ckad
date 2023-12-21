
## Chapter 29. Certification Tip: Formatting Output with kubectl

`kubectl` 명령어를 사용해 쿠버네티스 오브젝트의 값을 특정 포맷으로 변경할 수 있다. 예를 들어 다음과 같이 json 형태 데이터로 리소스 정보를 얻을 수 있다.

```
$ kubectl create namespace test-namespace --dry-run=client -o json
{
    "kind": "Namespace",
    "apiVersion": "v1",
    "metadata": {
        "name": "test-namespace",
        "creationTimestamp": null
    },
    "spec": {},
    "status": {}
}
```

--dry-run 옵션을 사용해 오브젝트를 실제로 생성하진 않았다. K8S 리소스 정의를 파일로 해야할 때 기본 템플릿을 만드는 용도로 사용하면 도움이 될 것이다. `name` 옵션은 단순 이름만 출력한다.

```
$ kubectl create namespace test-namespace --dry-run=client -o name
namespace/test-namespace
```

yml 파일로 만든다.

```
$ kubectl create namespace test-namespace --dry-run=client -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: test-namespace
spec: {}
status: {}
```

`-o` 옵션을 통해 출력할 수 있는 포멧은 다음과 같다.

- go-template
- go-template-file
- json
- jsonpath
- jsonpath-as-json
- jsonpath-file
- name
- template
- templatefile
- yaml
