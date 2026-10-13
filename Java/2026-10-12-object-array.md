# OOP

객체 지향 프로그래밍(Object Oriented Programing) : 객체들이 자신의 역할을 나눠 객체들의 협력으로 프로그램을 구성하는 방식

- 필드 : 객체가 가진 데이터
- 메소드 : 객체가 수행하는 동작

캡슐화

추상화

상속

**> 다형성**

## 오버로딩

이름이 같아도 매개변수의 타입이나 갯수, 순서가 다르면 여러개 만들 수 있다.

같은 의미의 동작인데 입력 값만 다를 때 사용

- 컴파일 타입 다형성 : 자바가 인자를 보고 알맞는 메세드를 '알아서' 찾아줌

### 오버로딩 규칙

메소드 시그니처 : 메소드를 식별하는 고유한 바코드 = '메서드 이름' + '매개변수 타입, 개수, 순서’

오버로딩 : 이름은 갖지만 시그니처가 다른 메서드를 여러 개 정의하는 기술

```java
public void test() {}

// 매개변수 '개수' 가 다른 경우
public void test(String str) {}

// 매개변수 '타입'이 다른 경우
public void test(int num) {}

// 매개변수 '순서'가 다른 경우
public void test(int num, String str) {}
public void test(String str, int num) {}
```

System.out.println : 온갖 종류의 매개변수로 메소드 오버로딩 적용되어 있음

성립하지 않는 경우

- 반환 타입만 다를 경우
- 접근 제한자만 다른 경우
- 매개 변수 이름만 다른 경우

```java
// 위 코드와 이어짐

/* 성립하지 않는 경우 */
// 1. 반환 타입만 다른 경우
// public int test() { return 0; }

// 2. 접근 제한자만 다른 경우
// private void test() {}

// 3. 매개변수 '이름'만 다른 경우
// public void test(int num2) {}
```

### 생성자 오버로딩

this() : 같은 클래스 내의 다른 생성자를 호출, 반드시 '첫번째 줄'에 작성해야 한다.

```java
// 2. 매개변수 있는 생성자
public User(String id, String pwd, String name) {
    this.id = id;
    this.pwd = pwd;
    this.name = name;
    System.out.println("id, pwd, name 을 초기화 하는 생성자 호출...");
}

public User(String id, String pwd, String name, Date enrollDate) {
    // this() : 같은 클래스 내의 다른 생성자를 호출, 반드시 '첫번째 줄'에 작성해야 한다.
    this(id, pwd, name);
    this.enrollDate = enrollDate;
    System.out.println("모든 필드를 초기화 하는 생성자 호출...");
}
```

## 파라미터 전달 방식

- 값 복사(Pass by Value) : 기본 자료형
- 주소 복사(Pass by Reference) : 배열, 객체(참조 자료형)
    - String : 불변성 때문에 값 복사처럼 동작한다.
    주소값이 복사되나 값을 변경할 시 불변성으로 인해 새로운 String 객체를 만들기 때문에 기존의 객체에는 영향을 주지 않는다.

```java
package com.ohgiraffers.section05.parameter;

import java.util.Arrays;

public class Application {
    public static void main(String[] args) {
        /*
        * 파라미터 전달 방식
        * 1. 값 복사(Pass by Value)
        * 2. 주소 복사(Pass by Reference)
        * */

        MethodParameter mp = new MethodParameter();

        // 값 복사 : 원본 불변
        int num = 20;
        System.out.println("호출 전 main num : " + num);
        mp.primitiveType(num);
        System.out.println("호출 후 main num : " + num);

        // 주소값 복사 : 원본이 변한다.
        int[] iarr = {1, 2, 3, 4, 5};
        System.out.println("호출 전 : " + Arrays.toString(iarr));
        mp.arrayParameter(iarr);
        System.out.println("호출 후 : " + Arrays.toString(iarr));

        // String : 불변성 때문에 값 복사처럼 동작한다.
        // 주소값이 복사되나 값을 변경할 시 불변성으로 인해 새로운 String 객체를 만들기 때문에 기존의 객체에는 영향을 주지 않는다.
        String str = "초기값";
        System.out.println("호출 전 값 : " + str);
        mp.stringType(str);
        System.out.println("호출 후 값 : " + str);

        // 가변 인자
        mp.orderPizza("홍길동");
        mp.orderPizza("판다", "불고기", "치즈", "대나무잎");
    }
}
```

