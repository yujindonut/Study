## Volume

### emptyDir

Pod 생성시 만들어지고, 삭제시 없어짐

### hostPath

각 pod이 올라가있는 Node의 path를 Volume으로 사용
각각의 Node는 pod 자신이 할당되어있는 Host에 있는 데이터를 읽거나 써야할때
![image](https://github.com/yujindonut/Study/assets/78431728/36c42101-dc63-42b8-9c3c-63f56187db0c)

외부에 pod이 재생성 될때, Node에 해당 volume에 대한 내용이 없을 수 있다. 이경우, 직접 매니저가 리눅스를 이용해 연결을 시켜줄 수는 있지만 추천하지 않음.

### PVC / PV

![image](https://github.com/yujindonut/Study/assets/78431728/c4ae4550-0d71-42b6-a91b-2d0281e565de)

- PV (퍼시스턴트 볼륨(PersistentVolume)) : 클러스터 안의 자원을 다룬다. 파드와 별개로 관리되며 별도의 생명주기가 있다.
- PVC (퍼시스턴트 볼륨 클레임(PersistentVolumeClaim))  : 사용자가 PV에 하는 요청. 사용량은 얼마인지, 읽기/쓰기는 어떤 모드로 설정하고 싶은지를 요청한다.

쿠버네티스 볼륨을 파드에 직접 할당하는 방식이 아니라 중간에 PVC를 두어 파드와 파드가 사용할 스토리지를 분리하는 것. 파드 각각의 상황에 맞춰 다양한 스토리지를 사용할 수 있게 된다. 
클라우드 서비스를 사용할 때, 본인이 사용하는 클라우드 서비스에서 제공해주는 볼륨 서비스를 사용할 수도 있고, 직접 구축한 스토리지를 사용할 수도 있다.


### PV/PVC의 생명주기
![image](https://github.com/yujindonut/Study/assets/78431728/77ef0b7c-6f7b-4112-9e27-480e11ec6ff6)

1. Provisioning(프로비저닝)
PV를 만드는 단계를 프로비저닝이라고 한다. PV를 미리 만들어두고 사용하는 정책방법과 요청이 있을때마다 PV를 만드는 동적 방법이 있다.
   - 정적 프로비저닝
     - 정적으로 PV를 프로비저닝할 때는 클러스터 관리자가 미리 정적 용량의 PV를 만들어두고 사용자의 요청이 있으면 미리 만들어둔 PV를 할당한다.
     - 사용할 수 있는 스토리지 용량에 제한이 있을때 유용.
     - 사용하도록 미리 만들어둔 PV의 용량이 100GB라면 150GB를 사용하려는 요청들은 실패. 1TB 스토리지를 사용하더라도 미리 만들어 둔 PV 용량이 150GB 이상인 것이 없으면 요청이 실패.

   - 동적 프로비저닝
     - 사용자가 PVC를 거쳐서 PV를 요청했을 때 생성해 제공한다.
     - 쿠버네티스 클러스터에 사용할 1TB 스토리지가 있다면 사용자가 원하는 용량만큼 생성해서 사용할 수 있다.
     - 정적 브로비저닝과 달리 필요하다면 한번에 200GB PV도 만들어서 사용할 수 있다.
     - PVC는 동적 프로비저닝할 때 여러가지 스토리지 중 원하는 스토리지를 정의하는 스토리지 클래스로 PV를 생성한다.
 
2. 바인딩 (Binding)
바인딩은 프로비저닝으로 만든 PV를 PVC와 연결하는 단계. PVC에서 원하는 스토리지의 용량과 접근 방법을 명시해서 요청하면 거기에 맞는 PV가 할당괸다. 이때 PVC에서 원하는 PV가 없다면 요청은 실패한다. PVC는 원하는 PV가 생길 때까지 대기하다가 바인딩합니다.
PV와 PVC는 1:1 관계. PVC하나가 여러개의 PV에 할당될 수 없다.

3. Using (사용)
   PVC는 파드에 설정되고 파드는 PVC 볼륨으로 인식해서 사용한다.
   할당된 PVC는 파드를 유지하는 동안 계쏙 사용하며 시스템에서 임의로 삭제할 수 없다. 이 기능을 'Storage Object in Use Protection' 라고 합니다. 사용중인 데이터 스토리지를 임의로 삭제하는 경우 치명적인 결과가 발생할 수 있으므로 이런 보호 기능을 사용하는 것
   
4. Reclaming (반환)
  사용이 끝난 PVC는 삭제되고 PVC를 사용하던 PV를 초기화(reclaim)하는 과정을 거친다.

  초기화 정책
  - Retain
     PV를 그대로 보존한다. PVC가 삭제되면 사용중이던 PV는 해제 상태(released) 여서 다른 PVC가 재사용이 불가능하다.
     PV안의 데이터는 그대로 남아있으므로, PV를 재사용하려면 관리자가 다음 순서대로 직접 초기화를 진행해야한다.

     1. PV 삭제. 만약 PV가 외부 스토리지와 연결되었다면 PV는 삭제되더라도 외부 스토리지의 볼륨은 그대로 남아있다.
     2. 스토리지에 남은 데이터를 직접 정리
     3. 남은 스토리지의 볼륨을 삭제하거나 재사용하려면 해당 볼륨을 이용하는 PV 재생성
    
  - Delete
    PV를 삭제하고 연결된 외부 스토리지 쪽의 볼륨도 삭제한다. 프로비저닝할 때 동적 볼륨 할당 정책으로 생성된 PV들은 기본 반환 정책이 Delete이다. 상황에 따라 처음에 Delete로 설정된 PV의 반환 정책을 수정해서 사용해야한다.
  - Recycle
    PV의 데이터들을 삭제하고 다시 새로운 PVC에서 PV를 사용할 수 있도록 한다. 


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

```
   cd mount1
   echo "file context" >> file.txt
```

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

PersistentVolume
  3개의 PV를 생성

##### spec.accessMode : 볼륨의 읽기/쓰기 옵션 설정
ReadWriteOnce: 노드 하나에만 볼륨을 읽기/쓰기하도록 마운트할 수 있음
ReadOnlyMany: 여러 개 노드에서 읽기 전용으로 마운트할 수 있음
ReadWriteMany: 여러 개 노드에서 읽기/쓰기 가능하도록 마운트할 수 있음
ReadWriteOncePod : 볼륨이 단일 파드에서 읽기-쓰기로 마운트될 수 있다. 전체 클러스터에서 단 하나의 파드만 해당 PVC를 읽거나 쓸 수 있어야하는 경우 ReadWriteOncePod 접근 모드를 사용한다. 이 기능은 CSI 볼륨과 쿠버네티스 버전 1.22+ 에서만 지원된다.

##### spec.storageClassName : 스토리지 클래스(StorageClass)를 설정하는 필드. 
특정 스토리지 클래스가 있는 PV는 해당 스토리지 클래스에 맞는 PVC만 연결. PV에 **spec.storageClassName** 필드 설정이 없으면 **spec.storageClassName** 필드 설정이 없는 PVC와만 연결된다.
  
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-01
spec:
  capacity:
    storage: 1G
  accessModes:
  - ReadWriteOnce
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-node1]}

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-02
spec:
  capacity:
    storage: 1G
  accessModes:
  - ReadOnlyMany
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-node1]}

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

