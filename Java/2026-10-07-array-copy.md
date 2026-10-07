# Java 기초
## API
자바는 필요한 기능을 클래스와 메소드로 미리 제공하는데, 이러한 기능을 사용하기 위한 규칙과 도구의 집합을 API 라고 한다. 

### java.lang
자주 쓰여서 import 구문을 쓰지 않아도 사용되도록 구현되어 있다.
- java.lang.Math : 수학에서 자주 사용하는 값과 메소드를 구현해 놓은 클래스. 모든 메소드는 static 메소드이디ㅏ.
    ```java
    System.out.println("원주율 : " + Math.PI);

    // Math.random() : 0.0 이상 1.0 미만의 실수 반환
    /* (공식) : (int) (Math.random() * (구하려는 난수의 갯수)) + (구하려는 난수의 최소값) */
    System.out.println("랜덤값 : " + ( (int)(Math.random() * 100) + 1 )); // 1~100 까지 난수 발생
    ```

### java.util
- java.util.Random
    ```java
    // 1. Random 객체 생성
    Random random = new Random();

    // 0 ~ 9 난수 발생
    // nextInt(bound) : 0부터 bound-1 까지의 정수 난수를 반환
    int randomVal = random.nextInt(10);
    ```

## Scanner
```java
import java.util.Scanner;

public class Application1 {
    public static void main(String[] args) {
        // Scanner 객체 생성
        Scanner sc = new Scanner(System.in);

        // nextLine() : 엔터 키 이전까지 한 줄 전체를 문자열로 읽음
        System.out.print("이름을 입력하세요 : ");
        String name = sc.nextLine();

        // next() : 공백문자나 개행문자 전까지를 문자열로 읽음
        System.out.print("인사말을 입력하세요 : ");
        String greeting = sc.next();

        // nextInt() : 공백 이전까지의 정수 값을 읽음
        System.out.print("나이를 입력하세요 : ");
        int age = sc.nextInt();
        // nextDouble() : 공백 이전까지의 실수 값을 읽음

        /*
        * 숫자 토큰만 읽고 엔터를 쳤을 경우 개행문자를 string buffer 에 남겨둔다.
        * 그렇기에 다음에 nextLint() 등으로 읽어올 경우 남겨진 개행문자로 입력을 안했는데 빈 문자열을 읽는다.
        * 그렇기에 숫자 다음 입력을 받을 때는 버퍼에 남이있던 개행 문자 처리가 필요하다.
        * */
        sc.nextLine();

        // 문자를 직접 입력 받는 기능은 제공하지 않는다
        // 문자열로 입력받고, 원하는 문자를 분리해서 사용해야 한다
        // java.lang.String 의 charAt(index)를 사용한다.
        System.out.println("인삿말의 첫 글자 : " + greeting.charAt(0));

        System.out.println("\n이름 : " + name + ", 나이 : " + age + "\n" + greeting);
        
        // 스캐너를 닫는다 (자원 정리)
        sc.close();
    }
}
```
※ nextInt() 나 nextDouble() 로 숫자 토큰을 읽을 경우 엔터친 후 개행 문자가 버퍼에 남아있을 것이기에 이후 문자열을 읽으려면 nextLine() 같은 것으로 처리가 필요하다. > 간단하게 처리하려면 nextLine() 으로 전부 입력 받고 Integer.parseInt(string) 등으로 형변환 처리

## 조건문
```java
public void testIfElseIf() {
    Scanner sc = new Scanner(System.in);
    System.out.print("학생의 점수를 입력 : ");
    int point = sc.nextInt();
    String grade = "-";

    if(point > 100 || point < 0) {
        System.out.println("입력받은 점수가 범위를 넘어섰습니다.");
    } else if(point >= 90) {
        grade = "A";
        if(point >= 95) grade += "+";
    } else if(point >= 70) {
        grade = "B";
        if(point >= 80) grade += "+";
    } else {
        grade = "F";
    }

    System.out.println("점수 : " + point + ", 등급 : " + grade);
    sc.close();
}

public void testSwitch() {
    Scanner sc = new Scanner(System.in);

    System.out.print("첫번째 정수 입력 : ");
    int first = sc.nextInt();

    System.out.print("두번째 정수 입력 : ");
    int second = sc.nextInt();

    sc.nextLine();

    System.out.print("연산 기호 입력 (+, -) : ");
    char op = sc.next().charAt(0);

    int result = 0;

    switch (op) {
        case '+' :
            result = first + second;
            break;
        case '-' :
            result = first - second;
            break;
        default:
            System.out.println("해당 연산자는 취급하지 않습니다.");
            return;
    }

    System.out.println(first + " " + op + " " + second + " = " + result);
    sc.close();
}
```

