# Lambda

## 람다식

하나의 추상 메소드를 가진 **함수형 인터페이스의 구현**을 간결하게 표현하는 문법

`(타입 생략한 매개변수명) -> { 실행할 코드 }`

- JavaScript 의 화살표 함수 `() => {}` 와 비슷하다.
- 함수형 인터페이스의 동작은 어차피 하나이기에 람다식으로 표현이 가능하다.
- 람다에 바깥의 변수도 사용 가능하지만 나중에 실행될 수 있기에 람다에 사용한 지역 변수는 변경할 수 없다.

```java
public static void main(String[] args) {
    /*
    * [람다식]
    * 하나의 추상 메소드를 가진 함수형 인터페이스의 구현을 간결하게 표현하는 문법
    * */
    Calculator lambdaAdd = (a, b) -> a + b;
    System.out.println("람다식 덧셈 : " + calculate(10, 20, lambdaAdd));
    
    calculate(10, 19, (a, b) -> a - b);
  
    // 람다 바깥의 변수도 사용 가능
    int bonus = 5;
    Calculator bonusCalculator = (a, b) -> a + b + bonus;
	  //        bonus = 10;
    /*
    * 람다가 참조하는 지역변수는 처음 정한 뒤 다시 대입할 수 없다.
    * 
    * 선언된 자리에서 지역 변수의 값을 가져가 나중에 실행될 수 있기 때문.
    * 그 사이 지역 변수 값이 계속 바뀐다면 람다가 어느 값을 사용해야 하는지 흐름을 이해하기 어려워진다.
    * 이러한 혼란을 막기 위해 자바는 람다가 참조하는 지역변수는 처음 정한 뒤 다시 대입하지 못하게 한다.
    * */
    calculate(10, 20, bonusCalculator);
}

public static int calculate(int first, int second, Calculator calculator) {
    return calculator.operate(first, second);
}
```

### 함수형 인터페이스

**추상 메서드가 하나**인 인터페이스

- `@FunctionalInterface`  : 함수형 인터페이스의 조건이 지켜지는지 체크하는 어노테이션
    
    ```java
    /* 함수형 인터페이스 : 추상 메소드가 하나인 인터페이스 */
    @FunctionalInterface
    public interface Calculator {
        int operate(int a, int b);
    }
    ```
    

## 익명 클래스

재사용 필요성이 없다면 **인터페이스나 추상 클래스**를 익명 클래스로 바로 구현 가능

```java
public static void main(String[] args) {
    /* 익명 클래스 덧셈 */
    // 원래 인터페이스는 new 로 생성할 수 없지만 재사용 필요성이 없다면 익명 클래스로 바로 구현할 수 있다.
    Calculator anonymousAdd = new Calculator() {
        @Override
        public int operate(int a, int b) {
            return a + b;
        }
    };

    System.out.println("익명 클래스 덧셈 : " + calculate(10, 19, anonymousAdd));
}
```

## Java 가 제공하는 표준 함수형 인터페이스

java.util.function 패키지 안에 존재

- Consumer<T> : 값을 받아서 소비하고 반환 값은 없음 `accept(인자)`
- Supplier<T> : 값은 받지 않고 결과를 반환 `get()`
- Function<T, R> : 값을 받아 다른 결과로 변환 `apply(인자)`
- Predicate<T> : 값을 받아 boolean 결과를 반환 `test(인자)`
- UnaryOperator<T> : 입력과 반환 타입이 같은 변환 `apply(인자)`

```java
import java.util.function.*;

public class Application {
    public static void main(String[] args) {
        String input = "   java    ";

        **Predicate<String> hasText = value -> value != null && !value.trim().isEmpty();**
        System.out.println("텍스트가 존재하는가? " + hasText.test(input)); // true
        
        **UnaryOperator<String> normalize = value -> value.trim().toUpperCase();**
        String normalizedText = normalize.apply(input);
        System.out.println("변환된 텍스트 : " + normalizedText); // JAVA
        
        **Consumer<String> printer = value -> System.out.println("처리 결과 : " + value);**
        printer.accept(input); // '   java    '
        
        // Function<String, Integer> length = String::length; // 메서드 참조
        Function<String, Integer> length = value -> value.length();
        System.out.println(length.apply(normalized)); // 4
        
        **Supplier<String> defaultValue = () -> "GUEST";**
        String blankInput = "     ";
        String result = hasText.test(blankInput) ? normalize.apply(blankInput) : defaultValue.get();
        printer.accept(result);
    }
}
```

### 매개변수 2개 사용

위의 표준 함수형 인터페이스 앞에 Bi 를 붙이면 두 개의 입력을 받는 인터페이스 이름으로 정의되어 있다.

- BiPredicate<T, U> : 값 두 개를 받아 boolean 으로 반환

