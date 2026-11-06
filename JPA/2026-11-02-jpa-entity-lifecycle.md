# JPA
JPA (Java Persistence API) : 자바 진영의 ORM (Obejct Relational Mapping) 기술 표준 인터페이스 모음

ORM : 자바 객체와 DB 테이블을 매핑하고 자바 객체 간의 관계를 토대로 SQL 을 생성 및 실행할 수 있다.

## JPA 필요성

### JDBC API 로 직접 SQL 을 다룰 경우 발생할 수 있는 문제점

1. 데이터 변환, SQL 작성, JDBC API 코드 등을 중복 작성 (= 개발시간 증가, 유지보수성 악화)
2. SQL 의존적인 개발
    - SQL 문과 Java 코드를 함께 관리해야 한다. (SQL 문이 바뀌면 관련 Java 코드도 같이 변경해야 함)
3. 패러다임 불일치 문제 (상속, 연관관계, 객체 그래프 탐색) : 테이블 구조와 java 객체 구조가 안 맞음
    - join 시 Java 코드에서 한 행 씩 꺼내어 일부를 객체로 만들어서 따로 넣어야 한다. (ex. Menu 객체 안에 Category 객체 존재) → 개발자가 직접 하나하나 조립
4. 동일성 보장 문제
    - 같은 쿼리문을 2번 실행하여 각각 객체에 넣었을 경우, 동일한 객체가 아니다. → JPA 는 캐시를 사용하여 변경 사항이 없을 경우 기존 정보 반환

### JPA 특징

- 영속성 컨텍스트가 엔티티를 생명주기를 통해 관리
- DBMS 별 dialect 를 제공
- native SQL 로 직접 SQL 작성 가능

### JPA 장점

- 객체지향(Java)과 관계지향(MySQL) 이라는 서로 다른 패러다임 불일치를 해소해 주며 SQL 중심이 아닌 객체지향 패러다임 중심의 개발이 가능하다.
- 개발자가 직접 SQL 을 작성하지 않아도 SQL 문을 작성해 주므로 생산성 향상된다.
- SQL 을 수정할 필요가 없으므로 **설정 및 필드를 변경 시 SQL 이 자동으로 수정**되어 유지보수가 향상된다.
- **DB 의 종류에 따라 JPA 가 자동으로** SQL 을 작성해준다.
- **캐시를 활용한 성능 최적화**로 트랜잭션을 처리하는 시간을 많이 단축해준다.

### JPA 단점

- 복잡한 SQL 을 작성하기에는 적합하지 않다. → 간단한 쿼리는 JPA 로 작성하고 복잡한 쿼리가 필요할 경우 Mybatis 등을 추가하여 같이 사용하는 편이 좋다.
- JPA 를 이해하지 못하고 작성 시 성능 저하가 발생할 수 있다.
- 객체지향 패러다임과 관계형 데이터베이스 패러다임에 대한 이해가 필요하다.

### Mybatis 와 JPA

JPA 는 ORM 기술이고, Mybatis 는 SQL Mapper 의 한 종류이다.

## JPA 설정

- JPA 를 구현하기 위한 의존성
    
    ```java
    implementation 'org.hibernate.orm:hibernate-core:7.2.25.Final'
    ```
    

- DB 연결 정보 및 설정 (resources/META_INF/persistence.xml)
    
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <persistence xmlns="https://jakarta.ee/xml/ns/persistence"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence https://jakarta.ee/xml/ns/persistence/persistence_3_1.xsd"
                 version="3.1">
        <!-- 엔티티 매니저 팩토리를 식별하기 위한 이름 설정 -->
        <persistence-unit name="jpatest">
    
    				<!-- 엔티티 클래스 매핑 -->
            <class>com.ohgiraffers.section02.crud.Menu</class>
            <class>com.ohgiraffers.section03.entity.Menu</class>
    
            <properties>
                <!-- 데이터 베이스 연결 정보 -->
                <property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/menudb"/>
                <property name="jakarta.persistence.jdbc.user" value="ohgiraffers"/>
                <property name="jakarta.persistence.jdbc.password" value="ohgiraffers"/>
                <!-- Hibernate가 실행하는 SQL을 확인하기 위한 설정 -->
    	          <!-- 실행되는 SQL 구문 출력 -->
                <property name="hibernate.show_sql" value="true"/>
                <!-- SQL 포맷을 예쁘게 출력 -->
                <property name="hibernate.format_sql" value="true"/>
            </properties>
        </persistence-unit>
    </persistence>
    ```
    

### EntityManager

엔티티를 저장하는 메모리 상의 데이터베이스. 엔티티 저장, 수정, 삭제, 조회 등 엔티티와 관련된 모든 일을 한다. 동시에 작업할 경우 섞일 수 있기에 **스레드 당 하나씩 생성**한다.

- Entity : DB 테이블과 매핑되는 객체

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;

public class EntityManagerGenerator {

    private EntityManagerGenerator() {}

    public static EntityManager getInstance() {
        EntityManagerFactory factory = EntityManagerFactoryGenerator.getInstance();
        
        // 각 스레드에서 EntityManager 를 요청할 때마다 새로운 EntityManager 를 만들어서 반환
        return factory.createEntityManager();
    }
}
```