PersistentVolumeClaim

'spec.resources.requests.storage' : 자원을 얼마나 사용할 것인지 요청
필드 값을 설정할 때, PV의 용량을 초과하면, 사용할 수 있는 PV가 없으므로 PVC를 생성할 수 없는 Pending상태가 된다.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-01
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-02
spec:
  accessModes:
  - ReadOnlyMany
  resources:
    requests:
      storage: 1G
  storageClassName: ""
  selector:
    matchLabels:
      pv: pv-02

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-03
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 3G -> 3G로 배포를 진행한다고 하면 적합한 Volume이 없어서 할당이 되지 않는다.
  storageClassName: ""
  selector:
    matchLabels:
      pv: pv-03

```
- POD

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: pod-volume-3
  spec:
    containers:
    - name: container
      image: kubetm/init
      volumeMounts:
      - name: pvc-pv
        mountPath: /mount3
    volumes:
    - name : pvc-pv
      persistentVolumeClaim:
        claimName: pvc-01
  ```
  


#### Label 과 selector를 이용해 연결

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-04
  labels:
    pv: pv-04
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

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-04
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2G
  storageClassName: ""
  selector:
    matchLabels:
      pv: pv-04
```

<img width="745" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/8d8e0748-8760-4726-a56c-aa3fec173ba8">


----------------------------------------------------------------------------------------------

## ConfigMap, Secret

### Config Map이 필요한 이유

<img width="835" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/8ebd6378-7361-44a5-84de-e8aafd736d44">

- Dev : 보안접근이 false가 되어도 가능 & User도 dev

- Prod환경 : 보안 접근이 true가 되어야함.

환경마다 다른 Setting 값을 container 자원에 저장해 놓는 것은 비효율적이다.
따라서 이를 ConfigMap, Secret 에 저장해두고 관리한다.

<img width="791" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/a5413513-f10d-45f6-bf0d-e48ef9cc3e23">

Env값을 변경하면서 따로 효율적으로 관리가 가능하다.

- Env (Literal)

  <img width="282" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/ba97c112-fda1-4530-9dc9-b6b1d3b584a0">

- Env (File)

  <img width="275" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/8bef1d02-e370-4bbc-b95d-76c2b9476c70">
  Pod가 재생성되어야지만 file의 변경 내용이 바뀐것이 적용이 된다.

- Volume Mount (File)

  <img width="272" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/46b319dc-42a6-425b-80b0-72d1e4ab0d47">
 
  파일이 Mount 되는 것이므로 file의 변경사항들이 Pod에도 적용이 그대로 된다.

### 실습

- Env (Literal)
  환경변수로 저장한 값은 변경될 시에 적용이 바로 되지 않는다.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-dev
data:
  SSH: 'false'
  User: dev

apiVersion: v1
kind: Secret
metadata:
  name: sec-dev
data:
  Key: MTIzNA== // base64 인코딩한 값만 들어감

apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/init
    envFrom:
    - configMapRef:
        name: cm-dev
    - secretRef:
        name: sec-dev
```
- Env (File)

