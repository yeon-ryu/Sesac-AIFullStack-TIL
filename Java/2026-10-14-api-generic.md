# JAVA
## 자주 사용하는 API (라이브러리)

Application Programming Interface : 서로 다른 소프트웨어나 프로그램이 데이터를 주고받고 기능을 서로 공유할 수 있도록 만들어진 연결 규약(인터페이스)

자주 사용되는 기능을 미리 만들어 제공하는 메소드 및 클래스의 사용 규칙

### Object

- 동일성 비교(==) : 주소값 비교
- equals() : **기본적으로 Object.equals() 는 동일성 비교**이기에 내용(값)이 같은지 비교하기 위해 오버라이딩 해서 동등성 비교로 바꾸어 사용
    - equals() 가 true 를 반환하면 hashCode() 도 일치해야한다는 규칙이 있기에 equals 를 오버라이딩할 때는 hashCode 도 오버라이딩 해준다.
- toString() : Object.toString()의 기본 형식은 '클래스명@16진수 형태의 해시값'이다. 사람이 보기에 의미없는 정보이기 때문에 객체의 필드값을 보여주도록 오버라이딩 하여 사용

```java
public class Application {
    public static void main(String[] args) {
        /* [API]
        * 프로그램을 만들 때 사용할 수 있도록 미리 제공되는 클래스와 메소드 사용 규칙
        *
        * [Object]
        * 자바의 최상위 객체로서, 모든 클래스는 자동으로 Object 클래스를 상속한다.
        * 따라서 Object 에 있는 메소드를 오버라이딩하여 사용할 수 있다.
        *  */

        Book book1 = new Book(1, "홍길동전", "허균", 50000);

        // 자바가 book1 을 문자열로 변경하기 위해 book1.toString()을 호출한다.
        /* Object.toString() 의 기본 형식은 '클래스명@16진수 형태의 해시값' 이다.
        * 사람이 보기에 의미없는 정보이기 때문에 객체의 필드값을 보여주도록 오버라이딩 하여 사용 */
        System.out.println("book1 = " + book1);

        Book book2 = new Book(2, "목민심서", "정약용", 30000);
        Book book3 = new Book(2, "목민심서", "정약용", 30000);

        System.out.println("동일성 비교(==) : " + (book2 == book3)); // 주소값 비교 : false
        // equals : 내용이 같은지 비교. 오버라이딩 해서 커스텀 객체 동등성 비교로 변경 (Object 는 기본적으로 동일성 비교로 해놓음)
        /* Object.equals() 의 기본 기능은 동일성 비교이다.
        * 하지만 서로 다른 객체라도 필드의 내용이 같으면 '같다(동등하다)' 고 판단해야 할 때가 있다.
        * 이런 '동등성 비교'를 위해 equals() 를 오버라이딩 한다. */
        System.out.println("동등성 비교(equals) : " + (book2.equals(book3))); // false // 오버라이딩 후 true
    }
}
```

```java
import java.util.Objects;

public class Book {
    private int number;
    private String title;
    private String author;
    private int price;

    @Override
    public String toString() {
        return "Book{" +
                "number=" + number +
                ", title='" + title + '\'' +
                ", author='" + author + '\'' +
                ", price=" + price +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (o == null || getClass() != o.getClass()) return false;
        Book book = (Book) o;
        return number == book.number && price == book.price && Objects.equals(title, book.title) && Objects.equals(author, book.author);
    }

    @Override
    public int hashCode() {
        return Objects.hash(number, title, author, price);
    }
}
```

### String

- trim() : 문자열의 앞뒤 공백 제거
- toUpperCase() / toLowerCase() : 대/소문자 변환
- indexOf(찾는 문자) : 특정 문자/문자열의 처음 위치 반환 (찾지 못하면 -1 반환)
- substring(시작 인덱스, 끝 인덱스) : 문자열의 특정 구간을 잘라내어 새로운 문자열로 반환 (시작 인덱스 포함, 끝 인덱스 포함 안 함)
- replace(이전 문자/열, 새로운 문자/열) : 문자열 내의 특정 문자/문자열을 다른 문자/문자열로 치환한 새로운 문자열 반환
- split(구분자) : 특정 구분자를 기준으로 문자열을 잘라 ‘배열’로 반환
    - 구분자를 정규식으로 해석한다.
    - 이스케이프 문자 : 문자열 내에서 줄바꿈, 탭, 따옴표 등 특별한 기능을 하는 문자. 역슬래시(\)로 시작한다. `\n (줄바꿈), \t (탭), \" (쌍따옴표), \' (홑따옴표), \\ (역슬래시)`
