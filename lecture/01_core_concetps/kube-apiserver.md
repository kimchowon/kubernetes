# kube-apiserver

### kube-apiserver

- 클러스터 내에서 ECTD 스토리지, 스케줄러, 컨트롤러 등의 구성 요소들의 통신 담당하고 오케스트레이션하는 역할

### kubectl 명령어를 실행하여 데이터를 조회하는 과정

1. kubectl 명령어를 실행, kube-apiserver로 도달
2. kube-apiserver가 요청을 인증하고 유효성을 검사
3. 인증및 유효성 검사에 통과하면 ETCD 스토리지에서 데이터를 조회하여 응답으로 반환
- kubectl 명령어 실행 없이 직접 kube-apiserver API를 호출하는 방법도 있음.

### 파드 생성 kube-apiserver API 호출 예시

1. 파드 생성 api 호출
    
    ```bash
    curl -X POST /api/v1/namespaces/defult/pods
    ```
    
2. 요청 인증 및 유효성 검사
3. 파드 생성
4. ETCD 스토리지에 데이터 업데이트
5. 사용자에게 파드 생성 성공 여부 응답
6. kube-schedular가 kube-apiserver를 지속적으로 모니터링, 신규 파드가 생성된 것을 확인
7. kube-schedular 신규 파드를 배치할 적절한 노드를 판단하여 kube-apiserver에 전달
8. kube-apiserver가 다시 ETCD 스토리지에 데이터를 업데이트
9. kube-apiserver가 해당 노드의 kubelet과 통신
10. kubelet이 노드안에 파드를 생성하고 컨테이너 런타임 엔진에게 지시하여 이미지를 배포
11. 작업을 수행 후에 kubelet이 다시 kube-apiserver와 통신하고, kube-apiserver는 ETCD 스토리지를 업데이트