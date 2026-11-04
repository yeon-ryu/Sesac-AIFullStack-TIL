# JUnit 테스트
Java 에서 사용하는 테스트

- `@Test` : 단위 테스트
- `@DisplayName` : 테스트 결과 창에 보일 테스트 이름 (IntelliJ 에서 실행 탭 → 체크 표시를 누르면 뜬다)
- `org.junit.jupiter.api.Assertions` : JUnit 검증 도구
    - assertFalse(결과) : false 가 맞는지 확인
    - assertEquals(예상값, 결과) : 예상값과 결과가 동일한지 확인
    - assertNotSame(결과1, 결과2) : 두 결과가 같지 않은지 확인
    - assertThrows(예상 Exception 클래스, 메소드나 catch한 Exception) : 예상한 예외가 발생하는지 확인
- `@BeforeEach`, `@AfterEach` : 각 `@Test` 메소드 수행 전후에 실행

```java
public class ProblemsOfUsingDirectSQLTests {

    private Connection con;

    @BeforeEach
    void setConnection() throws SQLException {
        String url = "jdbc:mysql://localhost:3306/menudb";
        String user = "ohgiraffers";
        String password = "ohgiraffers";

        con = DriverManager.getConnection(url, user, password);
        con.setAutoCommit(false);
    }

    @AfterEach
    void closeConnection() throws SQLException {
        if (con != null && !con.isClosed()) {
            con.rollback();
            con.close();
        }
    }
    
    @Test
    @DisplayName("직접 SQL을 작성하여 신규 메뉴 추가 시 발생하는 문제 확인")
    void testDirectInsertSQL() throws SQLException {
        //given
        Menu menu = new Menu();
        menu.setMenuName("고등어단백질쉐이크");
        menu.setMenuPrice(120000);
        menu.setCategoryCode(1);
        menu.setOrderableStatus("Y");

        String query = "INSERT INTO TBL_MENU(MENU_NAME, MENU_PRICE, CATEGORY_CODE, ORDERABLE_STATUS) VALUES (?, ?, ?, ?)";

        //when
        PreparedStatement pstmt = con.prepareStatement(query);
        pstmt.setString(1, menu.getMenuName());
        pstmt.setInt(2, menu.getMenuPrice());
        pstmt.setInt(3, menu.getCategoryCode());
        pstmt.setString(4, menu.getOrderableStatus());

        int result = pstmt.executeUpdate();

        //then
        Assertions.assertEquals(1, result);
        System.out.println("등록된 행 수 : " + result);

        pstmt.close();
    }
}
```

- `@SpringBootTest` : ApplicationContext 를 사용하고 실제와 유사한 환경에서 테스트 하기 위해 어노테이션 붙임
- `@Transactional` : 테스트가 트랜잭션을 시작한다.
- `@Commit` : 테스트 끝나면 롤백 안하고 커밋을 하기 위해 추가

```java
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Commit;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDateTime;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
@Transactional
public class MemberRegistServiceTests {

    // 필드 주입 할 때는 @Autowired 생략 불가!
    @Autowired
    private MemberRegistService memberRegistService;

    @Test
    @DisplayName("회원 엔티티 매핑과 저장 확인")
    @Commit
    void testRegistMember() {
        // given
        MemberRegistDTO newMember = new MemberRegistDTO(
                "user01", "pass01", "홍길동", "010-1234-5678", "서울시 종로구",
                LocalDateTime.now(), MemberRole.ROLE_MEMBER, "Y");

        // when
        Member savedMember = memberRegistService.registMember(newMember);

        // then
        assertTrue(savedMember.getMemberNo() > 0);  // AI 걸어둔게 제대로 들어가는지 확인
        assertEquals("홍길동", savedMember.getMemberName());
        assertEquals(MemberRole.ROLE_MEMBER, savedMember.getMemberRole());  // enum 으로 제대로 들어가나 확인
    }
}
```

## Spring Data JPA CURD 테스트
```java
@SpringBootTest
@Transactional
class MenuServiceTests {

    @Autowired
    private MenuService menuService;

    @PersistenceContext
    private EntityManager entityManager;

    @Test
    @DisplayName("Spring Data JPA로 메뉴 CRUD 수행")
    void menuCrudTest() {
        MenuResponse created = menuService.createMenu(
                new MenuRequest("돈까스 스파게티", 15000, 4, "Y")
        );
        assertTrue(created.getMenuCode() > 0);

        MenuResponse found = menuService.findMenu(created.getMenuCode());
        assertEquals("돈까스 스파게티", found.getMenuName());

        MenuResponse updated = menuService.updateMenu(
                created.getMenuCode(),
                new MenuRequest("치즈 돈까스 스파게티", 17000, 4, "Y")
        );
        entityManager.flush();
        assertEquals(17000, updated.getMenuPrice());

        menuService.deleteMenu(created.getMenuCode());
        entityManager.flush();
        entityManager.clear();

        assertThrows(
                ResourceNotFoundException.class,
                () -> menuService.findMenu(created.getMenuCode())
        );
    }
}
```