# Collection

여러 개의 데이터(객체)를 효과적으로 관리하기 위해 자바에서 제공하는 클래스들의 묶음

크게 List, Set, Map 인터페이스로 분류된다.

```
Iterable(I) - Collection(I) - List(I) - - - - - - - - - - - - - - - LinkedList
									    - ArrayList                     |
									    - Vector        - Stack         |
                                                                        |
							- Queue(I)  - Deque(I) - - - - - - - - - - -
												        - ArrayDeque

							- Set(I)    - SortedSet(I)  - TreeSet
										- HashSet
										- LinkedHashSet

Map(I)  - StortedMap(I) - TreeMap
		- Hashtable
	 	- LikedHashMap
		- HashMap
```

(I) : 인터페이스

## List 인터페이스

순서가 있는 데이터의 집합. 같은 데이터의 중복 저장을 허용한다.

- add(값) : 마지막에 값 추가
- get(인덱스) : 해당 위치의 값 조회
- set(인덱스, 값) : 해당 위치의 값 수정
- remove(인덱스 또는 값) : 해당 위치 또는 해당 값 삭제
    - Integer 타입의 리스트는 기본적으로 인덱스를 삭제한다. 값을 삭제하고 싶다면 `list.remove(Integer.valueOf(값));` 형식으로 작성
    - 첫번째 값이 아닌 모든 값을 제거하고 싶다면 `list.removeAll(Arrays.asList(값));` 형식으로 작성
- size() : 현재 저장된 요소 개수 확인

### ArrayList

내부적으로 배열을 이용하여 연결된 공간에 저장. **단순 조회는 빠르다**.

`List<T> alist = new ArrayList<>();`

앞을 상위 타입으로 해줘야 나중에 바뀌더라도 List 로 구현한 기능을 유지할 수 있다.

### LinkedList

이중 연결 리스트 방식으로 데이터를 관리하여 앞 뒤 요소의 링크 정보만 가진다. **중간에 데이터 추가나 삭제가 많을 경우** 유리. 값을 인덱스로 찾을 경우 연결을 따라서 가기에 **조회 속도가 느리다**.

`LinkedList<T> list = new LinkedList<>();`

## Collections 클래스

컬렉션을 다루는데 유용한 static 메소드들을 제공하는 유틸리티 클래스

