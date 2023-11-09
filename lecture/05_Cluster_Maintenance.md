# 05.Cluster Maintenance

Cluster Maintenance: 클러스터 유지관리

### 1. Operating System Upgrade

시나리오

- 클러스터의 일부 노드가 갑자기 죽는 상황
- 노드가 즉시 다시 복구되면 안의 파드도 활성화 됨.
- 하지만 노드가 5분 이상 복구되지 않을 경우 그 안에 파드도 종료됨.
    - 옵션: —pod-eviction-timeout (default: 5분)
- 5분이 지나서 노드가 복구되는 경우 안에 파드는 모두 사라진채로 복구됨.

시스템 업그레이드 도중 특정 노드를 내릴 일이 있다면 해당 노드의 작업들을 다른 노드에서 실행 가능하도록 운영. 심리스하게 노드를 내릴 수 있음.

- 특정 노드 내리는 명령어: kubectl drain node-1
- 노드가 다시 복구되었을 때 파드가 다시 바인딩 되도록 준비하는 명령어: kubectl uncordon node-1
- 특정 노드를 내리지는 않으나 이후부터는 새로운 파드가 바인딩 되지 않도록 하는 명령어:
kubectl cordon node-2

### **2.Cluster Upgrade Process**

- kubeadm: 쿠버네티스 업그레이드 명령어

kubeadm upgrade plan

- 현재 쿠버네티스 버전, 쿠버네티스 최신 버전 등의 정보 조회

kubeadm upgrade apply v1.12.0 

- 쿠버네티스 클러스터 업그레이드

apt-get upgrade -y kubelet=1.12.0-00

- 큐블렛 업그레이드

**작업자 노드 업그레이드**

1. 기존 노드를 내림
    1. kubectl drain node-1
2. kubeadm 설치
    1. apt-get upgrade -y kubeadm=1.12.0-00
3. kubelet 설치 
    1. apt-get upgrade -y kubelet=1.12.0-00
4. kubelet으로 노드 업그레이드
    1. kubeadm upgrade node config —kubelet-version v1.12.0
5. kubelet 서비스 재시작
    1. systemctl restart kubelet
6. 노드가 다시 복구되었을 때 파드가 다시 바인딩 되도록 준비
    1. kubectl uncordon node-1

### 3.백업 및 복구

쿠버네티스에서 백업해놔야 할 자원들: 자원 설정들(리소스 설정 xml 파일 등), ETCD, 영구적 저장소인 응용 프로그램

- 자원 설정 파일들
    - 디렉토리 내에 관리해도 되지만 kube-apiserver에서 설정 파일들을 조회해서 복사하여 사용하는 방법도 있음. (kubectl 명령어 사용)
    - ex) 네임스페이스에 있는 모든 자원 정보를 추출해서 yml 파일에 복붙
          kubectl get all —all-namespace -o yaml > all-deploy-services.yml
- ECTD 스토리지
    - ECTD 스토리지는 마스터 노드에 바인딩됨.
    - 스토리지의 스냅샷을 찍어놓을 수 있음.
        - 스냅샷 떠놓는 명령어: ETCDCTL_API=3 etcdctl snapshot save snapshot.db
        - 스냅샷의 상태 조회 명령어: ETCDCTL_API=3 etcdctl snapshot status snapshot.db
    - ECTD 복원 절차
        1. ECTD 복원: ETCDCTL_API=3 etcdctl snapshot restore snapshot.db —data-dir <복원하는 데이터가 저장될 경로>
        2. 데몬 재시작: systemctl daemon-reload
        3. ECTD 서비스 재시작: service etcd restart 

### [Practice Test]

### 1. [**OS Upgrade**](https://kodekloud.com/topic/practice-test-os-upgrades-2/?utm_source=udemy&utm_medium=labs&utm_campaign=kubernetes)

1. Try draining the node again using the same command as before: kubectl drain node01 —ignore-daemonsets
    1. 해석: kubectl drain node01 —ignore-daemonsets 명령을 실행해보는 문제
    2. 풀이:
        1. 위 명령어를 실행하면 다음과 같은 에러 메시지가 나온다. 
        cannot delete pods not managed by replicationController, ReplicaSet, Job, DaemonSet or StatefulSet
        2. 복제셋이 없는 단일한 파드는 삭제할 수 없다는 뜻
2. Why did the drain command fail on node01? It worked the first time!
    1. node01 노드를 내리는 것이 불가능한 이유
    2. 11번의 해석대로 노드내에 복제셋이 없는 단일한 파드가 존재하기 때문

### 2.Cluster Upgrade

