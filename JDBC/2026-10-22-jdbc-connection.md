# JDBC
Java Database Connectivity  
Java 에서 Database 에 접근 가능하도록 하는 Programming API  

Driver : JDBC 인터페이스 구현체  
DriverManager : Database 에 JDBC driver 를 통하여 Connection 을 만들어 반환  
Connection : 자바 어플리케이션과 DB 에 만들어지는 연결 통로

## 설정
1. 시스템 빌드 : Gradle, Gradle DSL : Groovy 로 프로젝트 작성
   - Gradle 프로젝트 작성 시 한글 깨짐 현상 : https://kangwlgns.tistory.com/486
   - 시스템 로캘 세계적인 UTF-8 로 변경 (Beta 기능)
2. build.gradle 에 의존성 추가
   1. build.gradle 파일의 dependencies 에 라이브러리를 복사한다.
   2. 코끼리 모양 아이콘을 눌러 적용  
    
    라이브러리 찾기 : https://mvnrepository.com/
    ```
    // Source: https://mvnrepository.com/artifact/com.mysql/mysql-connector-j
    implementation 'com.mysql:mysql-connector-j:9.7.0'

    // MySQL 8.4 이상을 사용하려면 아래 버전을 추가해야한다.
    implementation 'com.mysql:mysql-connector-j:26.7.0'
    ```

## Connection
### 설정 파일로 분리

java 코드가 바뀌면 빌드를 다시 해야하기에 바뀔 수 있는 설정 값은 properties 로 관리한다.

```java
// jdbc-config.properties

driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost/menudb
user=ohgiraffers
password=ohgiraffers
```

### 연결 함수 템플릿으로 분리

1. `Class.forName`  : 사용할 JDBC Driver 클래스 로딩
    - `Class.forName("com.mysql.cj.jdbc.Driver");` MySQL 드라이버를 사용하겠다고 선언
2. `DriverManager.getConnection(url, user, password)`  : 접근할 DB 주소와 user, 비밀번호 입력
    - `java.sql.Connection con = DriverManager.getConnection("jdbc:mysql://localhost/menudb", "ohgiraffers", "ohgiraffers");`

```java
package com.ohgiraffers.common;

import java.io.FileReader;
import java.io.IOException;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Properties;

public class JDBCTemplate {
    public static Connection getConnection() {
        Connection con = null;
        Properties prop = new Properties();

				// 설정 정보 담긴 프로퍼티 파일 읽기
        try(FileReader reader = new FileReader("src/main/java/com/ohgiraffers/config/jdbc-config.properties")) {
            prop.load(reader);

            String driver = prop.getProperty("driver");
            String url = prop.getProperty("url");
            String user = prop.getProperty("user");
            String password = prop.getProperty("password");

            Class.forName(driver);
            con = DriverManager.getConnection(url, user, password);

            System.out.println("Success create Connection! " + con);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        return con;
    }

    public static void close(Connection con) {
        try {
            if(con != null && !con.isClosed()) {
                con.close();
                System.out.println("Success Connection closed.");
            } else {
                System.out.println("Already Connection closed.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void close(ResultSet rset) {
        try {
            if(rset != null && !rset.isClosed()) {
                rset.close();
                System.out.println("Success ResultSet closed.");
            } else {
                System.out.println("Already ResultSet closed.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void close(Statement stmt) {
        try {
            if(stmt != null && !stmt.isClosed()) {
                stmt.close();
                System.out.println("Success Statement closed.");
            } else {
                System.out.println("Already Statement closed.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

- [JDBC](#jdbc)
  - [설정](#설정)
  - [Connection](#connection)
    - [설정 파일로 분리](#설정-파일로-분리)
    - [연결 함수 템플릿으로 분리](#연결-함수-템플릿으로-분리)
