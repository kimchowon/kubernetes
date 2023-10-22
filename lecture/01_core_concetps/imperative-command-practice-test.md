# practice-test(5) - imperative command

1. Deploy a pod named nginx-pod using the nginx:alpine image.
(Name: nginx-pod, Image: nginx:alpine)
nginx:alpine 이미지를 사용하여 nginx-pod 파드를 배포하라
    1. kubectl run nginx-pod --image=nginx:alpine

1. Deploy a redis pod using the redis:alpine image with the labels set to tier=db.
redis:alpine 이미지를 사용하고 tier=db 라벨링 된 redis 파드를 배포하라
    1. kubectl run redis --image=redis:alpine --labels="tier=db”

1. Create a service redis-service to expose the redis application within the cluster on port 6379.
클러스터 내에 있는 레디스 앱을 6379 포트로 노출하는 redis-service 서비스를 생성하라
    1. kubectl expose pod redis --port 6379 --name redis-service

1. Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas.
kodekloud/webapp-color 이미지를 사용하여 복제본이 3개인 webapp 이라는 배포를 만들어라 
    1. kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3

1. Create a new pod called custom-nginx using the nginx image and expose it on container port 8080.
    1. 파드 생성
        1. name: custom-nginx
        2. image: nginx
    2. 해당 파드내의 컨테이너는 8080 포트로 노출시킴
    
    → kubectl run custom-nginx --image=nginx --port 8080
    
2. Create a new namespace called dev-ns.
    1. dev-ns 라는 네임스페이스 신규 생성
    
    → kubectl create namespace dev-ns
    
3. Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas.
    1. 배포 생성 문제
        1. name: redis-deploy
        2. namespace: dev-ns
        3. image: redis
        4. replicas: 2
    
    → kubectl create deployment redis-deploy --namespace=dev-ns --image=rdis --replicas=2
    deployment.apps/redis-deploy created
    

1. Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80.
    1. 파드 생성 문제
        1. name: httpd
        2. image: httpd:alpine
        3. namespace: default
    2. 그 다음 서비스 생성
        1. type: ClusterIP
        2. name: httpd
        3. targetPort: 80
- 풀이 방법 2가지
    1. 파드, 서비스 각각 생성
        1. 파드: kubectl run httpd --image=httpd:alpine —port 80
        2. 서비스: kubectl expose pod httpd --type ClusterIP --name httpd --port 30080 --target-port 80 (ClusterIP의 포트는 30080으로 임의 지정)
    2. 파드와 서비스 생성을 동시에 하는 명령어
        1. kubectl run 옵션에 —expose 옵션이 있음
            
            ```bash
            --expose=false:
                    If true, create a ClusterIP service associated with the pod.  Requires `--port`.
            ```
            
            true로 설정할 경우 파드와 관련된 클러스터 ip 서비스를 자동으로 생성해줌. 
            
        2. 따라서 kubectl run httpd --image=httpd:alpine —port 80 —expose=true