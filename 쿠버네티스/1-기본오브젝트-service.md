## Service

### Cluster Ip
Pod라는 존재는 장애에 따라 삭제되거나 재생성된다. Pod에 할당된 ip주소는 신뢰할 수 없다. 
Service에 접근할 수 있는 ip주소는 바뀌지 않는다. Service는 재생성되지 않는다. 
서비스의 ip주소로 접근하는 pod들은 항상 접근이 가능하다.

#### 사용자
- 인가된 사용자 (운영자)
- 내부 대쉬보드 관리
- Pod의 서비스 상태 디버깅

Service의 ip주소도 cluster 내에서 접근이 가능한 정보이다.
<img width="252" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/8a6e41e5-12b4-4c42-ae98-84847cd6b919">

### Node Port
- Service에 연결된 모든 node에 포트가 생성된다.
- 1번 노드에 접근을 하더라도, 다른 노드에 트래픽을 전달할 수 있다.
- externalTrafficPolicy : Local (특정 노드로 정해진 트래픽이라면 해당 노드에만 트래픽이 가게됨)
<img width="251" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/190542be-5918-48ef-8523-a3f084a717dd">

#### 사용
- 내부망 연결
- 데모나 임시 연결용

### Load Balancer
타 외부 플러그인을 사용한다.
<img width="270" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/443f6cbb-91fc-4586-a80e-8b4cbcd38784">

#### 사용
- 외부 시스템 노출용


## 실습

### Cluster IP
- Pod 생성
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
     app: pod
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
```

- service : default ClusterIP
```
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    app: pod
  ports:
  - port: 9000 // 9000번으로 들어오는 port 를 8080으로 엮어준다.
    targetPort: 8080
```
클러스터 ip로 접속되는 것도 pod로 전달해준다.
pod의 ip는 변해도 service의 ip는 변하지 않는다.
<img width="452" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/8675d0ba-db51-4757-b113-11ccdd703cae">

### NodePort
Cluster의 ip로 pod 시스템 연결

Pod 2생성
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
     app: pod
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node2
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
```

```
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30000
  type: NodePort
  externalTrafficPolicy: Local
```
똑같은 service ip로 접속을 해도 다른 pod이 할당된다.
<img width="435" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/acf02179-8937-45b9-b411-cdf55c7a7dcd">
<img width="1499" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/a3c481e8-9bde-4b6d-b2a3-d936139dd9c3">

만약 직접적으로 pod으로 접속할시, 없는 pod에 접속시 멈춘다.

### Load Balancer

```
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
  type: LoadBalancer
```

$ kubectl get service svc-3
<img width="617" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/9bd3f87b-65a4-4a3f-bc77-71d3894b7191">
외부 IP를 할당해주는 loadbalancer가 없으면 pending 상태가 됨
