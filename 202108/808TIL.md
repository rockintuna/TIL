#### 이펙티브 자바, 아이템 1. 생성자 대신 정적 팩토리 메서드를 고려하라

객체를 생성해서 반환해주는 `static` 메서드를 이용하여 생성자의 역할을 대신한다.

- **정적 팩토리 메서드를 사용했을 때의 장점**

  - 이름을 가질 수 있다.
    메서드 이름에 객체의 생성 목적을 담아 낼 수 있다.

  - 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다. 
    인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다. 

    ```java
    class MiddleSchool extends School {
        private static final MiddleSchool INSTANCE = new MiddleSchool();
    
        private MiddleSchool() {
            super("middle");
        }
    
        public static MiddleSchool getInstance() {
            return INSTANCE;
        }
    }
    ```

    

  - 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
    생성자가 생성하는 객체의 타입은 정의하는 클래스 타입으로 제한되어있지만 정적 팩토리 메서드를 사용하면 하위 타입의 객체를 생성할 수 있다.
    인터페이스에 정적 팩토리 메서드를 구현하여 이런 특징을 볼 수 있다.

    ```java
    public interface School {
    
        public String getName();
    
        public static HighSchool highSchoolOf(String name) {
            return HighSchool.of(name);
        }
    }
    ```

    ```java
    class HighSchool implements School {
    
        private String name;
    
        private HighSchool(String name) {
            this.name = name;
        }
    
        @Override
        public String getName() {
            return this.name;
        }
    
        public static HighSchool of(String name) {
            return new HighSchool(name);
        }
    
        public static void main(String[] args) {
            HighSchool kojan = School.highSchoolOf("kojan");
            System.out.println(kojan.getName());
        }
    }
    ```

    

  - 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관 없다.

    ```java
    public class School {
        private String type;
    
        protected School(String type) {
            this.type = type;
        }
    
        public String getSchoolType() {
            return type;
        }
    
        //정적 팩토리 메서드
        public static School getSchoolByType(String type) {
            if (type.equals("middle")) {
                return MiddleSchool.getInstance();
            } else if (type.equals("high")) {
                return HighSchool.getInstance();
            } else {
                throw new IllegalArgumentException("잘못된 타입입니다.");
            }
        }
    
        public static void main(String[] args) {
            School middleSchool = School.getSchoolByType("middle");
            School highSchool = School.getSchoolByType("high");
            System.out.println(middleSchool.getSchoolType()+"\n"+highSchool.getSchoolType());
        }
    
    }
    
    class MiddleSchool extends School {
        private static final MiddleSchool INSTANCE = new MiddleSchool();
    
        private MiddleSchool() {
            super("middle");
        }
    
        public static MiddleSchool getInstance() {
            return INSTANCE;
        }
    }
    
    class HighSchool extends School {
        private static final HighSchool INSTANCE = new HighSchool();
    
        private HighSchool() {
            super("high");
        }
    
        public static HighSchool getInstance() {
            return INSTANCE;
        }
    }
    ```

  - 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    JDBC의 서비스 접근 API인 DriverManager.getConnection() 메소드가 적당한 예시가 될 수 있다. 클라이언트는 세부적인 구현 내용을 몰라도 서비스를 이용할 수 있다.
    `public static Connection getConnection(String url)`
    Connection 타입은 인터페이스이며, 실제로 구현하는 클래스가 존재하지 않아도 되고 구현하는 클래스가 계속 추가될 수 도 있다.

- **정적 팩토리 메서드를 사용했을 때의 단점**
  
  - 상속을 하려면 public이나 protected 생성자가 필요하기 때문에 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
  - 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
  생성자처럼 API 설명에 명확히 드러나지 않는다.
  
- **핵심 정리**
  정적 팩토리 메소드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋습니다.
  그렇다고 하더라도 정적 팩토리를 사용하는 게 유리한 경우가 더 많으므로 무작정 pulbic 생성자를 제공하던 습관이 있다면 고치도록 합니다.



