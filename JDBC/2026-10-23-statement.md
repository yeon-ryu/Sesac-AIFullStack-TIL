# JDBC
## Statement
java 에서 정적 SQL 문을 처리하는 메서드

쿼리 관련은 대문자로 쓰는게 관례 (ex. `SELECT * FROM TBL_MENU` )

- `stmt = con.createStatement();`  : Statement 생성
- ResultSet 의 `next()`  : 다음행으로 이동하여 해당 행에 값이 존재하는지 boolean 값으로 반환한다. (ResultSet 은 초기에는 첫 행보다 앞에 존재한다.)
    - `getXXX(컬럼명)` : 커서가 가리키는 행의 컬럼을 XXX 데이터 타입으로 반환한다.

```java
package com.ohgiraffers.section01.statement;

import com.ohgiraffers.common.JDBCTemplate;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class Application {
    public static void main(String[] args) {
        // 트랜잭션 처리를 위한 DBMS 연동용 커넥션 객체 생성
        Connection con = JDBCTemplate.getConnection();

        // 쿼리를 운반하고 결과를 반환하는 객체
        Statement stmt = null;
        // select 결과집합을 받아 올 용도의 인터페이스 (조회결과를 받아오는 객체)
        ResultSet rset = null;

        try {
            stmt = con.createStatement();
            rset = stmt.executeQuery("SELECT menu_name, menu_price FROM tbl_menu");

            /*
            * next() : ResultSet 의 커서 위치를 하나 내리면서 행이 존재하면 true 를 반환한다.
            * getXXX(컬럼명) : 커서가 가리키는 행의 컬럼을 XXX 데이터 타입으로 반환한다.
            * */
            while(rset.next()) { // 다음행으로 이동하여 읽을 값이 있는지 boolean 값 반환
                System.out.println(rset.getString("menu_name") + ", " + rset.getInt("menu_price"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            // Statement, ResultSet 도 닫아줘야하고 열린 역순으로 닫아준다.
            JDBCTemplate.close(rset);
            JDBCTemplate.close(stmt);
            JDBCTemplate.close(con);
        }
    }
}
```

## PreparedStatement
- `PreparedStatement pstmt = con.prepareStatement(미완성 쿼리문);`
- `?`  : 단일 값이 들어갈 위치 홀더
- `pstmt.setString(몇번째 위치 홀더, 변수);`

```java
package com.ohgiraffers.section02.preparedStatement;

import com.ohgiraffers.common.JDBCTemplate;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Scanner;

public class Application {
    public static void main(String[] args) {
        Connection con = JDBCTemplate.getConnection();

        PreparedStatement pstmt = null;
        ResultSet rset = null;

        Scanner sc = new Scanner(System.in);

        System.out.print("조회하실 메뉴 번호를 입력하세요 : ");
        String menuCode = sc.nextLine();
        /* ? : 위치 홀더 */
        String query = "SELECT MENU_NAME, MENU_PRICE FROM TBL_MENU WHERE MENU_CODE = ?";

        try {
            pstmt = con.prepareStatement(query);

            // sql 위치홀더 설정
            // pstmt.setString(넣을 위치, 넣을 값);
            pstmt.setString(1, menuCode);
            rset = pstmt.executeQuery();

            if(rset.next()) {
                System.out.println(rset.getString("MENU_NAME") + ", " + rset.getInt("MENU_PRICE"));
            } else {
                System.out.println("해당 코드 메뉴의 조회 결과가 없습니다.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            sc.close();
            JDBCTemplate.close(rset);
            JDBCTemplate.close(pstmt);
            JDBCTemplate.close(con);
        }
    }
}
```

### SQL 인젝션
SQL Injection : 입력값 들어가야할 자리에 쿼리문이 들어가는 악의적 공격  

> **Statement 는 SQL 인젝션을 방어할 수 없다.**
```java
package com.ohgiraffers.section03.sqlinjection;

import com.ohgiraffers.common.JDBCTemplate;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class Application {

    private static String empId = "210";
    private static String empName = "' OR 1=1 AND EMP_ID = '200";

    public static void main(String[] args) {
        Connection con = JDBCTemplate.getConnection();
        Statement stmt = null;
        ResultSet rset = null;

        // SELECT * FROM employeedb.EMPLOYEE WHERE EMP_ID = '210' AND EMP_NAME = '' OR 1=1 AND EMP_ID = '200'
        // OR 1=1 에서 true 가 되어 앞부분이 전부 true AND EMP_ID = '200' 으로 조건이 적용되었다.
        String query = "SELECT * FROM employeedb.EMPLOYEE WHERE EMP_ID = '" + empId + "' AND EMP_NAME = '" + empName + "'"; // 200 사원이 출력됨
        System.out.println("Created Query : " + query);

        try {
            stmt = con.createStatement();
            rset = stmt.executeQuery(query);

            if(rset.next()) {
                System.out.println(rset.getString("EMP_NAME") + "님 환영합니다.");
            } else {
                System.out.println("회원 정보가 없습니다.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCTemplate.close(rset);
            JDBCTemplate.close(stmt);
            JDBCTemplate.close(con);
        }
    }
}
```

