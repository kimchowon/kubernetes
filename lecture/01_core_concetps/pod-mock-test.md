# 모의 테스트(1) - Pod

### 모의테스트 링크

- https://kodekloud.com/topic/practice-test-pods-2/

1. How many `pods` exist on the system?

클러스터의 현재 네임스페이스에 존재하는 파드의 개수를 구하라

- 다음 명령어로 파드 조회
    - kubectl get pods
- 쿠버네티스의 네임스페이스의 디폴트명은 “default”임
    - kubectl get pods 로 검색하면 default 네임스페이스내의 파드들이 조회
    - 만약 네임스페이스가 문제에서 주어지면?
        - -n <namespce> 옵션을 사용
        - kubectl get pods -n chocho

1. Create a new pod with the `nginx` image.
    
    파드를 생성하라
    
- 파드 생성 명령어
    - kubectl run <파드이름> —image=<이미지명>
    - ex) kubectl run nginx —image=nginx

1. How many pods are created now?
현재 몇개의 파드가 생성되어 있는지 확인하라
- 다음 명령어로 파드 개수 확인
    - kubectl get pods

1. What is the image used to create the new pods?
new pods라는 파드를 만드는데 어떤 이미지를 사용했는지 확인하라
- 파드를 만드는데 어떤 이미지를 사용했는지 확인하려면 describe 명령어를 사용해야함
    - ex) kubectl describe pod new-pods
- 결과 예시
    
    ```bash
    Name:             newpods-qzhck
    Namespace:        default
    Priority:         0
    Service Account:  default
    Node:             controlplane/192.28.211.6
    Start Time:       Tue, 17 Oct 2023 14:31:35 +0000
    Labels:           tier=busybox
    Annotations:      <none>
    Status:           Running
    IP:               10.42.0.11
    IPs:
      IP:           10.42.0.11
    Controlled By:  ReplicaSet/newpods
    Containers:
      busybox:
        Container ID:  containerd://1727809f825b1f0598bcdbf80e18d162383dbc6e5a8c0a90518292c34d7bca00
        Image:         busybox # 이미지 확인!
        Image ID:      docker.io/library/busybox@sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79
        Port:          <none>
        Host Port:     <none>
        Command:
          sleep
          1000
        State:          Running
          Started:      Tue, 17 Oct 2023 14:31:39 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2q4m8 (ro)
    Conditions:
      Type              Status
      Initialized       True 
      Ready             True 
      ContainersReady   True 
      PodScheduled      True 
    Volumes:
      kube-api-access-2q4m8:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
    QoS Class:                   BestEffort
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
      Type    Reason     Age    From               Message
      ----    ------     ----   ----               -------
      Normal  Scheduled  4m46s  default-scheduler  Successfully assigned default/newpods-qzhck to controlplane
      Normal  Pulling    4m44s  kubelet            Pulling image "busybox"
      Normal  Pulled     4m44s  kubelet            Successfully pulled image "busybox" in 416.949361ms (416.962628ms including waiting)
      Normal  Created    4m44s  kubelet            Created container busybox
      Normal  Started    4m43s  kubelet            Started container busybox
    ```
    

1. Which nodes are these pods placed on?
파드들이 어떤 노드에 위치해있는가
- 전체 파드들의 노드를 확인하는 방법
    - kubectl get pods -o wide
    - Node 열의 “controlplane”이 현재 파드들이 위치한 노드
        
        ```bash
        NAME            READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
        newpods-9w46v   1/1     Running   0          8m9s    10.42.0.10   controlplane   <none>           <none>
        newpods-6fkq5   1/1     Running   0          8m9s    10.42.0.12   controlplane   <none>           <none>
        nginx           1/1     Running   0          8m14s   10.42.0.9    controlplane   <none>           <none>
        newpods-qzhck   1/1     Running   0          8m9s    10.42.0.11   controlplane   <none>           <none>
        ```
        
- 파드 개별로 노드를 확인하는 방법
    - kubectl describe pod <파드명>

1. How many containers are part of the pod `webapp`
    
    webapp라는 이름을 가진 파드안에 몇개이 컨테이너가 들어있는가
    
- 다음 명령어 실행
    - kubectl get pods -o wide
    - webapp이라는 파드에는 READY가 1/2라고 나와있음
    - 2개의 컨테이너가 있고 그 중 한개이 건테이너가 실행 중이라는 의미
        
        ```bash
        NAME            READY   STATUS             RESTARTS      AGE     IP           NODE           NOMINATED NODE   READINESS GATES
        nginx           1/1     Running            0             17m     10.42.0.9    controlplane   <none>           <none>
        newpods-6fkq5   1/1     Running            1 (52s ago)   17m     10.42.0.12   controlplane   <none>           <none>
        newpods-9w46v   1/1     Running            1 (52s ago)   17m     10.42.0.10   controlplane   <none>           <none>
        newpods-qzhck   1/1     Running            1 (52s ago)   17m     10.42.0.11   controlplane   <none>           <none>
        webapp          1/2     ImagePullBackOff   0             2m24s   10.42.0.13   controlplane   <none>           <none>
        ```
        

1. What images are used in the new webapp pod?
webapp 파드 생성에 사용된 이미지들을 확인하라
- 동일하게 다음 명령어로 확인
    - kubectl describe pod webapp

1. What is the state of the container `agentx` in the pod `webapp`?
`agentx` 컨테이너의 상태를 확인하라
- kubectl describe pod webapp 실행
- agentx 컨테이너의 states를 확인한다.

1. Why do you think the container `agentx` in pod `webapp` is in error?
왜 agentx 이미지가 에러인가.
- kubectl describe pod webapp을 실행하면 agentx 컨테이너 설명 아래에 에러 메시지가 나온다. 
에러메시지를 잘 읽고 선택하면 될 듯
- ex) 레파지토리가 존재하지 않거나 권한이 없다는 내용이다. 
답안지에 해당 이름의 도커이미지가 없다는 선택지가 답이다. 
**A Docker image with this name doesn't exist on Docker Hub**
    
    ```bash
    Warning  Failed     6m27s (x4 over 7m45s)   kubelet  
    Failed to pull image "agentx": rpc error: code = U
    nknown desc = failed to pull and unpack 
    image "docker.io/library/agentx:latest": 
    failed to resolve reference "docker.io/library/agentx:latest": p
    ull access denied, repository does not exist or may require authorization: 
    server message: insufficient_scope: authorization failed
    ```
    
1. What does the `READY` column in the output of the `kubectl get pods` command indicate?
kubectl get pods 명령어를 쳤을 때 READY 컬럼이 나타내는 의미는 무엇인가?
- 현재 실행중인 컨테이너 수 / 파드내의 전체 컨테이너 수
- **Running Containers in POD/Total Containers in POD**

1. Delete the `webapp` Pod.
webapp 파드를 제거하라
- kubectl delete pod webapp

1. Create a new pod with the name `redis` and the image `redis123`.
yaml 파일을 이용한 파드 생성 문제
- yaml 파일을 직접 작성하지 않고 복사하는 방법
    - kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
    - kubectl create -f redis.yaml

1. Now change the image on this pod to `redis`.
파드에 정의된 컨테이너의 이미지를 수정하라 
- 파드의 이미지 수정 방법
    - yaml 수정
        - yaml 파일을 수정하고 kubectl apply -f redis.yaml 명령어를 실행한다.