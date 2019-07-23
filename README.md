## RabbitMQ SSL
=========
### what is SSL?
모든 정보 전송이 https://로 암호화 프로토콜을 통해 안전하게 전송
### what is advatages of SSL
1) 스니핑 방지
2) 피싱 방지
3) 데이터 변조 방지
4) 기업 신회도 향상
----------

### TLS vs SSL
#### TLS
- 인터넷에서 정보를 암호화해서 송수신하는 프로토콜
- 표준
#### SSL
- 표준 아님

##### 즉, TLS 프로토콜은 SSL 프로토콜에서 발전한 것

### TLS 통신 과정
- 디지털 인증서 교환 및 유효성 검증에 의한 상호 인증
- 공유 비밀 키 생성을 위한 키 분배 문제점을 막아주는 비대칭 암호화 기술 사용
- SSL 또는 TLS는 메세지의 대칭 암호화에 공유 키를 사용함
![tls](https://user-images.githubusercontent.com/37536415/61677565-0c25f280-ad3b-11e9-8de3-a38b73d33e42.png)

출처 : boansecurity.blogspot.com

----------

### what is rabbitmq?
- AMQP(Advanced Message Queing Protocol)을 구현한 메시지 브로커
- mq : message queue – 신뢰성있는 메시지 전달
- 현업 분업을 위해서 필요
#### rabbitmq workflow
![rabbit](https://user-images.githubusercontent.com/37536415/61677823-e0573c80-ad3b-11e9-9945-fdd22d55a403.png)
![ra](https://user-images.githubusercontent.com/37536415/61677828-e2210000-ad3b-11e9-856d-333aa0ee3c25.png)

#### 현대의 server architecture
- micro service : 서버 하나씩 작은 단위 기능으로 줄이기
- 왜 서버로 쪼개냐 : 서로 독립된 실행을 위해, 분업을 할 수도 있음
- 정해진 규약에 맞춰서 개발만 하면 됨. 
###### 위의 하나씩 개발된 것을 합쳐주고 싶어서 rabbit MQ를 사용