> **PreparedStatment 는 ? 로 먼저 준비하고 값만 받을 수 있기에 SQL 인젝션 공격을 방어 가능**
```java
package com.ohgiraffers.section03.sqlinjection;

import com.ohgiraffers.common.JDBCTemplate;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.PreparedStatement;

public class Application2 {

    private static String empId = "210";
    private static String empName = "' OR 1=1 AND EMP_ID = '200";

    public static void main(String[] args) {
        Connection con = JDBCTemplate.getConnection();
        PreparedStatement pstmt = null;
        ResultSet rset = null;
        
        String query = "SELECT * FROM employeedb.EMPLOYEE WHERE EMP_ID = ? AND EMP_NAME = ?"; // 회원 정보가 없습니다 출력됨
        System.out.println("Created Query : " + query);

        try {
            pstmt = con.prepareStatement(query);
            pstmt.setString(1, empId);
            pstmt.setString(2, empName);
            rset = pstmt.executeQuery();

            if(rset.next()) {
                System.out.println(rset.getString("EMP_NAME") + "님 환영합니다.");
            } else {
                System.out.println("회원 정보가 없습니다.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCTemplate.close(rset);
            JDBCTemplate.close(pstmt);
            JDBCTemplate.close(con);
        }
    }
}
```

### 객체에 옮겨담기
1. 조회하고 `rset.next()` 로 행마다 돌면서 하나씩 대입하기 : ~~Statement~~, PreparedStatement 사용
2. 자동 매핑 : JPA(간결한 SQL 문을 작성할 시간을 줄임), Mybatis(복잡하거나 세세한 SQL문 작성) 등 사용 → 필요하다면 한 프로젝트에서 양쪽 다 사용 가능

### SQL 을 XML 파일에 분리
- 설정에서 Template 가져와서 사용 : 에디터 → 파일 및 코드 템플릿 → XML Properties File

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <!-- entry : key-value 쌍 -->
    <entry key="selectEmpByFamilyName">
        <!-- 한국 이름이어서 FamilyName 이 앞에 있다 -->
        SELECT E.* FROM employeedb.EMPLOYEE E
        WHERE E.EMP_NAME LIKE CONCAT(?, '%')
    </entry>
</properties>
```

```java
Properties prop = new Properties();
FileInputStream queryStream = new FileInputStream("xml 위치");
prop.loadFromXML(queryStream); // XML 읽어오므로 loadFromXML 사용
String query = prop.getProperty("selectEmpByFamilyName");
```

### INSERT / UPDATE / DELETE
`int result = pstmt.executeUpdate();`  동일하게 사용

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <entry key="insertMenu">
        INSERT INTO TBL_MENU(MENU_NAME, MENU_PRICE, CATEGORY_CODE, ORDERABLE_STATUS)
        VALUES(?, ?, ?, ?)
    </entry>

    <entry key="updateMenu">
        UPDATE TBL_MENU
        SET MENU_NAME = ?
        , MENU_PRICE = ?
        WHERE MENU_CODE = ?
    </entry>

    <entry key="deleteMenu">
        DELETE FROM TBL_MENU
        WHERE MENU_CODE = ?
    </entry>
</properties>
```

