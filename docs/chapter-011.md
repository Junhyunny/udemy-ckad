
## Chpater 11. Docker-vs-ContainerD

쿠버네티스는 초반에 시장을 지배하고 있는 컨테이너 환경인 도커(docker)만 지원했다. 이후 시장에서 쿠버네티스가 인기가 많아지면서 다른 컨테이너 런타임(runtime)을 지원하기 시작했다. 이때 다양한 컨테이너 런타임을 지원하기 위해 만들어진 것이 CRI(container runtime interface)이다. 

CRI는 OCI(open container initiative) 표준을 준수하는 컨테이너 런타임을 지원한다. OCI는 다음과 같이 구상되어 있다. 

- imagespec
    - 이미지가 어떻게 구축되어야 하는지 사양들을 정의
- runtimespec
    - 컨테이너 런타임 개발에 대한 표준을 정의

로켓 같이 OCI 표준을 준수한 컨테이너 런타임은 CRI를 통해 쿠버네티스 환경에서 동작할 수 있었다. 반대로 도커는 CRI가 나오기 전에 만들어진 기술이라 OCI 표준을 지키지 못 했지만, 여전히 중요한 역할을 수행하는 컨테이너 도구였다. 쿠버네티스는 `dockershim`을 도입하는 등 임시 방편으로 계속 도커를 지원할 수 밖에 없었다. 

`containerd`는 나중에 나온 도커 컨테이너 런타임이다. CRI 호환이 가능하고, 다른 런타임처럼 쿠버네티스와 직접적으로 작업할 수 있다. containerd는 도커와 별도로 자체 런타임으로 사용될 수 있다. 쿠버네티스는 도커 엔진을 계속 지원했지만, dockershim을 계속 지원하는 불필요한 수고를 줄이기 위해 쿠버네티스 1.24 버전에서 dockershim을 삭제하면서 도커 지원을 해제했다. 이전에 도커를 통해 만들어진 컨테이너들은 OCI 스펙을 준수하기 때문에 쿠버네티스 환경에서 동작하지만, 쿠버네티스 지원 런타임에서 도커가 제외된 것 뿐이다. 

containerd는 CNCF 회원 자격으로 졸업헀다. 도커를 설치할 필요 없이 컨테이너를 자체 설치할 수 있다. 도커가 있을 때는 docker 명령어를 사용하지만, containerd에서는 `ctr` CLI 명령어를 사용한다. 사용자 친화적인 명령어가 아니기 때문에 `nerdctl` 같은 명령어를 사용하는 것을 추천한다. 

`crictl`은 containerd, 로켓 같은 CRI 호환되는 컨테이너 런타임들과 상호 작용하는데 사용되는 명령어 도구이다. kubelet과 함께 작동한다. crictl 도구는 쿠버네티스 1.24 버전 이전에는 다음과 같은 소켓-포인트를 사용했다.

- unix:///var/run/dockershim.sock
- unix:///run/containerd/containerd.sock
- unix:///run/crio/crio.sock
- unix:///var/run/cri-dockerd.sock

쿠버네티스 1.24 버전부터는 다음과 같은 소켓-포인트를 사용한다.

- unix:///run/containerd/containerd.sock
- unix:///run/crio/crio.sock
- unix:///var/run/cri-dockerd.sock

### Summary

ctr 명령어는 containerd와 함께 제공되고 동작한다. 디버깅 목적으로만 사용되고 기능은 제한적이다. 실제로 거의 사용되지 않는다. 

nerdctl 명령어는 컨테이너 커뮤니티에서 등장했다. 컨테이너를 위한 docker 같은 CLI 명령어이다. 컨테이너를 만드는 데 일반적인 목적으로 사용되고 도커 CLI 명령어와 같거나 그 이상의 기능을 지원한다. 

crictl 명령어는 쿠버네티스 커뮤니티에서 등장했다. CRI 호환 가능한 모든 컨테이너 런타임과 상호 작용하는 데 사용된다. 주로 디버깅 목적으로 사용한다. 