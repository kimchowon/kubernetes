# 06. Security

### 1. 쿠버네티스 보안

1. 접근 가능한 사용자를 관리
    1. id, password를 파일로 관리
    2. id, 토큰을 파일로 관리
    3. 자격증명
    4. LDAP 인증
    5. 서비스 계정 생성  
2. 사용자가 행할 수 있는 기능을 권한에 따라 분류
    1. RBAC 권한
    2. ABAC 권한
    3. Node 권한
    4. 웹훅 모드
3. TLS Certificate
    1. 스케줄러, api 서버, kubelet, kube-proxy 등의 노드에서 실행되는 모든 행위는 TLS 암호화하여 보안 관리

### 2. 인증(Authentication)

- 계정 종류
    1. 사용자 계정
        1. 쿠버네티스 내부에서 생성 및 관리할 수 없음. 외부에 인증서가 있거나 LDAP 등으로 관리함.
        2. 사용자가 kube api server에 접근할 때 인증을 수행
            1. 패스워드 또는 토큰 인증 파일 
    2. 서비스 계정
        1. 쿠버네티스 내부에서 직접 계정을 생성 및 관리할 수 있음.
        2. 계정 생성 명령어
            1. kubectl create serviceaccount sa1
- 인증 메카니즘
    1. 패스워드 또는 토큰 파일 인증
        1. 인증 순서
            1. csv 파일에 사용자 패스워드 또는 토큰 정보를 저장
                
                ```yaml
                password, username, id, group
                
                or
                
                token, username, id, group
                ```
                
            2. 해당 파일을 kube-apisever 옵션에 지정(—basic-auth-file=user-details.csv)
                
                ```yaml
                apiVersion: v1
                kind: Pod
                metadata:
                	..
                spec:
                	containers:
                		- command:
                				- —basic-auth-file=user-details.csv // 패스워드 기반 파일 인증
                				- --token-auth-file-user-token-detail.csv // 토큰 기반 파일 인증
                ```
                
            3. 옵션을 지정한 뒤 kube-apiserver를 재시작
            4. kubeapi-server로 접근할 때 명령어에 계정 정보를 지정
                1. ex) 패스워드 기반: curl -v -k https://master-node-ip:6443/api/v1/pods -u “user1:password123”
                2. ex) 토큰 기반:  curl -v -k https://master-node-ip:6443/api/v1/pods —header “Authentication: Bearer <token>”
        
        ### 3. TLS 인증서 생성
        
        openssl 을 이용한 인증서 생성
        
        1. private key 생성
            1. openssl genrsa -out ca.key 2048
        2. CSR 파일 생성(인증서 서명 요청 파일)
            1. openssl req -new -key ca.key -subj “/CN=KUBERNETES-CA” -o
        3. CA에 인증서 서명 요청
            1. openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
        
        클라이언트 인증서 생성
        
        - admin, schedular, kube-controller-manager, kube-proxy → kube-apiserver
        
        - admin
            1. private key 생성
                1. openssl genrsa -out admin.key 2048
            2. CSR 파일 생성
                1. openssl req -new -key admin.key -subj “/CN=kube-admin” -out admin.csr
            3. CA에 인증서 서명 요청
                1. openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
        
        인증서로 kube-apiserver 접근하기
        
        ```yaml
        curl https://kube-apiserver:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt
        ```
        
        서버 인증서 생성
        
        - kube-apiserver → etcd server, kubelet
        
        - kube-apiserver
            1. private key 생성
                1. openssl genrsa -out [apiserver.ke](http://apiserver.ke)y 2048
            2. 관리자에게 CSR 파일 생성 요청
                1. openssl req -new -key apiserver.key -subj “/CN=kube-apiserver” -out apiserver.csr
            3. CA에 인증서 서명 요청
                1. openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -out apiserver.crt

### 4. 인증서 관리 & 인증서 API

- 인증서 키 파일은 마스터 노드에서 관리한다. (마스터 노드 = CA 관리 서버)
    - 정확히는 인증서에 대한 전반적인 작업을 마스터 노드의 컨트롤러 매니저에서 담당한다.
- 요청자가 인증서 서명을 받는 과정
    1. 요청자가 private key 생성
        1. openssl genrsa -out chocho.key 2048
    2. 관리자에게 CSR 파일 생성 요청
        1. openssl req -new -key chocho.key -subj “/CN=chocho” -out chocho.csr
    3. CSR 파일 base64로 인코딩
        1. cat chocho.csr | base64 -w 0
    4. 인증서 서명 요청 개체 생성(yaml 파일 이용)
        1. chocho-csr.yaml
        
        ```yaml
        apiVersion: certificates.k8s.io/v1beta1
        kind: CertificateSigningRequest
        metadata:
        	name: chocho
        spec:
        	groups:
        		- system:authenticated # 어떤 그룹에 대한 인증서 요청인지
        	usages:
        		- digital signature
        		- key encipherment
        		- server auth
        	request:
        		# 인증서 서명 요청을 기입
        		# base64로 인코딩한 값을 넣는다.
        		DKFJSKDJFSDLFKJDKSFKSDJ#$#@#@DFKSDJFLSKDJFKSDFJSD
        ```
        
    5. 인증서 서명 요청 객체가 생성되면 모든 서명 요청을 kubectl 명령어로 확인할 수 있음.
        1. kubectl get csr
    6. 인증서 승인 & 거절
        1. 승인: kubectl certificate approve chocho
        2. 거절: kubectl certificate deny chocho

- 현재 존재하는 인증 요청의 yaml 파일 조회 방법
    - k get csr <인증서 이름> -o yaml
        - ex) k get csr chocho -o yaml
