# 02.scheduling

### 1. 스케줄러의 역할

- 모든 파드가 하나의 노드에는 바인딩이 되도록 노드가 할당되지 않은 파드를 찾아내고 노드에 할당해주는 역할
- 스케줄링에 의해 파드를 순회하고, 노드가 할당되지 않는 파드에 적정한 노드를 바인딩함

**만약 이런 역할을 하는 스케줄러가 없는 경우**

- 파드 생성시 설정 파일에 노드를 직접 지정해주는 방법
    
    ```bash
    apiVersion: v1
    kind: Pod 
    metadata:
     name: nginx 
     labels:
      name: nginx
    spec:
     containers:
      - name: nginx
        image: nginx 
        ports:
         - containerPort: 8080 
     nodeName: node02 # 노드 지정
    ```
    
- 이미 생성된 파드에 노드를 지정하는 방법
    - 위 설정 파일 방법은 파드 신규 생성시에만 가능. 이미 생성된 파드를 이후에 노드 바인딩을 해주는 것은 불가능함.
    - 바인딩 개체를 생성하고 바인딩 API로 요청하는 방법으로 가능함.
        - Pod-bind-definition.yml
            
            ```bash
            apiVersion: v1
            kind: Binding
            metadata:
             name: nginx
            target:
             apiVersion: v1
             kind: Node
             name: node02 # 노드명 지정
            ```
            
        - 바인딩 API 호출
            
            ```bash
            curl --header "Content-Type:application/json" --request POST --data <json포맷> http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/
            ```
            
            해당 API의 —data 뒤에 바인딩 개체 생성 yml 파일의 내용을 JSON 형식으로 포맷핑하여 붙여준다. ([yml → JSON 변환 사이트](https://bfotool.com/ko/yaml-to-json))
            
            ```json
            '
            {
                "apiVersion": "v1",
                "kind": "Binding",
                "metadata": {
                    "name": "nginx"
                },
                "target": {
                    "apiVersion": "v1",
                    "kind": "Node",
                    "name": "node02"
                }
            }
            '
            ```
            

### 2. labels / Selector / Annotation

- 특정 라벨이 붙어있는 파드들만 조회하는 명령어
    - kubectl get pods —selector app=App1
- 자원 정의 yml 파일에 주석 다는 방법
    - metadata아래에 annotation 프로퍼티 사용
        
        ```json
        apiVersion: v1
        kind: ReplicaSet
        metadata:
         name: simple-webapp
         annotations:
          buildversion: 1.34
          auditor: chocho
        ```
        
    

### 3. 파드와 노드의 관계

- [taints(오점) & tolerations(관용)](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/taint-and-toleration/)
    - taints는 노드에, tolerations는 파드에 설정하여 파드가 허용한 조건 외에, 부적절한 노드에 할당되지 않도록 방지한다.
- 노드에 taints 설정하기
    - 명령어: kubectl taint nodes <node-name> key=value:taint-effect
    - taint-effect: taint로 인해 발생할 수 있는 이슈. 총 3가지가 있음.
        - NoSchedule: 스케줄러가 없어서 파드에 노드를 바인딩 할 수 없다.
        - PreferNoSchedule: 스케줄러가 있지만 파드를 노드에 바인딩하지 않는다.
        - NoExecute:
- taints & tolerations 적용 사례
    - 마스터 노드
        - 쿠버네티스 정책상 마스터 노드에는 어떤 서비스 응용 프로그램도 배포되지 않도록 해놓음. 즉, 어떤 파드도 바인딩 되지 않도록 함. (taint: NoSchedule)
        - 마스터 노드 taint 정책 확인
            - kubectl describe node kubemaster | grep Taint
        

### 4. Node Selector, Node Affinity

Node Selector

- 파드를 바인딩할 노드를 선택할 수 있는 옵션
- ex) 노드가 여러개인데 각자 사양이 다름. 큰 사양의 노드에만 바인딩 하고 싶을 때 nodeSelector 옵션을 yml 파일에 세팅해주면 된다.
- 세팅 방법
    1. 바인딩 하고 싶은 노드들에 라벨을 붙인다. 
        1. kubectl label nodes node01 size=Large
    2. pod yml 파일에 nodeSelector 옵션을 추가하고, 원하는 노드의 라벨을 붙여준다. 
    
    ```json
    apiVersion: v1
    kind: Pod
    ..
    sepc:
     nodeSelector: 
      size: Large # size=Large 는 노드의 라벨값이다. 
    ```
    
- 단점: 복잡한 노드 선택 조건이 들어갈 때 사용하는데 한계가 있다.
    - 조건을 2개 이상 추가 불가
    - 부정문 사용 불가(~가 아닌 노드)

Node Affinity

- 노드 바인딩에 있어서 복잡한 조건물을 추가할 때 사용하는 옵션
- ex) 조건을 2개 이상 추가: 사이즈가 Large 또는 Medium 인 노드 바인딩
    
    ```json
    apiVersion: v1
    kind: Pod
    ..
    spec:
     affinity:
      requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: size
           operator: In
           values:
           - Large # 라벨 1 
           - Medium # 라벨 2
    ```
    