- INSERT
    
    ```java
    package com.ohgiraffers.section01.insert;
    
    import com.ohgiraffers.common.JDBCTemplate;
    import com.ohgiraffers.model.dto.MenuDTO;
    
    import java.io.*;
    import java.sql.*;
    import java.util.*;
    
    public class Application2 {
        public static void main(String[] args) {
            Connection con = JDBCTemplate.getConnection();
            PreparedStatement pstmt = null;
            int result = 0;
    
            Scanner sc = new Scanner(System.in);
            System.out.print("메뉴 이름을 입력해주세요 : ");
            String menuName = sc.nextLine();
            System.out.print("메뉴 가격을 입력해주세요 : ");
            int menuPrice = sc.nextInt();
            System.out.print("카테고리 코드을 입력해주세요 : ");
            int categoryCode = sc.nextInt();
            sc.nextLine();
            System.out.print("판매 여부를 결정해주세요(Y/N) : ");
            String orderableStatus = sc.nextLine().toUpperCase();
    
            MenuDTO newMenu = new MenuDTO(null, menuName, menuPrice, categoryCode, orderableStatus);
    
            Properties prop = new Properties();
    
            try(FileInputStream queryStream = new FileInputStream("src/main/java/com/ohgiraffers/mapper/menu-query.xml")) {
                prop.loadFromXML(queryStream);
                String query = prop.getProperty("insertMenu");
    
                pstmt = con.prepareStatement(query);
                pstmt.setString(1, newMenu.getMenuName());
                pstmt.setInt(2, newMenu.getMenuPrice());
                pstmt.setInt(3, newMenu.getCategoryCode());
                pstmt.setString(4, newMenu.getOrderableStatus());
    
                // INSERT 실행
                result = pstmt.executeUpdate();
    
            } catch (IOException | SQLException e) {
                e.printStackTrace();
            } finally {
                JDBCTemplate.close(pstmt);
                JDBCTemplate.close(con);
            }
    
            if(result > 0) System.out.println("메뉴 등록을 성공하셨습니다.╰(*°▽°*)╯");
            else System.out.println("메뉴 등록을 실패하셨습니다.🤨");
        }
    }
    ```
    
- UPDATE
- DELETE


DTO(Data Transfer Object) : 계층들을 옮겨 다니는 객체

## DAO
DAO(Data Access Object) : 데이터베이스 접근용 객체. CRUD 연산을 담당하는 메소드의 집합으로 이루어진 클래스로, SQL 을 실행하고 결과를 java 객체(DTO)로 바꾸는 일을 담당한다.

PreparedStatement 는 하나 당 쿼리문 하나만을 실행 가능하기에 한 곳에서 쿼리 실행이 늘어나면 코드도 길어지고 반납해야 할 자원도 무수히 많아진다.

```java
package com.ohgiraffers.section02.model.dao;

import com.ohgiraffers.section02.model.dto.CategoryDTO;
import com.ohgiraffers.section02.model.dto.MenuDTO;

import java.io.FileInputStream;
import java.io.IOException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

import static com.ohgiraffers.common.JDBCTemplate.close;

/* DAO(Database Access Object) : 데이터베이스 접근용 객체
 * crud 연산을 담당하는 메소드들의 집합으로 이루어진 클래스
 * */
public class MenuDAO {

    private Properties prop = new Properties();

    public MenuDAO() {
        try (FileInputStream queryStream = new FileInputStream("src/main/java/com/ohgiraffers/mapper/menu-query.xml")) {
            prop.loadFromXML(queryStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /* 1. 메뉴 등록에 사용할 하위 카테고리 조회 */
    public List<CategoryDTO> selectAllCategory(Connection con) {

        PreparedStatement pstmt = null;
        ResultSet rset = null;
        List<CategoryDTO> categoryList = new ArrayList<>();
        String query = prop.getProperty("selectAllCategoryList");

        try {

            pstmt = con.prepareStatement(query);
            rset = pstmt.executeQuery();

            while(rset.next()) {
                CategoryDTO categoryDTO = new CategoryDTO();
                categoryDTO.setCategoryCode(rset.getInt("category_code"));
                categoryDTO.setCategoryName(rset.getString("category_name"));
                categoryDTO.setRefCategoryCode(rset.getInt("ref_category_code"));

                categoryList.add(categoryDTO);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            close(rset);
            close(pstmt);
        }

        return categoryList;
    }

    /* 2. 신규 메뉴 등록 */
    public int insertNewMenu(Connection con, MenuDTO newMenu) {

        PreparedStatement pstmt = null;
        int result = 0;
        String query = prop.getProperty("insertMenu");

        try {
            pstmt = con.prepareStatement(query);
            pstmt.setString(1, newMenu.getMenuName());
            pstmt.setInt(2, newMenu.getMenuPrice());
            pstmt.setInt(3, newMenu.getCategoryCode());
            pstmt.setString(4, newMenu.getOrderableStatus());

            result = pstmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            close(pstmt);
        }
        return result;
    }
}
```

- [JDBC](#jdbc)
  - [Statement](#statement)
  - [PreparedStatement](#preparedstatement)
    - [SQL 인젝션](#sql-인젝션)
    - [객체에 옮겨담기](#객체에-옮겨담기)
    - [SQL 을 XML 파일에 분리](#sql-을-xml-파일에-분리)
    - [INSERT / UPDATE / DELETE](#insert--update--delete)
  - [DAO](#dao)
