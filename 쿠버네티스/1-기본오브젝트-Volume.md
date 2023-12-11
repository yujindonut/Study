## Volume

### emptyDir

Pod 생성시 만들어지고, 삭제시 없어짐


### hostPath

각 pod이 올라가있는 Node의 path를 Volume으로 사용
각각의 Node는 pod 자신이 할당되어있는 Host에 있는 데이터를 읽거나 써야할때
![image](https://github.com/yujindonut/Study/assets/78431728/36c42101-dc63-42b8-9c3c-63f56187db0c)
외부에 pod이 재생성 될때, Node에 해당 volume에 대한 내용이 없을 수 있다. 이경우, 직접 매니저가 리눅스를 이용해 연결을 시켜줄 수는 있지만 추천하지 않음.

### PVC / PV
쿠버네티스가 자동으로 연결해준다. capacity와 accessMode를 보고
![image](https://github.com/yujindonut/Study/assets/78431728/c4ae4550-0d71-42b6-a91b-2d0281e565de)


## 실습

1. empty Dir
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-1
spec:
  containers:
  - name: container1
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount1
  - name: container2
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount2
  volumes:
  - name : empty-dir
    emptyDir: {}
```
2. mount | grep mount1 : mount가 된 폴더인지
   cd mount1
   echo "file context" >> file.txt

pod 삭제후, 두 container가 파일을 공유함. pod가 삭제되면 파일이 삭제되기에 삭제되도 괜찮은 것들만 삭제함


### hostPath

pod를 삭제해도, hostPath의 데이터는 유지가된다. 

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-3
spec:
  # nodeSelector:
    # kubernetes.io/hostname: k8s-node1 -> node를 지정해주지 않으면 다른 node에 할당되어 공유파일이 보이지 않음.
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate
```

### PVC / PV
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-03
spec:
  capacity:
    storage: 2G
  accessModes:
  - ReadWriteOnce
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-node1]}
```
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-02
spec:
  capacity:
    storage: 2G
  accessModes:
  - ReadOnlyMany
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-node1]}

```
