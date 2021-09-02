#### 이펙티브 자바, 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라.

생성자든 정적 팩토리 메서드든 매개변수가 많다면 적절하게 대응하기 쉽지 않다. 생성자에 매개변수가 많을 때 사용하는 방법들을 알아보았다.

- **점층적 생성자 패턴**
  필수적 매개변수만 받는 생성자부터 시작해서 선택적 매개변수 한 개씩을 늘려가면서 모든 매개변수를 받는 생성자까지 늘려가는 단순한 방식이다.

  ```java
  public class Book {
  
      private String title;
      private String author;
      private int price;
      private String publisher;
      private String description;
      private String subTitle;
  
      public Book(String title, String author, int price) {
          this(title, author, price, null, null, null);
      }
  
      public Book(String title, String author, int price, String publisher) {
          this(title, author, price, publisher, null, null);
      }
  
      public Book(String title, String author, int price, String publisher, String description) {
          this(title, author, price, publisher, description, null);
      }
  
      public Book(String title, String author, int price, String publisher, String description, String subTitle) {
          this.title = title;
          this.author = author;
          this.price = price;
          this.publisher = publisher;
          this.description = description;
          this.subTitle = subTitle;
      }
  }
  ```

  문제는 사용자가 설정하길 원하지 않는 매개변수 까지 포함할 수 있다는 것. 
  예를들어 아래와 같다.
  `Book myBook = new Book("이펙티브 자바 3/E", "조슈아 블로크", 36000, null, "자바를 공부하기에 적절한 책입니다.");`
  description 멤버를 설정하기 위해서는 원치 않더라도 publisher에 null이나 다른 값을 입력해야 한다.

  점층적 생성자 패턴을 사용할 수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려워진다.
  특히 매개변수의 순서를 잘 알아야하는데, 같은 타입이지만 서로 다른 매개변수의 순서를 다르게 입력하더라도 컴파일러는 실수인지 알지 못한다.

- **자바빈즈 패턴**
  우리가 흔히 볼수있는 세터를 사용하는 방법이다. 자바빈즈 패턴은 점층적 생성자 패턴에 비해서 읽고 사용하기 쉬운 코드가 된다.

  ```java
  public class Book {
  
      private String title;
      private String author;
      private int price;
      private String publisher;
      private String description;
      private String subTitle;
  
      public Book() {
      }
  
      public void setTitle(String title) {
          this.title = title;
      }
  
      public void setAuthor(String author) {
          this.author = author;
      }
  
      public void setPrice(int price) {
          this.price = price;
      }
  
      public void setPublisher(String publisher) {
          this.publisher = publisher;
      }
  
      public void setDescription(String description) {
          this.description = description;
      }
  
      public void setSubTitle(String subTitle) {
          this.subTitle = subTitle;
      }
  }
  ```

  그러나 자바빈즈 패턴은 객체 하나를 만들려면 메서드를 여러개 호출해야하고 객체가 완성되기 전 까지는 일관성이 무너진 상태에 놓이게 된다는 심각한 단점이있다.

  ```java
      public static void main(String[] args) {
          Book myBook = new Book();
          myBook.setTitle("이펙티브 자바 3/E");
          myBook.setAuthor("조슈아 블로크");
          myBook.setPrice(36000);
          //------- 여기까지 오기 전까지는 일관성이 무너진 상태이다.
        
          //...
      }
  ```

  

- **빌더 패턴**
  빌더 패턴은 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 둘다 가져갈 수 있는 방법이다. 빌더 패턴은 파이썬과 스칼라에 있는 `named optional parameter`를 흉내낸 것이다.
  클라이언트는 필수 매개변수로 빌더 생성자를 호출하여 빌더 객체를 만들고 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택적 매개변수를 설정한 뒤 build() 메서드를 통해 필요한 객체를 얻을 수 있다.

  빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는게 보통이다. 일반적으로 빌더 생성자와 메서드에서 개별 입력 매개변수를 검사하고 build() 메서드가 호출하는 생성자에서 불변식을 검사한다.

  ```java
  public class Book {
  
      private final String title;
      private final String author;
      private final int price;
      private final String publisher;
      private final String description;
      private final String subTitle;
  
      public Book(Builder builder) {
          this.title = builder.title;
          this.author = builder.author;
          this.price = builder.price;
          this.publisher = builder.publisher;
          this.description = builder.description;
          this.subTitle = builder.subTitle;
      }
  
      public static class Builder {
          private final String title;
          private final String author;
          private final int price;
  
          //선택적 매개변수는 기본값을 미리 설정한다.
          private String publisher = null;
          private String description = null;
          private String subTitle = null;
  
          public Builder(String title, String author, int price) {
              this.title = title;
              this.author = author;
              this.price = price;
          }
  
          public Builder publisher(String publisher) {
              this.publisher = publisher;
              return this;
          }
  
          public Builder description(String description) {
              this.description = description;
              return this;
          }
  
          public Builder subTitle(String subTitle) {
              this.subTitle = subTitle;
              return this;
          }
  
          public Book build() {
              return new Book(this);
          }
      }
  }
  ```

  객체를 만드는 클라이언트 코드는 아래와 같다. 쓰기 쉽고 읽기 쉽다.

  ```java
  Book myBook = new Book.Builder("이펙티브 자바 3/E", "조슈아 블로크", 36000)
          //빌더의 세터메서드들이 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. (메서드 체이닝)
          .publisher("인사이트").description("자바를 공부하기에 적절한 책입니다.").build();
  ```

  빌더 패턴은 객체를 생성하기 전에 빌더부터 생성해야되므로 성능적으로는 단점이 있다. 또한 점층적 생성자 패턴보다 코드가 장황해서 매개변수가 적을때는 적절하지 않다. 