```
// config Map
echo "Content" >> file-c.txt
kubectl create configmap cm-file --from-file=./file-c.txt

// Secret
echo "Content" >> file-s.txt
kubectl create secret generic sec-file --from-file=./file-s.txt

// Pod
apiVersion: v1
kind: Pod
metadata:
  name: pod-file
spec:
  containers:
  - name: container
    image: kubetm/init
    env:
    - name: file-c
      valueFrom:
        configMapKeyRef:
          name: cm-file
          key: file-c.txt
    - name: file-s
      valueFrom:
        secretKeyRef:
          name: sec-file
          key: file-s.txt
```

- Volume Mount (File)
```
// Pod
apiVersion: v1
kind: Pod
metadata:
  name: pod-mount
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: file-volume
      mountPath: /mount
  volumes:
  - name: file-volume
    configMap:
      name: cm-file
```

----------------------------------------------------------------------------------------------

## Namespace, ResourceQuota, LimitRange 

#### 왜 사용해야하는가!

쿠버네티스 클러스터는 사용할 수 있는 자원이 할당되어있다.
클러스터 안에는 여러개의 namespace를 만들 수 있다. namespace 안에는 여러 pod를 만들 수있다. pod들은 클러스트 안의 자원을 공유해서 사용한다.
만약 한 namespace가 모든 자원을 다 사용하면 다른 Namespace의 자원들이 부족하게 됨.

- resource quota - namespace마다 최대 한계의 pod자원을 설정.
- limitRange : namespace안에 들어올 수 있는 pod의 자원을 제한할 수 있음