```java
@Test
@DisplayName("엔티티 매니저 스코프 확인")
void testEntityManagerLifeCycle() {
    // when
    EntityManager manager1 = EntityManagerGenerator.getInstance();
    EntityManager manager2 = EntityManagerGenerator.getInstance();

    // then
    Assertions.assertNotSame(manager1, manager2);
}
```

### EntityManagerFactory

엔티티 매니저를 생성할 수 있다. 스레드 세이프 하기에 여러 스레드가 동시에 접근해도 되기에 **애플리케이션 당 하나의 EntityManagerFactory** 가 존재한다. 싱글톤 방식으로 구현.

```java
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;

public class EntityManagerFactoryGenerator {

		// 설정에 넣어둔 persistence-unit 이름
    private static final EntityManagerFactory FACTORY
            = Persistence.createEntityManagerFactory("jpatest");

    private EntityManagerFactoryGenerator() {}

		// 일종의 싱글톤 방식으로 외부에서 하나 존재하는 Factory 를 공유해서 사용
    public static EntityManagerFactory getInstance() {
        return FACTORY;
    }
}
```

### PersistenceContext

엔티티를 영구 저장하는 환경. 엔티티 매니저에 엔티티를 저장하거나 조회하면 앤티티 메니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다. key value 방식으로 저장하는 저장소 역할을 한다. **EntityManager가 생성 될 때마다 관리하는 PersistenceContext가 하나 생성**된다.
영속성 컨텍스트에서 발생한 일들은 실제로 DB 에서 일어난 것이 아니다. 그렇기에 변경사항을 동기화 하기 위해 `flush()` 를 해야한다.

## CRUD

find(), persist(), remove(), flush()

- PK로 단일 조회(**SELECT**) : `entityManager.**find(엔티티 클래스 타입, PK 값)**;`
- 저장(**INSERT**) : `entityManager.**persist(엔티티)**;` : 엔티티 객체를 영속성 컨텍스트에 저장
    1. 추가하고 싶은 메뉴를 persist 한다.
    2. `flush()` : 영속성 컨텍스트의 변경 내용을 SQL 로 DB 에 동기화 하라고 요청. commit 을 안했기에 이 시점에서는 DB 에 영구적으로 확정이 되지 않아 롤백이 가능하다.
    3. `clear()` : 영속성 컨텍스트를 비우기
- 수정(**UPDATE**) : 영속성 컨텍스트에서 관리하는 엔티티에 변경사항이 생기면 자동으로 update 를 한다.
    1. 변경하고 싶은 메뉴를 DB 에서 **find 를 이용해 찾아와서 영속성 컨텍스트가 관리**하게 한다. (=영속 상태) (find)
    2. 영속성 컨텍스트에 있는 데이터를 변경한다. (setter)
    3. flush (or commit) → clear
- 삭제(**DELETE**) : `entityManager.**remove(엔티티)**;`
    1. 삭제하고 싶은 메뉴를 DB 에서 **find 를 이용해 찾아와서 영속성 컨텍스트가 관리**하게 한다.
    2. 영속성 컨텍스트에 있는 엔티티를 삭제한다.
    3. flush (or commit) → clear

