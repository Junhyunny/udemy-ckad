
## Chapter 38. Define, build and modify container images

컨테이너를 만드는 과정을 이해하기 위해 애플리케이션을 수동으로 배포하는 과정을 먼저 살펴보자. 파이썬 웹 애플리케이션을 예로 들어본다.

1. Ubuntu 같은 OS 설치 
1. apt 저장소 업데이트
1. apt 의존성 설치
1. pip 명령어를 사용한 파이선 의존성 설치
1. 소스 코드를 /opt 폴더로 복사
1. flask 명령어를 통해 웹 애플리케이션 실행

도커 파일(dockerfile)에도 비슷한 과정을 작성한다. 작성 완료한 도커 파일을 기준으로 애플리케이션을 실행할 수 있는 컨테이너 이미지를 만들 수 있다. 

```
$ docker build Dockerfile -t junhyunny/my-app
```

생성된 이미지는 로컬에만 존재한다. 외부 저장소에 올리면 다른 호스트에서도 해당 이미지를 다운로드 받아서 사용할 수 있다. 다음과 같은 명령어를 실행한다.

```
$ docker push junhyunny/my-app
```

### Dockerfile

도커 파일은 도커(docker)가 이해할 수 있는 문법으로 작성된 파일이다. 명령어(instruction)과 인수(argument)로 구성된다. 다음과 같은 도커 파일이 존재할 때 명령어와 인수는 다음과 같이 구분할 수 있다.

- 명령어
    - FORM
    - RUN
    - COPY
    - ENTRYPOINT
- 인수
    - Ubuntu
    - apt-get update
    - apt-get install python
    - pip install flask
    - pip install flask-mysql
    - . /opt/source-code
    - FLASK_APP=/opt/source-code/app.py flask run  

```dockerfile
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

### Layered Architeture

도커 파일은 계층 아키텍처(layered architetrue)를 가진다. 각 명령문이 실행될 때마다 하나의 레이어가 생성된다. 다시 도커 파일을 기준으로 설명한다. 각 계층이 생길 때 발생하는 변화에 따라 이미지 용량이 기본 이미지에 추가된다.

1. 우분투 OS 레이어
    - 120MB
1. apt 패키지 업데이트 및 설치
    - 306MB
1. pip 의존성 설치
    - 6.3MB
1. 소스 코드 복사
    - 229B
1. flask 실행 명령어 업데이트
    - 0B

```dockerfile
FROM Ubuntu

RUN apt-get update && apt-get install python

RUN pip install flask && pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

레이어 아키텍처를 가졌을 때 가장 큰 장점은 캐싱이 가능하다는 점이다. 도커 이미지를 한 레이어씩 만드는 과정에서 중간에 실패했다고 가정해보자. 이전 레이어까진 정상적으로 성공했기 때문에 다음 빌드 때 성공한 이미지 레이어까진 재사용한다. 이를 레이어 캐시(layer cache)라고 한다.
