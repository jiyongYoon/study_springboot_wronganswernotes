# 설계와 의존성에 대해서

## 1. SOLID
### 1) 단일 책임 원칙
- 모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화해야 함을 일컫는다.
- 코드라인이 100줄을 넘어가면, 너무 많은 책임을 지고있지는 않는지 의심해보자.

### 2) 개방-폐쇄 원칙
- 확장에 대해 열려있어야 하고, 수정에 대해서는 닫혀있어야 한다.
    - 내가 추가 기능 개발을 해야하는데, 기존 코드를 건드려야 하는 상황이다? ==> 확장에 닫혀있다.
- 즉, 새로운 기능을 추가하려고 할 때, 기존 코드는 변경하지 않은 채(Close) 기능을 추가할 수 있어야 한다.
- 보통 추상화(인터페이스화)가 부족한 경우 발생한다.

### 3) 리스코프 치환 원칙
- 하위 자료형이 상위 자료형의 모든 동작을 완전히 대체 가능해야한다.
- 때문에 상속보다는 합성(컴포지션)을 활용하자.
> ?상속 vs 합성?
> 
> 1. 상속(Inheritance) 
> 
> Java에서 흔히 말하는 부모 - 자식 관계이다. <br>
> 의존성이 컴파일 타임에 해결이 된다. <br>
> is - a 의 관계로 `Cat is Animal`, `Dog is Animal`과 같이 표현될 때 사용할 수 있다. <br>
> 부모클래스의 구현에 의존 결합도가 매우 높아, 부모 클래스 코드를 재사용할 수 있다는 강력한 장점이 있지만, 때문에 모든 자식클래스가 영향을 받게 된다.
> 
> 2. 합성(Composition)
> 
> 두 객체 사이의 의존성이 런타임에 해결이 된다. <br>
> has - a 의 관계로 `Car has an engine` 과 같이 표현될 때 사용할 수 있다.
> `Car` 클래스에서 필드로 `Engine` 클래스 변수를 갖는다. <br>

### 4) 인터페이스 분리 원칙
- `public 메서드`라고 생각해도 된다.
- 자신이 이용하지 않는 메서드에 의존하지 않아야 한다. 즉, 내가 사용할 메서드만 가지고있어야 한다. => 인터페이스를 더 잘게 쪼개야 했을수도 있다.

### 5) 의존성 역전 원칙
- `코드를 안다`, `다른 객체나 함수를 사용한다`는 것이 의존한다는 것.
- 의존성을 역전시키려면
  1. 상위 모듈은 하위 모듈에 의존해서는 안된다. 두 모듈 모두 추상화에 의존해야한다.
  2. 추상화는 세부 사항에 의존해서는 안된다. 세부사항이 추상화에 의존해야 한다.
  > 여기서 말하는 추상화는 `인터페이스`, 세부 사항은 `클래스(구현체)` 라고 이해해도 좋다.

- 그러면 의존성을 줄이려면 어떻게 할까?
  - 외부에서 받아와서 주입하면 된다! 어떻게?
    1. 파라미터 주입
    2. 필드 주입
    3. 생성자 주입
- 그러면 의존성이 사라져? 아니! 여전히 코드를 알고 있잖아.

의존성 주입(Dependency Injection)와 의존성 역전(Dependency Inversion)는 다르다.
스프링 프레임워크가 해주는 것은 의존성 주입(DI)이고, 싱글톤 객체로 컨테이너에서 생성자로 주입을 해주는 방법을 사용한다.

의존성 주입은 해주지만, 의존성 역전을 만들어주지는 않는다!

## 2. 어떻게 처리할 것인가?

### 1) 의존성을 드러내라
- 보통, `시간`, `랜덤` 에서 의존성을 많이 숨기게 된다. 실행할 때마다 '변하는 값'이 보통 그렇다.
```java
class User {

  private long lastLoginTimestamp;

  public void login() {
    // ..
    this.lastLoginTimestamp = Clock.systemUTC().millis();
  }
}
```
내부 로직을 보면 `login()` 메서드가 `Clock` 클래스에 의존적이다.
그러나 호출하는 입장에서는 <br>
`user.login();` <br>
알 수 없다.

1. 이런 상황이 어떨때는 동작하고 어떨때는 동작하지 않는 경우가 많다.
2. 테스트가 어렵다.
    ```java
    class UserTest {
          
      @Test
      public void loginTest() {
          // given
          User user = new User();
          
          // when
          user.login();
          
          // then
          assertThat(user.getLastLoginTimestamp()).isEqualTo(___??);
                                              //테스트를 할 수가 없넹....
      }
    }
    ```
   
    
    `user.login()` 메서드에 시간을 파라미터로 주입해주면 해결이 된다. <br>
    `public void login(Clock clock) {...}` 이런 식으로 말이다. <br>

### 이제 된건가?
-> 아니. `User` 클래스를 사용하는 `UserService` 테스트는? `UserService`를 사용하는 `UserController`는? <br>
결국 내부에 숨어있는 의존성을 계속 폭탄돌리기를 하고 있을 뿐이다.

> 변하는 값을 추상화 시키면 해결이 가능하다.

```java
interface ClockHolder {
    long getMillis();
}
```

```java
class User {
    private long lastLoginTimestamp;
    
    public void login(ClockHolder clockHolder) {
      // ..
      this.lastLoginTimestamp = clockHolder.getMillis();
    }
}
```

```java
class UserService { 
    private final ClockHolder clockHolder;
  
    public void login(User user) {
      //..
      user.login(clockHolder);
    }
}
```
- 이제 `UserService` 는 더이상 현재 시간을 계산해서 `.login()` 메서드에 넘기지 않아도 된다. 그 책임을 `ClockHolder` 에게 전달했기 때문이다.

```java
// 프로덕션
class SystemClockHolder implements ClockHolder {

    @Override
    public long getMillis() {
      return Clock.systemUTC().millis();
    }
}

// 테스트
class TestClockHolder implements ClockHolder {
    
    private Clock clock;
    
    @Override
    public long getMillis() {
        return clock.millis();
    }
}

// 테스트 코드
class UserServiceTest {
    @Test
    public void loginTest() {
        //given
        Clock clock = Clock.fixed(Instant.parse("2000-01-01T00:00:00.00Z"). ZoneId.of("UTC"));
        User user = new User();
        UserService userService = new UserService(new TestClockHolder(clock));
        
        //when
        userService.login(user);
        
        //then
        assertThat(user.getLastLoginTimestamp()).isEqualTo(946684800000L);
    }
}
```
> 의존성 역전 원리를 이용하여, 컴파일 타임과 런타임 의존성을 다르게 하는 것이다.

## 3. CQRS

Command and Query Responsibility Segregation. <br>
명령과 질의의 책임을 분리해라. 즉, `메서드`를 `명령`과 `질의`로 나누자. (더 넓게는 클래스까지도)

- 명령 메서드?
  - 객체의 상태를 변경시킨다. 리턴값을 갖지 않는다.(이론적이네..)
- 질의 메서드?
  - 객체의 리턴값이 있다.