※ Collection 인터페이스와는 다르다.

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class Application {
    public static void main(String[] args) {
        List<String> stringList = new ArrayList<>();

        stringList.add("apple");
        stringList.add("banana");
        stringList.add("orange");
        stringList.add("mango");
        stringList.add("mango");

        // 문자열 목록 정렬하기
        /* [Collections 클래스]
        * 컬렉션을 다루는데 유용한 static 메소드들을 제공하는 유틸리티 클래스
        * */

				// 오름차순 정렬
        Collections.sort(stringList) // 문자열 오름차순
        System.out.println("정렬 후 stringList = " + stringList);

        // 내림차순 정렬
        // List 인터페이스에서 제공하는 sort 메소드에 Comparator.reverseOrder() 를 인자로 전달
        stringList.sort(Comparator.reverseOrder());
        System.out.println("내림차순 stringList = " + stringList);
    }
}
```

### 커스텀 정렬

- java.lang.Comparable 인터페이스를 구현하여 compareTo 구현

    ```java
    public class BookDTO implements Comparable<BookDTO>{
        private int number;
        private String title;
        private String author;
        private int price;

        @Override
        public int compareTo(BookDTO o) {
            /* 정렬기준 : 책 번호(number) 기준으로 오름차순으로 정하겠다.
            * - 현재 객체(this) 의 number 가 o 의 number 보다 작으면 -> 음수 반환(자리 유지)
            * - 현재 객체(this) 의 number 가 o 의 number 보다 크면 -> 양수 반환(자리 바꿈)
            * - 두 객체의 number 가 같으면 -> 0 반환(자리 유지)*/
            return Integer.compare(this.number, o.getNumber());
        }
    }
    ```

    ```java
    List<BookDTO> bookList = new ArrayList<>();

    bookList.add(new BookDTO(1, "홍길동전", "허균", 50000));
    bookList.add(new BookDTO(3, "삼국사기", "김부식", 46000));
    bookList.add(new BookDTO(2, "동의보감", "허준", 40000));

    Collections.sort(bookList);
    ```

- comparator

    ```java
    package com.ohgiraffers.section01.list.comparator;

    import com.ohgiraffers.section01.list.dto.BookDTO;

    import java.util.Comparator;

    public class AscendingPrice implements Comparator<BookDTO> {
        @Override
        public int compare(BookDTO o1, BookDTO o2) {
            return Integer.compare(o1.getPrice(), o2.getPrice());
        }
    }
    ```

    ```java
    package com.ohgiraffers.section01.list.run;

    import com.ohgiraffers.section01.list.comparator.AscendingPrice;
    import com.ohgiraffers.section01.list.dto.BookDTO;

    import java.util.ArrayList;
    import java.util.List;

    public class Application2 {
        public static void main(String[] args) {
            List<BookDTO> bookList = new ArrayList<>();

            bookList.add(new BookDTO(1, "홍길동전", "허균", 50000));
            bookList.add(new BookDTO(3, "삼국사기", "김부식", 46000));
            bookList.add(new BookDTO(2, "동의보감", "허준", 40000));

            /* 기본값과 다른 기준으로 정렬
            * 별도의 Comparator를 구현한 구현체인 AscendingPrice 를 전달한다. */
            bookList.sort(new AscendingPrice());

            System.out.println("Comparator : " + bookList);

        }
    }
    ```

## Queue 인터페이스 활용

### Stack

후입선출(LIFO : Last In First Out)

Deque 를 Stack 처럼 사용하는 것이 권장된다.

- push(값) : 값을 하나 앞에서부터 큐에 넣는다.
- peek() : 가장 마지막(맨 앞)에 들어간 요소를 확인
- pop() : 가장 마지막(맨 앞)에 들어간 요소를 꺼내고 스택에서는 제거 (비어 있을 경우 NoSuchElementException 발생)

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class Application4 {
    public static void main(String[] args) {
        /*
        * Stack
        * - 후입선출(LIFO) 방식의 자료구조이다.
        *
        * - 웹 브라우저의 '뒤로 가기' 기능
        * - 프로그램의 '실행 취소' 기능
        *
        * 자바에 오래된 Stack 클래스도 있지만, 현재는 Deque 인터페이스의 구현체인 ArrayDeque 를 스택처럼 사용하는 방식이 일반적이다.
        * Deque 는 양쪽 끝에서 값을 넣고 뺄 수 있지만 여기서는 한쪽 끝만 사용한다.
        * */
        Deque<Integer> integerStack = new ArrayDeque<>();

        integerStack.push(1);
        integerStack.push(2);
        integerStack.push(3);
        integerStack.push(4);
        integerStack.push(5);

        System.out.println(integerStack); // [5, 4, 3, 2, 1]

        /*
        * peek() : 가장 꼭대기 요소를 '확인'
        * pop() : 가장 꼭대기 요소를 '꺼내고' 제거한다.
        * */
        System.out.println("peek() : " + integerStack.peek());
        System.out.println("pop() : " + integerStack.pop());
        System.out.println("integerStack : " + integerStack);

        while(!integerStack.isEmpty()) {
            System.out.println("꺼낸 값 : " + integerStack.pop());
        }

        //        System.out.println("비어있을 때 pop() : " + integerStack.pop()); // NoSuchElementException

    }
}
```

### Queue

선입선출(FIFO : First In First Out)

