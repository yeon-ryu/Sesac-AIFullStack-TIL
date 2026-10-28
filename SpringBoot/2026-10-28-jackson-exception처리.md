# Spring Boot
## Request Mapping Handler
HTTP 요청 방식 + 경로 → Controller 메소드 연결

Spring 은 JSON 을 자동으로 매핑해서 객체로 변경시켜준다. (**기본 생성자 + Setter**)

- `@RequestParam` : Query String 에 있는 값 (?key=value&key2=value2)
- `@PathVariable` : 특정 자원을 가리키는 URL 경로 안의 값 (/{menuCode})
- `@RequestBody` : 요청 본문에 담긴 주요 데이터
- `@RequestHeader` : 본문 형식이나 클라이언트의 정보 같은 Header 값
- `@CookieValue` : Cookie 에 저장되어 다시 전달된 값

프로젝트 최상단 폴더/api-test.http
```java
### 1. @RequestParam - 모든 검색 조건 전달
GET http://localhost:8080/api/v1/menus?keyword=coffee&categoryCode=2&page=1

### 3. @RequestParam Map - query string 전체 받기
GET http://localhost:8080/api/v1/menus/search?name=latte&minPrice=3000&maxPrice=6000&orderableStatus=Y

### 4. @PathVariable - URL 경로 값 받기
GET http://localhost:8080/api/v1/menus/3

### 5. @RequestBody - JSON 요청 본문을 DTO로 받기
POST http://localhost:8080/api/v1/menus
Content-Type: application/json

{
  "name": "아이스 아메리카노",
  "price": 3000,
  "categoryCode": 2,
  "orderableStatus": "Y"
}

### 6. @RequestBody - 요청 본문을 문자열 그대로 받기
POST http://localhost:8080/api/v1/menus/raw
Content-Type: text/plain

plain text body sample

### 7. @RequestHeader, @CookieValue
GET http://localhost:8080/api/v1/menus/request-info
User-Agent: IntelliJ HTTP Client
Cookie: clientId=client-001
```

```java
@RestController
@RequestMapping("/api/v1/menus")
public class MenuRequestController {
    @GetMapping
    public ResponseEntity<Map<String, Object>> findMenus(
            /*
            * name 은 Query String 에서 찾을 이름이며, 생략하면 Java 매개변수명을 사용한다.
            * 기본적으로 required 는 true 이다.
            * required = false 는 누락시 null 을 전달하고, defaultValue 는 누락 시 기본 값을 전달한다.
            * */
            @RequestParam(name = "keyword", required = false) String keyword
            , @RequestParam(name = "categoryCode", defaultValue = "0") int categoryCode
            , @RequestParam(name = "page", defaultValue = "1") int page
    ) {
        // keyword 가 null일 수 있으므로 null을 허용하는 LinkedHashMap 사용
        Map<String, Object> response = new LinkedHashMap<>();

        response.put("message", "메뉴 목록 조회 요청 파라미터 확인");
        response.put("keyword", keyword);
        response.put("categoryCode", categoryCode);
        response.put("page", page);

        return ResponseEntity.ok(response);
    }

    @GetMapping("/search")
    public ResponseEntity<Map<String, Object>> searchMenus(
            @RequestParam Map<String, String> parameters
    ) {
        Map<String, Object> response = new LinkedHashMap<>();

        response.put("message", "검색 조건 전체 확인");
        response.put("parameter", parameters);

        return ResponseEntity.ok(response);
    }

    @GetMapping("/{menuCode}")
    public ResponseEntity<Map<String, Object>> findMenu(@PathVariable("menuCode") int menuCode) {
        Map<String, Object> response = new LinkedHashMap<>();

        response.put("message", menuCode + "번 메뉴 조회 요청");
        response.put("menuCode", menuCode);

        return ResponseEntity.ok(response);
    }

    /*
     * application/json 본문을 프로퍼티 이름이 같은 MenuDTO로 변환한다.
     * 빈 본문이나 잘못된 JSON을 400, 지원하지 않는 Content-type 은 415 가 발생할 수 있다.
     * */
    @PostMapping
    public ResponseEntity<Map<String, Object>> createMenu(@RequestBody MenuDTO menu) {
        Map<String, Object> response = new LinkedHashMap<>();

        response.put("message", "메뉴 등록 요청 성공");
        response.put("menu", menu);

        // 등록 요청이므로 응답 본문과 201 Created 를 반환
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    /*
    * String 타입의 @RequestBody 는 변환하지 않은 본문을 받는다.
    * @RequestHeader 는 지정한 Header 를 찾으며, 기본적으로 필수이다.
    * */
    @PostMapping("/raw")
    public ResponseEntity<Map<String, Object>> readRawBody(
            @RequestBody String body
            , @RequestHeader("content-type") String contentType
    ) {
        Map<String, Object> response = new LinkedHashMap<>();

        response.put("message", "요청 본문 문자열 확인");
        response.put("content type", contentType);
        response.put("body", body);

        return ResponseEntity.ok(response);
    }

    @GetMapping("/request-info")
    public ResponseEntity<Map<String, Object>> requestInfo(
            @RequestHeader(value="user-agent", required = false) String userAgent
            , @RequestHeader Map<String, String> headers
            , @CookieValue(value = "clientId", required = false) String clientId
    ) {
        Map<String, Object> response = new LinkedHashMap<>();

        response.put("message", "요청 헤더와 쿠키 확인");
        response.put("userAgent", userAgent);
        response.put("clientId", clientId);

        return ResponseEntity.ok(response);
    }
}
```

