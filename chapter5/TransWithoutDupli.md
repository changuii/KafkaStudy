# 5.3 중복 없는 전송


데이터 프로세싱 과정에서 메시지가 중복되지 않고 처리된다면 데이터 처리 업무를 담당하는 데이터 엔지니어나 개발자에게 매우 유용할 것입니다.  
왜냐하면 언제 발생할지 모르는 메시지 중복으로 인해 데이터 프로세싱 과정에서 모든 메시지에 대해 중복 처리 과정을 따로 거치지 않아도 되며 별도로 데이터 정합성을 체크할 필요도 없기 때문입니다.  

일부 서비스는 메시지 중복을 허용하는 경우도 있지만, 특정 서비스에서 메시지가 중복 처리된다면 치명적인 상황이 발생할 수 있습니다.  
이에 따라 카프카는 사용자들의 개발 편의를 높이기 위해 중복 없이 전송할 수 있는 기능을 제공합니다.  

'중복 없는 전송'(멱등성 전송)이라고 하면 얼핏 간단해 보이지만, 사실 이 기능을 실제 구현하는 개발자 입장에서는 상당히 어려운 작업입니다.  

메시지 시스템들의 메시지 전송 방식에는 '적어도 한 번 전송(at-least-once)', '최대 한 번 전송(at-most-once)', '정확히 한 번 전송(exactly-once)'이 있습니다.  
각 전송 방식은 서로 어떤 차이가 있고 카프카 프로듀서에 적용된 중복없는 전송과정도 살펴보겠습니다.  

## 5.3.1 적어도 한 번 전송

예상 시나리오  

1. 프로듀서가 브로커의 특정 토픽으로 메시지A를 전송합니다.  
2. 브로커는 메시지A를 기록하고, 잘 받았다는 ACK를 프로듀서에게 응답합니다.  
3. 브로커의 ACK를 받은 프로듀서는 다음 메시지인 메시지B를 브로커에게 전송합니다.  
4. 브로커는 메시지B를 기록하고, 잘 받았다는 ACK를 프로듀서에게 전송하려고 합니다. 하지만 네트워크 오류 또는 브로커 장애가 발생하여 결국 프로듀서는 메시지B에 대한 ACK를 받지 못합니다.  
5. 메시지B를 전송한 후 브로커로부터 ACK를 받지 못한 프로듀서는 브로커가 메시지B를 받지 못했다고 판단해 메시지B를 재전송합니다.  

프로듀서의 5번 동작을 조금 더 설명하겠습니다.  
프로듀서는 메시지B를 보내고 그에 대한 ACK를 받지 못했으므로 브로커가 메시지B를 받지 못했다고 판단합니다.  
하지만 4번 과정에서 브로커는 메시지B를 기록했으믈, 장애에서 복구된 브로커는 메시지B를 갖고 있을 것입니다.

현 상황을 다시 정리해보면, 프로듀서 입장에서는 브로커가 메시지를 저장하고 ACK만 전송하지 못한 것인지, 메시지를 저장하지 못해서 ACK를 전송하지 않은 것인지는 정확히 알 수 없습니다.  
하지만 메시지B에 대한 ACK를 받지 못한 프로듀서는 적어도 한 번 전송 방식에 따라 메시지B를 다시 한번 전송합니다.  

만약 브로커가 메시지B를 받지 못한 상황이었다면 브로커는 처음으로 메시지B를 저장할 것이고, 브로커가 메시지B를 저장하고 ACK만 전송하지 못한 상황이었다면 메시지B는 브로커에 중복 저장될 것입니다.  

이렇게 네트워크의 회선 장애나 기타 장애 상황에 따라 일부 메시지 중복이 발생할 수는 있지만, 최소한 하나의 메시지는 반드시 보장한다는 것이 적어도 한 번 전송 방식이며, 카프카는 기본적으로 이와 같은 적어도 한 번 전송 방식을 기반으로 동작합니다.  

## 5.3.2 최대 한 번 전송

예상 시나리오  

1. 프로듀서가 브로커의 특정 토픽으로 메시지A를 전송합니다.  
2. 브로커는 메시지A를 기록하고, 잘 받았다는 ACK를 프로듀서에게 응답합니다.  
3. 프로듀서는 다음 메시지인 메시지B를 브로커에게 전송합니다.  
4. 브로커는 메시지B를 기록하지 못하고, 잘 받았다는 ACK를 프로듀서에게 전송하지 못합니다.  
5. 프로듀서는 브로커가 메시지B를 받았다고 가정하고, 메시지C를 전송합니다.  

최대 한 번 전송은 ACK를 받지 못하더라도 재전송하지 않습니다.  
사실 최대 한 번 전송에서 ACK를 응답하는 과정은 없어도 되지만 적어도 한 번 전송과의 비교를 위해 추가한 것입니다.  

4번과정을 자세히 살펴보면 프로듀서 브로커에게 메시지B를 전송했지만 중간에 메시지B가 유실되어 브로커가 메시지B를 저장하지 못했을 수 있고, 브로커가 메시지B를 기록했지만 프로듀서에게 ACK만 응답하지 못했을 수도 있습니다.  
최대 한 번 전송 과정에서 프로듀서는 메시지의 중복 가능성을 회피하기 위해 재전송을 하지 않습니다.  
다시 말해 일부 메시지의 손실을 감안하더라도 중복 전송을 하지 않는 경우 입니다.  