- 인증서 삭제
    - k delete csr <인증서 이름>
        - ex) k delete csr chocho

### 5. kubeconfig

- kubeconfig 설정
    - kubectl 명령어로 kube-apiserver로 요청을 보낼때 사용자마다 권한을 인증하는 방법
- kubeconfig 파일 템플릿
    
    ```yaml
    apiVersion: v1
    kind: Config
    
    # default 컨텍스트
    # kubectl 명령을 할 때 current-context 기준으로 명령이 나간다. 
    current-context: my-kube-admin@my-kube-playground
    
    clusters: # 쿠버네티스 내 어떤 환경인지
    	- name: my-kube-playground
    		cluster:
    			certificate-authority: /etc/kubernetes/pki/ca.crt
    			server: https://my-kube-playground:6443
    contexts: # 어떤 환경에 어떤 사용자가 권한이 있는지 매핑
    	- name: my-kube-admin@my-kube-playground
    		context:
    			cluster: my-kube-playground
    			user: my-kube-admin
    users: # 사용자
    	- name: my-kube-admin
    		user:
    			client-certificate: /etc/kubernetes/pki/admin.crt
    			client-key: /etc/kubernetes/pki/admin.key
    ```
    

- default kubeconfig 파일 경로
    - 보통 .kube/ 아래에 config 파일로 있다.
- kubeconfig 파일 조회:
    - 현재 사용중인 kubeconfig 파일 조회: k config view
    - 특정 kubeconfig 파일 조회: k config view —kubeconfig=<파일명>

- 사용할 컨텍스트를 변경
    - k config —kubeconfig=<config파일경로> use-context <컨텍스트명>
        - ex) k config use-context prod-user@production

- 특정 kubeconfig 파일을 default 설정으로 변경하는 명령어
    - mv <변경라혀는 config 파일>  <기존 default config 파일>
- kubeconfig에 네임스페이스 설정
    - 특정 네임스페이스로내에서만 접근하도록 설정
        
        ```yaml
        ..
        contexts: # 어떤 환경에 어떤 사용자가 권한이 있는지 매핑
        	- name: my-kube-admin@my-kube-playground
        		context:
        			cluster: my-kube-playground
        			user: my-kube-admin
        			namespace: finance # 네임스페이스 명시
        
        ..
        ```
        

