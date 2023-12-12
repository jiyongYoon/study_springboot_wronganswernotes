# 테스트에 대해서

---

## 1. 테스트를 해야하는 이유?

## 2. 테스트의 종류
- Unit test / 통합 테스트 / API 테스트... 보통 구분 짓는 내용이지만 통합테스트 쪽의 범위가 다소 애매하다.
- `구글 엔지니어는 이렇게 일한다` 책에 제시된 테스트 종류: 소형 / 중형 / 대형
  - 소형: 단일 서버, 단일 프로세스, 단일 스레드, 디스크I/O 없음, Blocking call 허용 안됨.
  - 중형: 단일 서버, 멀티 프로세스, 멀티 스레드, 테스트용 DB(H2와 같은) 사용 가능
  - 대형: 멀티 서버, End to End 테스트
- Unit test - 80%, Integration test - 15%, E2E test - 5% 정도가 이상적인 비율이라고 함

## 3. 테스트 안티패턴
- 아이스크림 패턴
  - 테스트 비중이 역삼각형인 경우.
- 모래 시계 패턴
  - E2E 테스트와 Unit 테스트가 많은 경우.

## 4. 테스트시 사용하는 대역 객체들

### 1) Dummy
아무런 동작도 하지 않고, 그저 코드가 정상적으로 돌아가기 위해 전달하는 객체. <br>
ex) 이메일을 발송하는 로직이 들어있다고 한다면, 해당 객체를 Dummy 객체로 만들어서 제공하는 방법을 사용하여 메일 발송이 되지 않게 함.

### 2) Fake
Local에서 사용하거나 테스트에서 사용하기 위해 만들어진 가짜 객체. 자체적인 로직이 있다. <br>
ex) 메일을 실제로 발송하지는 않지만, 어떤 메일을 보내는지 메시지를 기록한다던가 하는 자체 로직을 구현해두어 테스트에 사용할 수 있음.

### 3) Stub
미리 준비된 값을 출력하는 객체. 주로 외부 연동하는 컴포넌트들에 많이 사용됨. <br>
ex) Mockito 의 given 으로 구현하는 것이 대표적인 Stub 객체

### 4) Mock
메서드 호출을 확인하기 위한 객체. 자가 검증 능력을 갖춤.
```java
final class MockMailer implements Mailer {
    private bool hasBeenCalled = false;
    public void sendWelcomeEmail(Long userId) {
        this.hasBeenCalled = true;
    }
    
    public boolean hasBeenCalled() {
        return this.hasBeenCalled;
    }
}
```
해당 객체는 메일을 보내는 역할을 하는게 아니라, 해당 메서드가 호출되었는지를 언제든 확인해볼 수 있는 객체임.

### 5) Spy
메서드 호출을 전부 기록했다가 나중에 확인하기 위한 객체

```java
import java.util.ArrayList;

final class EventDispatcherSpy implements EventDispatcher {

    private List<Object> events = new ArrayList<>();
    public void dispatch(Object event) {
        this.events.add(event);
    }
    
    public List<Object> dispatchedEvents() {
        return this.events;
    }
}
```

## 5. 테스트 시 생길 고민들에 대한 내용

### 1) private 메서드
[NO](https://shoulditestprivatemethods.com/)
- 주석에는 이렇게 적혀있음
```
Every time you consider testing a private method, 
your code is telling you that you haven't allocated responsibilities well.  
Are you listening to it?
```

메서드 자체를 테스트 대상으로 여기기 때문에 발생하는 고민인데, 우리가 테스트를 해야하는 대상은 `객체의 행위`이다. 그리고 그 상태 변화를 `검증`하는 것이 테스트이다. <br>

따라서, `public` 메서드가 보통 객체가 책임지는 하나의 행위가 되게 되고, 거기서 사용하는 여러 메서드들이 `private`로 들어가 있지만, 굳이 테스트 대상이 아니라는 말이 된다.
테스트의 의미와 대상을 잘 생각해보자. <br>

만약 그래도 필요한 상왕이라면, 그 코드가 다른 클래스로 분리되어 새로운 책임을 져야하는 상황일수도 있다.

### 2) final 메서드
`final` 메서드를 stub 해야하는 상황이 생긴다면, 무언가 설계가 잘못된 것이다.
만약 무언가 변화가 필요하다면, 해당 메서드를 가진 클래스를 따로 두고 의존성을 약하게 만드는 방법을 생각해보아야 함.

### 3) DRY < DAMP
테스트에서는 잘 읽히는 것에 먼저 집중하도록 하자. 중복 코드가 있어도 괜찮다.

### 4) 논리
테스트에 논리 로직(if, for, 사칙연산 등)을 넣지 말자.