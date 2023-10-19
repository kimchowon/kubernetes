# 모의 테스트(2) - ReplicaSets

### 테스트 링크

https://kodekloud.com/topic/practice-test-replicasets-2/

1. How many PODs exist on the system?
파드 개수를 구하는 문제
- kubectl get pods

1. How many ReplicaSets exist on the system?
복제셋 개수를 구하는 문제
- kubectl get rs

1. How many PODs are DESIRED in the `new-replica-set`?
새 복제셋에 파드가 몇개 필요한지 묻는 문제
- kubectl get rs 명령어를 치면 DESIRED 컬럼이 나옴.

```bash
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       2m5s
```

1. What is the image used to create the pods in the new-replica-set?
new-replica-set에 사용된 이미지는 무엇인지 묻는 문제
- kubectl get rs -o wide
    - -o wide 옵션을 붙이면 IMAGE 컬럼이 나옴
- 또는 kubectl describe replicaset new-replica-set 으로 상세 정보 확인

1. How many PODs are READY in the `new-replica-set`?
READY 상태인 파드 개수를 묻는 문제
- kubectl get rs에서 READY 컬럼 확인

1. Why do you think the PODs are not ready?
왜 준비중인 파드가 없는지 이유를 묻는 문제
- 복제셋에서 준비가 안되는 파드의 이유를 보려면 파드 하나를 선택해서 자세히 확인해야 한다.
    1. kubectl describe rs `new-replica-set`  복제셋의 파드를 확인
    2. kubectl describe pod <파드명>: 그 중 하나를 선택해서 자세하게 확인

1. Delete any one of the 4 PODs.
파드 삭제 문제
- kubectl describe rs `new-replica-set`  복제셋의 파드를 확인
- kubectl delete pod <파드명>: 그 중 하나를 선택해서 삭제

1. How many PODs exist now?
몇개의 파드가 남아있는지 묻는 문제
- kubectl get pods

1. Why are there still 4 PODs, even after you deleted one?
좀 전에 한 개를 삭제했는데도, 왜 여전히 4개의 파드가 있는지 묻는 문제
- **ReplicaSet ensures that desired number of PODs always run
복제셋은 항상 떠있기 원하는 파드의 개수를 보장한다.**

1. Create a ReplicaSet using the `replicaset-definition-1.yaml` file located at `/root/`.
/root/ 폴더 아래에 있는 `replicaset-definition-1.yaml` 을 이용하여 복제셋을 만들어라.
파일에 틀린 부분이 한 군데 있으니 고쳐서 만드는 문제
- yml 파일로 복제셋을 만들 때는 apiVersion이 그냥 v1이 아니라 apps/v1이어야 한다.
    - apiVersion: apps/v1 로 수정

1. Fix the issue in the `replicaset-definition-2.yaml` file and create a `ReplicaSet` using it.
`replicaset-definition-2.yaml` 파일의 틀린 부분을 수정하고 복제셋을 만드는 문제
- 파드 템플릿의 라벨과 복제셋 선택기에서 지정한 라벨이 동일해야 한다.

```bash
selector:
    matchLabels:
      tier: nginx # 동일 
  template:
    metadata:
      labels:
        tier: nginx # 동일
    spec:
      containers:
      - name: nginx
        image: nginx
```

1. Delete the two newly created ReplicaSets - `replicaset-1` and `replicaset-2`
두 복제셋을 삭제하는 문제
- kubectl delete rs <복제셋명> 을 두번 실행하여 각각 삭제
- 또는 kubectl delete rs <복제셋명>..<복제셋명> 여러개로 한번에 삭제

1. Fix the original replica set `new-replica-set` to use the correct `busybox` image.
busybox이미지를 이용하도록 new-replica-set을 수정하는 문제
- kubectl get rs -o wide 로 현재 복제셋의 이미지를 보면 busybox777을 사용 중임.
    
    ```bash
    NAME              DESIRED   CURRENT   READY   AGE   CONTAINERS          IMAGES       SELECTOR
    new-replica-set   4         4         0       41m   busybox-container   busybox777   name=busybox-pod
    ```
    
- kubectl edit rs <복제셋명> 을 실행하면 해당 복제셋을 수정하는 에디터가 나옴.
    - kubectl edit rs new-replica-set
    - 수정 후 저장
- 수정만 한다고 파드의 이미지가 수정되지 않는다. 기존 파드를 모두 삭제해야 수정한 이미지로 신규 파드가 생성됨.
    - 기존 파드 모두 제거
    - kubectl delete pod <파드명>…<파드명>
- 그리고 kubectl get pods를 해보면 모든 파드들이 RUNNING 상태인것을 확인

1. Scale the ReplicaSet to 5 PODs.
복제셋의 파드 복제 개수를 5개로 스케일업 하는 문제
- kubectl scale --replicas=5 rs new-replica-set

1. Now scale the ReplicaSet down to 2 PODs.
복제셋의 파드 복제 개수를 5개로 스케일 다운 하는 문제
- kubectl scale --replicas=2 rs new-replica-set