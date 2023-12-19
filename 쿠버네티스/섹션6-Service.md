# Headless, Endpoint, ExternalName

## Pod의 입장에서 연결 및 외부 서비스에 안정적인 연결방법

#### DNS 서버를 기본적으로 이용한다.
- 쿠버네티스 클러스터 안에 존재하는 서비스의 이름과 IP가 저장되어 있는 존재.
- Pod가 Service에 대한 도메인을 질의하면 해당 Service Ip를 알려주게된다.
- Pod가 유저를 찾으려고 할때, 쿠버네티스 내부 뿐만 아니라, 상위 DNS인 내부망 DNS, 외부 네트워크 DNS까지 조회가 가능하다.

- FQDN(Fully Qualified Domain Name)으로 구성되어 있다.
  - Pod의 경우 질의 이름에 IP address가 포함되어 있고 질의에 모든 FQDN을 입력해야하기 때문에 사용하기 어렵다.
  - Service의 경우 앞의 서비스명만으로 질의가 가능하다
    
## Headless
- Pod간 연결을 위해 사용하는 방법
- Cluster Ip가 None인 서비스여야한다.  (Service에 해당하는 pod을 만들지 않겠다는 의미)
- DNS Server에 Pod의 이름과 Service의 이름이 붙어져서 도메인 이름으로 등록됨
- 다른 Pod에 접근하기 위해 IP주소를 알 필요없이, 도메인 이름으로 접속가능하다.

### EndPoint 오브젝트

