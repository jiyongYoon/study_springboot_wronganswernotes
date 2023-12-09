# 스프링 프레임워크에 대해서

---
## 1. 비즈니스 로직을 도메인에게 위임하자

### 1) 컨트롤러
- 어떤 서비스를 실행할지 선택하는 정도의 역할

### 2) 서비스
> 소프트웨어가 수행할 작업을 정의하고 표현력있는 도메인 객체가 문제를 해결하게 하는 레이어,
> 다른 시스템의 응용 계층과 상호작용하는 데 필요한 것들.
> 
> 이 계층은 최대한 얇게 유지되어야 한다. 업무 규칙이나 지식이 포함되지 않아야 한다.
> **오직 작업을 조정하고 아래에 위치한 계층에 도메인 객체의 협력자에게 작업을 위임하자** <br>
> - 에릭 에반스, 도메인 주도 설계 소프트웨어의 복잡성을 다루는 지혜

이 계층이 두터울수록 트랜잭션 스크립트(트랜잭션을 실행하는 스크립트 역할이기 때문), 절차 지향적 코드가 되어있을 가능성이 높다.

> 작업을 조정하고 비즈니스 로직은 도메인에게 위임하자!
> 그리고, 비즈니스 로직 중 어떤 도메인에게 위임하기가 어려운 로직이 있다면, 그건 새로운 도메인 객체가 필요하다는 뜻이다!
> => `도메인 서비스(로직 자체가 목적인 객체)`

dependency 파트에서 사용했던 예시코드를 다시보면
```java
class User {

  private long lastLoginTimestamp;

  public void login() {
    // ..
    this.lastLoginTimestamp = Clock.systemUTC().millis();
  }
}
```
해당 코드는 `User`라는 객체가 로그인 로직을 들고 있는 것이다.
극단적인 예시지만, 만약 이게 `Service` 로 간다면 아래와 같겠다.

```java
class UserService {

    private final UserRepository userRepository;

    public void login(long id) {
        User user = userRepository.findById(id);
        user.lastLoginTimestamp = Clock.systemUTC().millis();
    }
}
```


### 그렇다면 기존에 내가 배웠고, 지금까지 짜고있던 `Transaction script` 방식은 과연 안티패턴인 것인가??! 
결국 `Trade-off`가 있는 것이다. 기존 절차지향적인 방식은 테스트를 하기 힘들고 OOP스럽지 않다는 특징이 있다. <br>
절차 지향적이기 때문에 `코드를 읽기 쉽고 개발속도가 빠르다`는 장점이 있다. 하지만, 서비스가 커진다면 점점 복잡하고 버거운 상황이 올 것이다!

---
## 2. 추상화, 어디까지 해야하나?

> **추상화**: 모듈을 격리하고 인터페이스로 만드는 과정

### 1) 시스템 외부 연동은 가능하면 모두 추상화 하자
- 대상: `DB`, `WebClient`, `RestTemplate` 등

**JPA**

`Repository` 인터페이스를 만들고`(A)`, 해당 인터페이스를 구현한 구현체`(C)`를 두고 그 구현체가 `JpaRepository`를 구현`(B)`하게 하자.

```java
//(A)
interface PostRepository {

    Post getById(long id);

    Optional<Post> findById(long id);

    List<Post> findAll();

    void deleteByIdIn(List<Long> ids);
}

//(B)
public interface PostRepository extends JpaRepository<PostEntity, Long> {

    void deleteByIdIn(List<Long> ids);

    List<PostEntity> findAll();
}

//(C)
@RequiredArgsConstructor
class PostRepositoryImpl implements PostRepository {

    private final PostJpaRepository postJpaRepository;
    
    @Override
    public Post getById(long id) {
        return postJpaRepository.findById(id)
            .orElseThrow()
            .toDomain();
    }
    
    @Override
    public Optional<Post> findById(long id) {
        return postJpaRepository.findById(id).map(PostEntity::toDomain);
    }
}
```
=> `Entity`는 실제로 `JPA`에 의존되어 있을 가능성이 높기 때문에 순수 객체인 `Domain`으로 한번 더 결합을 끊어주면 좋을 수 있다.