```java
package com.ohgiraffers.section05.parameter;

import java.util.Arrays;

public class MethodParameter {

    // 기본 자료형 : '값' 복사 -> 원본 불변
    public void primitiveType(int num) {
        System.out.println("전달받은 num : " + num);
        num = 99;
        System.out.println("변경 후 num : " + num);
    }

    // 배열, 객체(참조 자료형) : '주소값' 복사
    public void arrayParameter(int[] arr) {
        System.out.println("전달받은 arr : " + Arrays.toString(arr));
        arr[0] = 99;
        System.out.println("변경 후 arr : " + Arrays.toString(arr));
    }

    // String : '주소값' 복사, 불변성
    public void stringType(String str) {
        str = "변경됨"; // 이 시점에서 heap 에 새로운 String 객체를 만들어 해당 주소를 가리키게 되었다.
        System.out.println("전달받은 값 변경 : " + str);
    }

    /* 가변 인자 : 전달인자의 개수가 유동적일 때 사용
    * JS 의 ...rest 와 역할이 비슷하다.
    * 전달한 값 0개 이상을 받아 메소드 내부에서 배열로 취급한다. 매개변수의 마지막에 작성한다. */
    public void orderPizza(String customerName, String... toppings) {
        System.out.println(customerName + " 고객님! 주문하신 피자 토핑 : " + Arrays.toString(toppings));
    }
}
```

가변 인자 : 전달인자의 개수가 유동적일 때 사용 `자료형... 매개변수명`
전달한 값 0 개 이상을 받아 메소드 내부에서 배열로 취급한다. 매개변수의 마지막에 작성한다.

## static

static 멤버는 특정 객체가 아니라 클래스에 속한다.
객체를 생성하지 않고 클래스 이름으로 접근할 수 있으며 모든 인스턴스(객체)가 하나를 공유한다.

- non-static 필드 : **new** 로 객체를 만들 때마다 **Heap 에 새로 생성**되며, 객체마다 독립적이다.
- static 필드 : **프로그램 시작** 시 **static 영역에 단 하나만 생성**되며, 모든 객체가 공유한다. 객체에 속하지 않았기 때문에 this 로 접근할 수 없다.

```java
package com.ohgiraffers.section06.statickeyword;

public class Application {
    public static void main(String[] args) {
        /*
         * [static]
         * static 멤버는 특정 객체가 아니라 클래스에 속한다.
         * 객체를 생성하지 않고 클래스 이름으로 접근할 수 있으며 모든 인스턴스(객체)가 하나를 공유한다.
         * */

        Student student1 = new Student();
        System.out.println("학생1 non-static : " + student1.getPersonalLockerCount());
        System.out.println("학생1 static : " + student1.getSharedLockerCount());

        student1.increasePersonalLocker();
        student1.increaseSharedLocker();

        System.out.println("학생1 non-static : " + student1.getPersonalLockerCount());
        System.out.println("학생1 static : " + student1.getSharedLockerCount());

        Student student2 = new Student();
        System.out.println("학생2 non-static : " + student2.getPersonalLockerCount());
        System.out.println("학생2 static : " + student2.getSharedLockerCount()); // student1 에서 증가한 값이 유지됨

    }
}
```

