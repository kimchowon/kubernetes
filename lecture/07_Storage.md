07.Storage
1. 파드 & 볼륨
파드에 볼륨을 마운트하면 컨테이너가 kill 되어도 볼륨안에 저장된 데이터는 남는다. 
볼륨 구현 방법
설정 파일안에 작성
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
​
컨테이너 내부 디렉토리 ←→ 볼륨 저장소를 마운트
영구 저장 볼륨 요청(PVC: Persistent Volume Claim)
사용하고 싶은 용량은 얼마인지, 읽기/쓰기는 어떤 모드로 하고 싶은지 등등을 정해서 PV를 사용하겠다는 요청
PV는 쿠버네티스의 영구 저장소 그 자체, PVC는 PV를 사용하겠다는 사용자의 요청
사용 가능한 PV가 없으면 PVC는 보류 상태로 남는다. 그러다가 사용 가능한 PV가 생기면 자동으로 바인딩됨. 

PV 설정 파일
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
​
PVC 설정 파일
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
	name: myclaim
spec:
	accessModes:
		- ReadWriteOnce
	resources:
		requests:
			storage: 500Mi
	hostPath:
		path: /tmp/data
​
PV를 먼저 생성하지 않고 PVC를 먼저 생성하면 PVC의 상태값은 Pending임. 
이후 accessModes나 용량 등이 일치하는 PV가 생성되면, 자동으로 바인딩됨. 
PVC 삭제했을 때 PV를 어떻게 처리할지에 대한 정책
persistentVolumeReclaimPolicy: Retain
PV는 계속 유지한다. 단, 다른 PVC에 의해 재사용될 수는 없음
persistentVolumeReclaimPolicy: Delete
PVC가 삭제될 때 PV도 같이 삭제한다. 
persistentVolumeReclaimPolicy: Recycle
PVC가 해제되어도, 다른 PVC에 의해 재사용될 수 있음. 
파드에 PVC를 정의하는 설정
apiVersion: v1
kind: Pod
metadata:
	name: pod
spec:
	container:
	- name: myfrontend
		image: nginx
		volumeMounts:
		- mountPath: "/var/www/html"
			name: mypd
	volumes:
	- name: mypd
		persistentVolumeClaim:
			claimName: myclaim
​
