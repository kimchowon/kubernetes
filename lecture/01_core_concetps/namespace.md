# namespace

### 네임스페이스

- 단일 클러스터 내에서의 리소스 그룹 격리 메커니즘
- https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/namespaces/

### 네임스페이스에서 다른 네임스페이스의 자원에 접근하는 방법

- dev 라는 이름의 네임스페이스내의 db-service에 접근
    - mysql.connect(”db-service.dev.svc.cluster.local”)
        - db-service: 자원명
        - dev: 네임스페이스명
        - svc: 네임스페이스 하위의 도메인(service의 약자)
        - cluster.local: 쿠버네티스 클러스터의 기본 도메인명

### 네임스페이스 명령어

- kubectl get pods —namespace=네임스페이스명
    - kubectl get pods는 모든 파드를 조회하는 명령어지만 default 네임스페이스 내에서 조회
    - —namespace=네임스페이스명을 붙이면 해당 네임스페이스 내에서 파드들을 조회
- yml 파일을 사용한 파드 생성
    - kubectl create -f pod-definition.yml —namespace=네임스페이스명
    - 해당 네임스페이스 내에 파드 생성
    - 또는 yml 파일내에 metadata 섹션 하위에 namespace 키로 정의
        
        ```bash
        apiVersion: v1 
        kind: Pod
        metadata:
         name: myapp-pod
         namepspace: dev # 네임스페이스 지정
        
        ...
        ```
        
- 네임스페이스 생성
    - yml 파일
        
        ```bash
        apiVersion: v1
        kind: Namespace
        metadata:
         name: dev
        ```
        
    - 명령어로 바로 생성: kubectl create namespace dev
- 디폴트 네임스페이스 지정하는 법
    - —namespace=네임스페이스명 없이 kubectl 을 실행하면 default 네임스페이스가 기본값임
    - 다른 네임스페이스를 디폴트로 지정하고 싶으면 아래 명령어로 kubectl 설정을 변경
        - kubectl config set-context $(kubectl config current-context) —namespace=dev
- 모든 네임스페이스에 있는 파드들을 조회
    - kubectl get pods —all-namespaces

### 네임스페이스 자원 할당

- 네임스페이스마다 자원 할당량을 제한할 수 있음
- yml 파일을 생성하여 자원을 설정
    
    ```bash
    apiVersion: v1
    kind: ResourceQuota
    metadata:
     name: compute-quota
     namespace: dev
    spec:
     hard:
      pods: "10"
      requests.cpu: "4"
      requests.memory: 5Gi
      limits.cpu: "10"
      limits.memory: 10Gi
    ```