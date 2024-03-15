# 전략 패턴(Strategy Pattern)
- 행위 패턴 중 하나이다.
- **동일한 기능을 제공**하지만 **동적으로 알고리즘을 변경**할 때 사용하는 패턴이다.
- 특징은 context가 직접 전략을 선택하지 않는다는 점이다. concrete strategy는 context를 호출하는 client 코드가 생성 및 주입해준다.
  - 이때, client와 concrete strategy 간의 의존성이 생기는데 유지보수에 큰 영향이 생기진 않는다. 쌍을 이루기 때문에 기능이 제거되면 함께 제거되기 때문이다.
- 새로운 전략이 추가되어도 context를 변경되지 않기 때문에 OCP와 관련이 깊다.
![strategyPattern.png](..%2F..%2Fimg%2Fjava%2FstrategyPattern.png)
- Spring에서는 Repository 인터페이스를 만들 때 자주 사용하는 패턴이다.