## Jackson

Jackson : 직렬화와 역직렬화의 변화를 담당하는 라이브러리 (spring-boot-starter-webmvc 에 포함되어 있다.) (기본적으로 자동으로 변환해준다.)

- 직렬화 : 객체의 값을 JSON 같은 데이터 형태로 바꾼다. (서버 → 클라이언트)
- 역직렬화 : JSON 을 자바로 바꾼다. (클라이언트 → 서버)
    - 기본 생성자와 Setter 필요

자동 직렬화
```java
// Jackson 한테 직렬화를 알아서 맡긴다.
@GetMapping
public ResponseEntity<List<MemberDTO>> findMembers() {
    // 클라이언트에 반환되는 값 : [{"no":1, "name":"구준표", "age":18, "enrollDate":"2026-04-21"}, {}, ...]
    return ResponseEntity.ok(memberList);
}
```

### 클라이언트와 서버 커스텀 직렬화/역직렬화

- `@JsonProperty(”변환명”)` : 직렬화와 역직렬화 모두 Java 의 변수를 JSON 의 변화명과 연결
- `@JsonFormat(pattern = "yyyy-MM-dd")` : LocalDate 를 yyyy-MM-dd 문자열로 주고받으며 직렬화할 때는 해당 형식이 되고, 역직렬화할 때는 형식이 다르면 역직렬화에 실패한다.

```java
/* Jackson 한테 직렬화를 알아서 맡긴다. List 는 JSON 배열로, 각 MemberDTO 는 배열 안의 JSON 객체로 직렬화 */
public class MemberDTO {

    /* 직렬화와 역직렬화 모두 Java 의 no 을 JSON 의 memberNo 와 연결한다. */
    @JsonProperty("memberNo")
    private int no;
    private String name;
    private int age;
    /* LocalDate 를 yyyy-MM-dd 문자열로 주고받으며 직렬화할 때는 해당 형식 문자열이 되고, 역직렬화할 때는 형식이 다르면 역직렬화에 실패한다. */
    @JsonFormat(pattern = "yyyy-MM-dd")
    private LocalDate enrollDate;
}
```

커스텀 직렬화
```java
/* MemberDTO 하나는 @JsonProperty 와 @JsonFormat 이 적용된 JSON 객체로 직렬화 된다. */
@GetMapping("/{memberNo}")
public ResponseEntity<MemberDTO> findMember(@PathVariable int memberNo) {
    MemberDTO member = memberList.stream()
            .filter(m -> m.getNo() == memberNo)
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("해당 번호의 회원이 존재하지 않습니다."));
    // @JsonProperty("memberNo") 적용 후
    // {"memberNo":1, "name":"구준표", "age":18, "enrollDate":"2026-04-21"}
    return ResponseEntity.ok(member);
}
```

