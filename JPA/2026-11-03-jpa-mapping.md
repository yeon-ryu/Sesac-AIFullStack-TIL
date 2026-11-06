# JPA
## Mapping

#### application.yml

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/menudb
    username: ohgiraffers
    password: ohgiraffers
  jpa:
    show-sql: true # SQL 문을 보여준다.
    open-in-view: false # Controller 까지 영속성 컨텍스트를 열어두지 않겠다.
    database: mysql
    hibernate:
      ddl-auto: none # 없을 경우 자동으로 테이블 생성 여부
      naming:
        physical-strategy: org.hibernate.boot.model.naming.CamelCaseToUnderscoresNamingStrategy # Camel 과 UnderScore 로 자동으로 매핑되도록 설정
    properties:
      hibernate:
        format_sql: true # SQL 문을 예쁘게 보여준다.
```

### 객체 매핑 (Entity)

- `@Entity` : 해당 클래스를 JPA 가 관리할 대상으로 등록. 같은 이름의 Entity 가 존재할 경우 name 속성으로 엔티티 이름을 붙여줘야 함.
- `@Table(name=”테이블명”)` : 연결할 DB 테이블을 지정. 테이블이 없을 경우 테이블 생성도 해줌
- `@Id` : 필드 중 하나에는 반드시 PK(식별자)를 지정해줘야 함 (jakarta.persistence 에 존재)
- `@GeneratedValue(strategy = GenerationType.전략)` : 해당 멤버 생성할 때 전략
- `@Column(name = “컬럼명”)` : 필드와 컬럼 연결. 작성 방식에 따라 설정에 지정하고 생략 가능
    - `@Column(name = “컬럼명”, nullable = false, unique = true, length = 50)` 같은 식으로 제약 조건 설정 (length 는 최대 길이, nullable 은 디폴트 true)
- `@Enumerated(EnumType.STRING)` : enum 을 문자열로 저장한다 (기본적으로는 상수 번호 (ORDINAL))
- JPA 는 리플렉션 방식으로 값을 주입하기 때문에 setter 가 없어도 값이 들어간다.

```java
import jakarta.persistence.*;
import java.time.LocalDateTime;

// @Entity 는 이 클래스를 JPA 가 관리할 대상으로 등록, @Table 은 연결한 DB 테이블을 지정 (없으면 새로 생성도 해줌)
@Entity
@Table(name="tbl_member")
public class Member {

    // @Id 는 기본 키 필드이고, IDENTITY 방식에서는 INSERT 할 때 DB 가 회원 번호를 생성(auto-increment)
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_no")
    private int memberNo;
    @Column(name = "member_id", nullable = false, unique = true, length = 50)
    private String memberId;
    @Column(name = "member_pwd", nullable = false, length = 100)
    private String memberPwd;
    @Column(name = "member_name", nullable = false)
    private String memberName;
    @Column(name = "phone")
    private String phone;
    @Column(name = "address")
    private String address;
    @Column(name = "endroll_date", nullable = false)
    private LocalDateTime enrollDate;
    // enum 을 문자열로 저장한다 (기본적으로는 상수 번호 (ORDINAL)) -> enum 순서가 바뀌어도 DB 값의 의미가 유지됨
    @Enumerated(EnumType.STRING)
    @Column(name = "member_role", nullable = false)
    private MemberRole memberRole;  // enum 으로 memberRole 설정
    @Column(name = "status", nullable = false, length = 1)
    private String status;  // Y/N

    // JPA 가 조회 결과로 엔티티를 만들 때 기본 생성자가 필요하기에 protected 로 선언
    protected Member() {}

    // 개발자가 회원을 새로 만들 때
    public Member(
            String memberId, String memberPwd, String memberName,
            String phone, String address, LocalDateTime enrollDate,
            MemberRole memberRole, String status
    ) {
        this.memberId = memberId;
        this.memberPwd = memberPwd;
        this.memberName = memberName;
        this.phone = phone;
        this.address = address;
        this.enrollDate = enrollDate;
        this.memberRole = memberRole;
        this.status = status;
    }
}
```

- `@Embeddable` : 별도 테이블 없이 **소유 엔티티의 컬럼에 포함**되는 값 객체
- `@Embedded` : 해당 값 객체를 포함할 엔티티에서 컬럼에 붙인다.

```java
import jakarta.persistence.Column;
import jakarta.persistence.Embeddable;