![image](https://github.com/yujindonut/Study/assets/78431728/0ab2b0ca-5684-4716-b93a-ff733909e1a3)
- 사용자의 입장에서는 Service 와 Pod을 연결할때, label을 지정한다.
- 쿠버네티스상 안에서는 Service와 Pod의 연결을 위해 Endpoint를 지정해서 연결고리를 관리한다.
- Endpoint는 Service와 동일한 이름을 가지고 있고 Pod의 IP 정보도 담고 있다.
- Endpoint를 직접 만드는 것으로 label, Selector를 만들지 않고도 연결이 가능하다.
- Pod뿐만 아니라 외부의 IP주소를 입력하여 외부와도 연결이 가능하다.

### External Name
- 외부 사이트에서 데이터를 가지고 오려는 경우 사용
  - EndPoint로도 외부와 연결은 가능하지만, IP가 변동될 가능성이 있어 hostname으로 연결하는 방법이다.
- 쿠버네티스 DNS Server로 부터 상위 DNS서버를 계속 거쳐 외부 네트워크로 접근하는 방법
- Pod는 Service만 가르키면 되고, Service 에서 필요할 때마다 도메인 주소 변경이 가능하다. -> 외부 대상 IP가 변경이 될때마다 Pod을 수정하고 재배포 하지 않아도 된다.

----------------------------------------------------------------------------------------------------------------------------------------------------------

# Volume - Dynamic Provioning, Recliam Policy, StorageClass, Status

## Dynamic Provioning

Volume은 데이터를 안정적으로 사용하기 위해 필요한 Object이므로 쿠버네티스와 분리되어 관리한다.
이러한 Volume을 사용하기 위해서는 내부/외부망에 존재하는 Volume을 사용하기 위해서는 관리자가 PV를 만들고 사용자가 원하는 용량과 AccessMode로 PVC를 만들면 쿠버네티스 측에서 적절한 PV와 연결된다.

Volume이 사용방법에는 필요할때마다 PVC(PersistentVolumeClaim), PV(PersistentVolume)를 만들어야한다.
-> 사용방법이 복잡하다.
-> 동적 PV 생성을 이용한 간단한 Volume사용이 필요하다.

- 사용자가 PVC를 만들면 알아서 PV를 만들고 Volume과의 연결을 진행한다.
- 모든 PV에는 상태가 존재, PVC와의 연결 상태 또는 에러 유무를 확인가능하다.
  - Status
  - Policy 정책 존재

* 이러한 동적 PV 생성을 위해서는 Storage클래스가 필요하다.
  - 사용자가 직접 PV를 생성하는 경우에는 PVC의 StorageClassName항목에 ""을 기입하면 적절한 PV에 연결이 되었다.
  - StorageClassName 항목에 사전 생성된 StorageClass Object를 기입하는 것으로 동적 PV가 생성 가능하다.
  - StorageClass는 추가적으로 생성이 가능하고 Default 생성이 가능하다.
    - Storage name을 비워놓으면("" X) default 클래스가 할당이 된다.

## Status & Reclaim Policy

Pod가 삭제되면 PVC, PV의 데이터에는 문제가 생기지 않는다.
PVC가 삭제되면 PV의 policy에 따라서 상태가 달라진다.

PV는 상황에 따라 여러 상태로 나뉘게 된다.

### PV status
- Available : 최초 PV가 만들어진 상태
- Bound : PVC와 연결된 상태
- Released : 연결된 PVC가 삭제된 상태
- Failed : PV와 실제 데이터간 연결의 문제 발생

PV가 Released 상태일때, Reclaim Policy에 따라 PV에 대한 상태가 달라진다.

#### Reclaim Policy
- 연결된 PVC가 삭제되었을 때, PV 상태 변화를 위한 정책

1. Retain (default값)
   - PVC가 삭제되면, PV의 상태가 Released가 되는 옵션 
   - 실제 볼륨 데이터는 유지
   - 해당 PV를 다른 PVC에 연결이 불가능 (재사용이 불가능)
   - 수동으로 PV를 삭제해줘야함
  
2. Delete
   - PVC가 삭제되면, PV도 같이 지워지는 옵션
   - StorageClass로 생성된 PV의 기본정책
   - 볼륨의 종류에 따라, 실제 데이터가 삭제되기도 유지되기도 함. 재사용 불가

3. Recycle
   - PVC가 삭제되어도 PV의 상태가 Available이 되면서 다시 PVC에 연결할 수 있는 상태로 변경
   - 실제 데이터가 삭제됨
   - 재사용이 가능하다 (PV가 availabe 상태가 된다)
   - 사용권장X (Deprecated됨)

------------------------------------------------------------------------------------------------------------------------------------------------------

# Accessing API 

- Authentication
: 접속한 사람의 신분을 시스템이 인증하는 단계
![image](https://github.com/yujindonut/Study/assets/78431728/574db12d-672f-40b3-aeff-fabf612d798e)

- Authorization
: 어떤 권한을 가지고 어떤 행동을 할 수 있는지 확인하는 단계 (사용자별 권한 확인)

- Admission Control
: 인증과 권한 확인 이후에 추가적인 요청 내용에 대한 검증 또는 요청 내용 강제 변경
--------------------------------------------------------------------------------------------------------------------------------------------------------

# Authentication

## X509 Client Certs

쿠버네티스 클러스터에는 6443 포트로 API 서버가 열려있다. 
사용자가 이곳으로 https 접근하는 방법이며, Kubernetes 설치시, kubeconfig 파일 안의 인증서 내용으로 접근

![image](https://github.com/yujindonut/Study/assets/78431728/4940b825-7347-4eff-b51d-781dd416dfbe)


- kubeconfig에 Clinet crt, Client key를 사용자가 복사하여 kubernetes API Server에 접근 가능
- 이러한 kubeconfig는 kubectl이 설치되면서 내부에 복사됨
- kubectl내부에 복사된 kubeconfig을 이용하여 kubectl에서 Kubernetes API Server에 접근하여 리소스 조회가능
- kubectl에 accept hotst 기능으로 8001 포트를 열어두면 외부에서도 https로 접근 가능
- kubectl 내부에 Kubeconfig로 인증서를 가지고 있기 때문에 사용자는 아무런 인증없이 접근 가능


## Kubectl
외부 서버에 kubectl를 설치하여 멀티 클러스터접근
사전에 각 cluster에 대한 kubeconfig가 외부서버의 kubectl에 존재해야한다.
사용자는 외부 서버의 kubectl에 존재하는 kubeconfig를 통해 원하는 클러스터에 접근 가능

![image](https://github.com/yujindonut/Study/assets/78431728/dd2a2510-49d7-4d39-bb3f-a0aa7d378d73)

- kubeconfig의 내용 및 접근 방법
  - kubeconfig 내의 항목
    - cluasters : 클러스터 등록
    - users : 사용자 등록
    - contexts : cluster와 users를 연결, 이렇게 연결된 context를 이용해서 사용자가 접근가능하다.

## Serivce Account

![image](https://github.com/yujindonut/Study/assets/78431728/7da0cfcf-a634-49c6-bd68-13a512d7badc)
네임 스페이스 내부의 Secret의 token 값으로 사용자가 API Server에 접근 가능한 방법
- namespace 생성시, 자동적으로 Service account가 생성된다
  - 이러한 Service Account에는 Secret 값이 존재한다
  - 내부에 CA crt와 토큰 값이 존재한다.
- namespace 내부에 pod를 생성하면 Service Account와 연결되며, Pod는 Secret의 token값을 통해 API 서버에 접근이 가능하다.
--------------------------------------------------------------------------------------------------------------------------------------------------------

# RBAC, Role, RoleBinding(Authorization)

![image](https://github.com/yujindonut/Study/assets/78431728/05c2a76b-8555-4037-b8bb-6a7729ccb1ba)

## RBAC(Role-Based Access Control)

쿠버네티스가 자원에 대한 권한을 지원하는 방법에는 여러가지 방법 중 대표적인 방법
역할 기반으로 권한을 부여하는 방법으로 Role, RoleBinding 오브젝트를 사용한다.

- 누가, 무엇을, 어디에(namespace) 실행할 수 있는지 결정하는 권한 또는 템플릿 집합을 수반하는 identity 및 액세스 관리 형식
- 기존의 속성 기반 액세스 제어에서 발전한 형태

## Role과 RoleBinding
Namespace에 있는 ServiceAccount의 RoleBinding을 어떻게 설정하느냐에 따라 Namespace 내에 있는 자원만 접근할 수도 있고, 클러스터에 있는 자원까지 접근할 수 있다.

### 1. Namespace 내의 권한
![image](https://github.com/yujindonut/Study/assets/78431728/b6539bb1-56c9-42de-b2bc-d30756e4caa7)

- Role
  - Role은 여러개 만들 수 있고, Namespace에 있는 자원에 대해 Read, Write 권한을 줄 수 있다.
    
- RoleBinding
  - Role과 Service Account를 연결해주는 역할
  - RoleBinding은 하나의 Role만 연결할 수 있고, ServiceAccount는 여러개 지정가능
  - RoleBinding에 연결된 ServiceAccount들은 RoleBinding에 연결된 Role의 역할을 가지게 되어 API 서버에 접근이 가능하다.

Namespace 내부의 Role, RoleBinding 구성하였을때, Secret의 Token값으로 외부에서 API 서버에 접근 가능하다. 또한 Token의 권한에 따라 Namespace에서 인가된 권한만 행사 가능하다. 

### 2. ServiceAccount에서 Cluster 자원에 접근하는 권한
![image](https://github.com/yujindonut/Study/assets/78431728/79058473-db4a-4952-9efd-00c1013d20ef)

- ClusterRole
  - 클러스터 단위의 오브젝트들을 지정할 수 있다.
  - ClusterRoleBinding이 아닌 Namespace내부의 RoleBinding과도 연결 가능하다.
    - 일반 Rold와 RoleBinding을 사용하는 것과 동일하게 동작 -> 클러스터 자원에 접근 불가능하다.
    - 동일한 Role을 부여하고 관리할 때 한번에 쉽게 관리된다.
      - 한번의 변경으로 모든 namespace에 부여된 role이 변경된다 (관리가 용이해짐)
- ClusterRoleBinding
  - Namespace안에 있는 ServiceAccount를 연결해주면, ServiceAccount가 클러스터 안에 있는 자원에 접근할 수 있게 된다.

- Cluster Role/RoleBinding 구성하였을 때, Secret Token값으로 외부에서 API 서버에 접근이 가능
- 해당 네임스페이스와 다른 네임스페이스의 자원, 클러스터 단위의 자원에서도 조회나 생성 가능하다.


### 실습
```
kubectl create ns nm-01
```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: r-01
  namespace: nm-01
rules:
- apiGroups: [""]
  verbs: ["get", "list"]
  resources: ["pods"] // Pod에 대한 Get권한만 부여함

```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rb-01
  namespace: nm-01
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: r-01
subjects:
- kind: ServiceAccount
  name: default
  namespace: nm-01
```

#### Cluster Role 
- Role과는 다르게 namespace를 지정하지 않는다. 모든 자원에 대해 모든 권한 부여한다.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cr-02
rules:
- apiGroups: ["*"]
  verbs: ["*"]
  resources: ["*"]
```

ClusterRole과 ServiceAccount를 연결하는 ClusterRoleBinding을 생성한다.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: rb-02
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cr-02
subjects:
- kind: ServiceAccount
  name: sa-02
  namespace: nm-02
```