커스텀 역직렬화  
`Content-Type: application/json` → jackson 이 역직렬화 → Java 객체로 사용 → 응답을 보낼 때 직렬화
```java
/* @RequestBody 는 JSON 을 MemberDTO 로 역직렬화 한 뒤 메소드에 전달한다.
* 변환할 수 없는 JSON 은 400, 지원하지 않는 Content-Type 은 415 로 처리될 수 있다. */
@PostMapping
// 직렬화
public ResponseEntity<Map<String, Object>> createMember(@RequestBody MemberDTO member) {
    /* 변환된 객체를 목록에 추가한 뒤 응답해서 다시 JSON 으로 직렬화 한다. */
    memberList.add(member);

    Map<String, Object> response = new LinkedHashMap<>();
    response.put("message", "회원 등록 요청 성공");
    response.put("member", member);

    // 역직렬화
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

### ObjectMapper

- `ObjectMapper` : 날짜 모듈 등 Spring Boot 의 공통 설정이 적용된 JSON 변환기  
JSON 문자열 자체가 필요한 상황이나, 메세지로 보내야 하는 직접 변환해야 하는 상황이 있을 경우 필요하다. (기본적으로는 Jackson 이 알아서 해준다.)
```java
@GetMapping("/object-mapper")
public ResponseEntity<Map<String, Object>> objectMapperTest() {
    Map<String, Object> response = new LinkedHashMap<>();

    // 자바 문자열로 변환 (직렬화가 이루어진다)
    String jsonString = objectMapper.writeValueAsString(memberList);

    response.put("message", "ObjectMapper 직접 변환 결과");
    response.put("jsonString", jsonString);
    // [{\"memberNo\":1,\"name\":\"구준표\",\"age\":18,\"enrollDate\":\"2026-04-21\"}, ...]

    return ResponseEntity.ok(response);
}
```

## Exception 핸들러

여러가지 예외를 공통적인 구조로 클라이언트에 반환

- `@RestControllerAdvice` : Controller 처리 중 발생한 예외를 공통으로 관찰하고, Handler 의 반환 값을 HTTP 응답 본문으로 보낼 수 있게 한다.
- `@ExceptionHandler(클래스)` : 해당 클래스의 exception 이 발생했을 시 해당 Handler 가 실행된다.

```java
package com.ohgiraffers.exceptionhandler;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.WebRequest;

/* Controller 처리 중 발생한 예외를 공통으로 관찰하고, Handler 의 반환값을 HTTP 응답 본문으로 보낼 수 있다. */
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 전체 Controller 에서 지정한 exception 발생 시 handler 사용
    /* @ExceptionHandler 는 예외 타입에 맞는 메소드를 선택하여 JSON 실패 응답으로 바꾼다. */
    @ExceptionHandler(MemberNotFoundException.class)
    public ResponseEntity<ErrorResponse> handlerMemberNotFound(
            MemberNotFoundException exception, WebRequest request
    ) {
        /* 조회 대상이 없을 때는 서버 오류(500)이 아니라 리소스 없음인 404로 응답 */
        ErrorResponse response = new ErrorResponse(
                HttpStatus.NOT_FOUND.value()
                , HttpStatus.NOT_FOUND.getReasonPhrase()    // 이유 상세 (ex.Not Found)
                , exception.getMessage()
                , extractPath(request)
        );

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }

    private String extractPath(WebRequest request) {
		    // 요청에서 경로만 뽑아옴
        return request.getDescription(false).replace("uri=", "");
    }
}
```

```java
package com.ohgiraffers.exceptionhandler;

public class MemberNotFoundException extends RuntimeException {
    public MemberNotFoundException(String message) {
        super(message);
    }
}
```

```java
package com.ohgiraffers.exceptionhandler;

import com.fasterxml.jackson.annotation.JsonFormat;
import java.time.LocalDateTime;

public class ErrorResponse {

    /* REST API 실패 응답의 공통 JSON 구조. */
    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    private final LocalDateTime timestamp;
    private final int status;
    private final String error;
    private final String message;
    private final String path;

    public ErrorResponse(int status, String error, String message, String path) {
        this.timestamp = LocalDateTime.now();
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }

    public LocalDateTime getTimestamp() { return timestamp; }

    public int getStatus() { return status; }

    public String getError() { return error; }

    public String getMessage() { return message; }