#### 클린 코드, 2. 의미 있는 이름

- **의도가 분명하게 이름을 사용하라.**
  변수나 함수, 클래스 이름은 그 존재 이유와 수행 기능, 사용 방법을 설명할 수 있어야 한다.

- **코드의 의미를 흐리는 그릇된 정보를 피하라.**
  ex) `Map<int, Account> accountList`

- **의미있게 구분할 수 있는 이름을 사용하라.**
  a1, a2, ..., aN 같은 의도가 전혀 드러나지 않는 이름은 사용하지 않는다.
  Info, Data, a, an, the 같은 불용어를 별다른 의미없이 사용하는것은 적절하지 않다.

- **발음하기 쉬운 이름을 사용하라.**
  줄임말 같은 표현으로 발음이 이상해지는 이름은 좋지 않다.

- **검색하기 쉬운 이름을 사용하라.**
  `grep`이나 find 명령으로 찾을 때 쉽게 찾을 수 있는 단어를 사용하는 것이 좋다.

- **인코딩을 피하라.**

  - 헝가리식 표기법
    이름 첫글자에 그 요소의 타입을 주는 방법이다. 최근의 프로그래밍 언어는 컴파일러가 타입을 기억하고 제어하므로 타입 인코딩이 불필요하다.
  - 멤버 변수 접두어
    멤버 변수와 그 외 변수를 구분하기 위해서 멤버 변수에 `m_`라는 접두어를 붙이던 관습이 있었으나 이제는 필요가 없다.
  - 인터페이스 클래스와 구현 클래스
    추상화나 다형성을 사용하기 편하도록 인터페이스 이름에는 인코딩을 하지 않고 필요하다면 클래스 이름에 인코딩을 한다.
    ex) interface ShapeFactory , class ShapeFactoryImp

- **자신의 기억력을 자랑하지 마라.**
  코드를 읽을 때 자신이 이해할만한 다른 이름으로 변환해야 하는 이름은 피한다.
  루프를 제외한 다른 곳에서 문자 하나의 이름을 사용하는 것은 문제가 있다.
  남들이 이해하기 쉬울 정도로 명료한 것이 최고다.

- **클래스 이름은 명사나 명사구, 메서드 이름은 동사나 동사구로.**

- **기발한 이름은 피한다.**
  나만의 또는 어떤 집단만의 유머가 담긴 코드는 재치있긴 하지만 가독성 제로

- **한 개념에 한 단어만 사용하라.**
  같은 기능을 하는 요소를 다른 이름으로 사용하지 말자.
  ex) `get(), fetch(), retrieve()`, `Controller, Manager, Driver`

- **말장난을 하지 마라.**
  여기서의 말장난은 펀치라인을 말하는 것 같다. 한 단어를 두가지 뜻으로 말하는 바로 그것.
  예를 들면 기존에 add라는 메서드를 숫자 더하기로 구현하다가 새로 작성하는 add 메서드에 집합에 요소를 추가하는 기능을 구현하면 말장난이다.

- **해법 영역에서 가져온 이름을 사용하라.**
  적절한 프로그래밍 용어를 이름에 사용하는 것은 내용을 이해하기 쉽도록 도와준다.

- **문제 영역에서 가져온 이름을 사용하라.**
  도메인과 관련이 깊은 코드는 도메인에서 이름을 가져온다.

- **의미있는 맥락을 추가하라.**
  스스로 의미가 분명하지 못한 이름에는 맥락을 부여한다. 

  - 클래스, 함수, 이름공간에 넣어서 맥락을 부여한다.

  - 모든 방법이 실패하면 접두어를 사용한다.
    ex) firstName, lastName, state 대신 addrFirstName, addrLastName, addrState

- **불필요한 맥락을 없애라.**
  ex) 애플리케이션 이름을 모든 클래스 앞에 붙이는 등.