# Spring Boot
## File Upload

`Content-Type : multipart/form-data` : 파일 + 일반 입력값을 한번에 보낼 경우

1. Spring 의 `MultipartFile` 로 받을 수 있음
2. `transferTo(파일 경로)` : 실제 파일 저장
3. 서버의 uploads 디렉토리 (나중에 SFTP 등 사용하여 외부의 목표 디렉토리에 저장)

application.properties 설정

```java
# 파일 하나의 최대 업로드 크기
spring.servlet.multipart.max-file-size=10MB

# multipart 요청 전체의 최대 크기
spring.servlet.multipart.max-request-size=50MB

# 업로드 파일이 저장될 프로젝트 루트 기준 디렉터리 (커스텀 key)
file.upload-dir=uploads
```

FileDTO (실제 파일 데이터 자체가 아니라 저장된 파일의 메타 정보)

```java
public class FileDTO {

    /* 업로드 결과를 클라이언트에 JSON으로 내려주기 위한 DTO.
     * 실제 파일 데이터 자체가 아니라 저장된 파일의 메타 정보를 담는다. */

    /* 사용자가 업로드한 원래 파일명 */
    private String originFileName;

    /* 서버에 **충돌 방지를 위해 UUID로 저장**한 파일명 */
    private String savedFileName;

    /* 업로드 루트 디렉터리를 기준으로 한 상대 경로 */
    private String filePath;

    /* 사용자가 함께 보낸 파일 설명 */
    private String fileDescription;

    /* 파일 크기(byte) */
    private long fileSize;

    /* text/plain, image/png 같은 파일 MIME 타입 */
    private String contentType;
}
```

api-test.http (파일 테스트는 postman 추천)

```java
### 1. 단일 파일 업로드
POST http://localhost:8080/api/v1/files/single
Content-Type: multipart/form-data; boundary=singleBoundary

--singleBoundary
Content-Disposition: form-data; name="description"

단일 파일 설명입니다.
--singleBoundary
Content-Disposition: form-data; name="file"; filename="sample.txt"
Content-Type: text/plain

single file upload sample
--singleBoundary--

### 2. 다중 파일 업로드
POST http://localhost:8080/api/v1/files/multiple
Content-Type: multipart/form-data; boundary=multiBoundary

--multiBoundary
Content-Disposition: form-data; name="description"

다중 파일 설명입니다.
--multiBoundary
Content-Disposition: form-data; name="files"; filename="sample1.txt"
Content-Type: text/plain

first file sample
--multiBoundary
Content-Disposition: form-data; name="files"; filename="sample2.txt"
Content-Type: text/plain

second file sample
--multiBoundary--
```

### 단일 파일 저장

- `path.resolve(하위 경로)` : 상위경로/하위경로 로 Path 를 만든다.
- `path.normalize();` : ‘.’(현재 디렉토리), ‘..’(상위 디렉토리) 같은 불필요한 중복 요소를 제거하여 정규화
- `StringUtils.cleanPath(원본파일명)` : 구분자를 일정하게 정리해주고 파일명으로 디렉토리에 영향을 주지 않도록 변경
- `Files.createDirectories(경로);` : 실제 디렉토리 폴더 생성
- `file.transferTo(파일명 포함한 절대 경로);` : 실제 파일 생성
- `file.getSize()` : 바이트 단위 크기