```java
BiPredicate<String, String> sameText = (left, right) -> left.equals(right);
sameText.test("hello", "hello"); // true
```

## 메소드 참조

람다가 새로운 동작을 만들지 않고 이미 존재하는 메소드 하나를 그대로 호출할 때, 그 호출을 `클래스이름::정적메소드이름` 형태로 간결하게 표현하는 문법

```java
import java.util.function.*;

public class Application {
    public static void main(String[] args) {
        // 기존에 존재하는 Integer.parseInt(value) 를 그대로 사용
        Function<String, Integer> parseInt = value -> Integer.parseInt(value);
        System.out.println(parseInt.apply("123"));

        Function<String, Integer> parseIntReference = Integer::parseInt;
        System.out.println(parseIntReference.apply("1234"));

				// message -> System.out.println(message);
        Consumer<String> printer = System.out::println;
        printer.accept("hello method reference!");
        
        // (left, right) -> left.equals(right);
        BiPredicate<String, String> sameTextReference = String::equals;
        System.out.println(sameTextReference.test("hello", "world")); // false
    }
}
```

## 생성자 참조

`클래스이름::new`

```java
import java.util.function.*;

public class Application {
    public static void main(String[] args) {
		    // 매개변수 없는 기본 생성자
        Supplier<Account> newAccount = Account::new;
        
        // 매겨변수가 String 하나뿐인 생성자
        Function<String, Account> newAccountWithName = Account::new;
        
        // 매개변수가 String, int 인 생성자
        BiFunction<String, Integer, Account> newAccountFull = Account::new;
        
        System.out.println(newAccount.get());
        System.out.println(newAccountWithName.apply("홍길동"));
        System.out.println(newAccountFull.apply("판다", 10000));
    }
}
```

```java
public class Account {
    private String ownerName;
    private int balance;

    public Account() {
        System.out.println("기본 생성자 호출됨");
    }

    public Account(String ownerName) {
        this.ownerName = ownerName;
        System.out.println("String 생성자 호출됨");
    }

    public Account(String ownerName, int balance) {
        this.ownerName = ownerName;
        this.balance = balance;
        System.out.println("String, int 생성자 호출됨");
    }

    @Override
    public String toString() {
        return "Account{" +
                "ownerName='" + ownerName + '\'' +
                ", balance=" + balance +
                '}';
    }
}
```

# StreamAPI

I/O 의 Stream 과 다르다. 자바 8부터 추가된 기능으로, **Collection 을 처리하는 도구**.

최종 연산 필수

- 리스트.stream() : 요소를 차례로 처리할 `Steam<T>` 을 만든다.
    - **forEach(Consumer<? super T>) : 값 사용 (최종연산)**
    - filter(Predicate<? super T>) : 조건이 true 인 값들 반환
    - map(Function<? super T, ? extends R>) : 값을 다른 형식으로 반환
    - sorted(Comparator<? super T>) : 정렬
    - **collect(Collector<? super T, A, R>) : 결과물들을 새로운 List 로 수집 (최종 연산)**
    - **count() : 스트림에 남아있는 요소의 개수 세기 (최종 연산)**
    - **anyMatch(Predicate<? super T>) : 하나라도 조건을 만족하면 true (최종연산)**
    - **allMatch(Predicate<? super T>) : 모든 요소가 조건을 만족하면 true (최종연산)**

```java
List<String> names = Arrays.asList(
        "홍길동", "유관순", "이순신", "심사임당", "장보고", "강감찬", "이성계", "허균"
);

// 내용물은 람다식으로 작성
// filter, map, forEach 를 사용하여 파이프라인 만들기
// 이름이 세글자인 사람만 골라 대괄호를 붙인 뒤 출력
names.stream().filter(name -> name.length() == 3)
        .map(name -> "[ " + name + " ]")
			  .forEach(System.out::println);

List<String> leeFamily = names.stream().filter(name -> name.startsWith("이"))
	.sorted(Comparator.reverseOrder())
	.collect(Collectors.toList());
	
System.out.println(names); // [홍길동, 유관순, 이순신, 심사임당, 장보고, 강감찬, 이성계, 허균] -> 원본은 유지

long leeCount = names.stream().filter(name -> name.startsWith("이")).count(); // 중간 연산을 통과한 요소 수를 long 으로 반환하는 최종 연산

boolean hasJang = names.stream()
        .anyMatch(name -> name.equals("장보고"));

boolean isAllThreeLetters = names.stream()
        .allMatch(name -> name.length() == 3);
```

스트림은 최종 연산이 한 번 실행되면 소비되어 같은 스트림으로 다시 사용 불가하다.

```java
Stream<String> stream = names.stream();
System.out.println("첫 번째 최종 연산 결과 : " + stream.count()); // 8
// 컴파일러 경고 : 스트림이 이미 연결되거나 사용되었습니다
//        stream.forEach(System.out::println); // IllegalStateException (이미 동작했거나 닫혔음)
```

