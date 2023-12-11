## Pod

### Kubernetes Cluster
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

- 직접 선택해줄때
<img width="251" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/31be687e-4476-458f-8fbd-73122b4c3e99">


------------------------------------------------------------------------------------------

#### 쿠버네티스 실습 
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