```java
@RestController
@RequestMapping("/api/v1/files")
public class FileUploadController {

    private final Path uploadRoot;

    public FileUploadController(@Value("${file.upload-dir}") String uploadDir) {
        // Paths.get(경로 문자열) : 파일 경로로 바꿔준다.
        // toAbsolutePath() : 절대 경로로 바꿔준다.
        // 설정 값을 정규화된 "절대 경로"로 바꾸고 "모든 업로드의 저장 기준"으로 사용
        this.uploadRoot = Paths.get(uploadDir).toAbsolutePath().normalize();
    }
    
    @PostMapping("/single")
    public ResponseEntity<Map<String, Object>> uploadSingleFile(
				    /* 클라이언트에서 전달한 FormData 의 key 와 같아야한다 */
            @RequestParam("file") MultipartFile file    // React 에서 전달 예시 : formData.append("file", 선택한파일)
            , @RequestParam(value="description", required = false) String description
    ) throws IOException {
        FileDTO uploadFile = saveFile(file, "single", description);

        Map<String, Object> response = new LinkedHashMap<>();
        response.put("message", "단일 파일 업로드 성공");
        response.put("file", uploadFile);

        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    // 파일 저장
    private FileDTO saveFile(MultipartFile file, String directoryName, String description) throws IOException {
        validateFile(file);

        // 요청 종류별(directoryName) 하위 디렉토리를 만든다.
        // resolve() : 하위 경로를 붙인다. ([uploadRoot]/[directoryName])
        Path uploadPath = uploadRoot.resolve(directoryName).normalize();

        // Files.createDirectories(경로); : 실제 디렉토리 폴더 생성
        Files.createDirectories(uploadPath);

        // 원본 파일 이름 검증
        // StringUtils.cleanPath(원본파일명) : 구분자를 일정하게 정리해주고 파일명으로 디렉토리에 영향을 주지 않도록 변경
        String originFileName = StringUtils.cleanPath(file.getOriginalFilename());
        if(originFileName.contains("..")) {
            throw new IllegalArgumentException("파일명에 올바르지 않은 경로가 포함되어 있습니다.");
        }

        originFileName = Paths.get(originFileName).getFileName().toString();
        // 확장자 추출
        String extension = extractExtension(originFileName);

        // UUID 를 사용하여 중복되지 않는 문자를 만든 뒤, 원본 확장자를 붙여 파일 저장명을 완성 (UUID 는 중복될 확률이 한 없이 낮을뿐 0는 아니다)
        // UUID 에는 '-' 가 들어갈 수 있기에 해당 문자 제거
        String savedFileName = UUID.randomUUID().toString().replace("-", "") + extension;

        // 실제 디스크에 쓸 위치 Path
        // targetPath = [uploadRoot]/[directoryName]/[변경된파일명].확장자
        Path targetPath = uploadPath.resolve(savedFileName).normalize();

        // 일반적으로는 걸릴 일이 없지만 만에 하나 문제가 발생할 경우
        if(!targetPath.startsWith(uploadPath)) {
            throw new IllegalArgumentException("저장할 파일 경로가 올바르지 않습니다.");
        }

        // 실제 파일 저장 (생성)
        file.transferTo(targetPath);

        // 서버 내부 경로 노출을 줄이기 위해 절대 경로 대신 업로드 기준 상대 경로만 client 에게 전달
        // 운영체제에 따른 경로 구분자('//') 를 하나로 통일('\')
        // relativePath = [directoryName]/[변경된파일명].확장자
        String relativePath = uploadRoot.relativize(targetPath).toString().replace('\\', '/');

        // 저장 파일 메타데이터 반환
        return new FileDTO(originFileName, savedFileName, relativePath, description, file.getSize(), file.getContentType());
    }

    // 저장 1단계. 파일이 유효한지 검사
    private void validateFile(MultipartFile file) {
        // 빈 파일 요청은 정상 업로드 처리하지 않는다.
        if(file == null || file.isEmpty()) {
            throw new IllegalArgumentException("업로드할 파일이 비어있습니다.");
        }
        // 파일명 확인
        if(file.getOriginalFilename() == null || file.getOriginalFilename().isBlank()) {
            throw new IllegalArgumentException("원본 파일명이 없습니다.");
        }
    }

    // 저장 2단계. 확장자 추출
    private String extractExtension(String originFileName) {
        // 마지막 . 의 위치
        int lastDotIndex = originFileName.lastIndexOf(".");

        // . 이 존재하지 않을 경우
        if (lastDotIndex == -1) {
            return "";
        }

        // . 이 포함된 확장자
        return originFileName.substring(lastDotIndex);
    }
}
```

### 다중 파일 저장

여러 파일 업로드 하던 도중 실패 시 서버에 저장된 파일도 취소

```java
@PostMapping("/multiple")
public ResponseEntity<Map<String, Object>> uploadMultipleFiles(
        @RequestParam("files") List<MultipartFile> files
        , @RequestParam(value="description", required = false) String description
) throws IOException {
    if(files == null || files.isEmpty()) {
        throw new IllegalArgumentException("업로드할 파일 목록이 비어 있습니다.");
    }

    // 업로드 성공한 파일 목록
    List<FileDTO> uploadedFiles = new ArrayList<>();

    try {
        for (MultipartFile file : files) {
            uploadedFiles.add(saveFile(file, "multiple", description));
        }
    } catch (IOException | RuntimeException e) {
        // 중간 실패 시 앞서 저장한 파일을 정리한 뒤 원래 예외를 다시 던진다
        cleanupSavedFiles(uploadedFiles, e);
        throw e;
    }

    Map<String, Object> response = new LinkedHashMap<>();
    response.put("message", "다중 파일 업로드 성공");
    response.put("files", uploadedFiles);

    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}

// 다중 파일 업로드 중간 실패 시, 이미 저장된 파일을 정리
private void cleanupSavedFiles(List<FileDTO> uploadedFiles,
                               Exception originalException) {
    for (FileDTO uploadedFile : uploadedFiles) {
        Path savedPath = uploadRoot.resolve(uploadedFile.getFilePath()).normalize();
        try {
            if (savedPath.startsWith(uploadRoot)) {
                Files.deleteIfExists(savedPath);
            }
        } catch (IOException cleanupException) {
		        /* 정리 실패 시, 저장 실패의 원인을 덮어쓰지 않도록 부가 정보로 덧붙인다 */
            originalException.addSuppressed(cleanupException);
        }
    }
}
```

