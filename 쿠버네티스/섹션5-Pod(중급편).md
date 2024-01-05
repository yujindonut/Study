## Pod - Lifecycle

### Pod의 Status
1. Pending (Pod 의 최초 상태)
   - podScheduled -> initialized가 진행된다.
   - podScheduled : 어떤 node에 배치 될건지
   - pod이 initContainer 될때, container는 waiting 상태
   - container의 상태가 되면, containerReady: false, Ready: true가 상태됨
   - pod이 문제없이 보여도, 안의 container도 문제가 없는지 확인해야한다.
2. Running
   Waiting : CrashLoopBackOff
   ContainerReady: True
   Ready : True
   
3. Succeeded
4. Failed
5. Unknown
   통신장애가 네트워크 오류가 났을때 pending -> unknown 상태가 되기도함.

### ReadinessProbe, Livenessprobe
Pod의 상태를 확인하고 Pod가 정상적으로 작동하는지 결정하는데 사용한다. 유사하게 동작하지만, 목적과 사용방법이 다르다.

#### LivenessProbe
- Pod의 생명주기 중 Running 상태에서의 동작, 서비스 요청에 응답 가능한지 확인.
pod가 살아있는지 여부를 확인한다. Pod가 이 probe를 실패하면 Kubernetes는 pod를 삭제하고 다시 시작한다. 
이를 통해 시스템의 다른 부분에 문제를 일으키지 않고 문제가 있는 컨테이너를 다시 시작할 수 있다.

#### ReadinessProbe
- Pod의 생명주기 중 Pending 상태에서의 동작, 서비스 요청에 응답 가능한지 확인.
Pod가 이 probe를 실패하면 kubernetes는 pod를 서비스에서 제외하고, 다른 pod으로 요청을 전달한다. (App 구동 중 트래픽이 흐르지 않게 하기 위해)
이를 통해 요청을 처리할 준비가 되지 않은 Pod에 대한 요청을 방지할 수 있다.
pod가 실행이 되고, Probe가 성공하기 전에는 containerReady, Ready값이 false가 된다.
만약 이 상황이 지속되면 endpoint에서는 NotReadyAddr상태라고 서비스와 연결을 하지 않는다.
container상태가 ready가 된다면 initialDelaySecond 수만큼 체크하게 됨.

-> 두가지를 모두 사용하는 것이 권장된다. Pod상태를 조금 더 정확하게 파악하기 위해서.

1. 하나의 Service에 트래픽이 node1, node2에 50%, 50%의 확률로 가고 있는 중.
2. Node2에 앱이 다운이 되어서 팟이 Failed가 됨. 남은 트래픽이 모두 node1으로 가게됨
3. Pod2는 다른 node에 autohealing 기능으로 인해서 띄어지게 되고, app이 부팅중일때 50%의 트래픽이 pod2로 가게됨
4. 이럴 경우를 방지하기 위해서 ReadlinessProbe를 둠. (App이 구동 순간에 트래픽이 오지 않도록 설정값 지정)
5. Tomcat은 돌고있지만, 안의 service가 문제가 생길 수 있다. app에 대한 장애 상황을 파악하는 것이 Livenessprobe.
6. App이 장애시 pod을 다시 재실행하게 하여, 잠깐의 트래픽 장애는 발생하여도 지속적인 장애는 방지해줌. 

* Liveness Probe는 컨테이너 상태가 비정상이라고 판단하면, 해당 Pod를 재시작하는 반면 ReadinessProbe는 해당 Pod를 사용할 수 없음으로 체크하고 서비스 등에서 제외
<img width="1528" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/0d8abd91-fb93-43d1-85db-0ded6be6828f">

<img width="741" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/07c31d98-7ad5-4384-bdcd-e9863ce0f3a0">

- 필수적 설정 값
- httpProbe
  가장 많이 사용하는 Probe방식으로 Http GET을 이용하여 컨테이너 상태를 체크한다.
  - port
  - host
  - path
  - httpHeader
  - scheme
- Exec
  - command
- tcpSocket
  - port
  - host

- 선택적 설정값, 값이 안들어간다면 default값이 들어가게됨.
- initialDelaySecond (최초 Probe전의 지연시간)
- periodSecond (Probe를 체크하는 시간 간격)
- timeoutSeconds (결과를 대기하는 시간까지의 간격)
- successsThershold( 몇번의 성공 결과를 수신해야 성공으로 인정하는지)
- failureThershold (몇번의 실패 결과를 수신해야 실패라고 인정하는지)

<img width="788" alt="image" src="https://github.com/yujindonut/Study/assets/78431728/d45ef8a1-8b7e-43ba-9ce1-cf742339a767">


-----------------------------------------------------------------------------------------------------------------------------------

## QoS(Quality of Service) class

앱의 중요도에 따라 자원 배정을 관리한다.
노드에 있는 Pod가 모든 자원을 사용하고 있을때, 특정 Pod에서 추가 자원이 필요한 경우 중요도에 따라 자원을 우선순위를 가짐
Qos 우선순위 : BestEffort < Burstable < Guarnteed
pod의 Qos는 사용자가 선택하여 지정하지 않고 pod 생성시, Request, limits에 따라 지정된다.