- ex) 부정문 사용: 사이즈가 Small이 아닌 노드 바인딩
    
    ```json
    apiVersion: v1
    kind: Pod
    ..
    spec:
     affinity:
      requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: size
           operator: NotIn
           values:
           - Small
    ```
    
- exist 연산: 해당 라벨이 존재하기만 하면 바인딩
    
    ```json
    apiVersion: v1
    kind: Pod
    ..
    spec:
     affinity:
      requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: size
           operator: Exists
    ```
    

### 5. 리소스

- 파드 기본 리소스 값
    - 파드는 리소스양을 설정하지 않으면 기본적으로 자원 사용에 한계가 없음.
- CPU 리소스 배치 시나리오
    1. No Request, No Limit
        1. 설정된 리소스양 없음, 제한 없음.
        2. 노드내에 2개의 파드가 있을 때 하나의 파드가 너무 많음 자원을 사용해서 다른 파드에 영향을 줄 수 있음.
    2. No Rquest, No Limit
        1. 설정된 리소스양 없음, 제한 있음.
        2. 자동으로 requests = limits 로 세팅됨. 
    3. Request, Limit
        1. 리소스양 설정, 제한 있음. 
        2. 세팅한 리소스양 만큼 배치되고, 이후 초과되면 limit까지 추가 배치 허용
    4. Request, No Limit
        1. 가장 이상적인 시나리오
        2. 각각의 파드한 요구한 최소한의 자원은 보장받으면서 limit을 제한하지 않음. 
        요구한 양보다 더 많은 자원을 제한 없이 사용할 수 있지만 다른 파드의 최소 보장 자원양은 침범하지 않는 선에서 추가적으로 배치됨. 
- Memory 리소스 배치 시나리오
    - 아래는 CPU와 시나리오가 모두 동일
        1. No Request, No Limit
        2. No Rquest, No Limit
        3. Request, Limit
    1. Request, No Limit
        1. 메모리는 CPU와 달리 시스템에서 조절이 불가능해서 limit을 걸지 않으면 다른 파드의 자원을 침범하여 노드에서 퇴거 대상이 됨. 
- 리소스 제한(spec.containers.resources.limits)
    - 최초 할당받은 자원보다 크게 설정할 수 있음.
    
    ```json
    apiVersion: v1
    ..
    spec:
     containers:
      resources:
       requests:
        memory: "1Gi"
        cpu: 1
       limits:
        memory: "2Gi"
        cpu: 2
    ```
    
    - 제한한 리소스를 초과할 경우
        - 파드는 제한된 cpu만큼 사용할 수 없도록 시스템이 조절
        - 메모리는 한도를 초과하면 OOM 발생함.
- Limit Range
    - 오브젝트가 최초 생성될 때 디폴트로 지정되는 리소스 제한 정책
    - cpu config
        
        ```json
        apiVersion: v1
        kind: LimitRange
        metadata: 
        	name: cpu-resource-constraint
        spec:
        	limits:
        		- default:
        				cpu: 500m (limit)
        			defaultRequest:
        				cpu: 500m
        			max:
        				cpu: "1"
        			min:
        				cpu: 100m
        			type: Container
        ```
        
    - memory config
        
        ```json
        apiVersion: v1
        kind: LimitRange
        metadata: 
        	name: memory-resource-constraint
        spec:
        	limits:
        		- default:
        				memory: 1Gi (limit)
        			defaultRequest:
        				memory: 1Gi
        			max:
        				memory: 1Gi
        			min:
        				memory: 500Mi
        			type: Container
        ```
        
- Resource Quotas
    - 클러스터내에서 네임스페이스별 모든 파드의 리소스 제한 정책
    - config
        
        ```json
        apiVersion: v1
        kind: ResourceQuota
        metadata:
        	name: my-resource-quota
        spec:
        	hard:
        		requests.cpu: 4
        		requests.memory: 4Gi
        		limits.cpu: 10
        		limits.memory: 10Gi
        ```
        

### 6. Daemon Set

- Replica Set vs Daemon Set
    - Replica Set
        - 하나의 파드에 대한 복제 셋
    - Daemon Set
        - 클러스터내 노드들에 동일한 파드가 배포
    
    → 노드 단위로 데몬셋이 실행되고, 파드 단위로 복제셋이 실행
    
- 사용 사례
    - 모니터링 시스템, 로그 시스템
    - kube-proxy
    - 네트워킹 솔루션
- config
    
    ```json
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
    	name: monitoring-demon
    spec:
    	selector:
    		matchLabels:
    			app: monitoring-agent
    	template:
    		metadata:
    			labels:
    				app: monitoring-agent
    		spec:
    			containers:
    				- name: monitoring-agent
    					image: monitoring-agent
    ```
    