- 추가! 그리고 이렇게 되면, 테스트 시 H2와 같은 Repository가 아니라 In-memory 로직으로 `Fake Repository`를 만들어 데이터를 잡아서 테스트를 돌릴 수도 있다.

```java
@Repository
class LocalPostRepositoryImpl implements PostRepository {
    private int incrementId = 1;
    private final List<Post> postList = new ArrayList<>();
    
    // 필요한 메서드 Override
}
```
이렇게 DB역할을 할 Repository를 만들어서 테스트할 수 있음!

### 2) 서비스 레이어는 추상화가 필수적이지 않다.

Controller, Service, Entity, VO는 구현체로 구현되어도 상관이 없다. 왜냐하면, 어차피 그러한 객체들은 한번 생성하여 영원히 그 일을 하게 하려고 만든 객체이기 때문이다!
> 내가 이해한 말로 해석해보면,<br>
> 어차피 그 일은 그 객체에게 시킬 것이기 때문에 인터페이스를 만들어도 실제 그 인터페이스를 구체화 할 객체가 그 객체밖에 없을 것이라는 말인것 같다.

### 3) 도메인 영역을 만들자

도메인 레이어는 순서로 따지면 
`사용자 인터페이스` -> `응용 계층` -> `도메인 계층` -> `인프라 스트럭쳐 계층`으로 나누어서 3번째에 위치하게 되는데, 
의존성 역전의 기본 원칙인 `고수준 모듈이 저수준 모듈에 의존하지 않게 하는 것`에 따라 `Repository` 객체를 직접 사용할 수 없다. <br>
때문에, `도메인 객체`에서 `Repository`를 사용하려면, `Service` 에서 객체를 파라미터로 넘겨주어야 한다.
`Repository` 뿐만 아니라 `도메인 객체`에서 작업에 필요한 협력 객체들은 모두 `Service` 에서 파라미터로 넘겨주어야 한다.
> 여기에는 불필요한 쿼리가 나가는 부작용도 있어서, Domain 레이어에서도 Repository를 사용할 수 있도록 해야한다는 주장도 있다고 하는데,,, 나중에 추가로 학습해보자...!
> [관련링크](https://softwareengineering.stackexchange.com/questions/330428/ddd-repositories-in-application-or-domain-service)

![image](https://github.com/jiyongYoon/jiyongYoon.github.io/assets/98104603/f9ea5a8d-0350-4022-93d9-bb540e7d76f6)

---
## 3. 서비스란 무엇인가
> 스프링에서의 `@Service`에 대한 설명 <br>
> "Service", originally defined by Domain-Driven Design(Evans, 2003) ...
> May also indicate that a class is a "Business Service Facade"(in the Core J2EE patterns sence), or something similar. ...
> 
> **서비스는 DDD에서 가져온 개념이고, 비즈니스 서비스의 Facade이다.**



---

> **학습을 하며 느낀점**
>
> 서비스는 도메인들을 매니징하고 사용하는 레이어라는 생각이 든다.
> 도메인들은 각 도메인에 필요한 작업을 `직접` 수행할 줄 알아야 할 것이고,
> 서비스는 도메인들에게 적재적소에 필요한 `재료`를 `파라미터`로 넘겨주는 역할을 하게 하는 것이 맞겠다.
>
> 조금 더 나아가면, <br>
> 내가 사용하고 있는 `스프링 프레임워크`라는 말에서 나와있듯이,
> `프레임워크`는 `어플리케이션`에 해당하는 말이지 내 `비즈니스`와는 상관이 없어야 하겠다. <br>
> 만약 `어플리케이션 서비스` 클래스에 내 비즈니스 로직이 다 묶여있다면, 스프링 프레임워크를 탈출할 수 없게 된다. <br>
>
> `도메인 서비스`와 `도메인`이 비즈니스 로직을 갖게 되면 이 코드는 `스프링 프레임워크`와 무관한 코드가 되고, 언제든 `프레임워크` 밖으로 빠져나갈 수 있을 것이다.