// 별도 테이블 없이 소유 엔티티의 컬럼에 포함되는 값 객체
// 이 클래스의 필드들을 엔티티 내부에 포함해 매핑할 수 있다.
@Embeddable
public class Price {
    @Column(name="regular_price", nullable = false)
    private int regularPrice;
    @Column(name="discount_price", nullable = false)
    private double discountRate;
    @Column(name="sell_price", nullable = false)
    private int sellPrice;

    protected Price() {}

    public Price(int regularPrice, double discountRate) {
        validatePrice(regularPrice);
        validateDiscountRate(discountRate);
        this.regularPrice = regularPrice;
        this.discountRate = discountRate;
        this.sellPrice = calculateSellPrice(regularPrice, discountRate);
    }

    private void validatePrice(int regularPrice) {
        if (regularPrice < 0) {
            throw new IllegalArgumentException("가격은 음수일 수 없습니다.");
        }
    }

    private void validateDiscountRate(double discountRate) {
        if (discountRate < 0 || discountRate > 1) {
            throw new IllegalArgumentException("할인율은 0 이상 1 이하여야 합니다.");
        }
    }

    private int calculateSellPrice(int regularPrice, double discountRate) {
        return (int) (regularPrice - (regularPrice * discountRate));
    }
}
```

```java
import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
@Table(name="tbl_book")
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="book_no")
    private int bookNo;
    @Column(name="book_title", nullable = false, length = 100)
    private String bookTitle;
    @Column(name="author")
    private String author;
    @Column(name="publisher")
    private String publisher;
    @Column(name="published_date")
    private LocalDate publishedDate;
    // Price 의 값들을 Book 테이블의 컬럼으로 함께 저장 (Price : 값 객체) (값만 묶어서 따로 관리)
    @Embedded
    private Price price;

    protected Book() {}

    public Book(
            String bookTitle, String author, String publisher,
            LocalDate publishedDate, Price price
    ) {
        this.bookTitle = bookTitle;
        this.author = author;
        this.publisher = publisher;
        this.publishedDate = publishedDate;
        this.price = price;
    }
}
```

### 사용 (Repository, Service)

- `@PersistenceContext` : Spring 이 관리하고 있는 EntityManager 를 주입 받는다.
- `@Transactional` (springframework 소속) : 이미 진행 중인 트랜잭션이 없다면 트랜잭션을 시작한다. 진행 중 오류가 발생하면 자동으로 롤백하고 성공 시 커밋.
- Entity 는 DB 에 바로 영향을 줄 수 있기에 DB 에 결정하는 타이밍 외에는 DTO 를 사용

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import org.springframework.stereotype.Repository;

@Repository
public class MemberRepository {
    @PersistenceContext // Spring 이 관리하고 있는 EntityManager 를 주입 받는다.
    private EntityManager entityManager;

    public Member save(Member member) {
        entityManager.persist(member);
        return member;
    }
}
```

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class MemberRegistService {
    private final MemberRepository memberRepository;

    public MemberRegistService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    // Entity 자체를 옮겨다니면 안된다! <- 도중 변경 시 DB 에 영향을 줌
    @Transactional
    public Member registMember(MemberRegistDTO newMember) {
        Member member = new Member(newMember.getMemberId(), newMember.getMemberPwd()
                , newMember.getMemberName(), newMember.getPhone(), newMember.getAddress()
                , newMember.getEnrollDate(), newMember.getMemberRole(), newMember.getStatus());

        return memberRepository.save(member);
    }
}
```

### 연관 관계 매핑

데이터베이스 외래키를 객체로 참조한다.

공부용 프로젝트 10_jpa/chap03-association-mapping 참고.

- Entity 에서 Entity 를 필드로 가지고 있을 경우 카디널리티 연관 관계에 따라, `@ManyToOne` 등의 어노테이션을 작성해줘야 한다.
- `@JoinColumn(name="컬럼명")` : join 할 조건이 될 컬럼명
    - `nullable = false` 속성을 넣으면 반드시 해당 엔티티에 존재하는 외래키만 사용하란 의미

#### 단방향

메뉴에서 카테고리 참조 (N : 1 관계)

- `ManyToOne(fetch = FetchType.LAZY)`
    - fetch 속성 : 연결된 카테고리 조회를 실제 사용할 때까지 미룬다. 디폴트는 EAGER 로 join 을 하고, LAZY 일 때는 우선 자기 것을 select 하고 실제 사용할 때 해당 테이블만 select 해서 가져온다.

```java
import jakarta.persistence.*;