1. How many nodes can host workloads in this cluster? Inspect the applications and taints set on the nodes.
    1. 이 클러스터에서 워크로드를 호스팅할 수 있는 노드는 몇 개입니까? 노드에 설정된 애플리케이션 및 오염을 검사합니다.

<aside>
💡 풀이

- 앱을 호스트할 수 있는지 오염 여부를 확인해보는 문제
- 오염 여부 확인 명령어
    - k describe node | grep Taints
- <none>으로 뜨면 아무 오염도 없는 노드이므로 답은 2개
</aside>

### 3. [Backup & Restore](https://kodekloud.com/topic/practice-test-backup-and-restore-methods-2/)

1. What is the version of ETCD running on the cluster? Check the ETCD Pod or Process

> 실행 중인 ETCD의 버전을 묻는 문제. ETCD 파드를 확인해야함.
> 

<aside>
💡 풀이

- 파드를 확인해 보면됨.
- 모든 namespace에 있는 파드들을 확인
    - k get pod -A
    - 파드 이름 중에 etcd가 포함된 파드가 있으면 ETCD 파드임
    (kube-system etcd-controlplane 1/1 Running 0 5m46s)
- 해당 파드의 description을 확인
    - k describe pod etcd-controlplane -n kube-system
- description에서 파드내 컨테이너 이미지 버전을 확인
</aside>

1. At what address can you reach the ETCD cluster from the controlplane node? Check the ETCD Service configuration in the ETCD POD

> controlplane 노드로부터 ETCD로 접속할 수 있는 주소를 묻는 문제
> 

<aside>
💡 풀이

- —listen-client-urls: 클라이언트로부터 ECTD에 접근할 수 있는 주소
- describe명령어로 위 옵션을 확인
</aside>

1. The master node in our cluster is planned for a regular maintenance reboot tonight. While we do not anticipate anything to go wrong, we are required to take the necessary backups. Take a snapshot of the **ETCD** database using the built-in **snapshot** functionality. Store the backup file at location /opt/snapshot-pre-boot.db

> ETCD 스토리지 데이터를 백업 시켜놓는 문제
> 

<aside>
💡 풀이

- etcdctl snapshot 명령어를 써야하므로 ETCDCTL_API=3를 환경변수에 설정해 놓는다.
    - export ETCDCTL_API=3
- 원본 etcd 설정 파일을 찾는다.
    - etcd 설정 파일과 같이 쿠버네티스 구성 요소들에 관한 파일을 관리하는 경로
        - /etc/kubernetes/manifests/
        - /etc/kubernetes/manifests/ 에 etcd.yml 파일이 있음.
- 백업할 때 필요한 정보들
    1. 클라이언트로부터 접속 가능한 주소: —listen-client-urls
    2. ca 인증서 정보(cacert): —trusted-ca-file
    3. 인증서 정보(cert): —cert-file
    4. 보안 키 정보: —key-file
- 백업 명령어 실행
    
    ```json
    etcdctl snapshot save --endpoints=127.0.0.1:2379 \
    > --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    > --cert=/etc/kubernetes/pki/etcd/server.crt \
    > --key=/etc/kubernetes/pki/etcd/server.key \
    > /opt/snapshot-pre-boot.db
    ```
    
</aside>

1. Wake up! We have a conference call! After the reboot the master nodes came back online, but none of our applications are accessible. Check the status of the applications on the cluster. What's wrong?

> 클러스터상의 애플리케이션이 실행되지 않는데 무슨 문제가 있는건지 찾아내는 문제
> 

<aside>
💡 풀이

- deployment, service, pod 정보들을 모두 조회해본다.
- 위 문제에서는 3가지 다 없다고 나오는 상태
- 클러스터내 모든 구성 요소에 문제가 있으므로 정답은 “All of the above”
</aside>

1. Luckily we took a backup. Restore the original state of the cluster using the backup file.

> 백업 파일로 클러스터 원래 상태로 복구시키는 문제
> 

<aside>
💡 풀이

</aside>

<복원 순서>

1. 백업 파일을 이용해서 클러스터 상태를 복원하는 명령어
    - etcdctl snapshot restore —data-dir <복원 데이터가 저장될 경로> <백업파일>
    - etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/snapshot-pre-boot.db
2. /var/lib/etcd-from-backup 로 이동하면 member라는 폴더 생성되어 있음. (복원 파일 생성)
3. etcd.yml 파일에 데이터 저장 경로를 기존 → 복원한 데이터 경로로 변경해줌
    1. etcd.yml 파일의 volumes.hostPath.path 변경
        
        ```json
        volumes:
          - hostPath:
              path: /etc/kubernetes/pki/etcd
              type: DirectoryOrCreate
            name: etcd-certs
          - hostPath:
              path: /var/lib/etcd-from-backup // 변경
              type: DirectoryOrCreate
        ```