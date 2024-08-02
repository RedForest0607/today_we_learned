쿠버네티스 테스트를 위한 환경을 구축하다가 다음과 같은 에러가 등장했습니다.

``` sh
[root@k8s-worker02 ~]# kubectl get nodes
E0624 14:39:51.228730     688 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp [::1]:8080: connect: connection refused
E0624 14:39:51.230027     688 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp [::1]:8080: connect: connection refused
E0624 14:39:51.231077     688 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp [::1]:8080: connect: connection refused
E0624 14:39:51.232011     688 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp [::1]:8080: connect: connection refused
E0624 14:39:51.234365     688 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp [::1]:8080: connect: connection refused
```

워커 노드 환경에서 모든 노드들을 가져오는 명령어를 입력 했을 때 API 그룹 리스트를 가져올 수 없다 - connection refused 에러가 나타났습니다.  
이런 문제는 여러가지 요인이 있을 수 있지만 워커 노드의 `config`가 제대로 설정되어 있지 않을 수 있습니다.

```kubectl config view```
다음과 같은 명령어를 사용했을 때,  

``` sh
apiVersion: v1
clusters: []
contexts: []
current-context: ""
kind: Config
preferences: {}
users: []
```

다음과 같은 값이 나온다면 node의 config를 설정하지 않은 경우이다, 이런 경우 config를 잡아주어서, 워커 노드를 연결해줘야 한다.  
쿠버네티스의 클러스터링을 관리하는 툴인 `kubeadm`명령어를 통해서 설정을 잡아줘야한다.

``` sh
[root@k8s-master ~]# kubeadm reset //리셋이 필요한 경우
[root@k8s-master ~]# kubeadm token create --print-join-command
```

다음과 같은 명령어로 쿠버네티스 클러스터링을 초기화 하고 워커노드와의 연결을 위한 토큰을 생성한다.

``` sh
[root@k8s-master ~]# kubeadm token create --print-join-command
kubeadm join IP주소:포트 --token 토큰.값 --discovery-token-ca-cert-hash sha256:해시값
```

그리고 이렇게 나온 명령어 한 줄을 worker노드 측에다 입력해주면 된다.

``` sh
[root@k8s-worker01 ~]# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: 메인 서버주소
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```

다음과 같이 설정 결과가 나오면 완료!!