- JSON : application/json → Jackson 으로 역직렬화된 객체
- 파일 : multipart/form-data → MultipartFile, List<MultipartFile> (`@RequestParam` 으로 받음)
    - 실제 저장 : transferTo()

## Interceptor

클라이언트의 HTTP 요청이 Controller 에 도착하기 전과 Controller 실행이 끝난 후 공통 로직을 처리하는 SpringMVC 기능

- Interceptor :  **HTTP 요청 응답(Controller) 앞뒤로** 동작
- AOP : Spring bean **메소드 실행 앞뒤로** 동작

1. 요청
2. `preHandle()` : Controller 실행 전 호출한다. true 를 반환하면 다음 단계로 진행, false 를 반환하면 요청 처리 중단
3. Controller 핸들러 메소드 동작
4. `postHandle()` : Controller 가 정상적으로 실행된 후 호출. 예외가 발생하면 호출되지 않을 수 있다.
5. `afterCompletion()` : 요청 처리가 모두 끝난 후 호출. 예외 발생 여부와 관계 없이 마무리 작업에 활용.
6. 응답완료

인터셉터 메소드는 `HandlerInterceptor` 를 구현한 인터셉터에서 오버라이드로 사용 가능

```java
// 실행 시간 측정 인터셉터
@Component
public class StopwatchInterceptor implements HandlerInterceptor {
    private static final String START_TIME = "startTime";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        long startTime = System.nanoTime();
        request.setAttribute(START_TIME, startTime);

        System.out.println("[Stopwatch] preHandle " + request.getMethod() + " " + request.getRequestURI());

        // 검증하고 잘못 됐을 경우 예외 던질 거라서 true 로 박아둠
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
        Object startTimeAttribute = request.getAttribute(START_TIME);

        // Long 타입인지 확인하고 Long 타입이라면 startTime 에 넣는다.
        if(startTimeAttribute instanceof Long startTime) {
            // 단위 구분을 위해 언더바(_)를 넣어도 값에 영향을 주지 않는다.
            long elapsedTime = (System.nanoTime() - startTime) / 1_000_000;
            System.out.println("[Stopwatch] afterCompletion " + request.getMethod()
                    + " " + request.getRequestURI() + " " + elapsedTime + "ms");
        }
    }
}
```

