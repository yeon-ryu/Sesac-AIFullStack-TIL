# Spring Boot
## HTTP

HTTP (HyperText Transfer Protocol) : 클라이언트와 서버가 데이터를 주고 받기 위해 사용하는 **통신 규칙(프로토콜)**  
`클라이언트 → HTTP요청 → 서버 → HTTP응답` 의 단방향 구조

- 비연결성 (Connectionless) : 요청이 끝나면 서버는 클라이언트 정보를 유지하지 않는다. (쿠키, 세션 같은 기술이 필요)
- 무상태성 (Stateless) : 요청 간 서버가 상태를 기억하지 않는다. → 단순하고 확정성 좋음
- 텍스트 기반 프로토콜 : 사람이 읽어도 이해할 수 있는 구조로 되어 있음

### HTTP 구성

HTTP Request

1. Request Line
    - HTTP Method : GET, POST, PUT, PATCH, DELETE
    - 요청 url
    - HTTP 버전 (ex. HTTP/1.1)
2. HTTP Headers : HTTP 요청에 대한 부가 정보를 담는다.
    - Content-type : 요청 Body 에 담긴 데이터 형식을 설명 (ex. application/json)
    - Authorization : 인증 정보 (JWT 등)
    - Accept : 클라이언트가 원하는 응답 형식 (ex. Chrome/120)
3. HTTP Body : 요청 본문
    
    ```java
    {
    	"name" : "홍길동",
    	"age" : 20
    }
    ```
    

HTTP Response

1. Status Line
    - HTTP Version : ex. HTTP/1.1
    - Status Code : ex. 200
    - Response Phrase : ex. OK, Not Found
2. Headers
    - Content-Type : 응답 Body 형식을 지정
    - Set-Cookie : 서버가 클라이언트(브라우저)한테 쿠키 저장을 요청. 로그인 토큰 등. (ex. session=abc123)
    - Cache-Control : 캐시 정책 설정 (ex. no-cache)
3. Body :  서버가 클라이언트에 보내는 실제 데이터
    - JSON 데이터
    - HTML
    - 이미지 : PNG 파일의 byte 데이터

### HTTP 상태 코드

- 1xx : 정보
- 2xx : 성공
    - 200 OK (요청 성공)
    - 2001 Created (리소스 생성 성공)
- 3xx : 리다이렉트
- 4xx : 클라이언트 오류
    - 400 Bad Request (잘못된 요청)
    - 401 Unauthorized (로그인 필요 / 인증 안됨)
    - 403 Forbidden (권한 없음 / 인증은 됐지만 권한 없음)
    - 404 Not Found (리소스 없음)
- 5xx : 서버 오류
    - 500 Internal Server Error (서버 내부 오류)

### 데이터 전달 방식

- Query String : `/search?keyword=java&page=1`
- Path Variable : `/user/10` (`/user/{id}`)
- Body 데이터
    
    ```java
    요청
    POST /users
    Content-Type : application/json
    Accept : application/json
    
    {
    	"name" : "홍길동"
    }
    
    응답
    HTTP/1.1 201 Created
    Content-Type : application/json
    Location : /users/1
    ```
    

## JSON

JSON (JavaScript Object Notation) : JKavaScript 의 객체 표기법을 기반으로 한, 가볍고 읽기 쉬운 텍스트 기반의 데이터 교환 형식. **서로 다른 언어와 시스템 간에 데이터를 주고받는 것**을 주 목적으로 합니다.

### 작성 규칙

- 데이터는 `“key”:value` 의 쌍으로 구성되며, 콜론(`:`) 으로 구분된다.
- 여러 개의 항목을 넣을 때는 쉼표(`,`)로 구분한다.
- Json 객체 전체는 중괄호(`{}`)로 감싼다.

### Spring Boot 와 JSON

Spring Boot 는 `@RestController` 어노테이션을 붙이면 해당 컨트롤러에서 Jackson 라이브러리를 활용하여 자동으로 변환시킨다.

## REST API

클라이언트와 서버가 HTTP 를 통해 데이터를 주고받기 위한 설계 방식

React → HTTP 요청 → Spring Boot REST API → DB 처리 → JSON 응답 → React 화면 렌더링