- offer(값) : 기존 대기열 맨 뒤에 값을 추가
- peek() : 가장 처음(맨 앞)의 요소를 확인
- poll() : 가장 처음(맨 앞)의 요소를 꺼내고 제거한다. (비어있을 경우 null 반환)

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class Application5 {

    public static void main(String[] args) {
        /*
        * [Queue]
        * - 선입선출(FIFO) 방식의 자료구조
        * - Queue는 인터페이스이므로 실제 객체는 구현 클래스인 ArrayDeque 를 사용한다.
        * */

        Queue<String> queue = new ArrayDeque<>();

        // 데이터를 넣을 때는 offer()를 이용
        // offer(값) : 기존 대기열 맨 뒤에 값을 추가
        queue.offer("first");
        queue.offer("second");
        queue.offer("third");

        System.out.println(queue); // [first, second, third]

        // peek() : 맨 앞의 요소를 확인
        System.out.println("peek() : " + queue.peek());

        // poll() : 가장 처음(맨 앞)의 요소를 꺼내고 제거한다
        System.out.println("poll() : " + queue.poll());

        queue.poll();
        queue.poll();
        System.out.println("비어있을 때 poll() : " + queue.poll()); // null
    }
}
```

### Deque 기능

| push(값) | 값을 대기열 앞에 큐에 넣는다. |  |
| --- | --- | --- |
| pop() | 맨 앞 요소를 꺼내고 제거 | 비어 있을 경우 NoSuchElementException 발생 |
| Stack 유래 |  |  |
| offer(값) | 값을 대기열 뒤에 큐에 넣는다. |  |
| poll() | 맨 앞 요소를 꺼내고 제거 | 비어있을 경우 null 반환 |
| Queue 유래 |  |  |
| peek() | 큐의 맨 앞 요소를 확인 |  |

## Set 인터페이스

중복을 허용하지 않는다. → 중복 될 경우 해당 값은 무시한다.

- HashSet : 중복 제거, 순서 보장 X
- LinkedHashSet: 중복 제거, 입력 순서 유지
- TreeSet : 중복 제거, 비교 기준으로 정렬

공통 메서드

- add(값)
- size()
- iterator() : 컬렉션을 순회할 수 있는 Iterator 객체를 반환
    - hasNext() : 다음에 꺼낼 요소가 있다면 true
    - next() : 다음 요소를 꺼냄

### HashSet

중복 비허용, 순서를 보장하지 않음.

hashCode() 와 equals() 로 동등성을 판단하여 중복을 체크한다.

```java
import java.util.*;

public class Application {
    public static void main(String[] args) {
        /*
        * [Set]
        * - 순서를 보장하지 않는다.
        * - 중복을 허용하지 않는다.
        * */

        List<String> alist = new ArrayList<>();
        alist.add("java");
        alist.add("mysql");
        alist.add("java");

        Set<String> hset = new HashSet<>();
        hset.add("java");
        hset.add("html");
        hset.add("css");
        hset.add("java");

        System.out.println("ArrayList : " + alist);
        System.out.println("HashSet : " + hset);

        System.out.println("향상된 for 문으로 순회");
        for(String value : hset) {
            System.out.println(value);
        }

        System.out.println("Iterator 로 순회");

        // iterator() 를 호출하면 컬렉션을 순회할 수 있는 Iterator 객체를 반환한다
        Iterator<String> iter = hset.iterator();
        while(iter.hasNext()) { // hasNext() : 다음에 꺼낼 요소가 있다면 true
            System.out.println(iter.next()); // next() : 다음 요소를 꺼냄
        }
    }
}
```

### LinkedHashset

중복 불가, 요소가 입력된 순서를 기억하여 순서 유지.

```java
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.Set;

public class Application2 {
    public static void main(String[] args) {
        /*
        * [LinkedHashset]
        * - '중복 불가'의 특징은 그대로 유지
        * - 요소가 입력된 순서를 기억하여, 순서를 유지하는 특징이 있다.
        * */

        Set<String> hset = new HashSet<>();
        hset.add("java");
        hset.add("html");
        hset.add("css");
        hset.add("java");

        Set<String> lhset = new LinkedHashSet<>();
        lhset.add("java");
        lhset.add("html");
        lhset.add("css");
        lhset.add("java");

        System.out.println("HashSet : " + hset); // 중복 X, 순서 유지 X
        System.out.println("LinkedHashSet : " + lhset); // 중복 X, 순서 유지 O
    }
}
```

### TreeSet

중복 불가, 요소를 저장할 때마다 비교 기준에 따라 요소를 정렬해서 보관

- 요소가 Comparable 을 구현해 기본 정렬 기준을 제공하거나, TreeSet 을 만들 때 별도의 Comparator 을 전달해야 한다.
- 중복 여부를 비교 기준 값으로 판단한다. (compareTo() 의 결과 값이 0일 경우)

```java
import com.ohgiraffers.section01.list.dto.BookDTO;

import java.util.Iterator;
import java.util.TreeSet;

