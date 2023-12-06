# 일반적인 컨벤션

## 1. 이름
### 1) Java
- 변수이름: camelCase, 소문자 시작
- 함수이름: camelCase, 동사로 시작
- 클래스이름: PascalCase
- 패키지이름: alllowercase
- 상수: UPPER_SNAKE_CASE

### 2) 줄여쓰기
- 가능하면 풀어쓰자
  - message <-> msg
  - project <-> prj
  - object <-> obj
  - webSocket <-> ws
  - webServer <-> ws
- 길어도 괜찮다. ex) spring-security의 RequestMatcherDelegatingAuthenticationMAnagerResolver 클래스

### 3) 축약어
- userId? > userID?
- oidcId? > oicdID?
- restApi? > REST_API?
- ip? > iP?
- OAuth는 좀 곤란....

### 4) 클래스이름
- 유의미한 단어를 쓰자
  - Simple, Light, Base와 같은 표현이 애매한 것은 개발자가 많아질수록 뜻이 통용되기 어렵다.
- Util 보다 정확한 단어를 찾아보자
  ```java
  class ApplicationUtil {
        
        public static Application create() {
            return new Application();
        }
  }
    ```
    대신
  ```java
  class ApplicationFactory {
        
        public static Application create() {
            return new Application();
        }
  } 
  ```
  처럼 클래스의 의미가 명확해지도록 변경하자. Util은 온갖 static 메서드가 모이기 딱 좋다..

## 2. 동사
### 1) get vs find
- get은 일반적으로 항상 인스턴스를 돌려받는다는 의미. 따라서  데이터가 없을 경우 exception throw
  - ex) JPA의 `getById()`
- find는 일반적으로 데이터가 없을 경우를 가정하여 Optional로 리턴
  - ex) JPA의 `findById()`

### 2) get을 남발하지 않기
- 내가 가진 변수를 리턴한다는 의미가 큼. 어디서 계산을 해서 가져온다거나 하는 경우에 남발하지 말자.
  - ex) getter, setter 에서처럼 사용하는 경우
  - `getTotalPrice()` --> `sumPrice()`

## 3. 롬복과 Getter, Setter
- getter, setter는 남발하지 말자.
  - 사실상 public으로 사용이 가능해지다보니 캡슐화가 깨짐.
    - 그렇다 보니 간단한 변수들은 setter를 사용하려는 유혹이 강해짐
      - 그렇다 보니 객체를 수동적이게 만듦.
        ```java
        @Getter
        @Setter
        class User {
            private String name;
            private Status status;
            private long lastLoginTimestamp;
        }
        
        class UserService {
            
            public void doSomething(User user) {
                user.setStatus(Status.ACTIVATE);
                user.setLastLoginTimestamp(Clock.systemUTC().millis());
            }
        }
        ```
        이 유저 객체는 본인이 아무일도 안함. 이 유저 객체에 메서드를 주어 맴버 변수를 변하게 하는게 필요함.
        
        ```java
        class User {
            private String name;
            private Status status;
            private long lastLoginTimestamp;
        
            public void login() {
                this.status = Status.ACTIVATE;
                this.lastLoginTimestamp = Clock.systemUTC().millis();
            }
        }
        
        class UserService {
            
            public void doSomething(User user) {
                user.login();
            }
        }
        ```
        
## 4. 가독성
### 1) 주석
- 주석은 필요한 곳에 잘 사용하는게 중요하다.
  - 주석을 달아야 할 필요성이 있다면, 클래스나 메서드의 전체적인 설명에 다는 것이 좋고, 코드 한줄마다 필요한 경우는 오히려 `메서드 분리`의 신호로 받아들일 필요도 있다. 
    - 메서드가 분리되면서 메서드명이 해당 코드를 설명하는 역할을 할 수 있게 된다.

### 2) Collection.Map 사용 주의
- Key - Value 또는 Value에 또 다른 Collection 들을 넣을 수 있어서 클래스 내에 변수로 가지고 중요한 역할을 하는 경우가 생김
  - 이 경우에는 해당 데이터를 클래스를 분리하는게 맞는 경우도 있다.
  - 웬만하면 Collection.Map은 일급 클래스로 만들자.
  
`일급 클래스?` 
- Collection을 Wrapping 하면서, 그 외 다른 멤버 변수가 없는 상태.
    ```java
    Map<String, String> map = new HashMap<>();
    map.put("1", "A");
    map.put("2", "B");
    map.put("3", "C");
    ```

    ```java
    public class GameRanking {

        private Map<String, String> ranks;

        public GameRanking(Map<String, String> ranks) {
            this.ranks = ranks;
        }
    }
  ```
    위와 같은 클래스를 아래처럼 바꾸면 `비지니스에 종속적인 자료구조`가 되며, 이름, 상태, 행위 등의 여러가지 의미를 가지게 된다! => 객체 지향적이 된다. 


## 5. 관습
- 파라미터의 범위
  - 보통 시작부분은 포함, 끝부분은 제외
    `str.substring(startIndex, endIndex);` -> startIndex는 포함, endIndex는 제외.