```java
import jakarta.persistence.EntityManager;

public class EntityManagerCRUD {
    private final EntityManager entityManager;

    public EntityManagerCRUD(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    public Menu findMenuByMenuCode(int menuCode) {
        // find(엔티티 타입, pk)
        return entityManager.find(Menu.class, menuCode);
    }

    public void saveMenu(Menu newMenu) {
        entityManager.persist(newMenu);
    }

    // 영속성 컨텍스트에서 관리하는 엔티티에 변경사항이 생기면 자동으로 update 를 한다.
    public Menu modifyMenuName(int menuCode, String menuName) {
        // 1. 변경하고 싶은 메뉴를 DB 에서 find 를 이용해 찾아와서 영속성 컨텍스트가 관리하게 한다.
        Menu foundMenu = entityManager.find(Menu.class, menuCode);
        // 2. 영속성 컨텍스트에 있는 데이터를 변경한다.
        foundMenu.setMenuName(menuName);
        return foundMenu;
    }

    public void removeMenu(int menuCode) {
        // 1. 삭제하고 싶은 메뉴를 DB 에서 find 를 이용해 찾아와서 영속성 컨텍스트가 관리하게 한다.
        Menu foundMenu = entityManager.find(Menu.class, menuCode);
        if(foundMenu == null) return;
        // 2. 영속성 컨텍스트에 있는 엔티티를 삭제한다.
        entityManager.remove(foundMenu);
    }
}
```

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityTransaction;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

public class EntityManagerCRUDTests {
    private EntityManager entityManager;
    private EntityTransaction transaction;
    private EntityManagerCRUD crud;

    // 테스트는 기본적으로 단일 스레드로 순차적으로 진행되기에 각 테스트 마다 생성한 객체를 한 곳에 넣어도 문제 없다.
    @BeforeEach
    void setUp() {
        entityManager = EntityManagerGenerator.getInstance();
        transaction = entityManager.getTransaction();
        transaction.begin();
        crud = new EntityManagerCRUD(entityManager);
    }

    @AfterEach
    void tearDown() {
        if(transaction != null && transaction.isActive()) {
            transaction.rollback();
        }
        if(entityManager != null && entityManager.isOpen()) {
            entityManager.close();
        }
    }

    @Test
    @DisplayName("메뉴 코드로 메뉴 조회")
    void testFindMenuByMenuCode() {
        // when
        Menu foundMenu = crud.findMenuByMenuCode(1);

        // then
        assertNotNull(foundMenu);
        assertEquals(1, foundMenu.getMenuCode());
        System.out.println("foundMenu : " + foundMenu);
    }

    @Test
    @DisplayName("새로운 메뉴 저장")
    void testSaveMenu() {
        // given
        Menu newMenu = new Menu("신메뉴", 35000, 4, "Y");

        // when
        crud.saveMenu(newMenu);
        // flush 만으로는 commit 까지 되지 않았기에 DB 에 영구적으로 확정이 되지 않아 롤백 가능
        entityManager.flush();  // 영속성 컨텍스트에 쌓인 변경 내용을 SQL 로 DB 에 동기화하라고 요청
        int newMenuCode = newMenu.getMenuCode();
        entityManager.clear();  // 영속성 컨텍스트 비우기

        // then
        Menu savedMenu = crud.findMenuByMenuCode(newMenuCode);
        assertNotNull(savedMenu);
        assertEquals("신메뉴", savedMenu.getMenuName());
    }

    @Test
    @DisplayName("영속 상태 메뉴의 이름 변경")
    void testModifyMenuName() {
        // when
        crud.modifyMenuName(1, "변경된 이름");
        entityManager.flush();
        entityManager.clear();

        // then
        Menu modifyMenu = crud.findMenuByMenuCode(1);
        assertEquals("변경된 이름", modifyMenu.getMenuName());
    }