- kubeconfig 파일에 인증서를 넘기는 방법
    1. 파일 경로를 명시
        
        ```yaml
        ..
        
        clusters: # 쿠버네티스 내 어떤 환경인지
        	- name: my-kube-playground
        		cluster:
        			certificate-authority: /etc/kubernetes/pki/ca.crt # 파일 경로 명시
        			server: https://my-kube-playground:6443
        contexts: # 어떤 환경에 어떤 사용자가 권한이 있는지 매핑
        	- name: my-kube-admin@my-kube-playground
        		context:
        			cluster: my-kube-playground
        			user: my-kube-admin
        users: # 사용자
        	- name: my-kube-admin
        		user:
        			client-certificate: /etc/kubernetes/pki/admin.crt
        			client-key: /etc/kubernetes/pki/admin.key
        ```
        
    2. 인증서 내용 자체를 명시
        1. base64로 인코딩해서 넣어야함.
            
            ```yaml
            clusters: # 쿠버네티스 내 어떤 환경인지
            	- name: my-kube-playground
            		cluster:
            			certificate-authority-data: DKFJSLDJFSKLDJFSFJDLKFSJ...
            ```
            

### 6. kubernetes API

- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#api-groups
- 쿠버네티스 내에서 사용 가능한 api 목록 조회
    - 인증이 필요함
        1. curl http://localhost:6443 -k —key <private key> —cert <인증요청서> —cacert <ca 인증서>
        2. kube-proxy에 kubeconfig 파일에 인증 내용 저장
            1. user → kube-proxy → kube-api 

### 7. 권한 부여

권한 부여 종류

1. 노드 접근 권한 부여
    1. 노드내 kubelet 에 인증 시스템이 있음. 
2. ABAC: 특성 기반 권한 부여
3. RBAC: 역할 기반 권한 부여
    1. 각 역할마다 권한 집합을 부여하는 방법
        1. ex) developer 역할에 A 권한 집합 부여
    2. 설정 방법
        1. 설정 파일 생성
        - developer-role.yaml
            
            ```yaml
            apiVersion: rbac.authorization.k8s.io/v1
            kind: Role
            metadata:
             name: developer
            rules:
            - apiGroups: [""]
              resources: ["pods"]
              verbs: ["list", "get", "create", "update", "delete"]
              resourceNames: ["blue", "orange"] #네임스페이스내 리소스에 권한 부여
            
            - apiGroups: [""]
              resources: ["ConfigMap"]
              verbs: ["create"]
            ```
            
        2. k create -f developer-role.yaml
        3. 사용자를 권한과 매핑
            - role binding 이라는 설정 파일 생성
            - devuser-developer-binding.yaml
                
                ```yaml
                apiVersion: rbac.authrization.k8s.io/v1
                kind: RoleBinding
                metadata:
                 name: devuser-developer-binding
                 namespace: controlplane
                subjects:
                 - kind: User
                   name: dev-user
                   apiGroup: rbac.authorization.k8s.io
                roleRef:
                 kind: Role
                 name: developer
                 apiGroup: rbac.authorization.k8s.io
                ```
                
    3. 생성된 롤 목록 조회: k get roles
        1. k get roles : default 네임스페이스 내의 롤 조회
        2. k get roles -n <네임스페이스명>: 네임스페이스 내 롤 조회
        3. k get roles -A: 모든 네임스페이스내 롤 조회
    4. 특정 롤에 대한 상세 정보 조회: k describe role <롤 이름>
    5. 생성된 롤 바인딩 목록 조회: k get rolebindings
    6. 특정 롤 바인딩에 대한 상세 정보 조회: k describe role <롤 바인딩 이름>
    7. 롤 설정 수정: k edit role <롤 이름>
        1. k edit role developer

1. 웹훅
    1. 외부에서 권한 정보를 관리할 때 외부에서 웹훅을 보내면 승인
2. 항상 승인 or 항상 거절

특정 행위에 권한이 있는지 확인

- k auth can-i <행위> —as <롤> —namespace <네임스페이스>
    - ex) k auth can-i create deplyments —as dev-user —namespace test
    - ex) k auth can-i delete nodes —as dev-user —namespace test

