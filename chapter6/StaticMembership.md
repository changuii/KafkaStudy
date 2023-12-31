# 6.3 스태틱 멤버십


때로는, 하드웨어 점검이나 소프트웨어 업데이트 등의 이유로 관리자는 컨슈머 그룹 내의 컨슈머들을 하나씩 순차적으로 재시작하고 싶은 경우가 있을 것입니다.  
하지만 하트비트 주기, 세션 타임아웃 등의 설정으로 인해 하나의 컨슈머가 재시작될 때마다 전체 리밸런싱이 일어나며, 리밸런싱 작업이 일어나는 동안 컨슈머들은 일시 중지하므로 이는 매우 번거러운 일이 아닐 수 없습니다.  

10개의 컨슈머가 있는 그룹이라면 최소 10번 이상의 리밸런싱이 발생하고 10번 이상의 컨슈머 일시 중지가 일어납니다.  
그렇다면 컨슈머의 재시작으로 인해 리밸런싱이 일어나는 배경을 알아보겠습니다.  

일반적으로 컨슈머 그룹 동작에서는 각 컨슈머를 식별하기 위해 엔티티ID를 부여하게 됩니다.  
이렇게 생성된 ID들은 컨슈머 그룹 내에서 임시로 사용되는 값입니다.  

따라서 컨슈머의 설정 변경이나 소프트웨어 업데이트로 인해 컨슈머가 재시작되면, 컨슈머 그룹 내의 동일한 컨슈머임에도 새로운 컨슈머로 인식해 새로운 엔티티ID가 부여되고 이로 인해 컨슈머 그룹의 리밸런싱이 발생하는 것입니다.  
대용량 메시지들을 처리하는 컨슈머 그룹이라면 리밸런싱 동작으로 인해 원래 상태를 복구하는데 상당한 시간이 소요될 수 있습니다.  

따라서 카프카는 이러한 불필요한 리밸런싱을 방어하기 위해 아파치 카프카 2.3버전 부터 `스태틱 멤버십(static membership)`이라는 개념을 도입했습니다.  
스태틱 멤버십이란 컨슈머 그룹 내에서 컨슈머가 재시작 등으로 그룹에서 나갔다가 다시 합류하더라도 리밸런싱이 일어나지 않게 합니다. 즉, 컨슈머마다 인식할 수 있는 ID를 적용함으로써 다시 합류하더라도 그룹 코디네이터가 기존 구성원임을 인식할 수 있게 하는 것입니다.  

그뿐 아니라 스태틱 멤버십 기능이 적용된 컨슈머는 그룹에서 떠날 때 그룹 코디네이터에게 알리지 않으므로 불필요한 리밸런싱도 발생하지 않습니다.  

## 6.3.1 스태틱 멤버십이 적용된 경우의 재시작 과정

기본적으로는 컨슈머 그룹에서 컨슈머가 떠날 때 리밸런싱이 일어나는데, 스태틱 멤버십이 적용된 컨슈머는 그룹에서 떠날 때 그룹 코디네이터에게 알리지 않으므로 여기서 한 번의 리밸런싱을 피할 수 있습니다.  
그리고 해당 컨슈머가 다시 합류할 때 그룹 코디네이터가 컨슈머의 ID를 확인하고 리밸런싱이 일어나는데, 이때도 기존 구성원임을 인지하므로 리밸런싱은 발생하지 않습니다.  

이렇게 스태틱 멤버십을 적용하면 컨슈머 재시작 시 총 두번의 불필요한 리밸런싱을 회피할 수 있습니다.  

스태틱 멤버십을 적용하기 위해서는 아파치 카프카 2.3 이상이며 group.instance.id 옵션에는 그룹 코디네이터가 컨슈머를 식별하기 위해 컨슈머 인스턴스 별로 고유한 값을 입력해야 합니다.  
예를 들어 접두어로 consumer-라고 지정하고, 접미어는 호스트 네임이나 서버의 IP 등을 이용해 consumer-hostname1, consumer-hostname2와 같이 입력하는 방법이 있습니다.  

만약 스태틱 멤버십 기능을 적용한다면, session.timeout.ms를 기본 값보다는 큰 값으로 조정해야 합니다.  
컨슈머를 재시작한 후 session.timeout.ms 값에지정된 시간 동안 그룹 코디네이터가 하트비트를 받지 못한다면 강제로 리밸런싱이 일어나므로, 불필요한 리밸런싱 동작을 최소화하기 위한 스태틱 멤버십의 목적에 위배되기 때문입니다.  