- isEmpty() : 문자열의 길이가 0 이면 true, 아니면 false 반환

```java
public class Application {
    public static void main(String[] args) {
        /*
        * [String
        * 문자열 처리
        * */

        String rawData = "     [RPOD-001] Apple iPhone 16 Pro | a17 bionic | 2500000    ";
        System.out.println("rawData = " + rawData);

        // trim() : 문자열의 앞/뒤 공백 제거
        String trimmedData = rawData.trim();

        // toUpperCase() / toLowerCase() : 대/소문자 변환
        String upperData = trimmedData.toUpperCase();

        /* indexOf() : 특정 문자/문자열의 처음 위치(index) 반환 (찾지 못하면 -1 반환) */
        int codeOpen = upperData.indexOf('[');
        int codeEnd = upperData.indexOf(']');

        /* substring(시작 인덱스, 끝 인덱스) : 문자열의 특정 구간을 잘라내어 새로운 문자열로 반환 (시작 인덱스 포함, 끝 인덱스 포함 안 함) */
        String productCode = upperData.substring(codeOpen + 1, codeEnd);

        /* replace() : 문자열 내의 특정 문자/문자열을 다른 문자/문자열로 치환한 새로운 문자열 반환 */
        String replaceName = upperData.replace("APPLE IPHONE 16 PRO", "GALAXY S26");

		    /* split() : 특정 구분자를 기준으로 문자열을 잘라 ‘배열’로 반환 */
        String emp = "100/홍길동//서울/교육부";
        String[] empArr = emp.split("/");

        /* .isEmpty() : 문자열의 길이가 0 이면 true, 아니면 false 반환 */
        for(String info : empArr) {
            if(info.isEmpty()) {
                System.out.print("정보없음 ");
            } else {
                System.out.print(info + " ");
            }
        }

        /* 자바의 split() 은 구분자를 정규식으로 해석한다.
        *
        * [이스케이프 문자]
        * 문자열 내에서 줄바꿈, 탭, 따옴표 등 특별한 기능을 하는 문자. 역슬래시(\)로 시작한다.
        * \n (줄바꿈), \t (탭), \" (쌍따옴표), \' (홑따옴표), \\ (역슬래시)
        * */
        String fileName = "report.final.pdf";
        // 그냥 "." 으로 넘으면 제대로 동작하지 않는다.
        System.out.println("fileName Array = " + Arrays.toString(fileName.split("\\.")));
    }
}
```

#### String 의 특징

- 리터럴 방식으로 문자열 생성 : String Constant Pool 에 실제값을 보관 (같은 문자열을 재사용)
- 생성자 방식으로 문자열 생성 : 새로운 String 객체를 생성

```java
// 리터럴 방식 : String Constant Pool 에 실제값을 보관
String str1 = "java";
String str2 = "java";
// 생성자 방식
String str3 = new String("java");
String str4 = new String("java");

System.out.println(str1 == str2);   // true
System.out.println(str3 == str4);   // false

System.out.println(str1.equals(str4));  // true
System.out.println(str3.equals(str4));  // true
```

**String 이라는 클래스에서 이미 equals() 가 동등성 비교(값 비교)를 하도록 오버라이딩 해놓았기에** equals() 를 사용하면 동등성 비교가 행해진다.

- 문자열은 불변(immutable) 객체이다 : 원본을 변경하지 않고 새로운 문자열을 만들어서 해당 문자열을 가리킨다.
    - 문자열을 너무 자주 바꾸면 성능에 영향을 끼친다. (메모리와 시간 둘 다)
    - 수정이나 삭제가 많으면 StringBuilder 를 사용하자.

### StringBuilder

가변 객체이기에 문자열을 많이 수정해야 할 경우 사용한다. 새로운 문자열을 추가할 때 새 객체를 만들지 않고 StringBuilder 가 관리하는 공간에 추가해준다.

`StringBuilder sb = new StringBuilder();`

`sb.append("추가");`

`String result = sb.toString();` 을 사용하여 일반 문자열로 반환 가능