### switch-case 전용 화살표 함수
```java
int result = switch (op) {
    case '+' -> first + second;
    case '-' -> first - second;
    default -> {
        System.out.println("해당 연산자는 취급하지 않습니다.");
        yield -9999; // 중괄호로 작성한 case 에서 값을 반환할 때 yield 를 사용 (딱히 특별한 표시가 되진 않음)
        // return 은 메소드 전체를 종료하지만 yield 는 switch 결과값만 정한다
    }
};

System.out.println("향상된 스위치 결과 : " + first + " " + op + " " + second + " = " + result);
```

## 반복문
- for 문 : `for(int i = 0; i < 10; i++) { }`
- while 문 : 반복횟수가 불명확할 때 `while(조건식) { }`
- do-while 문 : 최소 한 번 실행해야 할 때 `do{ } while(조건식);`
- break : 가장 가까운 switch 문 또는 반복문 종료
  - 다른 반복문도 빠져나가고 싶을 경우
    - boolean flag 사용
    - label 사용 `라벨명 : for() { for() { break 라벨명; } }`
- continue : 가장 가까운 반복문의 현재 회차만 중단. 다음 조건 검사/증감 단계로 이동
- return : 반복문이 아니라 현재 메소드 전체 종료

## 배열
선언한 자료형만 저장 가능하고 한번 생성 후에 길이도 변경할 수 없다. 
`int[] scores = new int[5];`  
`int scores[] = new int[5];` (권장되지 않음)

1. 배열 선언 : int 배열 객체를 가리킬 수 있는 참조 변수만 준비 (Stack 에만 준비해놓은 상태 / Heap 에는 아직 생성되지 않았다.) `int[] iarr;`
2. 배열 할당 : Heap 영역에 int 값 다섯 개를 저장할 배열 객체를 만든다. new 가 반환한 참조값은 iarr 에 저장하므로 iarr 을 통해 배열 객체에 접근할 수 있다. `iarr = new int[5];`
    - 선언과 동시에 할당하는 경우 new 연산자 생략 가능
    `int[] iarr = new int[]{11, 22, 33, 44, 55}; ->int[] iarr = {11, 22, 33, 44, 55};`

```java
import java.util.Arrays;

public class Application2 {
    public static void main(String[] args) {
        /*
        * 1. 배열 선언
        * int 배열 객체를 가리킬 수 있는 참조 변수만 준비 (Stack 에만 준비해놓은 상태 / Heap 에는 아직 생성되지 않았다.)
        * */
        int[] iarr; // 더 권장되는 방식
        char carr[];

        /* 2. 배열 할당
        * new int[5] : Heap 영역에 int 값 다섯 개를 저장할 배열 객체를 만든다.
        * new 가 반환한 참조값은 iarr 에 저장하므로 iarr 을 통해 배열 객체에 접근할 수 있다.
        *  */
        iarr = new int[5];

        // 선언과 동시에 할당
        int[] iarr2 = new int[5]; // 다섯칸의 배열을 만들고 기본값으로 초기화
        // 선언과 동시에 할당하는 경우 new 연산자 생략 가능
        int[] iarr3 = {11, 22, 33, 44, 55}; // int[] iarr3 = new int[]{11, 22, 33, 44, 55};

        /* 값을 넣지 않으면 자료형에 맞는 기본값으로 채워짐
        * 정수는 0, 실수는 0.0, 논리형은 false, 문자형은 \u0000, 참조형은 null 이다. */

//        iarr[5] = 60; // 배열의 크기를 넘어가면 ArrayIndexOutOfBoundsException 발생

        // 문자열 배열
        String[] sarr = {"apple", "banana", "orange"};
        System.out.println(sarr); // 배열 타입과 식별정보 출력됨

        // 반복문이나 Arrays.toString() 을 사용
        for(String s : sarr) {
            System.out.println(s);
        }
        System.out.println(Arrays.toString(sarr)); // [apple, banana, orange]
    }
}
```

### Heap 과 Stack
`int[] scores = new int[5];`  
new 로 만들어진 실체는 Heap 영역에 저장됨  
stack 에 heap 의 실체 주소값을 참조하는 app 이 존재 

### 다중 배열
다중 배열의 경우 행마다 열의 개수가 달라도 된다. (안쪽 배열 가변 배열 가능)  
`int[][] iarr = new int[3][];`