따라서 컨슈머의 재시작 시간을 고려해서 적절한 시간값으로 조정해둬야 합니다.  
예를들어 컨슈머 재시작 시간이 총 2분 소요된다면, session.timeout.ms 값은 2분보다 큰 값으로 설정해야 불필요한 리밸런싱 동작을 사전에 방지할 수 있습니다.  

## 6.3.2 일반 컨슈머 그룹의 리밸런싱 동작

현재 peter-test06 토픽의 파티션이 컨슈머의 프로세스가 실행된 브로커와 맵핑되어 있습니다.  
(각 파티션 0, 1, 2는 컨슈머 3, 2, 1과 각각 연결되어 있다.)  

이제 peter-kafka01의 컨슈머 프로세스를 강제로 종료합니다.  
브로커 peter-kafka01의 컨슈머가 peter-consumer01 그룹에서 떠나자 그룹 코디네이터는 이 상황을 인지했으며 컨슈머 그룹 내부적으로 리밸런싱이 일어났습니다.  

브로커 peter-kafka01의 컨슈머가 종료되면서 컨슈머 그룹에서는 리밸런싱이 일어났고, 파티션 수보다 컨슈머 수가 작기 때문에 브로커 peter-kafka03에서 실행 중인 컨슈머가 2개의 파티션을 담당하게 되었습니다.  
여기서 중요한 사실은 브로커 peter-kafka02의 컨슈머가 담당하고 있던 파티션도 변경되었다는 점입니다.  

peter-kafka02의 컨슈머는 파티션1을 담당하고 있었는데, peter-kafka01의 컨슈머가 종료되면서 리밸런싱 동작을 통해 파티션2를 담당하게 되었습니다.  
이처럼 컨슈머의 리밸런싱은 재시작되거나 그룹에서 떠나는 컨슈머만 대상으로 동작하는 것이 아니라, 컨슈머 그룹 내 전체 컨슈머를 대상으로 동작합니다.  

컨슈머 리밸런싱 동작 과정 중 일시적으로 모든 컨슈머가 일시 중지하게 되는데, 대량의 메시지를 컨슘하는 컨슈머 그룹에게 이러한 일시 중지 동작은 매우 부담이 크며 고비용이 드는 작업입니다.  
따라서 불필요한 리밸런싱은 최대한 줄여야 합니다.  

게다가 브로커 peter-kafka01의 컨슈머가 장애에서 복구되어 다시 합류한다면 또 다시 리밸런싱 동작이 일어납니다.  

## 6.3.3 스태틱 멤버십 컨슈머 그룹의 리밸런싱 동작  

현재 peter-test06 토픽의 파티션이 컨슈머의 프로세스가 실행된 브로커와 매핑되어 있습니다.  
(각 파티션 0, 1, 2는 컨슈머 2, 1, 3과 각각 연결되어 있다.)  

그럼 이제 브로커 peter-kafka01에서 실행중인 컨슈머 프로세스를 강제로 종료하겠습니다.  
브로커 peter-kafka01의 컨슈머 프로세스를 종료했음에도 불구하고, 변경된 내용이 없음을 알 수 있습니다.  

즉 리밸런싱 동작이 일어나지 않습니다.  
하지만 앞서 session.timeout.ms에 지정한 30여 초가 지난 뒤 다시 확인해보면 리밸런싱 동작이 일어나 새로운 파티션이 할당됐음을 알 수 있습니다.  

peter-kafka01에서 동작 중이던 컨슈머 프로세스를 종료하기 전에는 peter-kafka01의 컨슈머가 파티션 1번을 담당하고 있었습니다.  
peter-kafka01에서 동작중이던 컨슈머 프로세스를 종료한 다음, 일정 시간이 지난 후 1번 파티션의 담당이 peter-kafka02로 변경되었습니다.  

다시 정리해보면, 표준 컨슈머를 사용하는 경우 컨슈머의 오류가 발생해 해당 컨슈머를 잠시 제거하고, 오류 수정 후 다시 컨슈머 그룹에 합류하면 총 두 번의 리밸런싱이 발생합니다.  
하지만 스태틱 멤버십을 적용한 컨슈머를 활용하는 경우에는 session.timeout.ms에 지정된 시간을 넘어가지 않는다면 컨슈머를 잠시 제외하더라도 리밸런싱 동작이 발생하지 않게 됩니다.  






