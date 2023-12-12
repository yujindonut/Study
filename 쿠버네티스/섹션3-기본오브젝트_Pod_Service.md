# Pod


### Container
- 컨테이너들이 port 번호가 중복될 수 없다.
- container에 휘발성의 ip주소가 자동 할당된다. 재생성시 변경된다.

<img width="139" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/845ed2ae-7772-45d3-b432-688351720d72">


### Label
<img width="222" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/89f333b1-a04c-4c6b-94bd-985cdf1ed54d">

- 사용목적에 따라 label 등록
Key : value
자신이 원하는 pod만 손쉽게 검색이 가능하다.

### Node Schedule
Pod은 한 Node에 스케줄링이 되어야한다.

직접 선택시 & 스케줄러가 판단해서 Node를 선택해주는 경우
<img width="268" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/161d4c4d-3fd5-44af-ad35-6adb07f292b3">

직접 선택해줄때

<img width="251" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/31be687e-4476-458f-8fbd-73122b4c3e99">


------------------------------------------------------------------------------------------

## POD 실습 
1. https://192.168.64.30:30000/#/login 접속
2. Pod 생성
   ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-1
    spec:
      containers:
      - name: container1
        image: kubetm/p8000
        ports:
        - containerPort: 8000
      - name: container2
        image: kubetm/p8080
        ports:
        - containerPort: 8080
    ```
3. master에 Pod 연결
   master로 가서 curl [ip주소]:8080
   <img width="782" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/51373353-c726-4c6d-84cd-0924b0cb7c27">
4. container에 연결이 잘되었는지 확인

Label
1. Pod 생성시 label 설정
  ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-2
      labels:
        type: web
        lo: dev
    spec:
      containers:
      - name: container
        image: kubetm/init
  ```
2. Service
   selector 에 따른 label 이 붙은 pod만 필터
   ```
     apiVersion: v1
    kind: Service
    metadata:
      name: svc-1
    spec:
      selector:
        type: web
      ports:
      - port: 8080
   ```

- 노드 스케줄링
  1. 자원량에 따라 node에 점수가 매겨진다.
  2. pod을 생성하면 자원이 많이 남는 노드(점수가 높은 node)에 스케줄링이 된다.

-------------------------------------------------------------------------------------------------------

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

-----------------------------------------------------------------------

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

-----------------------------------------------------------------------

## Service 실습

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

```
$ kubectl get service svc-3
```

<img width="617" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/9bd3f87b-65a4-4a3f-bc77-71d3894b7191">

외부 IP를 할당해주는 loadbalancer가 없으면 pending 상태가 됨