- **핵심정리**
  생성자나 정적 팩터리가 처리해야 할 매개변수가 많거나 많아질 것으로 예상된다면 빌더 패턴을 선택하는 게 더 낫다.
  매개변수 중 다수가 필수가 아니거나 같은 타입이라면 더더욱.
  빌더는 점층적 생성자 패턴보다 간결하고 자바빈즈보다 안전하다.



#### 클린 코드, 3. 함수

함수를 잘 만드는 법!

- **작게 만들어라.**
  조건문/반복문에 들어가는 블록을 한줄로 줄인다. 보통 다른 함수를 호출한다.
- **한 가지만 해라.**
  함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.
  여기에서 한 가지 작업이란 지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행하는 것이다.
  만약 의미 있는 이름으로 다른 함수로 추출할 수 있다면 그 함수는 여러 작업을 하고 있는 것이다.
- 함수 당 추상화 수준은 하나로.
  TO 문단을 읽어내려 가듯이 코드를 구현하여 각 함수가 일정한 추상화 수준을 유지하면서 다음 함수를 참고하고 소개하도록 한다. 
- Switch 문
  switch 문은 작게 만들거나 한 가지 작업만 하게 만들기 어렵다. 그러므로 switch 문을 추상 팩토리에 담아서 다형성을 통해 다른 코드에서 같은 switch 문이 발생하지 않고 switch 문을 숨길 수 있도록 할 수 있다.
- 서술적인 이름을 사용하라.
  길고 서술적인 이름이 짧고 어려운 이름보다 좋으니 여러 단어를 사용하여 함수의 기능을 잘 표현하는 이름을 선택하자.
  서술적인 이름을 사용하면 설계가 뚜렷해지므로 코드를 개선하기 쉬워진다.
- 함수 인수
  최선은 함수 인수가 없는 경우이며, 차선은 1개, 늘어날 수록 좋지 않다. 함수 인수로 사용하는 것 대신 클래스의 인스턴스 변수로 선언하여 사용하자.
  - 많이 쓰는 단항 형식
    인수에 질문을 던지는 함수, 인수를 변환하여 결과를 반환하는 함수, 입력 인수로 시스템 상태를 바꾸는 이벤트 함수 세가지 경우가 아닌 함수는 단항 함수를 피한다.
  - 플래그 인수는 피한다. ex) `render(true)`
  - 이항 함수
    이항 함수는 단항 함수보다 이해하기 어렵다. 가능하면 단항 함수로 바꾸도록 애써야한다.
  - 인수 객체
    인수가 2-3개 필요하다면 일부를 독자적인 클래스 변수로 선언해보는 것을 고려한다.
    `Circle makeCircle(double x, double y, double radius);` ====>
    `Circle makeCircle(Point center, double radius);`
  - 동사와 키워드
    단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다. ex) `writeField(name)`
    함수 이름에 키워드를 추가할 수도 있다. ex) `assertExpectedEqualsActual(expected, actual)`
  - 출력 인수
    출력 인수 쓰지말고 클래스 변수를 this로 변경하는 방식을 사용하자.
- 부수 효과를 일으키지 마라.
  메서드의 이름으로는 확인되지 않는 부수 효과를 숨기고 있으면 안된다.
- 명령과 조회를 분리하라.
  객체 상태를 변경하거나 객체 정보를 반환하거나 둘 중 하나만 해야 한다. 둘다 하면 혼란스럽다.
- 오류 코드보다 예외를 사용하라.
  오류 코드를 사용하면 조건문이 여러 단계로 중첩될 수 있고 오류 처리 코드까지 같이 작성해야 한다.
  오류 코드 대신 예외를 사용하면 오류 처리 코드를 원래 코드에서 분리할 수 있다.
  try/catch 블록은 코드 구조를 혼란스럽게 하므로 별도의 함수로 뽑아내는 편이 좋다.
- 반복하지 마라.
- 구조적 프로그래밍
  함수가 아주 크다면 되도록 단일 입/출구 규칙을 지킨다. (break, continue, goto를 사용하지 않고 return 문은 한번만 있어야 한다.)
- 함수 짜기
  처음부터 위의 규칙에 맞춰 짜는 것은 쉽지 않다. **리펙터링**이 중요하다.

