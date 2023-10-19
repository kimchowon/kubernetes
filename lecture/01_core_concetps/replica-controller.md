# replication-controller

### 복제 컨트롤러(Replication Controller) vs 복제셋(Replica Set)

- 파드를 복제해두고, 파드가 죽었을 때 복제 파드를 띄워 늘 서비스가 실행 가능하도록 하는 역할
- 서비스의 고가용성 제공
- 복제 컨트롤러가 구식 방식으로 deprecated 되었고 복제셋을 사용하는 것으로 옮겨가고 있음.

### 복제 컨트롤러 생성 방법

1. yaml 파일 작성
- re-definiton.yml

```bash
apiVersion: v1
kind: ReplicationController
metadata:
 name: myapp-rc
 labels:
  app: myapp
  type: front-end
spec:
 template:
  metadata:
   name: myapp-pod
   labels:
    app: myapp
    type: front-end
  spec:
   containers:
    - name: nginx-controller
      image: nginx
 replicas: 3
```

- template: 복제를 하기위한 파드 템플릿
    - pod를 생성하는 yaml 파일에서 apiVersion, kind 를 제외한 나머지를 그대로 복사 붙여 넣는다.
- replicas: 복제할 파드의 개수

1. yaml 파일을 사용하여 복제 컨트롤러 생성
- kubectl create -f re-definiton.yml

1. 복제 컨트롤러 확인
- kubectl get replicationcontroller

### 복제 셋(Replica Set) 생성 방법

1. yaml 파일 작성

```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: myapp-replicaset
 labels:
  app: myapp
  type: front-end
spec:
 template:
  metadata:
   name: myapp-pod
   labels:
    app: myapp
    type: front-end
  spec:
   containers:
    - name: nginx-controller
      image: nginx
 replicas: 3
 selector:
  matchLabels:
   type: front-end
```

- apiVersion: apps/v1 이어야 함.
- selector: 어떤 파드를 복제본으로 관리할지 설정
    - labels 로 파드를 지정

1. 복제셋 생성
- kubectl create -f replicaset-defiintion.yml

1. 생성된 복제셋 확인
- kubectl get replicaset

### 복제본 개수 변경

1. yaml 파일 수정
- replicas를 수정
- kubectl replace -f replicaset-definition.yml 실행
1. 다음 명령어 실행
- kubectl scale —replicas=6 -f replicaset-definition.yml
1. type과 name을 이용한 명령어 실행
- kubectl scale —replicas=6 <type> <name>
    - kubectl scale —replicas=6 replicaset myapp-replicaset