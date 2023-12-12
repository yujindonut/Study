### Namespace, ResourceQuota, LimitRange 왜 사용해야하는가!

쿠버네티스 클러스터는 사용할 수 있는 자원이 할당되어있다.
클러스터 안에는 여러개의 namespace를 만들 수 있다. namespace 안에는 여러 pod를 만들 수있다. pod들은 클러스트 안의 자원을 공유해서 사용한다.
만약 한 namespace가 모든 자원을 다 사용하면 다른 Namespace의 자원들이 부족하게 됨.

resource quota - namespace마다 최대 한계의 pod자원을 설정.
limitRange : namespace안에 들어올 수 있는 pod의 자원을 제한할 수 있음

<img width="803" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/c37adc0c-49cd-4661-90db-72fc0979caf9">

LimitRange
![image](https://github.com/yujindonut/Study/assets/78431728/7dee2e71-7ed0-4aac-b018-467a5e4e0b2d)

maxLimitRequestRatio.memory : request, limit의 배수가 3을 넘으면 pod이 통과하지 못함
pod이 접속시, reqeust, limit값을 설정해주지 않으면 자동으로 defaultReqeust와 default 값이 들어오게 됨


## 실습

### Namespace

한 namespace에는 같은 이름의 pod이 생성이 되지 않는다.
service 생성할때, namespace를 지정하지 않으면 pod이 해당 service 와 연결되지 않는다.

// name space   
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

// Pod
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

// Service

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

ResourceQuota는 Namespace 뿐만 아니라 Cluster 전체에 부여할 수 있는 권한이지만, LimitRange의 경우 Namespace내에서만 사용 가능합니다.