<img width="803" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/c37adc0c-49cd-4661-90db-72fc0979caf9">

### LimitRange

![image](https://github.com/yujindonut/Study/assets/78431728/7dee2e71-7ed0-4aac-b018-467a5e4e0b2d)

maxLimitRequestRatio.memory : request, limit의 배수가 3을 넘으면 pod이 통과하지 못함
pod이 접속시, reqeust, limit값을 설정해주지 않으면 자동으로 defaultReqeust와 default 값이 들어오게 됨


## 실습

### Namespace

한 namespace에는 같은 이름의 pod이 생성이 되지 않는다.
service 생성할때, namespace를 지정하지 않으면 pod이 해당 service 와 연결되지 않는다.

- name space
```
apiVersion: v1
kind: Namespace
metadata:
  name: nm-1

apiVersion: v1
kind: Namespace
metadata:
  name: nm-2
```

- Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-1
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080

apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-2
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: kubetm/init
    ports:
    - containerPort: 8080
```

- Service

```
apiVersion: v1
kind: Service
metadata:
  name: svc-1
  namespace: nm-1
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
```

```
curl 10.16.36.115:8080/hostname
curl 10.96.92.97:9000/hostname // service랑 Pod이랑 연결할때 
```

### Namespace Exception

```
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30000 -> nodePort는 namespace로는 나뉠 수 없는 부분. 다른 namespace에 이미 할당된 Nodeport는 namespace가 다르더라도 할당될 수 없다.
  type: NodePort
```

```
apiVersion: v1
kind: Pod
metadata:
 name: pod-2
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1 -> pod끼리 namespace가 달라도 공유하고 있는 파일 자원을 볼 수 있다.
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate
```

### ResourceQuota

```
apiVersion: v1
kind: Namespace
metadata:
  name: nm-3
```

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-3
spec:
  hard:
    requests.memory: 1Gi
    limits.memory: 1Gi
```

root master에서 Namespace 적용

```
kubectl describe resourcequotas --namespace=nm-3
```

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
spec:
  containers:
  - name: container
    image: kubetm/app

// resource 를 할당하지 않으면 에러가 나온다

apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.5Gi
      limits:
        memory: 0.5Gi
```

한계 -
pod가 생성되어있는 상태에서(resource에 대한 정보 없이) resourceQuata를 만들면, 생성이 된다.

### LimitRange

```
apiVersion: v1
kind: Namespace
metadata:
  name: nm-5

apiVersion: v1
kind: LimitRange
metadata:
  name: lr-1
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.4Gi
    maxLimitRequestRatio:
      memory: 3
    defaultRequest:
      memory: 0.1Gi
    default:
      memory: 0.2Gi
```

```
kubectl describe limitranges --namespace=nm-5
```

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.1Gi
      limits:
        memory: 0.5Gi // request와 limit의 비율이 5이상 차이가 나서, 오류가 남
```

#### 에러 상황

```
apiVersion: v1
kind: Namespace
metadata:
  name: nm-6

apiVersion: v1
kind: LimitRange
metadata:
  name: lr-5
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.5Gi
    maxLimitRequestRatio:
      memory: 1
    defaultRequest:
      memory: 0.5Gi
    default:
      memory: 0.5Gi

apiVersion: v1
kind: LimitRange
metadata:
  name: lr-3
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.3Gi
    maxLimitRequestRatio:
      memory: 1
    defaultRequest:
      memory: 0.3Gi
    default:
      memory: 0.3Gi

apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/app
```
한 namespace 안에 limit range를 여러개 생성하려고 하면, default request는 큰 것으로, default memory는 작은값으로 ~ 등의 이유로 pod 생성이 안될 수 있다.
limit range를 여러개 두는것은 위험하니 주의해서 작성해야한다.

ResourceQuota는 Namespace 뿐만 아니라 Cluster 전체에 부여할 수 있는 권한이지만, LimitRange의 경우 Namespace내에서만 사용 가능