- 조회: kubectl get daemonsets

## [Practice Test]

### 1. [Manual Scheduling(수동 스케줄링)](https://uklabs.kodekloud.com/topic/practice-test-manual-scheduling-2/)

1. Why is the POD in a pending state?
    1. 파드의 상태를 보니 pending 상태인데 왜인지. 
    2. 풀이
        1. kubectl describe pod nginx 명령어를 실행해서 에러 메시지를 확인하려 했지만 아무 에러메시지가 나와있지 않았음.
        2. node 키의 값이 <None>이길래 “No node avaliable”을 답으로 선택했지만 오답이었음.
        옳은 답이 아닌 이유가, 사용 가능한 노드가 없을 수도 있지만 사용 가능한 노드들에 파드를 할당해줄 스케줄러가 없는 것일 수도 있음. 
        3. 시스템에 스케줄러가 있는지 확인
            - kubectl get pods -n kube-system
            - 스케줄러는 kube-system 네임스페이스에 위치한다.
            - 확인되는 스케줄러가 없으므로 정답은 “No Schedular Present”
2. Manually schedule the pod on `node01`.
    1. nginx 파드에 node01 노드를 바인딩해라
    2. 이미 nginx 파드가 존재하므로 기존 파드를 삭제하고 다시 생성해야함. 
        1. 기존 파드 삭제: kubectl delete pod nginx
        2. yaml 파일 수정 후 create
            
            ```json
            apiVersion: v1
            kind: Pod
            metadata:
              name: nginx
            spec:
              nodeName: node01
              containers:
              -  image: nginx
                 name: nginx
            ```
            

### 2. Taints and Tolerations

1. Do any taints exist on `node01` node?
    1. node01 노드에 어떤 taints가 있는지
    2. describe로 taints 정보도 확인할 수 있음.!
    3. kubectl describe node node01
2. Create a taint on `node01` with key of `spray`, value of `mortein` and effect of `NoSchedule`
    1. node01 노드에 spray=mortein:NoSchedule 인 taint를 만들어라.
    2. kubectl taint node node01 spray=mortein:NoSchedule
3. Create another pod named `bee` with the `nginx` image, which has a toleration set to the taint `mortein`.
    1. 이미지 nginx의 bee 라는 파드를 만들고, `mortein` taint를 관용하도록 설정
    2. 파드까지는 명령어로 생성 가능하지만, toleration부터는 명령어로 세팅 불가능. yml 파일을 이용해야함. 
    3. [tolerations 설정 양식](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/pod-with-toleration.yaml)
4. Remove the taint on `controlplane`, which currently has the taint effect of `NoSchedule`.
    1. `controlplane` 노드에서 `NoSchedule` taint를 제거해라.
    2. taint 삭제 방법
        1. describe에서 노드의 taint의 value값을 복사한다. 
            1. ex) Taints: node-role.kubernetes.io/control-plane:NoSchedule 
        2. kubectl taint node `controlplane` node-role.kubernetes.io/control-plane:NoSchedule- (맨 끝에 -를 붙임)

### 3. Static Pod

1. Create a static pod named static-busybox that uses the busybox image and the command sleep 1000
    1. 정적 파드를 yaml 파일을 만들어서 생성하는 문제
    2. 풀이
        1. k run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > static-busybox.yaml 실행
            1. 선언적 명령어를 실행할 때 —command 옵션은 가장 마지막에 붙어야 한다. 뒤에 다른 옵션이 붙으면 안됨. 
            2. 정적 파드는 /etc/kubernetes/manifest 폴더 내에 파일을 생성하면 자동으로 파드가 생성된다. k create -f 명령어를 하지 않아도 된다.
2. We just created a new static pod named **static-greenbox**. Find it and delete it.
    1. 정적 파드를 삭제하는 문제 
    2. 풀이
        1. 정적 파드의 경로가 항상  /etc/kubernetes/manifest 인 것은 아님. /etc/kubernetes/manifest 안에 아무 파일도 없으면 다른 경로를 찾아야함. 
        2. kubelet 설정 파일 경로: /var/lib/kubelet/config.yaml 
            - config.yaml 에서 staticPodPath 옵션을 확인하면 된다.

## 명령어 꿀팁

1. —watch
    1. 파드의 상태값을 계속 모니터링 하는 옵션
    2. kubectl get pods --watch
2. yml 파일 내용을 새로운 yml 파일에 봍사붙여넣는 명령어
    1. kubectl get pod webapp -o yaml > my-new-pod.yaml
3. 모든 네임스페이스의 자원을 조회하는 옵션
    1. —all-namespace 또는 -A
    2. ex) kubectl get daemonsets -A
4. pod내 애플리케이션의 로그를 확인
    1. k logs <appname> -n <namespace>