```java
public class Application {
    public static void main(String[] args) {
        long startTime = System.nanoTime(); // 현재 시간을 나노 형태로 반환

        /* [String]
        * - 불변 객체
        * - 연결 결과로 새로운 문자열이 만들어짐
        * - 반복적인 문자열 연결에는 비효율적일 수 있음
        *
        * [StringBuilder]
        * - 가변 객체
        * - append() 로 내부 내용을 누적
        * - 반복적인 문자열 조립에 적합
        * */
        String str = "";
        for(int i = 0; i < 10000; i++) {
            str += i;
        }

        long endTime = System.nanoTime();
        System.out.println("문자열 길이 : " + str.length() + ", 작업 시간 : " + (endTime - startTime)); // 문자열 길이 : 38890, 작업 시간 : 61621900

        startTime = System.nanoTime();

        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < 10000; i++) {
            sb.append(i);
        }

        endTime = System.nanoTime();
        System.out.println("문자열 길이 : " + sb.length() + ", 작업 시간 : " + (endTime - startTime)); // 문자열 길이 : 38890, 작업 시간 : 1439400
    }
}
```

- delete(start, end) : 시작 인덱스부터 종료 인덱스 전까지의 문자열 제거
- insert(index, value) : 지정한 인덱스에 값 삽입
- reverse() ; 문자열의 순서를 뒤집는다.

```java
StringBuilder sb2 = new StringBuilder("javamysql");

/* delete(start, end) : 시작 인덱스부터 종료 인덱스 전까지의 문자열 제거 */
System.out.println(sb2.delete(4, 9));   // java

/* insert(index, value) : 지정한 인덱스에 값 삽입 */
System.out.println(sb2.insert(0, "my "));   // my java

/* reverse() : 문자열의 순서를 뒤집는다 */
System.out.println(sb2.reverse());  // avaj ym

/* append() 는 자기 자신을 반환하므로 메서드 체이닝이 가능하다 (불변 객체인 String 과 달리) */
StringBuilder query = new StringBuilder();
query.append("SELECT ").append("* ").append("FROM product ")
        .append("WHERE price > ").append(10000).append(";");
String result2 = query.toString();
System.out.println(result2); // SELECT * FROM product WHERE price > 10000;
```

### Wrapper 클래스

기본 자료형의 값을 객체 형태로 감싸는 클래스

- Boxing : 기본형 → Wrapper
    - Auto-Boxing : `Integer age = 20;`
- Unboxing : Wrapper → 기본형
    - Auto-Unboxing : `int unboxedAge = age;`

age.intValue(); 를 호출해서 자동 언박싱

| 기본 타입 | 레퍼 클래스 |
| --- | --- |
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
| float | Float |
| double | Double |
| char | Character |
| boolean | Boolean |

**Wrapper 는 객체이므로 값 비교에 equals() 를 사용한다.**

```java
Integer num1 = 128;
Integer num2 = 128;

System.out.println(num1.equals(num2)); // true
System.out.println(num1 == num2); // false
```

**Parsing : 문자열 데이터를 원하는 기본 자료형으로 변환하는 과정**

문자열 → 숫자

- Wapper클래스.parse자료형(문자열)
    - Integer.parseInt(문자열)

숫자 → 문자열

- String.valueOf(값)
- 값 + “”

### Time

java.time.* 패키지

- LocalDate : 날짜
- LocalTime : 시간
- LocalDateTime ; 날짜 + 시간
- ZonedDateTime, ZoneId : 국가별 시간대가 필요할 경우

java.time 패키지 안에 있는 객체는 불변 객체이다.

- 시간 계산 : `plus...(), minus...()`
- 시간 비교 : `isBefore(), isAfter(), isEqual()`
- 포매팅
    - Format : LocalDateTime → String `DateTimeFormatter.ofPattern()` 과 `시간객체.format(패턴)` 사용
    - Parse : String → LocalDate `DateTimeFormatter.ofPattern()` 과 `LocalDate.parse(날짜 문자열, 패턴)` 사용

