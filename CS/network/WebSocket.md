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


### webSocket 세션 vs HTTP 세션
- 웹소켓세션은 HTTP 세션의 보안이 적용되지 않는다.
#### WebSocket 세션에 굳이 HTTP 세션을 저장하는 이유
- 로그인 하지 않은 사용자가 웹소켓 세션에 접근할 수 있다.
- 사용자 식별이 어렵다. 어떤 사용자가 메세지를 보냈는지 식별하기 어렵다.
```
3/1 추가사항
Spring Security를 사용하면 세션 정보를 Principal로부터 가져올 수 있다.
Spring Security를 사용하지 않으면 Interceptor를 통해 가져올 수 있는데 중간에 HTTP 세션 정보가 변경되면 update 해줘야 할 수도 있음.
세션은 어차피 세션 쿠키에 저장되기 때문에 브라우저를 닫지 않는 이상 업데이트를 해줘야할 필요성을 못 느끼겠다.
+ 일부러 세션 쿠키를 제거한다면 어차피 로그아웃 될거다.
```


참고
- [ably](https://ably.com/topic/websockets-vs-http)
- [wikipedia](https://ko.wikipedia.org/wiki/%EC%9B%B9%EC%86%8C%EC%BC%93)
- [geeksforgeeks](https://www.geeksforgeeks.org/what-is-web-socket-and-how-it-is-different-from-the-http/)
- https://velog.io/@koseungbin/WebSocket#annotated-controllers
- https://jinn-blog.tistory.com/152
- https://blog.leaphop.co.kr/blogs/56
- https://javakyu4030.tistory.com/entry/SpringFrameworkWebSocketSession%EC%97%90%EC%84%9C-HttpSession-%EA%B0%92-%EA%B0%80%EC%A0%B8%EC%98%A4%EA%B8%B0
