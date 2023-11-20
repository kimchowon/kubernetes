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
        
        서버 인증성 생성
        
        - kube-apiserver → etcd server, kubelet
        
        - kube-apiserver
            1. private key 생성
                1. openssl genrsa -out [apiserver.ke](http://apiserver.ke)y 2048
            2. CSR 파일 생성
                1. openssl req -new -key apiserver.key -subj “/CN=kube-apiserver” -out apiserver.csr
            3. CA에 인증서 서명 요청
                1. openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -out apiserver.crt
    

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
    
    - 2379 포트는 통상적으로 ETCD 서버 포드이다.
5. ETCD 상태를 모니터링
    1. crictl ps -a | grep etcd
6. 5에서 조회한 ps id로 로그 확인
    1. crictl logs fca3f827cc0ba
7. 인증서 파일이 존재하지 않는다는 에러 확인
    
    ```yaml
    "error":"open /etc/kubernetes/pki/etcd/server-certificate.crt: no such file or directory"
    ```