```java
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;

public class Application {
    public static void main(String[] args) {
        /*
        * java.time 주요 클래스
        *
        * LocalDate : 날짜
        * LocalTime : 시간
        * LocalDateTime : 날짜 +
        *
        * 국가별 시간대가 필요한 서비스는 ZonedDateTime 과 ZoneId 를 사용한다.
        * */

        LocalDate departureDate = LocalDate.of(2026, 12, 25);
        LocalTime departureTime = LocalTime.of(14, 30);

        LocalDateTime trainDeparture = LocalDateTime.of(departureDate, departureTime);
        System.out.println("기차 출발 시각 ; " + trainDeparture); // 2026-10-13T14:30

        // 현재 날짜와 시간
        LocalDateTime now = LocalDateTime.now();
        System.out.println("now = " + now);

        System.out.println(trainDeparture.getYear() + "년 " + trainDeparture.getMonth() + " "
                + trainDeparture.getMonthValue() + "월 " + trainDeparture.getDayOfWeek()); // 2026년 DECEMBER 12월 FRIDAY

        /* plus...(), minus...() 이용하여 시간 계산을 할 수 있다.
        * java.time 패키지의 객체는 불변이다. 원본 객체를 변경하지 않고 계산된 새로운 객체를 반환한다.
        * 따라서 반환값을 변수에 저장해야 계산 결과를 사용할 수 있다. */
        LocalDateTime arrivalDeadLine = trainDeparture.minusMinutes(20); // 20분 전
        System.out.println("arrivalDeadLine = " + arrivalDeadLine); // 2026-12-25T14:10
        System.out.println("trainDeparture = " + trainDeparture); // 2026-12-25T14:30

        /* 날짜 비교하기
        * isBefore(), isAfter(), isEqual() */
        boolean isPast = trainDeparture.isBefore(now); // false
        System.out.println("기차 떠났나요? " + isPast);

        /* 포매팅
        * DateTimeFormatter 를 사용하면 날짜/시간 객체를 원하는 형식의 문자열로 변환하거나,
        * 특정 형식의 문자열을 날짜/시간 객체로 변환할 수 있다.
        * */

        // Format : LocalDateTime -> String
        // 1. 원하는 출력 형식의 패턴을 만든다.
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("YYYY년 MM월 dd일 HH시 mm분 출발");
        // 2. 날짜 객체에 format() 메소드를 호출하며 패턴을 전달한다.
        String ticket = trainDeparture.format(formatter);
        System.out.println("ticket = " + ticket); // 2026년 12월 25일 14시 30분 출발

        // Parse : String -> LocalDate
        String christmas = "20261225"; // yyyyMMdd 형식의 문자열
        // 1. 분석할 문자열의 패턴을 만든다.
        DateTimeFormatter parser = DateTimeFormatter.ofPattern("yyyyMMdd");
        // 2. LocalDate.parse() 메소드를 호출하여 문자열과 패턴 전달
        LocalDate parsedDate = LocalDate.parse(christmas, parser);
        System.out.println("parsedDate = " + parsedDate);   // 2026-12-25
    }
}
```

## Generic

제네릭 : 클래스나 메소드에서 사용할 자료형을 미리 고정하지 않고 사용하는 시점에 결정하는 문법

타입별 중복코드를 줄이고 잘못된 타입의 데이터가 들어오는 것을 방지하기 위해 제네릭 타입 (T, E, K, V)을 활용한다.

```java
public class GenericBox<T> {
    /*
    * [제네릭 클래스]
    * T 는 아직 정해지지 않은 타입을 나타내는 타입 매개변수이다.
    *
    * T 는 Type 을 뜻하는 관계적인 이름이며 반드시 T만 사용해야 하는 것은 아니다.
    * 컬렉션에서는 Element 의 E, Key 의 K, Value 의 V 같은 이름도 자주 사용한다.
    * */

    private T content;

    public void setContent(T content) {
        this.content = content;
    }

    public T getContent() {
        return content;
    }
}
```

```java
public class Application {
    public static void main(String[] args) {
        /*
        * [제네릭]
        * 클래스나 메소드에서 사용할 자료형을 미리 고정하지 않고 사용하는 시점에 결정하는 문법
        *
        * 타입별 중복코드를 줄이고 잘못된 타입의 데이터가 들어오는 것을 방지한다.
        * */

        // 제네릭을 사용하지 않고 상위 타입인 Object 타입을 사용했을 경우
        NormalBox normalBox = new NormalBox();
        normalBox.setContent("안녕하세요"); // Object 타입으로 업캐스팅 되어 저장된다.
        String content = (String) normalBox.getContent();

//        normalBox.setContent(123); // Integer 타입이 Object 타입으로 업캐스팅 되어 저장된다.
        // 숫자도 저장 가능하기에 자체 컴파일 오류가 발생하지 않지만 런타임 시점에서 숫자를 String 으로 바꾸려는 순간 에러 발생
//        String worngContent = (String) normalBox.getContent(); // ClassCastException

        GenericBox<String> stringBox = new GenericBox<>();
        // String 으로 정해줬기에 String 으로 받는 자료형 고정 -> 컴파일러가 다른 자료형을 막아준다.
        stringBox.setContent("안녕");
//        stringBox.setContent(123);

        String strContent = stringBox.getContent();

        /* 제네릭 타입 인자에는 int 같은 기본 자료형이 아니라 참조 자료형을 사용한다.
        * 따라서 int 가 아닌 Integer로 지정한다. */
        GenericBox<Integer> integerBox = new GenericBox<>();
        integerBox.setContent(123); // Auto-Boxing 으로 Integer 타입으로 들어간다.
        int intContent = integerBox.getContent(); // Auto-Unboxing
        System.out.println(intContent + 1);
    }
}
```