public class Application3 {
    public static void main(String[] args) {
        /* TreeSet
         * - '중복 불가'의 특징과 함께, 요소를 저장할 때마다 비교 기준에 따라 요소를 정렬해서 보관
         * */

        /*
        * 요소가 Comparable 을 구현해 기본 정렬 기준을 제공하거나,
        * TreeSet 을 만들 때 별도의 Comparator 를 전달해야 한다.
        * String 클래스와 Integer 같은 클래스는 이미 Comparable 을 구현하여 자동 정렬된다.
        * */
        TreeSet<String> tset = new TreeSet<>();

        tset.add("java");
        tset.add("html");
        tset.add("mysql");
        tset.add("css");
        tset.add("java");

        System.out.println(tset);

        TreeSet<BookDTO> bookSet = new TreeSet<>();
        bookSet.add(new BookDTO(2, "목민심서", "정약용", 30000));
        bookSet.add(new BookDTO(1, "홍길동전", "허균", 50000));
        bookSet.add(new BookDTO(4, "동의보감", "허준", 40000));
        bookSet.add(new BookDTO(2, "다른 제목", "다른 저자", 10000));
        // BookDTO 의 compareTo() 는 number 만 비교한다. 그렇기에 number 가 2 로 중복 판단되어 "다른 제목" 객체는 들어가지 않는다.

        for(BookDTO book : bookSet) {
            System.out.println(book);
        }

        /* TreeSet 을 활용한 로또 번호 생성기 */
        Set<Integer> lotto = new TreeSet<>();
        while(lotto.size() < 6) {
            lotto.add((int)(Math.random() * 45) + 1);
        }
    }
}
```

## Map

**Collection 프레임워크에 속하지만 Collection 인터페이스의 자식은 아니다.**

**key 와 value 를 하나의 쌍(Entry)로 묶어서 관리**한다. `{four=java, one=java, two=html, three=mysql}`

key 는 중복 저장을 허용하지 않고, value 는 중복 저장을 허용한다. → key 가 중복될 경우 덮어씌워진다.

저장 순서를 유지하지 않는다(HashMap 의 경우)

### HashMap

- put(key, value)
- get(key) : value 를 반환한다. 해당 key 키 값을 가진 Entry 가 없을 경우 null 을 반환
- containsKey(key) : key 의 존재 여부를 boolean 값으로 반환 (value 가 null 일 경우도 있으니 존재 확인)
- remove(key) : key 값으로 요소를 삭제한다. 없는 key 를 넣어도 에러는 발생하지 않는다.

Collection 의 자식이 아니기에 직접 향상된 for 문을 사용할 수 없다.

- keySet() : 모든 키를 Set 으로 받는다.
- values() : 모든 값들만 Collection 으로 반환받는다.
    - value 는 중복이 허용되기에 Set 이 아닌 Collection 으로 반환받는다.
- entrySet() : 모든 키-값 쌍(Entry)를 Set 으로 반환한다.
    - key, value 를 전부 사용해야 할 때

```java
import java.util.*;

