# Messaging
메시징 시스템 관련 학습

비동기 기반의 메시지 처리 시스템
구독, 발행 구조로 구성
메시지 브로커 구현 필요

client ----> message broker ----> consumer ----> user

메시지 브로커에서 메시지 발행 
메시지에는 구독, 발행과 관련된 메타데이터 포함
구독자는 구독한 메시지 큐에 데이터가 쌓이면 pull

장점
1. 서비스 결합도가 낮아진다.
2. client 가 직접 전달하는 과정을 구현하지 않고 message broker 로 던지기만 하면 되서 간편하다.

단점
1. 비동기 시스템이여서 안정성 문제
2. 메모리 문제 

## Rabbit MQ
Amqp 를 구현한 오픈소스 메시지 브로커

amqp 
(exchange, queue, binding )

<img width="824" alt="스크린샷 2023-12-13 오전 10 09 05" src="https://github.com/tmdrb/Messaging/assets/31639082/5973d65c-b497-4938-9f72-89dee2e959c0">

메시지를 전달할 때 공급자는 특정한 메시지 속성(메타데이터)를 broker 에게 보내고 브로커는 이 메시지를 속성에 따라 전송 

exchange Type -> 4가지 지원


default -> direct exchange 를 사용하며 이름 x, 큐들이 생성되면 자동으로 큐의 이름과 같은 라우팅 키를 사용하여 바인딩


<img width="760" alt="스크린샷 2023-12-13 오전 10 19 37" src="https://github.com/tmdrb/Messaging/assets/31639082/39952f12-67f5-435d-ae5d-b0f516599648">


direct -> 라우팅 키를 큐와 바인딩해서 사용, 사용자가 보낸 라우팅 키와 바인딩 한 키의 이름을 비교해서 메시지 전송, 하나의 exchange 에 같은 이름을 가진 라우팅 키로 여러개의 큐가 연결 됐을경우 모두 전송


<img width="707" alt="스크린샷 2023-12-13 오전 11 11 47" src="https://github.com/tmdrb/Messaging/assets/31639082/979746b6-5327-46d2-83da-cfc6cf426bf8">


fanout -> 라우팅 키를 무시하고 받은 메시지를 복사해서 바인딩 된 모든 큐에 메시지 전송, 브로드캐스트 라우팅에 가장 이상적인 exchange   
   

topic -> 라우팅 키와 큐에 바인딩 된 패턴들 사이에서 매칭을 통해서 메시지 전송, 다수의 컨슈머들이 선택적으로 메시지 받을 때 사용   


headers -> 라우팅 큐를 사용하지 않고 header 를 사용해서 라우팅 한다, any, all 옵션중 선택해서 header 속성에 맞는지 체크 한 다음 라우팅


### exchange attributes   

- name   
- Durability(서버가 재시작해도 살아있음)   
- auto-delete (마지막 큐가 바인딩에서 해제되면 자동으로 삭제)   
- Arguments (선택적으로 사용, 프러그인이나 브로커의 특정 기능을 사용 할 때)   

*********************************************

### queue   

- name   
- Durability(서버가 재시작해도 살아있음)
- Exclusive(오직 하나의 커낵션만 사용 가능하고 커넥션이 종료되면 큐는 지워짐)   
- auto-delete (구독한 consumer 가 없을때 자동으로 삭제)   
- Arguments (선택적으로 사용, 플러그인이나 브로커의 특정 기능을 사용 할 때)

  ***************

### Acknowledgements

비동기 메시징 시스템에서는 메시지가 피어에 정상적으로 도달했는지 잘 처리 되었는지에 대한 확신이 없음   

따라서 이것을 보완하기 위한 승인 메카니즘을 사용

- 소비자로 부터의 승인   
   consume 를 사용해서 소비자를 등록했을 경우 or get 메소드를 사용해서 메시지를 가져 온 경우

   - auto
  
        fire and forget 이라 불리며 전송 된 즉시 배송에 성공했다고 고려
        
        완벽하게 보내지기전에 connection 이 종료 될 수도 있어서 안전하지는 못함
        
        처리되지 못한 메시지에 대한 제한이 없어서 memory 오류 발생 할 가능성이 있음
        
        안정적으로 처리 할 수 있는 consumer 에게만 추천   
 
   - manual
     
     ack, nack 응답 메시지로 판단   
     
     ack 가 온 경우 정상적으로 받았다고 판단, deliveryTag 를 multi 로 처리 가능
     
     nack 전송한 경우 메시지가 버려지거나 다시 큐에 들어가거나 Dead Letter Exchange 에 들어가거나 선택 ( Dead Letter Exchage 가 구성 안되어 있으면 버린다 )
     
     메시지가 다시 큐에 적재된 경우 원래 position 으로 위치
     
     prefetch 카운트를 설정하여 바로 응답할 건지 대기 할 건지 선택
     
     qos = 0 이면 limit 제한 없음, qos = 1 이면 delivery_tag 5,6,7,8 을 보냈을 때 승인 응답이 하나라도 와야 다시 메시지 전송


  consumer 에서 승인을 해서 서버로 보냈다고 하더라도 서버에서 확실하게 받아서 처리했다는 보장이 없을 수 있음.

  그래서 이걸 보장하기 위한 방법으로 transaction 을 사용하면 되는데 너무 무겁고 처리량이 250배 가량 떨어진다.

  no-wait 옵션을 사용해서 client 가 select 함수를 전송하면 broker 는 select-ok 라는 응답을 줌

  channel 이 confirm mode 로 되어 있으면 broker 에서도 client 와 마찬가지로 count 한다

  같은 channel 에서 broker 는 basic.ack 를 전송해서 메시지를 확인

  delivery-tag, multiple 옵션 사용   


********************

### Rabbit MQ 흐름   

publisher 가 브로커에 메시지 전송   

broker 에서 바인딩 된 큐에 메시지 전송   

consumer 는 큐에서 메시지를 읽음   

메시지를 받으면 ack 를 다시 전송

broker 는 ack 를 받으면 메시지 삭제   

만약 ack 를 받지 못했다면 삭제, 재전송 등 설정에 따라 실행   