### 8. 클러스터 롤 바인딩

- 롤, 롤 바인딩의 기본적으로 네임스페이스 기준으로 생성됨.
    - 그러나 네임스페이스 종속적이지 않고 클러스터 종속적인 자원들이 있음.
        - kubectl api-resources —namespaced=true/false 로 확인
        - 대표적으로 node가 있음.
- 클러스터 scope 자원 롤 및 바인딩 설정 파일
    1. 롤 설정 파일
    - cluster-admin-role.yaml
        
        ```yaml
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRole
        metadata:
         name: cluster-administrator
        rules:
        - apiGroups: [""]
          resources: ["nodes"]
          verbs: ["list", "get"]
        ```
        
    2. 롤 바인딩 설정 파일
        
        ```yaml
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
         name: cluster-admin-role-binding
        subjects:
        - kind: User
          name: cluster-admin
          apiGroup: rbac.authorization.k8s.io
        roleRef:
         kind: ClusterRole
         name: cluster-administrator
         apiGroup: rbac.authorization.k8s.io
        ```
        

### 9. 서비스 계정

- 쿠버네티스 계정은 2개: 사용자 계정, 서비스 계정
- 서비스 계정 생성
    - k create serviceaccount <계정이름>
- 서비스 계정 목록 조회
    - k get serviceaccount
- 서비스 계정 토큰
    - 서비스 계정을 생성하면 토큰이 자동 생성됨.
    - 서비스 계정의 비밀 객체가 생성되고  그 안에 토큰이 저장됨.
    - 토큰 상세 조회
        - k describe secret <토큰명>
    - curl로 API 호출시 인증이 필요할 때 토큰을 실어서 요청할 수 있음.
        - curl http://../api -insecure —header “Authorization: Bearer <토큰>”
- 파드가 생성되면 자동으로 default 서비스 계정이 만들어짐
    - 토큰 위치: /var/un/secret/kubernetes
    - 기본 권한: API 쿼리 실행
- 기존 파드의 서비스 계정은 수정할 수 없음. 파드를 삭제하고 새로 만들어야 함.
- 파드에 서비스 계정 설정 하는 법
    - /pod-definitoin.yaml
        
        ```yaml
        apiVersion: v1
        kind: Pod
        ..
        spec:
         containers:
          ..
         serviceAccountName: dashboard-sa
        ```
        
    - 아무것도 설정하지 않으면 default 서비스 계정이 파드에 자동으로 마운팅 되는데 이 기본설정을 끄는 옵션
        
        ```yaml
        apiVersion: v1
        kind: Pod
        ..
        spec:
         containers:
          ..
         amountServiceAccountToken: false
        ```
        
- 파드내 디렉토리 열거
    - k exec -it <파드명> ls <디렉토리 경로>
    - ex) k exec -it my-kubernetes-dashboard ls /var/run/secrets/kubernetes.io/serviceaccount

### 10. 이미지 보안

- 쿠버네티스에서 특정 개인 레파지토리 내의 이미지 접근 권한을 얻는 방법
    1. 자격증명이 포함된 비밀 개체 생성
        
        ```yaml
        # 비밀 개체 이름은 regcred라고 명명
        k create secret docker-registry regcred \
        --docker-server= \
        --docker-username= \
        --docker-password= \
        --docker-email=
        ```
        
    2. 파드 설정 파일에 비밀 개체를 지정
        
        ```yaml
        apiVersion: v1
        kind: Pod
        ..
        spec:
         imagePullSecrets:
          - name: regcred
        ```
        

### [Practice Test]

### 1. View Certificate Details

1. Identify the certificate file used for the kube-api server.
- kube-apiserver에서 사용되는 인증서의 경로 및 이름은?

<aside>
💡 풀이

- 인증서에 대한 정보를 보려면 메니페스트 파일을 학인하면 된단.
    - 메니페스트 파일 경로: /etc/kubernetes/manifests/kube-apiserver.yaml
- 인증서 경로: -tls-cert-file
</aside>