```java
package com.ohgiraffers.section06.statickeyword;

public class Student {
    // non-static 필드 : 개인 사물함
    // new 로 객체를 만들 때마다 Heap 에 새로 생성되며, 객체마다 독립적이다.
    private int personalLockerCount;

    // static 필드 : 공용 사물함
    // 프로그램 시작 시 static 영역에 단 하나만 생성되며, 모든 객체가 공유한다.
    private static int sharedLockerCount;

    public int getPersonalLockerCount() {
        return this.personalLockerCount;
    }

    public int getSharedLockerCount() {
        return Student.sharedLockerCount;
    }

    public void increasePersonalLocker() {
        this.personalLockerCount++;
    }

    public void increaseSharedLocker() {
        Student.sharedLockerCount++;
    }
}
```

## final

상수

`public static final` = 공유하고(static) + 절대 못 바꾸는(final) 값

이름은 전부 대문자, 단어는 언더스코어로 구분 - SNAKE_CASE 관례

```java
public static final double PI = 3.141592;
```

final 인스턴스 필드 : 객체 생성 시 딱 한 번 결정, 이후 변경 불가 → 생성자에서 초기화 필수

```java
public class Person {
    // final 인스턴스 필드 : 객체 생성 시 딱 한 번 결정, 이후 변경 불가 -> 생성자에서 초기화 필수

    final String ssn; // 주민등록번호
    private String name;

    public Person(String ssn, String name) {
        this.ssn = ssn;
        this.name = name;
    }

    public String getSsn() {
        return ssn;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

## 디자인 패턴

자주 등장하는 문제를 효과적으로 해결하기 위한 검증된 설계 템플릿

### 싱글톤(Singleton) 패턴

프로그램 전체에서 특정 클래스의 인스턴스(객체)가 단 하나만 존재하도록 보장한다.

- 구현 용도 : DB 연결 설정, 로그 시스템, 앱 전역 설정값 등
- 구현 방식
    1. 생성자를 private 으로 제작 → 외부에서 new 로 객체를 못 만들게 함
    2. private static 필드 → 클래스 스스로 자기 객체를 딱 하나 만들어 보관
    3. public static 메서드 → 그 하나의 객체를 외부에 건네주는 창구를 만듦 (생성자를 잠궜으니 static 이 아니면 외부에서 접근 불가)

singleton = static + private 생성자

이른 초기화 : 프로그램 시작 시에 인스턴스를 만들어 놓는다. (사용하지 않을 경우에도 만들어짐)

```java
public class EagerSingleton {
    // 이른 초기화
    // 1. 클래스가 로딩되는 즉시 인스턴스를 딱 하나 미리 만들어 보관
    private static EagerSingleton eager = new EagerSingleton();

    // 2. 외부에서 new 로 만들지 못하게 생성자를 private 으로 잠근다
    private EagerSingleton() { }

    // 3. 유일한 객체를 건네주는 통로 (외부에서 instance 를 못 만들기에 static 이어야 접근 가능)
    public static EagerSingleton getInstance() {
        return eager;
    }
}
```

게으른 초기화 : 처음 호출될 때 인스턴스를 만든다.

```java
public class LazySigleton {
    // 게으른 초기화

    // 처음에는 비워둔다 (null)
    private static LazySigleton lazy;

    private LazySigleton() {}

