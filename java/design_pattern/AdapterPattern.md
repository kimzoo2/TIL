# 어댑터 패턴(Adapter pattern)
- 클라이언트가 사용하려는 인터페이스와 재사용 모듈이 사용하려는 인터페이스가 일치하지 않을 때 사용하는 패턴이다.
- OCP 원칙을 따를 수 있도록 도와준다. (클라이언트의 변경을 최소화하고 기능을 확장할 수 있게 해주기 때문이다.)
- 대표적인 예 : 
  - SLF4J - 단일 로깅 API를 사용하면서 log4k, LogBack 등의 로깅 프레임워크를 선택적으로 사용할 수 있게 해 줌
  - InputStreamReader - BufferedReader에 System.in을 직접적으로 사용할 수 없어서 InputStreamReader가 어댑터 역할을 한다.

## 구현 방법
- 조합
- 상속 : 자바는 단일 상속만 지원하는 언어이기 때문에 클래스 상속을 이용하면 제약이 있다.

## 구조
![adapterPattern.png](..%2F..%2Fimg%2Fjava%2FadapterPattern.png)

## 구현
```java
// client
public class Client {
	
	private ClientService clientService;
	
	public Client(ClientService clientService){
		this.clientService = ClientService;
    }
	public void callAdapter(){
		clientService.callService();
    }
}

// client interface(target)
public interface ClientService {
	void callService();
}

// adapter-composition : 사용하고 싶은 모듈을 호환해주는 어댑터
public class Adapter implements ClientService {
	
	private Service service;

	public Adapter(Service service){
		this.service = service;
	}
	
	@Override
    public void callService(){
		service.callAnotherService();
    }
}

// adaptee : 사용 불가했으나 Adapter로 인해 사용 가능해진 모듈
public class Service {
	public void callAnotherService() {
		System.out.println("Adaptee 호출, 원래 사용 못 했으나 어댑터로 사용가능할 수 있게 되었다.");
	}
}

// adapter-inheritance
public class Adapter extends Service implements ClientService {

	private Service service;

	public Adapter(Service service){
		this.service = service;
	}

	@Override
	public void callService(){
		super.callAnotherService();
	}
}
```
