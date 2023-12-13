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

### Bindings   



