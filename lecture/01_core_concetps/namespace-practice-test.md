# practice-test(4) - Namespace

1. How many Namespaces exist on the system?
시스템에 총 몇개의 네임스페이스가 있는가
    1. kubectl get namespaces
2. How many pods exist in the `research` namespace?
research 네임스페이스에 몇 개의 파드가 있는가
    1. kubectl get pods --namespace=research
    2. kubectl get pods -n=research
3. Create a POD in the `finance` namespace.(파드명: redis, 이미지: redis)
finance 네임스페이스에 파드를 생성하라. 
    1. kubectl run redis --image=redis --namespace=finance
4. Which namespace has the `blue` pod in it?
blue 파드는 어떤 네임스페이스에서 가지고 있는가?
    1. 모든 네임스페이스를 조회하여 각 네임스페이스에서 가지고 있는 파드를 확인
    2. kubectl get pod --all-namespaces
5. Access the `Blue` web application using the link above your terminal!!
From the UI you can ping other services.
6. What DNS name should the Blue application use to access the database db-service in its own namespace - marketing? You can try it in the web application UI. Use port `6379`.
marketing 네임스페이스 안에서 Blue 앱이 db-service 데이터 베이스에 접근하려면 어떤 DNS 이름을 사용해야 하는가. 웹 앱의 화면에서 6379 포트로 접근해라
    1. marketing 네임스페이스내에 있는 서비스들을 조회
    2. kubectl get svc --namespace=marketing
    3. 서비스명이 db-service이고 포트가 6379인 서비스를 선택
7. What DNS name should the `Blue` application use to access the database `db-service` in the `dev` namespace?
Blue 앱이 dev 네임스페이스의 db-service 데이터베이스에 접근하기 위해 사용되는 DNS 이름은 무엇인가
    1. Blue앱은 marketing 네임스페이스에 있으므로 다른 네임스페이스에 존재하는 db-service에 접근해야함. 
    2. 그러려면 서비스 전체 주소를 사용해야 함.
    3. db-service.dev.svc.cluster.local