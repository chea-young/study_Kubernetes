### Pod - Lifecycle
- 파드의 라이프사이클: Pending - Running - Suceeded -Failed
    - Pending: 파드의 최초 상태
        - ReadinessProbe
        - Policy
    - Running
        - LivenessProbe
        - Qos
    - Succeeded
        - Policy
    - Failed
        - Policy
- 파드 생성 후 내용 하단을 보면 status가 존재한다.
    - 구조
    <img />
    #### Pod
    -  Status
        - Phase: 파드의 전제 상태를 대표하는 속성 [Pending, Running, Succeeded, Failed, Unknown]
        - Conditions: 파드가 생성되면서 실행하는 단계들이 있는데 그 단계와 상태를 알려주는 것이다. [Initialized, ContainerReady, PodScheduled, Ready]
            - False인 경우 False인 이유를 알아야 되기 때문에 Reason이 존재한다.
        - Reason: Conditions의 세부내용을 알려준다.[ContainersNotReady, PodCompleted]
            - ContainersNotReady: 컨테이너 작업이 아직 진행 중이다.
            - PodCompleted
    #### Containers
    -  Status
        - Phase - [Wating, Running, Terminated]
        - Reason - [ContainerCreating CrashLoopBackOff, Error, Completed]
#### Pending
- 파드의 최초 상태이다.
- Pending 중에 바로 Failed 혹은 Unknown(통신장애로 인하여)으로 빠질 수도 있다.
- 
<img />

- Wating(Reason: ContainerCreaing)
    - `Initialized: True`: initContainer - 본 컨테이너가 기동되기 전에 초기화시켜야 되는 내용이 있을 경우 그 내용을 담는 컨테이너를 만드는 항목이다.
    - `PodScheduled: True`: Node Schediling - 파드가 어느 노드에 올라갈 지 직접 지정하였을 때는 해당 노드에 혹은 자원 사용량을 판단해서 노드를 결정하는데 이것이 완료되었을 때이다.
    -  Image Downloading: 컨테이너의 이미지를 다운로드하는 동작이다.
#### Running
- 잡이나 크론잡으로 생성된 파드의 경우 자신의 일을 수행 중에는 러닝이지만 일을 맞치게 되면 Failed나 Succeeded로 바뀌게 된다.
- Running 중에 통신장애로 인하여 Unknown으로 빠질 수도 있다. 바로 해결이 되면 원래 상태로 돌아가지만 그것이 지속된다며 Failed가 된다.
<img />

- Wating(Reason: CrashLoopBackOff): 하나 또는 모든 컨테이너가 기동 중에 문제가 발생해서 재시작될 때이다.
    - `ContainerReady: False`
    - `Ready: False`
- 정상적으로 기동이 되었을 때이다.
    - `ContainerReady: True`
    - `Ready: True`
#### Failed
<img />

- 작업을 하고 있는 컨테이너 중에 하나라도 문제가 생겨서 에러가 되었을 때이다.
    - `ContainerReady: False`
    - `Ready: False`

#### Succeeded
<img />

- 컨테이너들이 모두 해당 일들을 다 맞쳤을 때이다.
    - `ContainerReady: False`
    - `Ready: False`

### Pod - ReadinessProbe, LivenessProbe
- 이 둘은 사용 목적은 다르지만 설정할 수 있는 것은 같다.
- 파드를 만들면 그 안에 컨테이너가 생기고 파드와 컨테이너 상태가 러닝이 되면서 그 안의 앱도 정상적으로 구동이 되고 서비스에 연결이 되고 그 서비스의 외부 IP를 통해서 사용자들이 실시간으로 접근을 하게 된다. (노드2, 파드2)그 상태에서 한 노드가 문제가 생기면 그 노드에 올라가 있던 파드도 Failed된다. 그러게 되면 한 파드에 트래픽이 집중되게 된다. 이제 오토힐링 기능을 통해서 파드는 재생성되고자 하고 파드와 컨테이너가 러닝상태이고 앱이 구동중인 상태에서 서비스와 연결이 된다. 사용자는 50퍼센트의 확률로 에러 페이지를 보게 된다. 이때 ReadinessProbe를 주게 되면 이러한 문제점을 해결할 수 있다.
    - ReadinessProbe: 앱 구동 순간에 트래픽 실패를 없앤다. (구동 중에는 파드를 서비스와 연결하지 않는다.)
- 파드와 컨테이너는 러닝인 상태에서 앱만이 다운이 되는 상황이 있는데 이때 앱의 장애 상황을 감지해주는 것이 LivenessProbe
    - LivenessProbe: App 장애 시 지속적인 트래픽 실패를 없앤다. 파드의 앱에 문제가 되면 파드를 restart하게 만들어준다.
