# 05.Cluster Maintenance

Cluster Maintenance: 클러스터 유지관리

### 1. Operating System Upgrade

시나리오

- 클러스터의 일부 노드가 갑자기 죽는 상황
- 노드가 즉시 다시 복구되면 안의 파드도 활성화 됨.
- 하지만 노드가 5분 이상 복구되지 않을 경우 그 안에 파드도 종료됨.
    - 옵션: —pod-eviction-timeout (default: 5분)
- 5분이 지나서 노드가 복구되는 경우 안에 파드는 모두 사라진채로 복구됨.

시스템 업그레이드 도중 특정 노드를 내릴 일이 있다면 해당 노드의 작업들을 다른 노드에서 실행 가능하도록 운영. 심리스하게 노드를 내릴 수 있음.

- 특정 노드 내리는 명령어: kubectl drain node-1
- 노드가 다시 복구되었을 때 파드가 다시 바인딩 되도록 준비하는 명령어: kubectl uncordon node-1
- 특정 노드를 내리지는 않으나 이후부터는 새로운 파드가 바인딩 되지 않도록 하는 명령어:
kubectl cordon node-2

### [Practice Test]

### 1. [**OS Upgrade**](https://kodekloud.com/topic/practice-test-os-upgrades-2/?utm_source=udemy&utm_medium=labs&utm_campaign=kubernetes)

1. Try draining the node again using the same command as before: kubectl drain node01 —ignore-daemonsets
    1. 해석: kubectl drain node01 —ignore-daemonsets 명령을 실행해보는 문제
    2. 풀이:
        1. 위 명령어를 실행하면 다음과 같은 에러 메시지가 나온다. 
        cannot delete pods not managed by replicationController, ReplicaSet, Job, DaemonSet or StatefulSet
        2. 복제셋이 없는 단일한 파드는 삭제할 수 없다는 뜻
2. Why did the drain command fail on node01? It worked the first time!
    1. node01 노드를 내리는 것이 불가능한 이유
    2. 11번의 해석대로 노드내에 복제셋이 없는 단일한 파드가 존재하기 때문