public class Application {
    public static void main(String[] args) {
        /*
        * [Map]
        * - key 와 value 를 하나의 쌍(Entry)로 묶어서 관리한다.
        * - key 는 중복 저장을 허용하지 않는다.
        * - value 는 중복 저장을 허용한다.
        * - 저장 순서를 유지하지 않는다. (HashMap 의 경우)
        * */

        Map<String, String> hmap = new HashMap<>();
        hmap.put("one", "java");
        hmap.put("two", "jdbc");
        hmap.put("three", "mysql");
        hmap.put("two", "html");
        hmap.put("four", "java");

        System.out.println(hmap); // {four=java, one=java, two=html, three=mysql}

        System.out.println(hmap.get("one"));
        System.out.println(hmap.get("five")); // 없는 키는 null 반환

        System.out.println(hmap.containsKey("five")); // 키의 존재 여부 확인

        hmap.remove("four");
        hmap.remove("six"); // 없는 key 를 넣어도 에러가 발생하지 않는다.
        System.out.println(hmap);

        // keySet() : 모든 키를 Set 으로 받아, 키로 값을 다시 조회한다.
        System.out.println("keySet() 이용");
        Set<String> keys = hmap.keySet();
        for(String key : keys) {
            System.out.print(key + " = " + hmap.get(key) + "\t");
        }
        System.out.println();

        // values() : 모든 값들만 Collection 으로 반환받아 순회하는 방법
        Collection<String> values = hmap.values();
        for(String value : values) {
            System.out.print(value + " ");
        }
        System.out.println();

        // entrySet() : 모든 키-값 쌍(Entry)를 Set 으로 반환한다.
        // key 와 value 모두 필요할 때는 key 로 다시 조회하지 않는 이 방식이 가장 직접적이다.
        Set<Map.Entry<String, String>> entrySet = hmap.entrySet();
        for(Map.Entry<String, String> entry : entrySet) {
            System.out.print(entry.getKey() + " = " + entry.getValue() + " / ");
        }
    }
}
```

## Collection 차이

| List | Set | Map |
| --- | --- | --- |
| 순서 O, 중복 O | 순서 X, 중복 X | key=value Entry, key : 중복 X, value : 중복 O |
| ArrayList, LinkedList | HashSet |  |
| Stack (LIFO) | LinkedHashSet : 순서 O |  |
| Queue (FIFO) | TreeSet : 비교 기준 정렬 O |  |
| Collection 인터페이스 상속 |  | Collection 인터페이스 상속 X (직접적으로 향상된 for 문 사용 불가) |

# Exception

Exception Handling : 프로그램 실행 중 발생 문제를 감지하고 안내/기록/복구 방법을 정하는 것

- Error(오류) : 시스템 상에서 프로그램에 심각한 문제가 발생하여 실행 중인 프로그램이 종료되는 것(JVM 에러, 정전, 컴퓨터 하드웨어 적인 문제 등)
- Exception(예외) : 개발자가 미리 예측하고 처리할 수 있는 미약한 오류

오류나 예외는 모두 Throwable 을 상속 받는다.

예외의 최상위 클래스는 Exception 클래스이다.

Unchecked Exception 계열은 기본적으로 이미 처리되어 있고 실행 중인 프로그램이 종료되게 작성되어 있다. (RuntimeException이 포함되어 있다.)

```
Throwable - Error
					- Exception (Unchecked Exception) - RuntimeException - NullPointerException
																															 - ...
												(Checked Exception) - IOException
																						- ...
```

- Checked Exception : 컴파일러가 처리 또는 throws 선언을 확인한다.
- UnChacked Exception : 컴파일러가 처리를 강제하지 않으며 주로 실행 중 발견된다.

## 예외 처리 방법

1. throws 로 위임 : 상위 메소드한테 처리를 위임 `public void 메소드명() throws Exception { }`
    - throw ; 현재 위치에서 예외 객체를 직접 던진다. (예외 발생)
    - throws : 이 메소드가 처리하지 않은 예외를 호출한 곳으로 위임한다. (예외 위임)
2. try-catch 로 처리 `try{ } catch(Exception e){ }`
- e.printStackTrace() : 프로그램이 종료되게 하지는 않지만 예외 발생을 추적. 실제 서비스에서는 보통 로깅 도구를 이용하여 필요한 정보를 기록한다. → 가장 마지막에 출력됨
- e.getClass().getSimpleName() : 발생한 예외 클래스 간단한 이름

```java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.NoSuchElementException;

public class Application2 {
    public static void main(String[] args) {
        ExceptionTest et = new ExceptionTest();

        try {
            et.checkEnoughMoney(10000, 50000);
            et.checkEnoughMoney(10000, 5000);
        } catch (Exception e) { // Exception 타입의 예외가 전달 되었을 때 실행
            System.out.println(e.getMessage());

            /* 예외가 발생해 전달된 호출 경로 확인
            * 실제 서비스에서는 보통 로깅 도구를 이용하여 필요한 정보를 기록한다. */
            e.printStackTrace(); // 가장 마지막에 출력됨

        }

        System.out.println("프로그램을 정상적으로 종료합니다.");

        Deque<Integer> stack = new ArrayDeque<>();

        try {
            stack.pop(); // NoSuchElementException 발생!!
        } catch (NoSuchElementException e) {
            System.out.println("빈 스택에서는 값을 꺼낼 수 없습니다.");
        }

        /*
        * - Checked Exception : 컴파일러가 처리 또는 throws 선언을 확인한다.
        * - UnChacked Exception : 컴파일러가 처리를 강제하지 않으며 주로 실행 중 발견된다.
        * */
    }
}
```

## 커스텀 Exception

```java
public class NegativeException extends Exception {
    public NegativeException(String message) {
        super(message);
    }
}
```

```java
public class PriceNegativeException extends NegativeException{