1. Guaranteed class
   
   ![image](https://github.com/yujindonut/Study/assets/78431728/913e482f-7f61-4b4c-a2a3-b599ae0d56d2)
   - 모든 Container에 Request와 limit이 설정된다.
   - Request와 limit에는 Memory 와 CPU가 모두 설정
   - 각 container의 memory, cpu는 request와 limit의 값이 같다.
2. Burstable class
   
   ![image](https://github.com/yujindonut/Study/assets/78431728/8a548e02-9b12-4592-8c14-b90344f419f1)
   아래의 기준에 속한다면 Burstable
   - Container마다 Request와 limit은 설정되어있지만, 수치가 같지 않음
   - Container에 Request 또는 limit만 설정되어있음
   - Pod내에 하나라도 완벽하게 (Guaranteed의 기준) 설정되어있지 않은 경우
3. BestEffort class
   
   ![image](https://github.com/yujindonut/Study/assets/78431728/a3fca3ec-5cbc-49d2-b9a7-d1d91d3f4086)
   - 어떤 Container 내에도 Request, Limit이 미설정
   - Burstable class 내에서의 우선순위
     - OOM score가 낮을 수록 우선 순위가 높음(OOM Score가 높은 Pod가 먼저 제거)
     - OOM score: request와 실제 사용에 따른 메모리 사용량을 판단


-----------------------------------------------------------------------------------------------------------------------------------

## Node Scheduling

Pod을 생성할 경우, 생성할 Pod이 어떤 노드에 할당되어야할지 유저가 지정하거나 쿠버네티스에서 자동적으로 할당 가능 또는 운영자가 특정 노드를 사용하지 못하도록 관리 가능.
이를 Scheduling이라고 부르며 쿠버네티스는 다양한 기능으로 스케줄링을 지원한다.

![image](https://github.com/yujindonut/Study/assets/78431728/32475a26-6e99-4616-afc6-4d0802ae6923)


### Node Scheduling을 지원하는 다양한 기능

#### 1. 특정 Node를 선택하는 방법
- Node name
  - Pod를 생성하면서 특정 Node를 지정 Scheduler와 상관없이 Node에 할당됨
  - 명시적으로 사용할 수 있어 좋아보이지만, 실 사용에서는 Node의 이름이 계속해서 변경되기 때문에 자주 사용하지는 않음
- Node selector
  - Node의 label(Key, value)를 지정하여 해당 Node에 할당됨
  - 같은 Label을 가진 Node 중에서는 Scheduler에 의한 자원이 많은 Node에 할당
  - 매칭이 되는 Label이 없다면 어떤 Node에도 할당이 되지 않는다.
 
- Node Affinity
  - Node selector 를 보완하여 사용할 수 있는 기능
  - Node Affinity는 matchExpressions와 Required, Preferred의 옵션을 사용하여 복잡하고 세부적인 조건 구성이 가능하다.
    - Required : 해당 조건에 반드시 부합하는 Node에만 할당
    - Preferred : Required에 비해 유연한 선택이 가능
      - 해당 조건에 부합하지 않더라도 preferred Weight(가중치)에 의해 결정
      - Preferred Weight : Preferred 옵션에서 조건에 따른 가중치를 부여함

#### 2. Pod간 집중/분산
- Node가 아닌 Pod를 기준으로 할당하는 방법
- Pod Affinity
  - 같은 PV 호스트패스를 사용한다면 Master Pod가 최초 할당된 Node에 Slave도 함께 생성
  - 다수의 Pod가 네트워크와 같은 리소스 효율을 위해 동일한 Node에 동작하는 것이 효율적일 때 사용
 
- Anti-Affinity
  - Master와 Slave를 서로 다른 Node에 생성
  - Pod들이 같이 있으면 해당 Node가 다운될때, 서비스가 다운될 수도 있는 경우 사용

#### 3. 특정 Node에 대해 할당 제한
- Toleration / Taint

  - 일반적인 경우에는 Node 할당을 컨트롤하여, 해당 Node에는 Pod 생성이 되지 않도록 한다.
  - Pod가 해당 Node를 직접 지정해도 할당 불가능
  - Toleration 옵션이 있어야함나 할당 가능
  - GPU와 같이 특별한 하드웨어 옵션을 가진 Node를 이용하는 특정 Pod 배치에 유용하게 사용가능하다

- Operator
  - Equal : Key, value, Effect가 모두 같은지 확인
  - Exists : Key, Value, Effect 중 1개 이상 설정 가능. 해당 설정을 가지고 있는지만 확인
    
- Effect
  - No Schedule : 노드에 해당 설정이 있으면 다른 Pod들이 배치 되지 않는다.
  - PreferNoSchedule : 가급적 다른 Pod들이 배치 되지 않게 함. 배치할 노드가 없는 경우 일반 Pod배치를 허용
  - NoExecute : 해당 설정이 있으면 일반 Pod 배체 금지 뿐만 아니라 기존 Pod도 삭제시킨다.