REST API 중요 포인트  
- URL 은 리소스를 표현한다.
    - url 은 동사가 아니라 리소스 중심으로 작성 (동작은 HTTP Method 가 표현하고, URL 은 대상을 표현한다)
    - 리소스 이름은 보통 복수형으로 통일한다. (ex. 게시글 하나 등록 `POST /posts)`
- HTTP Method 는 행위를 표현한다.
    - GET : 조회
        - 멱등성 : 같은 요청을 여러번 보내도 최종 상태가 같다.
    - POST : 생성
    - PUT : 전체 수정
        - 멱등성 : 같은 요청을 여러번 보내도 최종 상태가 같다.
    - PATCH : 부분 수정
        - 변경값을 지정하는 방식이면 멱등해 지지만, 조회수를 1 올리는 것 같을 경우 멱등하지 않다.
    - DELETE : 삭제
        - 멱등성 : 같은 요청을 여러번 보내도 최종 상태가 같다.
- 응답은 상태 코드와 JSON body 로 표현한다.

### 중첩 리소스 설계

리소스 사이에 부모-자식 관계가 있을 경우 부모의 문맥이 필요하다면 url 을 계층적으로 표현할 수 있다.

ex. 게시글 하나에 댓글 여러개 (`GET /posts/{postId}/comments`)  
자체의 식별자가 중요할 경우 (`DELETE /comments/{commentId}`)

### Lombok

의존성 추가

```java
compileOnly 'org.projectlombok:lombok'
annotationProcessor 'org.projectlombok:lombok'
```

사용

```java
package com.ohgiraffers.restapi.section01.responseentity;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@AllArgsConstructor // 모든 필드 있는 생성자
@NoArgsConstructor  // 기본 생성자
@Getter
@Setter
@ToString(exclude = "pwd")  // pwd 는 빼고 toString
public class UserRequestDTO {

    /* 클라이언트가 회원 등록·수정 요청으로 보내는 값만 담는다. */
    private String id;
    private String pwd;
    private String name;
    private String status;
}
```

### REST API 예시

공통 Response 커스텀 객체

```java
@AllArgsConstructor
@Getter
@Setter
@ToString
public class ResponseMessage {
    private int httpStatus;
    private String message;
    private Map<String, Object> results;
}
```

- `ResponseEntity.created(URI)` : 상태를 201 로 만들고, 전달한 URI 를 Header 에 Location 값으로 넣는다.
- `ResponseEntity.noContent()` : 상태를 204로 정하고, body가 없으므로 build()로 완성

```java
@RestController
@RequestMapping("/api/v1/response/users")
public class ResponseEntityTestController {

    private final List<UserResponseDTO> users;
    private int nextUserNo = 4;

    public ResponseEntityTestController() {
        users = new ArrayList<>();

        users.add(new UserResponseDTO(1, "user01", "고릴라", new Date(), "active"));
        users.add(new UserResponseDTO(2, "user02", "호랑이", new Date(), "active"));
        users.add(new UserResponseDTO(3, "user03", "원숭이", new Date(), "inactive"));
    }

    @GetMapping
    public ResponseEntity<ResponseMessage> findUserByConditions(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String status
    ) {
        List<UserResponseDTO> foundUsers = users.stream()
                .filter(user -> {
                    // 값이 존재하면 상태가 일치하거나 이름을 포함하는 것만 뽑아내겠다
                    if(status != null && !user.getStatus().equals(status)) return false;
                    if(name != null && !user.getName().contains(name)) return false;
                    return true;
                })
                .collect(Collectors.toList());

        Map<String, Object> responseMap = new HashMap<>();
        responseMap.put("users", foundUsers);

        ResponseMessage responseMessage = new ResponseMessage(HttpStatus.OK.value(), "조회 성공", responseMap);

        return new ResponseEntity<>(responseMessage, HttpStatus.OK);
    }

    @GetMapping("/{userNo}")
    public ResponseEntity<ResponseMessage> findUserByNo(@PathVariable int userNo) {
        UserResponseDTO foundUser = users.stream()
                .filter(user -> user.getNo() == userNo)
                .findFirst()
                .get();

        ResponseMessage responseMessage = new ResponseMessage(HttpStatus.OK.value(), "조회 성공"
                , Map.of("user", foundUser));

        return ResponseEntity.ok(responseMessage);
    }

    @PostMapping
    public ResponseEntity<Void> registUser(@RequestBody UserRequestDTO request) {
        UserResponseDTO newUser = new UserResponseDTO(
                nextUserNo++, request.getId(), request.getName(), new Date(), request.getStatus());

        users.add(newUser);

        // ResponseEntity.created(URI) : 상태를 201 로 만들고, 전달한 URI 를 Header 에 Location 값으로 넣는다.
        return ResponseEntity.created(URI.create("api/v1/response/users/" + newUser.getNo())).build();
    }

    // Put 은 활용 여부와 별개로 기본적으로 모든 필드를 보내야 한다.
    @PutMapping("/{userNo}")
    public ResponseEntity<Void> modifyUser(@PathVariable int userNo, @RequestBody UserRequestDTO modifyInfo) {
        UserResponseDTO foundUser = users.stream()
                .filter(user -> user.getNo() == userNo)
                .findFirst()
                .get();

        if(modifyInfo.getId() != null) foundUser.setId(modifyInfo.getId());
        if(modifyInfo.getName() != null) foundUser.setName(modifyInfo.getName());
        if(modifyInfo.getStatus() != null) foundUser.setStatus(modifyInfo.getStatus());
        // 비밀번호는 없으니 변경 코드 없음

        // ResponseEntity.noContent() 가 상태를 204로 정하고, body가 없으므로 build()로 완성
        // body 를 넣을거면 ok 로 하고 넣으면 된다.
        return ResponseEntity.noContent().build();
    }

    @DeleteMapping("/{userNo}")
    public ResponseEntity<Void> removeUser(@PathVariable int userNo) {
        UserResponseDTO foundUser = users.stream()
                .filter(user -> user.getNo() == userNo)
                .findFirst().get();
        users.remove(foundUser);

        return ResponseEntity.noContent().build();
    }
}
```

### Valid

- NotBlank() : 문자열 전용
- NotNull() : 객체의 존재 여부
- Past() : 날짜를 과거만 허용
- Size() : 문자열 사이즈 제한

```java
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@ToString(exclude = "pwd")
public class UserDTO {

    private Integer no;
    @NotBlank(message = "아이디는 반드시 입력되어야 합니다.")
    private String id;
    @NotBlank(message = "비밀번호는 반드시 입력되어야 합니다.")
    @Size(max = 15, min = 4, message = "비밀번호는 4글자 이상 15글지 이하여야 합니다.")
    private String pwd;
    @NotBlank(message = "이름은 반드시 입력되어야 합니다.")
    @Size(max = 15, message = "이름은 15글지 이하여야 합니다.")
    private String name;
    @NotNull(message = "가입일은 반드시 입력되어야 합니다.")
    @Past(message = "가입일은 현재보다 과거 날짜가 입력되어야 합니다.")   // 현재 날짜보다 과거인지 검증
    private Date enrollDate;
}
```

발생할 수 있는 예외를 묶어서 한꺼번에 처리

- `@RestControllerAdvicse` : RestController 의 Advice 가 아니라 JSON 을 반환하는 ControllerAdvice
- `getFieldError()` : 첫번째 필드 오류를 꺼낸다.
    - 필요하다면 모든 필드 오류를 꺼내는 메소드를 사용한다.

```java
// RestController 의 Advice 가 아니라 JSON 을 반환하는 ControllerAdvice
@RestControllerAdvice
public class ExceptionController {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodValid(MethodArgumentNotValidException e) {
        String code = "";
        String description = "";

        // getFieldError() : 첫번째 필드 오류를 꺼낸다.
        FieldError fieldError = e.getBindingResult().getFieldError();

        // DTO 클래스에 사용자 정의로 valid 용 어노테이션을 만들어 달아놓을 수도 있다.
        // 그 경우 여러 필드를 같이 검사할 경우 특정 필드 오류가 아니라 객체 전체 오류로 하여 필드 에러가 없을 수도 있다.
        String detail = fieldError != null ? fieldError.getDefaultMessage() : "요청 값을 확인해주세요.";
        String bindResultCode = fieldError != null ? fieldError.getCode() : "";

        switch(Objects.requireNonNull(bindResultCode)) {
            case "NotBlank", "NotNull" :
                code = "ERROR_CODE_00001";  // 커스텀 에러 코드
                description = "필수 값이 누락되었거나 공백입니다.";
                break;
            case "Size" :
                code = "ERROR_CODE_00002";
                description = "글자 수를 확인해주세요";
                break;
            default:
                code = "ERROR_CODE_99999";
                description = "요청 값 검증에 실패했습니다.";
        }

        ErrorResponse errorResponse = new ErrorResponse(code, description, detail);
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
}
```

### Swagger

의존성 추가

Web → SpringDoc OpenAPI

```java
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:3.0.3'
```

Swagger URL : `http://localhost:8080/swagger-ui/index.html`

application.properties

```java
# swagger 범위를 특정 패키지로만 한정
springdoc.packages-to-scan=com.ohgiraffers.restapi.section03
```

#### Swagger 세부 설정

문서 전체 설명 설정

```java
package com.ohgiraffers.restapi.section03.swagger;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    // OpenAPI : Swagger 문서 설정 정보
    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI().info(swaggerInfo());
    }

    private Info swaggerInfo() {
        return new Info()
                .title("Ohgiraffers API")
                .description("SpringBoot Swagger 연동 테스트")
                .version("1.0.0");
    }
}
```

!["문서전체설정"](/Reference/swagger-page.png)

DTO 설명 설정

```java
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@ToString(exclude = "pwd")
@Schema(description = "회원 요청 정보")
public class UserRequestDTO {

    @Schema(description = "회원ID", example = "user01")
    private String id;
    private String pwd;
    @Schema(description = "회원 이름", example = "신규 회원")
    private String name;
    @Schema(description = "활동 상태", example = "inactive")
    private String status;
}
```

!["dto예시"](/Reference/swagger-dto-1.png) !["dto스키마"](/Reference/swagger-dto-2.png)


API 메소드 설명 설정

```java
@Tag(name = "회원 관련 API", description = "회원 CRUD API")
@RestController
@RequestMapping("/api/v1/swagger/users")
public class SwaggerTestController {

    private final List<UserResponseDTO> users;
    private int nextUserNo = 4;

    public SwaggerTestController() {
        users = new ArrayList<>();
        users.add(new UserResponseDTO(1, "user01", "고릴라", new Date(), "active"));
        users.add(new UserResponseDTO(2, "user02", "호랑이", new Date(), "active"));
        users.add(new UserResponseDTO(3, "user03", "원숭이", new Date(), "inactive"));
    }

    @Operation(summary = "회원 목록 검색", description = "이름 혹은 상태로 검색")
    @ApiResponses({@ApiResponse(responseCode = "200", description = "회원 목록 조회 성공")})    // 응답 예시 리스트
    @GetMapping
    public ResponseEntity<ResponseMessage> findUserByConditions(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String status) {

        List<UserResponseDTO> foundUsers = users.stream()
                .filter(user -> {
                    if (status != null && !user.getStatus().equals(status)) {
                        return false;
                    }
                    if (name != null && !user.getName().contains(name)) {
                        return false;
                    }
                    return true;
                })
                .collect(Collectors.toList());

        Map<String, Object> responseMap = new HashMap<>();
        responseMap.put("users", foundUsers);
        ResponseMessage responseMessage = new ResponseMessage(
                HttpStatus.OK.value(), "조회 성공", responseMap);

        return ResponseEntity.ok(responseMessage);
    }
    
    @ApiResponses({@ApiResponse(responseCode = "200", description = "회원 조회 성공")
            , @ApiResponse(responseCode = "404", description = "회원 정보 없음")})
    @GetMapping("/{userNo}")
    public ResponseEntity<ResponseMessage> findUserByNo(@PathVariable int userNo) {
        UserResponseDTO foundUser = findUser(userNo);

        Map<String, Object> responseMap = new HashMap<>();
        responseMap.put("user", foundUser);
        ResponseMessage responseMessage = new ResponseMessage(
                HttpStatus.OK.value(), "조회 성공", responseMap);

        return ResponseEntity.ok(responseMessage);
    }
    
    private UserResponseDTO findUser(int userNo) {
        return users.stream()
                .filter(user -> user.getNo() == userNo)
                .findFirst()
                .orElseThrow(() -> new SwaggerUserNotFoundException(
                        userNo + "번 회원 정보를 찾을 수 없습니다."));
    }
}
```
!["메소드설명"](/Reference/swagger-method.png)
!["메소드결과"](/Reference/swagger-response.png)


- [Spring Boot](#spring-boot)
  - [HTTP](#http)
    - [HTTP 구성](#http-구성)
    - [HTTP 상태 코드](#http-상태-코드)
    - [데이터 전달 방식](#데이터-전달-방식)
  - [JSON](#json)
    - [작성 규칙](#작성-규칙)
    - [Spring Boot 와 JSON](#spring-boot-와-json)
  - [REST API](#rest-api)
    - [중첩 리소스 설계](#중첩-리소스-설계)
    - [Lombok](#lombok)
    - [REST API 예시](#rest-api-예시)
    - [Valid](#valid)
    - [Swagger](#swagger)
      - [Swagger 세부 설정](#swagger-세부-설정)
