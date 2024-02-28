# WebSocket vs HTTP
## HTTP
특징
- 비연결성
- 매번 연결을 맺고 끊는다
- 요청 - 응답 (client - server) 구조
## WebSocket
- 연결 지향
- 한 번 연결 한 뒤 유지된다
- 양방향 통신


## WebSocket 이전에는 어떻게?
### poling 방식
- 클라이언트가 주기적으로 요청하여 응답을 주고 받는 방식이다. 
#### 폴링의 단점
- 폴링의 주기가 짧으면 서버에 부담이 된다.
- 주기가 길면 실시간성이 떨어진다.

### LongPoling 방식
- 한 번 요청 시 서버에서 접속을 열어두는 시간을 길게 만든다. 대기 시간 동안 업데이트가 되면 서버는 응답을 전달하고 클라이언트는 다시 요청하는 방식이다. 업데이트 되는 주기가 짧으면 폴링과 동일하다는 단점이 있다.

### Streaming 방식
- 웹 접속을 계속 열어두고, 이벤트가 발생하면 부분 응답을 전달하는 방식이다.


문제는 websocket을 지원하지 브라우저들이 있음. 이런 경우 SockJs나 Socket.io 같은 걸 사용하면 라이브러리에서 지원을 해준다.
스프링은 text/binary 웹소켓 핸들러를 지원한다. STOMP는 메세지 브로커를 이용하여 pub/sub 매커니즘 기반으로 메세지를 주고 받는 프로토콜이다.

##### 메세지 브로커란?
- 발신자의 메세지를 받아서 수신자에게 전달해주는 객체

##### STOMP 쓰는 이유
- 메세지 기반 웹소켓 핸들러를 구현하지 않아도 된다.
- 웹소켓은 HTTP와 같이 정해진 프레임이 없다. 그래서 웹소켓 간에 프레임을 정해서 통신을 주고 받는데 STOMP는 프레임을 제공해준다.
- 메세지 브로커를 이용하는 pub/sub 기반이기 때문에 외부 브로커를 사용할 수 있다. (카프카, rabbitMQ 등)



참고
- [ably](https://ably.com/topic/websockets-vs-http)
- [wikipedia](https://ko.wikipedia.org/wiki/%EC%9B%B9%EC%86%8C%EC%BC%93)
- [geeksforgeeks](https://www.geeksforgeeks.org/what-is-web-socket-and-how-it-is-different-from-the-http/)