    public static LazySigleton getInstance() {
        // 처음 호출될 때만 만들고, 이후에는 이미 만들어진 것을 반환
        if(lazy == null) lazy = new LazySigleton();
        return lazy;
    }
}
```

Spring Boot 에서 사용하는 어노테이션 `@Service @Repository` 등을 클래스 위에 붙여서 선언하면 스프링이 알아서 싱글톤으로 관리해준다.

## 변수의 종류

클래스 영역에 작성하는 변수를 필드라 한다.

- 인스턴스 변수 (**non-static** 필드)
    - **Heap** 영역에 저장된다 (new 실행 시 생성 / stack 에서 참조하고 있지 않을 경우 GC 가 제거)
    - 같은 클래스 내 어디서든 접근 가능
    - 객체마다 따로 만들어지며 객체가 존재하는 동안 유지된다.
- 클래스 변수 (**static** 필드)
    - **Static** 영역에 저장된다 (프로그램 시작 시 생성 / 프로그램 종료 시 사라짐)
    - 프로그램 전체에서 접근 가능
    - 클래스가 초기화될 때 하나 만들어지며 같은 클래스 모든 객체가 공유한다.

메소드 안에 선언하는 변수를 지역 변수라 한다.

- 지역 변수
    - **Stack** 영역에 저장된다. (메소드 호출 시 생성 / 메소드 종료 시 사라짐)
    - 해당 지역(블럭)에서만 사용 가능
    - 매개 변수도 지역 변수의 일종이다.
    - 기본 값이 없기에 초기화 전 사용 불가

## 객체 배열

```java
public class Application {
    public static void main(String[] args) {
        // 객체 배열 선언
        Car[] carArray = new Car[3];
        System.out.println(carArray[0]); // 참조 자료형은 기본값이 null 이다.
        // carArray[0].driveMaxSpeed(); // NullPointerException : 객체가 현재 null 인 상태인데 참조 연산자를 사용했을 경우 발생

        // 자동차 객체를 찾을 수 있는 참조값이 배열에 저장되어 있다.
        carArray[0] = new Car("페라리", 300);
        carArray[1] = new Car("람보르기니", 350);
        carArray[2] = new Car("롤스로이스", 250);

        carArray[0].driveMaxSpeed();

        for(int i = 0; i < carArray.length; i++) {
            carArray[i].driveMaxSpeed();
        }
        System.out.println("============");

        // 객체 배열도 선언과 동시에 객체 배열 초기화가 가능하다
        Car[] carArray2 = {
                new Car("페라리", 300)
                , new Car("람보르기니", 350)
                , new Car("롤스로이스", 250)
        };

        for(Car car : carArray2) {
            car.driveMaxSpeed();
        }

        // 가장 빠른 자동차 찾기 : 객체 배열에 최댓값 탐색 방법 적용
        Car fastestCar = carArray[0];
        for(int i = 1; i < carArray.length; i++) {
            if(carArray[i].getMaxSpeed() > fastestCar.getMaxSpeed()) {
                fastestCar = carArray[i];
            }
        }
        System.out.println(fastestCar.getModelName() + "이 가장 빠르며, 시속은 " + fastestCar.getMaxSpeed() + "km/h 입니다.");
    }
}
```

### 계층 분리 예제

Repository : 데이터를 보관하고 저장과 조회하는 역할을 담당

```java
package com.ohgiraffers.section01.uses;

// 회원을 저장하기 위한 MemberRepository
public class MemberRepository {

    /*
    * Repository 는 데이터를 저장하고 조회하는 역할을 담당한다.
    * 이후 데이터베이스를 배우면 이 역할ㅇ르 데이터베이스 접근 코드로 확장할 수 있다.
    * */

    // final : 새로운 배열을 가리키는 것만 불가능하고 안의 값을 바꾸는 건 가능하다.
    private static final Member[] members;

    // 이미 저장된 회원 수
    private static int count;

    // static 초기화 블럭 : MemberRepository 클래스가 처음 초기화 될 때 한 번 실행된다.
    static {
        members = new Member[10]; // null 로 채워져있다.
    }

    // 매개변수로 전달 받은 Member 객체 배열을 Repository 의 members 배열에 저장
    public static boolean store(Member[] newMembers) {
        // 남은 공간보다 저장할 회원 수가 많으면 저장하지 않음
        if(count + newMembers.length > members.length) return false;

        for(int i = 0; i < newMembers.length; i++) {
            members[count++] = newMembers[i];
        }

        return true;
    }

    /* 저장된 members 배열 반환 */
    public static Member[] findAllMembers() {
        return members;
    }
}
```

Service : 프로그램의 기능을 처리

```java
package com.ohgiraffers.section01.uses;

import java.util.Scanner;

