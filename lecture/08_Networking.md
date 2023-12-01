# 08.Networking

### 1. 라우팅

- 라우터: 다른 네트워크 간에 경로를 연결하는 장치
- 시스템에서 라우팅 구성 확인 명령어: ip route
- 라우터에 경로 추가 명령어
    - ip route add <도착지IP> <출발지IP>
    - ex) ip route add 192.168.2.0/24 via 192.168.1.1
    - ex) ip route add default via 192.168.1.1 / 또는 ip route add 0.0.0.0 via 192.168.1.1
        - 도착지에 대한 정보가 라우터에 없는 경우 default 게이트웨이를 통하게 한다.

### 2. 네트워크 네임스페이스

- 네트워크 네임스페이스: 하나의 호스트에서 여러 개의 격리된 네트워크 공간을 형성하는 기술
- 컨테이너가 생성되면, 해당 컨테이너를 위한 네트워크 네임스페이스가 생성됨.
    - 각 네트워크 스페이스는 자기만의 veth, 라우팅 테이블, ARP 테이블을 가지고 있다.
        - veth(가상 이더넷 인터페이스):
            - 로컬 이더넷 터널
            - 자신과 연결된 다른 네트워크 네임스페이스와 패킷을 주고 받는 식으로 동작
        - 라우팅 테이블: 다른 네트워크 간에 경로를 연결 정보를 저장하는 테이블
        - ARP 테이블
            - Adress Resolution Protocol
            - IP 주소를 MAC 주소와 매칭하는 정보를 저장하는 테이블
    - 호스트에 네트워크 네임스페이스 추가하는 명령어
        - ip netns add <네트워크 네임스페이스명>
            - ip netns add red
    - 네트워크 네임스페이스 목록 조회
        - ip netns
    - 특정 네트워크 네임스페이스 내에서 명령어 실행
        - ip netns exec <네트워크네임스페이명> <실행 명령어>
            - ex) ip netns exec red ip link
    - 가상 케이블을 이용한 호스트내 네트워크 네임스페이스간 연결
        1. ip link add <가상 케이블명A> type veth peern name <가상 케이블명B>
        2. B에도 A를 따로 설정해줘야 함.
        3. 가상 케이블과 네크워크 네임스페이스를 연결
            1. ip link set veth-blue netns blue
        4. 네트워크 네임스페이스에 ip 주소 할당
            1. ip -n red addr add 192.168.15.1 dev veth-red
        5. 네트워크 네임스페이스에 네트워크 인터페이스 설치
            1. ip -n <네트워크 네임스페이스> link set <가상케이블명> up
            2. ex) ip -n red link set veth-red up
        6. A 네트워크 네임스페이스에서 B 네트워크 네임스페이스의 ip로 ping 날리기
            1. ip netns exec <A 네트워크네임스페이스명> ping <B ip>
            2. ex) ip netns exec red ping 192.168.15.2
- 하나의 호스트에 격리된 네트워크 네임스페이스가 많을 때
    - 네트워크 네임스페이스가 단 2개일 때는 1대1 매핑으로 연결해주면 되지만, 여러개일 때는 중앙에 가상 스위치를 하나 만들고 네트워크 네임스페이스들을 해당 스위치에 연결
    - 생성 및 연결 순서
        1. 호스트내 가상 스위치(브릿지) 추가
            1. ip link add <브릿지명> type bridge
        2. 가상 스위치 활성화
            1. ip lilnk set dev <브릿지명> up
        3. 네임스페이스들과 스위치 연결
            1. 네트워크 네임스페이스를 연결할 케이블 생성
                1. ip link add <케이블 말단 A> type veth peer name <케이블 말단 B>
                2. 케이블 말단A(네트워크 네임스페이스에 붙임) ←→ 케이블 말단B(가상 스위치에 붙임)
            2. 네트워크 네임스페이스와 케이블 연결
                1. ip link set <케이블 말단A> netns <네트워크 네임스페이스>
                2. ip link set <케이블 말단B> master <가상 스위치>
        4. 네트워크 네임스페이스에 ip 주소 할당
            1. ip -n <네트워크 네임스페이스> addr add <ip> dev <케이블>
        5. 장치 활성화
            1. ip -n <네트워크 네임스페이스> link set <케이블> up
        6. 호스트와 가상 스위치 ip 연결
            1. ip add add 192.168.15/24 dev <가상 스위치명>
    

### 3. 도커 네트워킹

- 도너캐 컨테이너 네트워크 설정 종류
    1. 도커 내 컨테이너에 아무런 네트워크 설정도 하지 않고 실행
        1. docker run —network none nginx
    2. 호스트 네트워크를 사용
        1. docker run —network host nginx
        2. 호스트의 ip와 port를 그대로 사용한다.
    3. 브릿지
        1. 컨테이너에 사설 네트워크 생성하고 내부 브릿지에 연결
- 브릿지 설정
    - 도커에서 브릿지 존재 여부 확인: docker network ls 또는 ip link
    - 도커 내에 브릿지를 추가
        - ip link add <브릿지명> type bridge
    - 브릿지에 할당된 ip 주소를 확인
        - ip addr 명령어를 통해 브릿지에 해당하는 정보를 확인
    - 도커 내 컨테이너 실행하여 네트워크 네임스페이스 생성
        - 컨테이너를 실행하면 네트워크 네임스페이스는 자동으로 생성됨.
        - docker run nginx
        - 도커내 생성된 네트워크 네임스페이스 조회: ip netns
    - 브릿지 설정을 끝내도 여전히 호스트 내에서만 컨테이너 ip로 접속 가능함. 외부에서 컨테이너에 접속할 수 없음.
        - 포트 매핑 필요!
            - 컨테이너=ip: 172.17.0.3, port: 80
            - 호스트= ip: 192.168.1.10, port: 8080
            - docker run -p 8080:80 nginx
        - 이후 외부에서 8080 포트로 접속
            - curl http://192.168.1.10:8080

### 4. Container Networking Interface(CNI)

- CNI
    - 컨테이너 런타임 환경에서 네트워킹 문제를 해결하기 위해 프로그램이 어떻게 개발되어야 하는지 정의한 표준
    - docker는 자체 표준인 CNM이 있어서 CNI를 따르지 않음

### 쿠버네티스 네트워크 명령어 TIP

- 노드 ip 주소 조회
    - k get nodes -o wide
- ip에 연결된 네트워크 인터페이스 조회
    - ip address
    - ip link
- 노드에 매핑된 MAC 주소 조회
    1. 네트워크 인터페이스로 조회
        - ip address 로 네트워크 인터페이스 먼저 조회
        - ip link show <네트워크 인터페이스명>
        - ex) ip link show eth0
    2. ssh 로 노드에 직접 접속해서 조회
        1. ssh <노드명>
        2. ip address
- 쿠버네티스 시스템들이 노드의 어떤 포트로 통신하고 있는지 조회
    - ex) kube-scheduler인 경우
    - netstat -npl | grep -i scheduler