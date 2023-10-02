# kube-controller-manager

### 쿠버네티스 컨트롤러

- 지속적으로 시스템내의 다양한 구성 요소의 상태를 모니터링
- 전체 시스템을 원하는 상태로 유지하는 역할

### 컨트롤러 종류

1. 노드 컨트롤러
    1. 노드 상태 모니터링, 새 노드를 클러스터에 추가
    2. 노드 상태 모니터링 및 재배포 방법
        1. 노드 컨트롤러가 kube-apiserver를 통해 5초에 한 번씩 노드를 모니터링 함.
        (option: node-monitor-period: 5s)
        2. 만약 노드로부터 하트비트 신호가 중단되면 40초 동안 노드의 응답이 오지 않는 것을 허용하면서 기다림.
        (option: node-monitor-grace-period: 40s)
        3. 40초를 기다렸는데도 신호가 오지 않으면 노드의 상태를 unreachable(수신불가)로 표시
        4. 노드가 죽으면 노드 내의 파드를 종료하기 전에 5분동안 기다림.
         (option: pod-eviction-timeout: 5m)
        5. 5분이 지나면 해당 노드를 종료, 노드 내의 파드를 복제 노드에 재배포
2. 복제 컨트롤러
    1. 원하는 수 만큼의 복제본을 확보하고 관리
    2. 파드가 종료되면 다른 파드를 생성
3. 기타
    1. 노드 컨트롤러, 복제 컨트롤러 외에도 다양한 컨트롤러들이 쿠버네티스 안에 존재함. 

### 컨트롤러 위치

- Kube-Controller-Manager라는 하나의 프로세스에 패키지화 되어 있음.

### kube-controller-manager 설정 정보

(mac os 용 [minikube](https://www.morenice.kr/273) 설치하여 확인)

- kubectl -n kube-system describe pod kube-controller-manager-minikube
- configure
    
    ```bash
    Name:                 kube-controller-manager-minikube
    Namespace:            kube-system
    Priority:             2000001000
    Priority Class Name:  system-node-critical
    Node:                 minikube/192.168.59.100
    Start Time:           Mon, 02 Oct 2023 23:00:49 +0900
    Labels:               component=kube-controller-manager
                          tier=control-plane
    Annotations:          kubernetes.io/config.hash: 0db32c964cb33c04731572b16e7bf0af
                          kubernetes.io/config.mirror: 0db32c964cb33c04731572b16e7bf0af
                          kubernetes.io/config.seen: 2023-10-02T14:00:32.886522925Z
                          kubernetes.io/config.source: file
    Status:               Running
    SeccompProfile:       RuntimeDefault
    IP:                   192.168.59.100
    IPs:
      IP:           192.168.59.100
    Controlled By:  Node/minikube
    Containers:
      kube-controller-manager:
        Container ID:  docker://8dcbe65ad964a6738501465169ee8cda657d27d25d0b7e2918bc74b9f86478c7
        Image:         registry.k8s.io/kube-controller-manager:v1.25.2
        Image ID:      docker-pullable://registry.k8s.io/kube-controller-manager@sha256:f961aee35fd2e9a5ee057365e56c5bf40a39bfef91f785f312e51891db41876b
        Port:          <none>
        Host Port:     <none>
        Command:
          kube-controller-manager
          --allocate-node-cidrs=true
          --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
          --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
          --bind-address=127.0.0.1
          --client-ca-file=/var/lib/minikube/certs/ca.crt
          --cluster-cidr=10.244.0.0/16
          --cluster-name=mk
          --cluster-signing-cert-file=/var/lib/minikube/certs/ca.crt
          --cluster-signing-key-file=/var/lib/minikube/certs/ca.key
          --controllers=*,bootstrapsigner,tokencleaner
          --kubeconfig=/etc/kubernetes/controller-manager.conf
          --leader-elect=false
          --requestheader-client-ca-file=/var/lib/minikube/certs/front-proxy-ca.crt
          --root-ca-file=/var/lib/minikube/certs/ca.crt
          --service-account-private-key-file=/var/lib/minikube/certs/sa.key
          --service-cluster-ip-range=10.96.0.0/12
          --use-service-account-credentials=true
        State:          Running
    ```
    
- 설정에 위의 옵션들 추가 가능
    - node-monitor-period
    - option: node-monitor-grace-period
    - pod-eviction-timeout