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
