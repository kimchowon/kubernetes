# 04. Application Lifecycle Management

### 1. 배포시 업데이트 및 롤백

- Rollout과 Versioning
    - Rollout
        - 배포의 롤아웃 확인 명령어
            - kubectl rollout history depeloyment/<deploy name>
- 쿠버네티스 배포 전략
    1. 기존 인스턴스들을 모두 롤아웃 시키고, 새로운 인스턴스들을 배포하는 방법
    2. 롤링 배포 - 기본 전략
- 신규 버전으로 배포하기
    1. 배포 yml 파일 수정
        - yml 파일에 명시된 template의 커네이너 이미지를 버전업한다.
            
            ```json
            apiVersion: apps/v1
            kind: Deployment
            ..
            spec:
             template:
              metadata:
               ..
               spec:
                containers:
                 - name: nginx-container
                   image: nginx:1.7.1 -> 1.7.2 로 변경 
            ```
            
    2. kubectl 명령어로 버전업하기
        1. kubectl set image deployment/myapp-deployment nginx-container=nginx:1.7.2
- 배포 동작 방식
    - 현재 버전의 파드 복제셋이 하나씩 내려감과 동시에 새로운 버전의 복제셋에 파드가 하나씩 뜸(롤링 배포)
    - 기존 버전과 신규 버전의 복제셋의 확인
        - kubectl get replicasets
        기존 버전은 모두 내려가 있고 신규 버전이 새롭게 뜬 것을 확인
            
            ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2356c823-06cb-4e16-be3a-7eaa83f4a147/e72e6e60-a1f2-4768-a452-3e035c83901a/Untitled.png)
            
    - 롤백
        - 신규 버전을 다시 내리고 이전 버전으로 복원
        - kubectl rollout undo deployment/myapp-deployment

### 2. 도커 환경 변수

- 도커 컨테이너 환경 변수 설정하는 법
    1. 각각의 yaml 파일에서 관리
        
        ```json
        apiVersion: v1
        kind: Pod
        ..
        spec:
         containers:
          env: # 환경 변수 추가
           - name: APP_COLOR
             value: pink
        ```
        
        - = docker run -e APP_COLOR=pink simple-webapp-color
    2. 여러 yaml 파일에서 사용하도록 중앙에서 관리하는 configMap 생성
        1. configMap 생성 방법
            1. 명령어 방식도 있지만 환경 변수가 많아지게 되면 선언적 방식이 더 편리할 듯
            2. yaml 파일(config-map.yaml)
                
                ```json
                apiVersion: v1
                kind: ConfigMap
                metadata:
                 name: app-config
                data:
                 APP_COLOR: blue
                 APP_MODE: prod
                ```
                
                - 이런식으로 여러 애플리케이션의 설정 파일을 만듬. ex) redis-config, mysql-config ..
        2. 애플리케이션 설정 파일에 환경 변수 설정 파일 주입하는 방법
            1. ex) pod-definition.yml
                
                ```json
                apiVersion: v1
                kind: Pod
                ..
                spec:
                 containers:
                  envFrom:
                   - configMapRef:
                      name: app-config // configMap의 name을 추가
                ```
                

### 3. secrets 설정

- 패스워드 같은 민감한 정보를 관리하는 설정
- secrets 생성 방법 (선언적(파일) 생성)
    - 민감 정보를 plain 텍스트가 아닌 인코딩 형태로 변환한다.
        - 인코딩 명령어 echo -n ‘password’ | base64 를 실행, 출력된 결과를 secret yaml 파일의 value에 추가한다.
    - ex) secret-definition.yml
        
        ```json
        apiVersion: v1
        kind: Secret 
        metadata:
         name: app-secret
        data:
         DB_HOST: <encoding value>
         DB_USER: <encoding value>
         DB_PASSWORD: <encoding value>
        ```
        
    - 암호화된 값 해독하는 명령어
        - echo -n ‘cm9vdA==’ | base64 —decode
- 파드에 secret 설정 파일 적용
    
    ```json
    apiVersion: v1
    kind: Pod
    ..
    spec:
     containers:
      - name: ..
        envFrom:
         - secretRef:
            name: app-secret
    ```
    

### [Pracetice Test]

### 1. Rolling Update & Rollout

1. Up to how many PODs can be down for upgrade at a time
롤링배포 할 때 최대 몇개까지 다운시킬 수 있는가
    1. kubectl describe 로 보면 아래와 같은 옵션이 있다. 
        
        ```json
        RollingUpdateStrategy:  25% max unavailable, 25% max surge
        ```
        
        - max unavailable(최대 불가)
            - 업데이트 실행 중에 사용할 수 없는 최대 파드 수
        - max surge(최대 서지)
            - 의도한 파드 수보다 더 생성할 수 있는 최대 파드 수
    2. max unavailable이 25%이므로 4개 중에 1개만 다운시킬 수 있다. 
2. Change the deployment strategy to `Recreate`
배포 전략을 중단 배포로 변경해라
    1. k edit deployment frontend
    2. strategy 옵션을 Recreate로 변경한다.
        
        ```json
        // AS-IS
        strategy:
            rollingUpdate:
              maxSurge: 25%
              maxUnavailable: 25%
            type: RollingUpdate
        
        // TO-BE
        strategy:
            type: Recreate
        ```
        
    
    ### 2. Docker Commnad
    
    1. Create a pod with the ubuntu image to run a container to sleep for 5000 seconds. Modify the file `ubuntu-sleeper-2.yaml`.
    2. 파드에 sleep 5000 커맨드를 추가
    
    ```json
    apiVersion: v1
    kind: Pod 
    metadata:
      name: ubuntu-sleeper-2
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command:
         - "sleep"
         - "5000"
    ```
    
    1. Create a pod with the given specifications. By default it displays a `blue` background. Set the given command line arguments to change it to `green`
        1. 파드를 생성하고 —color=green 컨매드를 앱 구동시 실행하는 문제
        2. k run webapp-green --image=kodekloud/webapp-color -- --color green

### 3. Multi Container Pod

1. Create a multi-container pod with 2 containers.

Use the spec given below.

If the pod goes into the `crashloopbackoff` then add the command `sleep 1000` in the `lemon` container.

1. 

The application outputs logs to the file `/log/app.log`. View the logs and try to identify the user having issues with Login.

Inspect the log file inside the pod.