    // 부모인 NegativeException 에 기본 생성자가 존재하지 않기에 부모와 똑같은 인수 받는 생성자를 필수로 만들어야 함
    public PriceNegativeException(String message) {
        super(message);
    }
}
```

```java
package com.ohgiraffers.section02.userexception;

import com.ohgiraffers.section02.userexception.exception.MoneyNegativeException;
import com.ohgiraffers.section02.userexception.exception.NotEnoughMoneyException;
import com.ohgiraffers.section02.userexception.exception.PriceNegativeException;

public class ExceptionTest {
    // 메소드 위임
    public void checkEnoughMoney(int price, int money)
            throws PriceNegativeException, MoneyNegativeException, NotEnoughMoneyException {
        System.out.println("가지고 계신 돈은 " + money + "원 입니다.");

        if(price < 0) {
            throw new PriceNegativeException("상품 가격은 음수일 수 없습니다.");
        } else if(money < 0) {
            throw new MoneyNegativeException("소지금은 음수일 수 없습니다.");
        } else if(money < price) {
            throw new NotEnoughMoneyException("돈이 부족합니다.");
        }

        System.out.println("구입 감사합니다. 즐거운 쇼핑 하세요~");
    }
}
```

```java
package com.ohgiraffers.section02.userexception;

import com.ohgiraffers.section02.userexception.exception.MoneyNegativeException;
import com.ohgiraffers.section02.userexception.exception.NotEnoughMoneyException;
import com.ohgiraffers.section02.userexception.exception.PriceNegativeException;

public class Application {
    public static void main(String[] args) {

        ExceptionTest et = new ExceptionTest();
        try {
            et.checkEnoughMoney(30000, 5000);
        } catch (PriceNegativeException e) {
            System.out.println(e.getClass().getSimpleName() + " : " + e.getMessage());
        } catch (MoneyNegativeException e) {
            System.out.println(e.getClass().getSimpleName() + " : " + e.getMessage());
        } catch (NotEnoughMoneyException e) {
            System.out.println(e.getClass().getSimpleName() + " : " + e.getMessage());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- 멀티 캐치 : 같은 내용물일 경우 `catch (PriceNegativeException | MoneyNegativeException | NotEnoughMoneyException e)` 이렇게 묶을 수 있다.

## 상속에서 Exception

- 자식은 부모가 열어둔 Exception 가능성을 선언하지 않아도 된다.
- 자식은 부모가 열어둔 Exception 와 같거나 보다 더 구체적인 범위는 허용한다.
- 자식은 부모가 열어둔 Exception 보다 더 넓은 범위는 불가능하다.

```java
public class SuperClass {
    public void method() throws IOException { }
}
```

```java
public class SubClass extends SuperClass{

    /*
    // 부모가 Exception 가능성을 열어뒀더라도 자식에서는 선언 안할 수 있다.
    @Override
    public void method() { }

    // 자식은 부모가 열어둔 Exception 와 같거나 보다 더 구체적인 범위는 허용한다.
    @Override
    public void method() throws FileNotFoundException { }

    // 오류 발생 : 부모 자리에 대체될 수 있어야하기에 부모보다 더 넓은 범위는 허용하지 않는다.
    @Override
    public void method() throws Exception { }
    */
}
```

- [Collection](#collection)
  - [List 인터페이스](#list-인터페이스)
    - [ArrayList](#arraylist)
    - [LinkedList](#linkedlist)
  - [Collections 클래스](#collections-클래스)
    - [커스텀 정렬](#커스텀-정렬)
  - [Queue 인터페이스 활용](#queue-인터페이스-활용)
    - [Stack](#stack)
    - [Queue](#queue)
    - [Deque 기능](#deque-기능)
  - [Set 인터페이스](#set-인터페이스)
    - [HashSet](#hashset)
    - [LinkedHashset](#linkedhashset)
    - [TreeSet](#treeset)
  - [Map](#map)
    - [HashMap](#hashmap)
  - [Collection 차이](#collection-차이)
- [Exception](#exception)
  - [예외 처리 방법](#예외-처리-방법)
  - [커스텀 Exception](#커스텀-exception)
  - [상속에서 Exception](#상속에서-exception)
