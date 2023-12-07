# 객체에 대한 오답노트

## 1. 객체의 종류
### 1) VO (Value Object)

```java
class UserInfo {

    // 값이 할당된 후 변하지 않음
    private final long id;
    private final String username;
    private final String email;

    // 생성자의 2가지 역할, 값 검증 & 값 할당
    public UserInfo(long id, String username, String email) {
        // 값 검증
        assert id > 0;
        assert StringUtils.hasText(username);
        assert EmailValidator.isValid(email);
        // 값 할당
        this.id = id;
        this.userName = username;
        this.email = email;
    }
}
```
VO 객체는 생성되면, 소멸될 때까지 항상 그 멤버변수의 값은 `유효`하고 `불변`한다는 것을 보장한다.
이 보장만으로도 프로그램의 복잡도를 엄청나게 낮추어준다!

- 만약 VO가 값이 변경될 필요가 있는 상황이라면, 그 때는 `새로운 VO를 반환`하는 것이어야 한다.
  - 그리고 그러한 동작을 하는 메서드는 전치사로 시작하는 메서드가 보통이다. `ex) UserInfo withNewEmail(String newEmail)`

### 2) DTO (Data Transfer Object)

```java
@Getter
@Setter
class UserInfo {
    // 사실상 Getter, Setter로 인해 변수가 노출된거나 마찬가지임.
    private long id;
    private String username;
    private String email;
}
```

데이터베이스와는 관련이 없다. 메서드간, 클래스간, 하여간 데이터를 주고받는곳에 사용하는 객체를 말한다. 모든 값을 다 파라미터로 넘길수는 없으니까...

그렇기 때문에 변수에 대한 어떠한 보장도 없다. 사실상 public 변수라고 봐도 무방하다.

### 3) Entity

```java
@Entity // jpa
class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String username;
    private String email;
}
```
`식별자`, `생명주기`가 있으며, `보통 DB에 저장`된다.
여기서 주의할 점은 `꼭 DB에 저장되는 객체를 Entity라고 하는건 아니다`라는 것이다.

### 4) DAO (Data Access Object)
이거는 DTO같은 Object가 아니고, 현재는 Repository라고 통용되는 객체다...

### 5) 기타
- BO (Business Object)
- SC (Service Object)

> !중요!
> 
> 어떤 종류의 객체를 만드는지가 중요한것은 아니다.
> 결국, 객체를 만들 때 고민해야할 것은 
> 1. 어떤 값을 불변으로 만들 것인가?
> 2. 어떤 인터페이스(메서드)를 노출할 것인가?
> 
> 여야 한다.


## 2. 디미터 법칙
최소 지식의 법칙. 모듈은 자신이 조작하는 객체의 속사정을 몰라야한다. 객체 내부 체이닝으로 들어가서 조작하는 코드가 있다면 그러한 것!

무슨말이냐고? 아래 코드를 보자

```java
class ComputerManager {
    public void printSpec(Computer computer) {
        long size = 0;
        for (int i = 0; i < computer.getDisks().size(); i++) {
            size += computer.getDisks().get(i).getSize();
        }
        System.out.println(size);
    }
}
```
ComputerManager가 Computer 객체의 속사정(disk)에 대해 너무 깊게 알고있다.

```java
class ComputerManager {
    public void printSpec(Computer computer) {
        System.out.println(computer.getDiskSize());
    }
}
```
얘는 속사정은 모른다. 그렇지만 일을 수동적으로 한다.

```java
class Computer {
    // 멤버변수 존재
    public void printSpec() {
        System.out.println(this.diskSize);
    }
}
```
Computer 객체에 일을 시키도록 하여 객체를 능동적으로 만들자!


## 3. 행동
행동 위주의 사고를 하자. 객체 지향적일 확률이 훨씬 높다.

```java
duck typing, 덕 타이핑?

> 만약, 어떤 새가 오리처럼 걷고, 헤엄치고, 꽥꽥 소리를 낸다면, 나는 그 새를 오리라고 부를 것이다.class 

실제 타입(클래스)가 아니라 구현된 메서드로만 판단하는 방식.

```

## 4. 순환참조
- 순환참조가 생긴다는 것은, 사실 그 두 클래스가 하나의 클래스여야 했을수 있다는 뜻. (결합도가 매우매우 높기 때문)
- Serialize가 불가능해짐.

### 그럼 어떻게 해결하나?

**1. 간접참조 이용**

```java
class User {
    private long id;
    private String username;
    private List<Feed> feeds;
}

class Feed {
    private long id;
    private String content;
    private User writer;
}
```
위 클래스를 아래 클래스로
```java
class User {
    private long id;
    private String username;
    private List<Feed> feeds;
}

class Feed {
    private long id;
    private String content;
    private long writerId; // 간접참조 이용하여 필요할때 Repo에서 가져오자
}
```

**2. 컴포넌트 분리**

`A`와 `B`가 서로를 참조하고 있다면, 공통적으로 필요로 하는 것을 `C`로 다시 빼서 `A`, `B`에서 `C`를 참조하도록 변경하자.