- 속성
    - httpGet: [port, host, path, httpheader, scheme]를 체크할 수 있다.
    - Exec: [command]를 날려 특정 결과를 체크한다.
    - tcpSocket: [port, host]를 날려 성공여부를 확인한다.
- 옵션
    - initialDelaySeconds: 최초로 실행하기 전에 지연될 시간 간격이다.
    - periodSeconds: 프로브를 체크하는 시간의 간격이다.
    - timeoutSeconds: 지정된 시간까지 결과를 기다릴 시간이다.
    - successTreshols: 몇 번 성공 결과를 받아야 성공으로 인정할 횟수이다.
    - failureThreshold: 몇 번 실패 결과를 받아야 실패로 인정할지 결정하는 횟수이다.


### Pod - ReadinessProbe, LivenessProbe 실습


### Pod - QoS Classes
- 컨테이너의 리소스 설정에서 requests와 limits에 따라서 메모리와 cpu를 어떻게 설정하는지에 따라서 결정이 된다.
- 노드에 리소스가 있고 노드 위에 파드가 3개가 만들어져서 균등하게 자원을 사용하고 있을 때 파드1이 추가적으로 자원을 더 사용해야 하지만 추가적으로 더 사용할 수 있는 자원이 없는 상황 일 때 쿠버네티스에서는 앱의 중요도의 따라 이런 것을 관리할 수 있도록 3가지 단계로 Qos를 지원해주고 있다.
    - 이 상황에서는 BestEffort인 파드가 가장 먼저 다운이 되서 자원이 반환되서 파드1이 사용할 수 있게 된다.
    - 노드에 어느 정도의 자원이 남아있지만 파드2에서 더 많은 자원을 요구하는 상황이 된다면 Burstable로 된 자원이 그 다음으로 다운이 되면서 자원을 반환한다.
- 속성
    - Guaranteed
    <img />
     - 파드에 여러 컨테이너가 있다면 모든 컨테이너마다 requests와 limit가 설정되고 그 안의 메모리와 cpi가 있어야 한다.
     - 각 컨테이너의 메모리와 CPU의 리쿼스트와 limit의 값이 같아야 한다.
    - Burstable
    <img />
     - 컨테이너마다 requests와 limit는 설정되어 있지만 requests와 limit가 다 다른 경우이다
     - requests와 limit  중에 하나만이 설정되어 있는 경우이다
     - 컨테어너가 2개인데 하나는 완벽하게 설정이 되어있고 하나는 아무것도 설정되어 있지 않은 경우이다

    - BestEffort
    <img />
     - 파드의 어떤 컨테이너에도 requests와 limit가 설정되어 있지 않는 경우이다.
- OOM Score(Out of Memory Score)
    - 현재 이 파드의 리쿼스트 메모리의 사용량을 계산해서 이 OOMscore가 더 큰 파드를 먼저 제거한다.

### Pod - QoS Classes 실습

### Pod - Node Scheduling
- Node 선택: 파드가 특정 노드에 할당되도록 선택하기 위한 용도로 사용이 된다. 노드가 여러개가 있을 때 파드를 스케줄러에 연결하게 되면 노드가 가지고 있는 자원과 태그에 따라 연결을 해준다.
    - NodeName: 파드가 스케줄러를 거치지 않고 바로 노드에 올라간다.
    - NodeSelector: 파드에 key, value를 달면 해당 라벨이 달려있는 노드에 할당이 된다. 만약 같은 태그를 여러 개의 노드들이 가지고 있을 경우 노드들 중에서 자원이 가장 많이 남은 노드에 할당이 된다.
    - NodeAffinity: NodeSelector를 보완한 것으로 파드에 키만 설정을 해주어도 해당 그룹 중에 스케줄러를 통해서 자원이 많은 노드에 파드가 할당이 된다. 만약 해당 조건에 맞는 노드가 없더라도 스케줄러가 자원이 많이 남은 노드에 할당이 되도록 옵션을 줄 수 있다.
- Pod간 집중/분산: 여러 파드들을 한 노드에 집중해서 할당을 하거나 파드들 간에 겹치는 노드 없이 분산을 해서 할당한다.
    - Pod Affinity: 만약 두 개의 파드가 같은 노드에 있어 PV를 공유하는 상황이여 할 때 1번 파드가 한 노드에 생기고 PV가 생기면 2번 파드가 1번 파드에 있는 노드에 들어가게 하기 위해서 1번 파드의 라벨을 그대로 지정한다.
    - Anti-Affinity: 두 개의 파드가 각 다른 노드에 배치되어야 하는 상황일 경우 1번 노드가 스케줄러에 의해 한 노드에 들어가게 되면 


### Pod - Node Scheduling 실습