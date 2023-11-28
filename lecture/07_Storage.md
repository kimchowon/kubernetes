# 07.Storage

### 1. 파드 & 볼륨

- 파드에 볼륨을 마운트하면 컨테이너가 kill 되어도 볼륨안에 저장된 데이터는 남는다.
- 볼륨 구현 방법
    - 설정 파일안에 작성
        
        ```yaml
        apiVersion: v1
        kind: Pod
        ..
        spec:
         containers:
          - image: alpine
            name: alpine
            command: ["/bin/sh", "-c"]
            args: ["shuf -i 0-100 -n 1 >> /opt/number.out"]
            volumeMounts: # 컨테이너 내부 디렉토리
            - mountPath: /opt
              name: data-volume
        
         volumes: # 볼륨 저장소
         - name: data-volume
           hostPath:
            path: /data
            type: Directory
        ```
        
    - 컨테이너 내부 디렉토리 ←→ 볼륨 저장소를 마운트
- 영구 저장 볼륨(PVC: Persistent Volume Claim)
    - 클러스터내 파드가 여러개일 때 각 파드끼리 영역을 나눠 마운트할 수 있는 중앙 집중 저장소
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2356c823-06cb-4e16-be3a-7eaa83f4a147/0fc8e150-854c-4ad4-95a9-6f1a98ab2152/Untitled.png)
        
    - 설정 파일
        
        ```yaml
        apiVersion: v1
        kind: PersistentVolume
        metadata:
        	name: pv-voll
        spec: 
        	accessModes:
        		- ReadWriteOnce
        	capacity:
        		storage: 1Gi
        	hostPath:
        		path: /tmp/data
        ```