    @Test
    @DisplayName("메뉴 삭제")
    void testRemoveMenu() {
        // when
        crud.removeMenu(1);
        entityManager.flush();
        entityManager.clear();

        // then
        assertNull(crud.findMenuByMenuCode(1));
    }
}
```

## Entity 생명주기

detach(), merge(), clear(), close()

### Entity 상태 (=생명주기)

- **비영속** - 만들어졌지만 아직 JPA 과 관리하지 않는 객체
- **영속** - 현재 영속성 컨텍스트가 관리하는 객체. 이 상태에서 엔티티를 수정하면 자동으로 데이터베이스에 반영된다.
    - `find(엔티티타입, pk값)` 으로 찾아 관리한다.
- **준영속** - 한때 관리됐지만 현재는 분리된 객체. 엔티티를 수정해도 데이터베이스에 반영되지 않는다.
    - `detach(엔티티);` : 준영속화
        - **준영속인 상태에서는 관리하지 않기 때문에** 이 상태에서 다시 find 하면 DB 에서 새로 찾아온다.
    - `merge(준영속화 되어 있는 엔티티);` : 영속화. 관리될 엔티티에 준영속화 엔티티로 덮어씌워지면서 병합된다. **merge 는 전달한 객체를 다시 관리하지 않고 관리되는 객체를 반환**한다.
    - `clear()` : 내가 관리 중인 엔티티를 모두 준영속 상태로 만든다.
- **삭제** - 영속성 컨텍스트에서 삭제 대상으로 지정된 객체. 엔티티 객체가 영속성 컨텍스트에서 제거 되어 있기 때문에 이 상태에서는 엔티티를 수정해도, 데이터베이스에는 반영되지 않는다.

```java
@Test
@DisplayName("비영속 객체와 영속 객체 비교")
void testTransientAndManaged() {
    // when
    Menu managedMenu = lifeCycle.findMenuByMenuCode(2);
    Menu transientMenu = new Menu(managedMenu.getMenuCode(), managedMenu.getMenuName()
            , managedMenu.getMenuPrice(), managedMenu.getCategoryCode(), managedMenu.getOrderableStatus());

    // then
    assertTrue(lifeCycle.isManaged(managedMenu));
    assertFalse(lifeCycle.isManaged(transientMenu));
}
```

```java
@Test
@DisplayName("준영속에서 merge가 반환한 새로운 영속 객체 확인")
void testMergeEntity() {
    // given
    Menu detachedMenu = entityManager.find(Menu.class, 10);
    entityManager.detach(detachedMenu); // 준영속화
    // 준영속화 상태에서 이름을 바꿨기에 영속 컨텍스트에는 변경 사항이 등록되어 있지 않다
    detachedMenu.setMenuName("병합된 메뉴 이름");

    // when
    // 영속화 되면서 준영속화 됐던 객체가 덮어씌우는 식으로 병합된다.
    Menu mergedMenu = entityManager.merge(detachedMenu);    // 다시 영속화
    // merge 는 전달한 객체를 다시 관리하지 않고 관리되는 객체를 반환한다.
    entityManager.flush();

    // then
    assertFalse(entityManager.contains(detachedMenu));
    assertTrue(entityManager.contains(mergedMenu));
    assertNotSame(detachedMenu, mergedMenu);
    assertEquals("병합된 메뉴 이름", mergedMenu.getMenuName());
}
```

### Entity 동일성 보장

- 각자 다른 EntityManager 가 관리하는 Entity 는 똑같이 find 를 해도 같은 객체가 아니다.
- 영속성 콘텍스트는 캐싱 기능이 있기에 같은 EntityManager 가 같은 find 를 하면 캐시 반환

```java
@Test
@DisplayName("서로 다른 EntityManager 가 조회한 엔티티 비교")
void testManagedByDifferentEntityManagers() {
    // given
    EntityManager otherEntityManager = EntityManagerGenerator.getInstance();

    // when
    Menu menu1 = entityManager.find(Menu.class, 3);
    Menu menu2 = otherEntityManager.find(Menu.class, 3);

    // then
    assertNotSame(menu1, menu2);
}

@Test
@DisplayName("같은 EntityManager 에서 동일한 엔티티 조회")
void testManagedBySameEntityManagerㅁs() {
    // when
    Menu menu1 = entityManager.find(Menu.class, 3);
    Menu menu2 = entityManager.find(Menu.class, 3);

    // then
    assertSame(menu1, menu2);
}
```

**1차 캐시** : 영속성 컨텍스트 내부에 Map 으로 관리되는 캐시로 조회 성능도 올라가고, 변경사항을 1차 캐시와 비교하여 감지한다. 영속 상태의 엔티티에만 적용된다.

- [JPA](#jpa)
  - [JPA 필요성](#jpa-필요성)
    - [JDBC API 로 직접 SQL 을 다룰 경우 발생할 수 있는 문제점](#jdbc-api-로-직접-sql-을-다룰-경우-발생할-수-있는-문제점)
    - [JPA 특징](#jpa-특징)
    - [JPA 장점](#jpa-장점)
    - [JPA 단점](#jpa-단점)
    - [Mybatis 와 JPA](#mybatis-와-jpa)
  - [JPA 설정](#jpa-설정)
    - [EntityManager](#entitymanager)
    - [EntityManagerFactory](#entitymanagerfactory)
    - [PersistenceContext](#persistencecontext)
  - [CRUD](#crud)
  - [Entity 생명주기](#entity-생명주기)
    - [Entity 상태 (=생명주기)](#entity-상태-생명주기)
    - [Entity 동일성 보장](#entity-동일성-보장)