최대 한 번 전송을 이용하는 곳이 있을까 하는 생각이 들 수도 있겠지만, 일부 메시지가 손실되더라도 높은 처리량을 필요로하는 대량의 로그 수집이나 IoT 같은 환경에서 사용하곤 합니다.  

두 가지 전송 방식의 특징을 다시 정리해보면 적어도 한 번 전송은 메시지 손실 가능성은 없지만 메시지 중복 가능성이 존재하고, 최대 한 번 전송은 메시지 손실 가능성은 있지만 메시지 중복 가능성은 없습니다.  


## 5.3.3 카프카의 중복 없는 전송

예상 시나리오  

1. 프로듀서가 브로커의 특정 토픽으로 메시지A를 전송합니다. 이때 PID(Producer ID)0과 메시지 번호 0을 헤더에 포함해 함께 전송합니다.
2. 브로커는 메시지A를 저장하고, PID와 메시지 번호 0을 메모리에 기록합니다. 그리고 메시지를 잘 받았다는 ACK를 프로듀서에게 응답합니다.
3. 프로듀서는 다음 메시지인 메시지B를 브로커에게 전송합니다. PID는 동일하게 0이고, 메시지 번호는 1이 증가하여 1이 됩니다.
4. 브로커는 메시지B를 저장하고, PID와 메시지 번호 1을 메모리에 기록합니다. 그리고 메시지를 잘 받았다는 ACK를 프로듀서에게 전소하려고 합니다. 하지만 네트워크 오류 또는 브로커 장애가 발생하여 프로듀서는 메시지B에 대한 ACK를 받지 못합니다.
5. 브로커로부터 ACK를 받지 못한 프로듀서는 브로커가 메시지B를 받지 못했다고 판단해 메시지B를 재전송합니다.  

지금까지의 과정은 적어도 한 번 전송 과정과 동일합니다.  
하지만 프로듀서가 메시지B를 다시 전송한 후 브로커의 동작은 차이가 있습니다.  

프로듀서가 재전송한 메시지B의 헤더에서 PID(0)와 메시지 번호(1)를 비교해서 메시지B가 이미 브로커에 저장되어 있는 것을 확인한 브로커는 메시지를 중복 저장하지 않고 ACK만 보냅니다.  
이러한 브로커의 동작 덕분에 브로커에 저장된 메시지는 중복을 피할 수 있게 됩니다.  

카프카에서 이용하는 PID와 메시지 번호가 바로 중복없는 전송의 핵심인데, 이에 대해 좀 더 살펴봅시다.  
프로듀서가 중복 없는 전송을 시작하면, 프로듀서는 고유한 PID를 할당 받게 되고, 이 PID와 메시지에 대한 번호를 메시지의 헤더에 포함해 메시지를 전송합니다.  
여기서 메시지 번호를 시퀀스 번호라고도 합니다.  

시나리오에서 PID는 0이고, 메시지A의 시퀀스 번호는 0입니다.  
브로커에서는 각 메시지마다 PID값과 시퀀스 번호를 메모리에 유지하게 되며, 이 정보를 이용해 브로커에 기록된 메시지들의 중복 여부를 알 수 있습니다.  

메시지B의 경우 PID 값 0과 시퀀스 번호 1이라는 값을 지닌 메시지가 이미 브로커에 저장되어 있기 때문에 만약 프로듀서가 동일한 메시지를 재전송하더라도 브로커에는 메시지가 중복 저장되지 않습니다.  
PID는 사용자가 별도로 생성하는 것이 아니며 프로듀서에 의해 자동 생성됩니다.  
또한 이 PID는 프로듀서와 카프카 사이에서 내붝으로만 이용되므로 사용자에게 따로 노출되지 않습니다.  

또한 메시지마다 부여되는 시퀀스 번호는 0번부터 시작해 순차적으로 증가합니다.  
프로듀서에서 시퀀스 번호를 메시지마다 순차적으로 증가시키는 방법과 동일하게, 브로커에서도 기록되는 메시지들에 대해 시퀀스 번호를 증가시킵니다.  

따라서 프로듀서가 보낸 메시지의 시퀀스 번호가 브로커가 갖고 있는 시퀀스 번호보다 정확하게 하나 큰 경우가 아니라면, 브로커는 프로듀서의 메시지를 저장하지 않습니다.  
바로 이 동작 때문에 메시지 중복을 피할 수 있는 것입니다.  

이렇게 메시지 중복을 피하기 위해 사용되는 PID와 시퀀스 번호 정보는 브로커의 메모리에 유지되고, 리플리케이션 로그에도 저장됩니다.  
따라서 예기치 못한 브로커의 장애 등으로 리더가 변경되는 일이 발생하더라도 새로운 리더가 PID와 시퀀스 번호를 정확히 알 수 있으므로 중복 없는 메시지 전송이 가능합니다.  