@Entity(name = "Section01Menu")
@Table(name = "tbl_menu")
public class Menu {

    @Id
    @Column(name = "menu_code")
    private int menuCode;

    @Column(name = "menu_name")
    private String menuName;

    @Column(name = "menu_price")
    private int menuPrice;

    @ManyToOne
    @JoinColumn(name="category_code", nullable = false)
    private Category category;

    @Column(name = "orderable_status")
    private String orderableStatus;

    public Menu() {
    }

    public Menu(int menuCode, String menuName, int menuPrice, Category category, String orderableStatus) {
        this.menuCode = menuCode;
        this.menuName = menuName;
        this.menuPrice = menuPrice;
        this.category = category;
        this.orderableStatus = orderableStatus;
    }
}
```

```java
// 읽기 전용
@Transactional(readOnly = true)
public Menu findMenu(int menuCode) {
    return manyToOneRepository.find(menuCode);
}
```

카테고리에서 메뉴 참조 (1 : N 관계)

- `@OneToMany` : 1 대 다 관계로 리스트로 가져온다. (fetch 속성 디폴트가 LAZY)
- `@JoinColumn(name="category_code", insertable = false, updatable = false)` : 이 목록으로 insert나 update가 되지 않도록 한다 (안전을 위해)

```java
@Entity(name = "Section02Category")
@Table(name = "tbl_category")
public class Category {

    @Id
    @Column(name = "category_code")
    private int categoryCode;

    @Column(name = "category_name")
    private String categoryName;

    @Column(name = "ref_category_code")
    private Integer refCategoryCode;

		/* @OneToMany 는 기본이 LAZY 이며, 실제 사용할 때 조회한다.
    * @ManyToOne 은 기본이 EAGER 이며, 처음부터 조인해서 조회한다. */
    @OneToMany
    @JoinColumn(name="category_code", insertable = false, updatable = false)    // 이 목록으로 insert 나 update 가 되지 않도록 한다 (안전을 위해)
    private List<Menu> menuList;
    
    // 이하 생성자
}
```

실무에서는 **연관 관계는 대부분 LAZY 방식으로 조회**해 온다. 필요할 경우만 EAGER 를 사용한다.
다만 LAZY 에서는 N+1 문제가 발생할 수 있다.

#### 양방향

단방향을 서로 참조하여 양방향처럼 설정하여 사용한다.

연관 관계의 주인 : fk 를 가진 쪽 (Menu)

- Menu Entity
연관 관계의 주인은 원래 하던 방식으로 작성하면 된다.
    
    ```java
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_code", nullable = false)
    private Category category;
    ```
    
- Category Entity
연관 관계의 주인이 아니면 mappedBy 에 주인에서 관리하는 필드명을 작성하여 해당 관계를 뒤집는 식으로 동작
    
    ```java
    // mappedBy 에는 DB 컬럼명이 아니라 반대편 Menu 에 존재하는 연관관계 필드명 category 를 적는다.
    @OneToMany(mappedBy = "category")
    private List<Menu> menuList = new ArrayList<>();
    ```
    

Category Entity 에 Menu 를 추가하는 메소드 추가

```java
// 연관 관계 편의 메서드
public void addMenu(Menu menu) {
    // Menu 와 Category 양쪽 다 맞춰줘야 한다.
    menuList.add(menu); // 현재 카테고리의 메뉴 목록에 전달받은 메뉴 목록 추가
    menu.setCategory(this); // 전달받은 메뉴에도 현재 카테고리를 설정
}
```

- [JPA](#jpa)
  - [Mapping](#mapping)
      - [application.yml](#applicationyml)
    - [객체 매핑 (Entity)](#객체-매핑-entity)
    - [사용 (Repository, Service)](#사용-repository-service)
    - [연관 관계 매핑](#연관-관계-매핑)
      - [단방향](#단방향)
      - [양방향](#양방향)