1. What is the Common Name (CN) configured on the Kube API Server Certificate?
- kube-apiserver 인증서의 공동 이름은?

<aside>
💡 풀이

- CN 확인 명령어
    - openssl x509 -in <file-path.crt> -text -noout
    - kube-apiserver의 인증서 파일 경로가 /etc/kubernetes/pki/apiserver.crt 이므로
    openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
- Issuer: CN = kubernetes : 인증서 발급 주체
- Subject: CN = kube-apiserver: 인증서를 발급받은 대상
- 답은 kube-apiserver
</aside>

1. Which of the below alternate names is not configured on the Kube API Server Certificate?
- kube-apiserver 인증서에 나와있지 않은 대체제명은 무엇인가?

<aside>
💡 풀이

- SAN(Subject Alternative Name)
    - ip 주소를 SAN 필드에 추가하여 쿠버네티스가 여러 노드들의 ip에서 작동할 수 있도록 설정
- CN 확인 명령어 openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout 실행
- Subject Alternative Name 에 존재하지 않은 이름이 정답
</aside>

1. Kubectl suddenly stops responding to your commands. Check it out! Someone recently modified the /etc/kubernetes/manifests/etcd.yaml file
You are asked to investigate and fix the issue. Once you fix the issue wait for sometime for kubectl to respond. Check the logs of the ETCD container.
- kubectl 명령어가 먹통이라 etcd.yaml 파일에 잘못된 내용을 찾아 고치는 문제

<aside>
💡 풀이

1. kubectl 명령어가 안된다는 것은 kube-apiserver 통신이 불가능한 상태
    - kube-apiserver의 상태를 모니터링 한다.
2. kube-apiserver의 상태를 모니터링 명령어
    - crictl ps -a | grep kube-apiserver
        
        ```yaml
        controlplane kubernetes/pki/etcd ➜  crictl ps -a | grep kube-apiserver
        44185712c3334       6f707f569b572       2 minutes ago       Exited              kube-apiserver            7                   cd8a092c7bacb       kube-apiserver-controlplane
        ```
        
3. 2에서 조회한 ps id 44185712c3334 로 로그를 확인한다.
    1. crictl logs 44185712c3334
4. 로그를 확인해보면 127.0.0.1:2379 와 통신이 거절 된다는 로그가 나옴
    
    ```yaml
    Error while dialing dial tcp 127.0.0.1:2379: connect: connection refused
    ```
    
    - 2379 포트는 통상적으로 ETCD 서버 포트이다.
5. ETCD 상태를 모니터링
    1. crictl ps -a | grep etcd
6. 5에서 조회한 ps id로 로그 확인
    1. crictl logs fca3f827cc0ba
7. 인증서 파일이 존재하지 않는다는 에러 확인
    
    ```yaml
    "error":"open /etc/kubernetes/pki/etcd/server-certificate.crt: no such file or directory"
    ```
    
8. /etc/kubernetes/pki/etcd 로 가보면 server-certificate.crt 가 아닌 server.crt가 있다. 
9. /etc/kubernetes/manifest/etcd.yml 파일에 —cert-file 프로퍼티를 수정한다. 
</aside>

### 8. Cluster Role Binding

1. A new user `michelle` joined the team. She will be focusing on the `nodes` in the cluster. Create the required `ClusterRoles` and `ClusterRoleBindings` so she gets access to the `nodes`.
- 새로운 팀원 michelle이 노드 관련 접근을 못함. 클러스터 롤 및 바인딩 정책을 추가해서 michelle에게 권한을 부여하는 문제

<aside>
💡 풀이

1. 먼저 k get nodes —as michelle 을 해보면 forbidden 에러 발생함.
2. 설정 파일(yaml)을 만들어도 되지만, 간단하게 명령어로 롤 및 롤바인딩 설정을 추가할 수도 있음.
    1. 롤 추가: kubectl create clusterrole michelle-role --verb=get,list,watch --resource=nodes
    2. 롤 바인딩 추가: kubectl create clusterrolebinding michelle-node-binding --clusterrole=michelle-role --user=michelle
</aside>