※ 제네릭은 타입 위치에 기본 자료형을 사용할 수 없고 참조 자료형만 들어올 수 있다.

ArrayList 등도 이런 제네릭 타입을 사용해서 구현되어 있다.

### 타입 제한하기

```java
/*
* <T extends Rabbit> Rabbit 또는 Rabbit 의 자식 타입만 오도록 제한
* */
public class RabbitFarm<T extends Rabbit> {
    private T animal;

    public RabbitFarm() {
    }

    public RabbitFarm(T animal) {
        this.animal = animal;
    }

    public T getAnimal() {
        return animal;
    }

    public void setAnimal(T animal) {
        this.animal = animal;
    }
}
```

```java
public class Application {
    public static void main(String[] args) {

        RabbitFarm<Rabbit> farm1 = new RabbitFarm<>(new Rabbit());
        RabbitFarm<Bunny> farm2 = new RabbitFarm<>(new Bunny());
        RabbitFarm<DrunkenBunny> farm3 = new RabbitFarm<>(new DrunkenBunny());

//        RabbitFarm<Snake> farm4 = new RabbitFarm<Snake>(new Snake());

        Rabbit rabbit = farm1.getAnimal();
        Bunny bunny = farm2.getAnimal();
        DrunkenBunny drunkenBunny = farm3.getAnimal();
    }
}
```

### 제네릭 와일드카드

호출할 때 들어오는 정확한 요소 타입은 달라도 메소드가 허용할 타입의 범위를 정하는 문법

와일드카드 : 메소드의 매개변수에 제네릭을 사용 가능(기본적으로 Object 로 받음) : `?`

- **extends** : Number 이거나 Number 의 자식 타입 : 읽는 것에 유리
    - `private static double sum(List<? extends Number> list) { }`
- **super** : Integer 이거나 Integer 의 부모 타입 : 값을 받아서 저장하는데 유리
    - `private static void addInteger(List<? super Integer> list)`

```java
import java.util.ArrayList;
import java.util.List;

public class Application6 {
    public static void main(String[] args) {
        /*
        * [제네릭 와일드카드]
        * 호출할 때 들어오는 정확한 요소 타입은 달라도 메소드가 허용할 타입의 범위를 정하는 문법
        * */

        List<Integer> integerList = new ArrayList<>();
        integerList.add(10);
        integerList.add(20);

        List<Double> doubleList = new ArrayList<>();
        doubleList.add(1.5);
        doubleList.add(2.5);

//        List<Number> numberList = integerList;

        printAnyList(integerList);
        printAnyList(doubleList);

        System.out.println("정수 합계 : " + sum(integerList));
        System.out.println("실수 합계 : " + sum(doubleList));

        List<Number> numberList = new ArrayList<>();
        addInteger(numberList);
        System.out.println(numberList);
    }

    private static void printAnyList(List<?> list) {
        for(Object value : list) {
            System.out.print(value + "\t");
        }
        System.out.println();
    }

    // Number 이거나 Number 의 자식
    private static double sum(List<? extends Number> list) {
        double total = 0;

        // java.lang.Number (Integer 나 Double 등의 부모 클래스)
        for(Number number : list) {
            // Integer / Double 을 전부 기본 자료형 double 로 바꾸는 메소드
            total += number.doubleValue();
        }
        return total;
    }

    // Integer 이거나 Integer 의 부모
    private static void addInteger(List<? super Integer> list) {
        list.add(100);
    }
}
```

- [JAVA](#java)
  - [자주 사용하는 API (라이브러리)](#자주-사용하는-api-라이브러리)
    - [Object](#object)
    - [String](#string)
      - [String 의 특징](#string-의-특징)
    - [StringBuilder](#stringbuilder)
    - [Wrapper 클래스](#wrapper-클래스)
    - [Time](#time)
  - [Generic](#generic)
    - [타입 제한하기](#타입-제한하기)
    - [제네릭 와일드카드](#제네릭-와일드카드)