인터셉터는 `WebMvcConfigurer` 의 `addInterceptors()`에서 등록하고 적용할 URL 패턴을 지정해야 한다.

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

    private final StopwatchInterceptor stopwatchInterceptor;

    public WebConfiguration(StopwatchInterceptor stopwatchInterceptor) {
        this.stopwatchInterceptor = stopwatchInterceptor;
    }

    // stopwatchInterceptor 를 /api/v1/** 요청에서 높은 우선 순위로 사용한다.
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(stopwatchInterceptor)
                .addPathPatterns("/api/v1/**")
                .order(1);
    }
}
```

### Interceptor로 권한 검증

```java
@PostMapping("/admin/menus")
public ResponseEntity<Map<String, Object>> registMenu(@RequestBody MenuDTO menu) {
    /* /admin 경로는 ApiKeyInterceptor 적용 대상이다. */
    menus.add(menu);

    Map<String, Object> response = new LinkedHashMap<>();
    response.put("message", "관리자 메뉴 등록 성공");
    response.put("menu", menu);

    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

```java
@Component
public class ApiKeyInterceptor implements HandlerInterceptor {
    private static final String API_KEY_HEADER = "X-API-KEY";
    private static final String VALID_API_KEY = "secret-keyㄴ";

    private final ObjectMapper objectMapper;

    public ApiKeyInterceptor(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String apiKey = request.getHeader(API_KEY_HEADER);

        if(VALID_API_KEY.equals(apiKey)) {
            System.out.println("[ApiKey] authorized " + request.getMethod() + " " + request.getRequestURI());
            return true;
        }

        System.out.println("[ApiKey] unauthorized " + request.getMethod() + " " + request.getRequestURI());

        // false 로 Controller 를 막을 것이므로 상태, 형식, 본문을 여기서 직접 작성
        response.setStatus(HttpStatus.UNAUTHORIZED.value());    // 401
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("UTF-8");

        Map<String, Object> errorResponse = new LinkedHashMap<>();
        errorResponse.put("status", HttpStatus.UNAUTHORIZED.value());
        errorResponse.put("error", HttpStatus.UNAUTHORIZED.name());
        errorResponse.put("message", "관리자 API 를 호출하려면 X-API-KEY 헤더가 필요합니다.");
        errorResponse.put("path", request.getRequestURI());

        // ObjectMapper 로 오류 정보를 JSON 문자열로 직렬화하여 PrintWriter 로 그 문자열을 HTTP 응답 본문에 쓴다.
        // response.getWriter() -> PrintWriter.write(직접써줌)
        response.getWriter().write(objectMapper.writeValueAsString(errorResponse));

        return false;
    }
}
```

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

    private final StopwatchInterceptor stopwatchInterceptor;
    private final ApiKeyInterceptor apiKeyInterceptor;

    public WebConfiguration(StopwatchInterceptor stopwatchInterceptor, ApiKeyInterceptor apiKeyInterceptor) {
        this.stopwatchInterceptor = stopwatchInterceptor;
        this.apiKeyInterceptor = apiKeyInterceptor;
    }

    // stopwatchInterceptor 를 /api/v1/** 요청에서 사용한다. (숫자가 낮을수록 우선순위 높음)
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(stopwatchInterceptor)
                .addPathPatterns("/api/v1/**")
                .order(1);

        registry.addInterceptor(apiKeyInterceptor)
                .addPathPatterns("/api/v1/admin/**")
                .order(2);
    }
}
```

## 클라이언트-서버 연결

### CORS

Cross-Origin Resource Sharing (교차 출처 리소스 공유) : 브라우저가 출처가 다른 서버와 연결할 때 보안 규칙

서버에서 CORS 허용을 해줘야 클라이언트에 반환해준다. (혹은 프론트에서 proxy 로 우회)

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

    // CORS 허용
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000")    // 응답을 읽도록 허용할 출처
                .allowedMethods("GET", "POST", "OPTIONS")   // 허용할 메소드 (OPTIONS : 사전 확인 요청)
                .allowedHeaders("Content-Type");
    }
}
```

```jsx
useEffect(() => {
  async function loadMenus() {
      try {
          const response = await fetch(`${API_BASE_URL}/api/v1/menus`);

          if(!response.ok) {
              throw new Error("메뉴를 불러오지 못했습니다.");
          }

          const data = await response.json();

          setMenus(data);
      } catch(e) {
          setError(e.message);
      } finally {
          setLoading(false);
      }
  }

  loadMenus();
}, []);

const handleSubmit = async (e) => {
    e.preventDefault(); // form submit 기본 새로고침 막기
    setError("");

    try {
        const res = await fetch(`${API_BASE_URL}/api/v1/menus`, {
            method: "POST",
            headers: {"Content-Type" : "application/json"},
            // 단축 문법 {name : name, price : Number(price)}
            body: JSON.stringify({name, price : Number(price)})
        });

        if(!res.ok) throw new Error("메뉴를 등록하지 못했습니다.");

        const createdMenu = await res.json();

        // 서버가 돌려준 새 메뉴를 기존 배열 뒤에 추가한 새 배열로 상태를 갱신한다.
        setMenus(prev => [...prev, createdMenu]);

        setName("");
        setPrice("");
    } catch(e) {
        setError(e.message);
    }
}
```

```java
@RestController
@RequestMapping("/api/v1/menus")
public class MenuController {

    private final List<MenuResponse> menus = new ArrayList<>();
    private int nextCode = 3;

    public MenuController() {
        menus.add(new MenuResponse(1, "아메리카노", 3000));
        menus.add(new MenuResponse(2, "카페라떼", 4000));
    }

    @GetMapping
    public ResponseEntity<List<MenuResponse>> findAllMenus() {
        return ResponseEntity.ok(menus);
    }

    @PostMapping
    public ResponseEntity<MenuResponse> registMenu(@RequestBody MenuRequest request) {
        MenuResponse createdMenu = new MenuResponse(nextCode++, request.getName(), request.getPrice());
        menus.add(createdMenu);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdMenu);
    }
}
```

- [Spring Boot](#spring-boot)
  - [File Upload](#file-upload)
    - [단일 파일 저장](#단일-파일-저장)
    - [다중 파일 저장](#다중-파일-저장)
  - [Interceptor](#interceptor)
    - [Interceptor로 권한 검증](#interceptor로-권한-검증)
  - [클라이언트-서버 연결](#클라이언트-서버-연결)
    - [CORS](#cors)
