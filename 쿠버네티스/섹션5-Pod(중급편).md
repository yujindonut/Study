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