```java
import java.util.Arrays;

public class Application {
    public static void main(String[] args) {
        // 2차원 배열 선언 및 할당
        int[][] iarr = new int[3][5];

        // 안쪽 배열은 가변 배열이 가능하다 (행마다 열의 개수가 달라도 된다)
        int[][] iarr2 = {{1, 2}, {3, 4, 5}, {6, 7}};

        // 중첩 반복문을 이용한 값 대입
        int value = 1;
        for(int i = 0; i < iarr.length; i++) {
            for(int j = 0; j < iarr[i].length; j++) {
                iarr[i][j] = value++;
            }
        }

        // 값 확인
        for (int[] i : iarr) {
            System.out.println(Arrays.toString(i));
        }

        // 행만 먼저 생성하는 가변 배열
        int[][] iarr3 = new int[3][];
        for(int i = 0; i < iarr3.length; i++) {
            iarr3[i] = new int[i + 2];
            System.out.println(Arrays.toString(iarr3[i]));
        }
    }
}
```
```java
Scanner sc = new Scanner(System.in);

// 학생 수 입력 받고 과목 점수 가변으로 받기
System.out.print("학생 수를 입력해주세요 : ");
int studentCnt = sc.nextInt();
int[][] testStu = new int[studentCnt][];
sc.nextLine();
for(int i = 0; i < studentCnt; i++){
    System.out.print((i + 1) + "번째 학생의 과목 점수를 입력해 주세요(띄어쓰기로 구분) : ");
    String[] readLine = sc.nextLine().split(" ");
    testStu[i] = new int[readLine.length];
    for(int j = 0; j < testStu[i].length; j++) {
        testStu[i][j] = Integer.parseInt(readLine[j]);
    }
}
for(int[] s : testStu) {
    System.out.println(Arrays.toString(s));
}

sc.close();
```

## 복사
리터럴한 값은 그냥 값 자체이기에 대입 연산자를 사용하면 값 자체가 들어간다.  
객체는 Heap 에 생성되고 Stack 에서는 참조값만 가지고 있기에 대입 연산자를 사용하면 참조값이 들어가 Heap 안의 같은 객체를 가리킨다. 

### 얕은 복사
```java
public static void main(String[] args) {
    int[] originArr = {1, 2, 3, 4, 5};
    
    // 얕은 복사 : 배열 자체가 복사 되는 것이 아니라 배열을 가리키는 참조값이 복사됨
    int[] copyArr = originArr;
    System.out.println("같은 배열인가? " + (originArr == copyArr)); // true
  
    copyArr[0] = 10;
    System.out.println(originArr[0]); // 같은 객체를 가리키고 있기에 copy 에서 바꾼게 origin 에서도 같게 보인다.
}
```
메소드에 인자로 배열을 전달하거나, 메소드가 배열을 반환할 때도 얕은 복사가 발생한다. 

### 깊은 복사
1차원 배열 값 복사
- for 문을 이용한 수동 복사
- Arrays.copyOf(원본 배열, 복사할 길이)
- System.arraycopy(원본 배열, 원본 시작 인덱스, 사본 배열, 사본 시작 인덱스, 복사할 길이)
- clone()
```java
import java.util.Arrays;

public class Application {
    public static void main(String[] args) {
        int[] originArr = {1, 2, 3, 4, 5};

        /* 깊은 복사 (1차원 기본형 배열 기준)
        * 새로운 배열을 생성하고 기존 배열의 int값 복사 (값 복사) */

        // 1. for 문을 이용한 수동 복사
        int[] copyFor = new int[originArr.length];
        for(int i = 0; i < originArr.length; i++) {
            copyFor[i] = originArr[i];
        }
        copyFor[1] = 100;

        // 2. Arrays.copyOf(원본배열, 복사할 길이)
        int[] copyOf = Arrays.copyOf(originArr, originArr.length);
        copyOf[2] = 30;

        // 3. System.arraycopy(원본, 원본 시작 위치, 사본, 사본 시작 위치, 복사할 길이)
        int[] arrayCopy = new int[originArr.length];
        System.arraycopy(originArr, 0, arrayCopy, 0, originArr.length);
        arrayCopy[3] = 40;

        // 4. clone() : 간단하지만 크기 조절 불가
        int[] copyClone = originArr.clone();
        copyClone[4] = 88;

        print("origin", originArr);
        print("copyFor", copyFor);
        print("copyOf", copyOf);
        print("arrayCopy", arrayCopy);
        print("clone", copyClone);
    }

    public static void print(String name, int[] arr) {
        System.out.println(name + " : " + Arrays.toString(arr));
    }
}
```

- [Java 기초](#java-기초)
  - [API](#api)
    - [java.lang](#javalang)
    - [java.util](#javautil)
  - [Scanner](#scanner)
  - [조건문](#조건문)
    - [switch-case 전용 화살표 함수](#switch-case-전용-화살표-함수)
  - [반복문](#반복문)
  - [배열](#배열)
    - [Heap 과 Stack](#heap-과-stack)
    - [다중 배열](#다중-배열)
  - [복사](#복사)
    - [얕은 복사](#얕은-복사)
    - [깊은 복사](#깊은-복사)