## Stream API 활용

- **findFirst() : 첫번째 요소를 `Optional` 로 반환하는 최종 연산**
    - Optional<T> : 객체가 있을 수도 있고 없을 수도 있음
        - ifPresent() : Optional 안에 값이 있을 때만 전달한 Consumer 를 실행
        - orElse() : Optional 안에 값이 없을 경우 디폴트 값
- distinct() : 중복 제거
- **sum() : 남은 요소 합계 (최종 연산)**
- **average() : 남은 요소 평균 (최종 연산)**
- mapToInt() : 최종적으로 반환하는 값이 IntStream → IntStream 이어야 sum() 같은 연산이 가능하다.

- Collectors.groupingBy(맵의 키 값이 될 값) : 값이 같은걸 기준으로 묶어서 Map<키, List<객체>> 형태로 반환. 맵의 key 순서는 보장하지 않는다.

```java
import java.util.*;
import java.util.stream.Collectors;

public class Application {
    public static void main(String[] args) {

        List<OnlineCourse> courses = Arrays.asList(
                new OnlineCourse("java의 정석", "Programming", 120, false),
                new OnlineCourse("Spring 완전 정복", "Programming", 300, false),
                new OnlineCourse("JPA 끝장내기", "Programming", 240, false),
                new OnlineCourse("포토샵 마스터", "Design", 90, true),
                new OnlineCourse("일러스트레이터 시작하기", "Design", 110, true),
                new OnlineCourse("SQL 첫걸음", "Database", 80, true),
                new OnlineCourse("데이터베이스 모델링", "Database", 150, false)
        );

        // Programming 강의 중 200분을 초과하는 강의 찾아 타이틀 출력
        courses.stream()
                .filter(c -> "Programming".equals(c.getCategory()))
                .filter(c -> c.getDuration() > 200)
                .map(OnlineCourse::getTitle)
                .forEach(System.out::println);

        // 조건에 맞는 첫번째 추천 강의 찾기
        // Optional<T> : 객체가 있을 수도 있고 없을 수도 있음
        Optional<OnlineCourse> firstRecommended = courses.stream()
                .filter(c -> "No-Programming".equals(c.getCategory()))
                .filter(c -> c.getDuration() > 200)
                .findFirst(); // 첫번째 요소를 반환하는 최종 연산

        // ifPresent() : Optional 안에 값이 있을 때만 전달한 Consumer 를 실행
        firstRecommended.ifPresent(c -> System.out.println("추천 강의 타이틀 : " + c.getTitle()));

        System.out.println("첫 번째 추천 강의 : " + firstRecommended.orElse(new OnlineCourse("정보 없음", "", 0, true)));

        // 무료 강의를 수강 시간 순으로 정렬하기
        courses.stream()
                .filter(OnlineCourse::isFree)
//                .sorted((c1, c2) -> Integer.compare(c1.getDuration(), c2.getDuration()))
                .sorted(Comparator.comparingInt(OnlineCourse::getDuration))
                .forEach(System.out::println);

        // 중복 없이 카테고리 목록 만들기
        courses.stream()
                .map(OnlineCourse::getCategory)
                .distinct() // 중복 제거
                .forEach(System.out::println);

        // 무료 강의의 전체 수강 시간 계산하기
        /*
        * count() : 스트림에 남아있는 요소 개수 확인
        * sum() : 남은 요소 합계
        * average() : 남은 요소 평균
        * */
        int totalDurationOfFree = courses.stream()
                .filter(OnlineCourse::isFree)
                // 일반 map 을 사용하면 Integer 객체의 스트림이 된다.
                .mapToInt(OnlineCourse::getDuration)    // mapToInt : IntStream 으로 변환 -> sum() 수행 가능
                .sum();
        System.out.println("무료 강의 총 시간 : " + totalDurationOfFree);

        // 카테고리 별로 강의 묶기
        Map<String, List<OnlineCourse>> courseByCategory = courses.stream()
                .collect(Collectors.groupingBy(OnlineCourse::getCategory));

        courseByCategory.forEach((category, courseList) -> {
            System.out.println("[" + category + "]");
            courseList.forEach(c -> System.out.println(" - " + c.getTitle()));
        });
    }
}
```

- [Lambda](#lambda)
  - [람다식](#람다식)
    - [함수형 인터페이스](#함수형-인터페이스)
  - [익명 클래스](#익명-클래스)
  - [Java 가 제공하는 표준 함수형 인터페이스](#java-가-제공하는-표준-함수형-인터페이스)
    - [매개변수 2개 사용](#매개변수-2개-사용)
  - [메소드 참조](#메소드-참조)
  - [생성자 참조](#생성자-참조)
- [StreamAPI](#streamapi)
  - [Stream API 활용](#stream-api-활용)