    public String getPath() { return path; }
}
```

### 유효성 검증

의존성 추가  
- I/O → Validation (검증 기능에 필요한 어노테이션)
    ```java
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    ```
    

1. `@Valid` : 값 유효성 검사. Valid 어노테이션이 있어야 아래 어노테이션들이 실행된다.
    - `@Min(value = 숫자, message = “유효성을 지키지 않았을 때 메세지”)` : 최솟값 검증
    - `@NotBlank(message =””)` : 비어 있을 수 없음
2. Controller 업무 규칙 검사 : ex. 중복된 회원 존재

```java
public class MemberDTO {
    @Min(value = 1, message = "회원 번호는 1 이상이어야 합니다.")
    private int no;
    @NotBlank(message = "회원 이름은 비어 있을 수 없습니다.")
    private String name;
    @Min(value = 0, message = "회원 나이는 0 이상이어야 합니다.")
    private int age;
}
```

```java
@PostMapping
public ResponseEntity<Map<String, Object>> createMember(@RequestBody @Valid MemberDTO member) { // 1. Valid 값 유효성 검사
    // 2. 업무 규칙 검사
    boolean duplicatedMemberNo = members.stream()
            .anyMatch(savedMember -> savedMember.getNo() == member.getNo());
    if(duplicatedMemberNo) {
        throw new InvalidMemberRequestException("이미 등록된 회원입니다.");
    }

    members.add(member);

    Map<String, Object> response = new LinkedHashMap<>();

    response.put("message", "회원 등록 성공");
    response.put("member", member);

    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

공통 에러 형태 적용
```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleMethodArgumentNotValid(
        MethodArgumentNotValidException exception, WebRequest request
) {
    // 필요한 메세지를 추출하기 위해 exception 을 타고 내려간다.
    String message = exception.getBindingResult()
            .getFieldErrors()   // 필드 오류 반환 받음
            .stream()
            .findFirst()
            .map(DefaultMessageSourceResolvable::getDefaultMessage)
            .orElse("요청 값이 올바르지 않습니다.");

    ErrorResponse response = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value()
            , HttpStatus.BAD_REQUEST.getReasonPhrase()
            , message
            , extractPath(request));

    return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
}

@ExceptionHandler(InvalidMemberRequestException.class)
public ResponseEntity<ErrorResponse> handleInvalidMemberRequest(
        InvalidMemberRequestException exception, WebRequest request
) {
    ErrorResponse response = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value()
            , HttpStatus.BAD_REQUEST.getReasonPhrase()
            , exception.getMessage()
            , extractPath(request)
    );

    return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
}

private String extractPath(WebRequest request) {
    // 요청에서 경로만 뽑아옴
    return request.getDescription(false).replace("uri=", "");
}
```

### 그 외 모든 에러 처리 (+로그 남기기)

`org.slf4j.Logger` : 프로그램 실행 중 발생한 정보를 기록하는 객체

```java
package com.ohgiraffers.exceptionhandler;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestControllerAdvice
public class GlobalExceptionHandler {

    /* 예상치 못한 500 오류의 실제 원인은 서버 로그에 남긴다. */
    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    private String extractPath(WebRequest request) {
        // 요청에서 경로만 뽑아옴
        return request.getDescription(false).replace("uri=", "");
    }
    
    // 위에서 구체적으로 처리하지 않은 남은 모든 Exception 을 처리
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(
            Exception exception, WebRequest request
    ) {
        // 서버에 에러 로그를 남김
        log.error("처리하지 못한 예외가 발생했습니다", exception);

        ErrorResponse response = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value()    // 500 에러
                , HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase()
                , "서버 내부 오류가 발생했습니다."
                , extractPath(request)
        );

        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
}
```

- [Spring Boot](#spring-boot)
  - [Request Mapping Handler](#request-mapping-handler)
  - [Jackson](#jackson)
    - [클라이언트와 서버 커스텀 직렬화/역직렬화](#클라이언트와-서버-커스텀-직렬화역직렬화)
    - [ObjectMapper](#objectmapper)
  - [Exception 핸들러](#exception-핸들러)
    - [유효성 검증](#유효성-검증)
    - [그 외 모든 에러 처리 (+로그 남기기)](#그-외-모든-에러-처리-로그-남기기)