// 회원의 기능
public class MemberService {
    /*
    * Service : 회원 등록, 조회, 검색 등 프로그램의 기능을 처리한다.
    * */

    // 회원 등록
    public void signUpMembers() {
        Member[] members = {
          new Member(1, "user01", "pwd01", "판다", 3),
          new Member(2, "user02", "pwd02", "코알라", 2),
          new Member(3, "user03", "pwd03", "원숭이", 5)
        };

        if(MemberRepository.store(members)) {
            System.out.println("총 " + members.length + "명의 회원 등록 성공!!");
        } else {
            System.out.println("정원 초과로 등록 실패!");
        }
    }

    // 전체 회원 조회
    public void showAllMembers() {
        Member[] allMembers = MemberRepository.findAllMembers();

        for(Member member : allMembers) {
            if(member != null) {
                System.out.println(member.getInformation());
            }
        }
    }

    // 회원 아이디로 회원 검색
    // id 를 받아와서 사용하는게 아니라 그냥 스캐너 통째로 받아오는구나.. 근데 이게 전부 백엔드에서 동작하면 로직 모아야하니 이게 맞긴 한 것 같기도
    // id 를 매개변수로 받아서 찾아오는 것 자체는 Repository 에 만드는 것도 좋음
    public void searchMemberById(Scanner sc) {
        // 검색할 아이디 입력받기
        System.out.print("검색할 아이디를 입력하세요 : ");
        String searchId = sc.nextLine();

        // Repository 에 있는 모든 회원 정보를 가져온다.
        Member[] foundMembers = MemberRepository.findAllMembers();

        // 검색 결과를 찾았느지 여부를 확인하기 위한 변수 선언
        boolean isFound = false;

        // 반복문을 이용해 전체 회원 배열 탐색
        for(Member member : foundMembers) {
		        if(member == null) continue;
            if(searchId.equals(member.getId())) {
                System.out.println("---- 검색 결과 ----");
                System.out.println(member.getInformation());
                isFound = true;
                break;
            }
        }

        if(!isFound) System.out.println("해당 id 를 가진 회원은 존재하지 않습니다.");
    }
}
```

Application : 실행

```java
package com.ohgiraffers.section01.uses;

import java.util.Scanner;

public class Application {
    public static void main(String[] args) {
        /*
        * Application 은 사용자 입력과 메뉴 진행, Service 는 회원 관련 처리, Repository 는 회원 데이터 보관을 담당한다.
        * 이후 Spring 에서 배우는 계층 분리의 기초 형태를 미리 경험하는 예제이다.
        * */

        Scanner sc = new Scanner(System.in);

        MemberService memberService = new MemberService();

        while(true) {
            System.out.println("====회원 관리 프로그램====");
            System.out.println("1. 회원 등록 2. 회원 전체 조회 3. 회원 아이디로 검색 0. 프로그램 종료");
            System.out.print("메뉴 선택 : ");
            int no = sc.nextInt();
            sc.nextLine();

            switch(no) {
                case 1:
                    memberService.signUpMembers();
                    break;
                case 2:
                    memberService.showAllMembers();
                    break;
                case 3:
                    memberService.searchMemberById(sc);
                    break;
                case 0:
                    System.out.println("프로그램을 종료합니다.");
                    sc.close();
                    return;
                default:
                    System.out.println("잘못된 번호를 입력하셨습니다.");
            }
        }
    }
}
```

- [OOP](#oop)
  - [오버로딩](#오버로딩)
    - [오버로딩 규칙](#오버로딩-규칙)
    - [생성자 오버로딩](#생성자-오버로딩)
  - [파라미터 전달 방식](#파라미터-전달-방식)
  - [static](#static)
  - [final](#final)
  - [디자인 패턴](#디자인-패턴)
    - [싱글톤(Singleton) 패턴](#싱글톤singleton-패턴)
  - [변수의 종류](#변수의-종류)
  - [객체 배열](#객체-배열)
    - [계층 분리 예제](#계층-분리-예제)
