링크

https://uklabs.kodekloud.com/topic/practice-test-services-2/

1. How many Services exist on the system?
현재 시스템이 서비스가 몇 개 있는가?
    1. kubectl get services
2. That is a default service created by Kubernetes at launch.
3. What is the type of the default `kubernetes` service?
디폴트 서비스인 kubernetes의 타입은 무엇인가?
    1. kubectl get services 실행하여 TYPE 컬럼 확인
4. What is the `targetPort` configured on the `kubernetes` service?
kubernetes 서비스의 타켓포트는 몇 인가?
    1. kubectl edit service kubernetes 실행하면 yml 파일이 나온다. 거기서 targetPort를 확인
    2. 또는 kubectl describe svc kubernetes (음.. 서비스는 edit보다 describe가 더 나은듯)
5. How many labels are configured on the `kubernetes` service?
kubernetes 서비스에서 몇 개의 라벨이 설정되어 있는가?
    1. 마찬가지로 kubectl edit service kubernetes로 yml 파일 조회
    2. metadata.labels 하위에 라벨들을 확인한다. 
6. How many Endpoints are attached on the `kubernetes` service?
    1. kubectl describe svc kubernetes 실행
    2. Endpoints 키에서 ip 개수 확인
7. What is the image used to create the pods in the deployment?
배포에서 파드를 생성할 때 사용한 이미지는?
    1. 먼저 배포 정보를 조회
        1. kubectl get deployment
    2. 그 다음 상세 정보 조회
        1. kubectl describe deployment <배포명>
    3. Pod Template.Containers.<파드명>.Image 키값 조회
8. Are you able to accesss the Web App UI?
지금 web app의 UI에 접속할 수 있는가?
    1. 갑뿐 오른쪽 상단에 링크를 클릭해야함. 이런 이미지를 클릭하면 Bad Gateway 등장. 고로 접속할 수 없다. 
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2356c823-06cb-4e16-be3a-7eaa83f4a147/98e217eb-e4f9-459a-82d5-bc7d5ca1ee83/Untitled.png)
        
9. Create a new service to access the web application using the `service-definition-1.yaml` file.
주어진 내용대로 서비스 생성 yml 파일을 작성하는 문제
10. Access the web application using the tab `simple-webapp-ui` above the terminal window.
터미널로 simple-webapp 서비스의 UI에 접속하라
    1. 9에서 만들고 다시 8번의 이미지를 클릭하면 UI에 접속됨.