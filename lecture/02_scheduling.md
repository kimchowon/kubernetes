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
            

## 명령어 꿀팁

1. —watch
    1. 파드의 상태값을 계속 모니터링 하는 옵션
    2. kubectl get pods --watch