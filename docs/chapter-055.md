 
 ## Chapter 55. Security Contexts

다음은 컨테이너 보안에 관련된 사용자 변경, 권한 추가, 삭제 방법들이다.

- 호스트 변경

```
$ docker run --user=1001 ubuntu sleep 3600
```

- 권한 추가 

```
$ docker run --cap-add MAC_ADMIN ubuntu
```

- 권한 삭제

```
$ docker run --cap-drop KILL ubuntu
```

쿠버네티스도 동일한 작업이 가능하다. 

