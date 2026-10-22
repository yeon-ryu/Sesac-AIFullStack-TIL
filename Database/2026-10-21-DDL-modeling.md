# Database
## DDL

Data Definition Language : 데이터 정의 언어 (CREATE, ALTER, DROP)

**DDL 은 롤백 불가**

### CREATE

`CREATE TABLE 테이블명 (컬럼명 데이터타입(길이), 컬럼명 데이터타입(길이)) ENGINE = ...;`

테이블 정보 출력 : `DESCRIBE 테이블명;`

pk 키 지정 및 AI 추가 : `칼럼명 INT AUTO_INCREMENT PRIMARY KEY`

```sql
# DDL (Data Definition Language)
-- 데이터 정의 언어 (CREATE, ALTER, DROP)

# CREATE
-- 테이블 생성 구문

-- IF NOT EXISTS : 존재하지 않을 때만 생성하기 위한 구문
CREATE TABLE IF NOT EXISTS tb1(
	pk INT,
    fk INT,
    col1 VARCHAR(255)
) ENGINE=INNODB; -- 테이블이 사용할 저장 엔진. INNODB : MySQL 대표 엔진

-- 테이블 정보 출력
DESCRIBE tb1;

INSERT INTO tb1 VALUES(1, 10, 'Y');
SELECT * FROM tb1;

-- AUTO_INCREMENT
-- INSERT 시 PRIMARY KEY 에 해당하는 컬럼에 자동으로 번호를 생성시켜 저장할 수 있다.

-- tb2 생성
CREATE TABLE IF NOT EXISTS tb2(
	pk INT AUTO_INCREMENT PRIMARY KEY,
    fk INT,
    col1 VARCHAR(255)
) ENGINE = INNODB;

DESCRIBE tb2;

INSERT INTO TB2 VALUES(NULL, 10, 'Y'), (NULL, 20, 'N');
SELECT * FROM TB2;
```

### ALTER

테이블에 추가/변경/수정/삭제하는 모든 것

- 컬럼 추가 : `ALTER TABLE 테이블명 ADD 컬럼명 타입-같은-정의;`
- 컬럼 삭제 : `ALTER TABLE 테이블명 DROP COLUMN 컬럼명;`
- 컬럼 수정 : `ALTER TABLE 테이블명 CHANGE COLUMN 변경할컬럼명 변경한컬럼명 타입-같은-정의;`
- 컬럼 수정(컬럼명 유지) : `ALTER TABLE 테이블명 MODIFY 컬럼명 타입-같은-정의;`

```sql
# ALTER
-- 테이블에 추가/변경/수정/삭제하는 모든 것은 ALTER 명령어를 사용해 적용한다.
-- ADD : 컬럼 추가
-- DROP COLUMN : 컬럼 삭제
-- CHANGE COLUMN : 이름을 포함한 정의 변경
-- MODIFY : 이름을 유지한 정의 변경

-- 컬럼 추가
ALTER TABLE tb2 ADD col2 INT;
DESCRIBE tb2;
SELECT * FROM tb2;

-- 컬럼 삭제
ALTER TABLE TB2 DROP COLUMN col2;

-- 컬럼 수정
ALTER TABLE TB2 CHANGE COLUMN fk change_fk BIGINT;

-- 컬럼명만 수정
ALTER TABLE tb2 MODIFY change_fk INT;
```

### DROP

`DROP TABLE 테이블명;`

테이블 구조와 데이터를 영구적으로 삭제

```sql
# DROP
-- 테이블 구조와 데이터를 영구적으로 삭제
CREATE TABLE IF NOT EXISTS tb3 (
	pk INT AUTO_INCREMENT PRIMARY KEY,
    fk INT,
    col1 VARCHAR(255)
) ENGINE = INNODB;

DROP TABLE IF EXISTS tb3;
```

### TRUNCATE

`TRUNCATE TABLE 테이블명;`

테이블 내용물만 비우고 초기화(구조는 남겨두고, 데이터만 비우고 싶을 때 사용)

**완전 초기화이기 때문에 데이터를 다시 넣으면 pk 가 1부터 시작**

```sql
# TRUNCATE
-- 테이블 내용물만 비우고 초기화(구조는 남겨두고, 데이터만 비우고 싶을 때 사용)
CREATE TABLE IF NOT EXISTS tb4 (
	pk INT AUTO_INCREMENT PRIMARY KEY,
    fk INT,
    col1 VARCHAR(255)
) ENGINE = INNODB;

INSERT INTO TB4 VALUES (NULL, 10, 'Y'), (NULL, 20, 'N');
SELECT * FROM TB4;

TRUNCATE TABLE TB4;
INSERT INTO TB4 VALUES (NULL, 10, 'Y'), (NULL, 20, 'N');
SELECT * FROM TB4; -- 완전히 초기화 되었기 때문에 PK 도 1부터 다시 시작
```

## CONSTRAINTS (제약 조건)

### NOT NULL

NULL 을 INSERT, UPDATE 를 하려할 때 에러코드 1048 로 허용하지 않는다.

```sql
# NOT NULL
-- NULL 값을 허용하지 않음
CREATE TABLE IF NOT EXISTS user_notnull (
	user_no INT NOT NULL,
    user_id VARCHAR(255) NOT NULL,
    user_pwd VARCHAR(255) NOT NULL,
    gender CHAR(1)
) ENGINE = INNODB;

DESCRIBE user_notnull;

INSERT INTO user_notnull VALUES(1, 'user01', 'pass01', '남');

SELECT * FROM USER_NOTNULL;

-- INSERT INTO user_notnull VALUES(2, 'user02', NULL, '여');
INSERT INTO user_notnull VALUES(2, 'user02', 'pass02', NULL);
```

### UNIQUE

중복값을 허용하지 않음

duplicate 됐을 때 에러 코드 1062 로 허용하지 않는다.

제일 처음 NOT NULL UNIQUE 를 적용한 컬럼은 PK 키가 된다.

```sql
# UNIQUE
-- 중복값 허용하지 않음
CREATE TABLE IF NOT EXISTS user_unique (
	user_no INT NOT NULL UNIQUE,	-- 컬럼 레벨 방식으로 제약 조건 정의
    user_id VARCHAR(255) NOT NULL,
    user_pwd VARCHAR(255) NOT NULL,
    gender CHAR(1),
    UNIQUE(user_id)	-- 테이블 레벨 방식으로 제약 조건 정의
) ENGINE = INNODB;

DESCRIBE user_unique;

INSERT INTO user_unique VALUES(1, 'user01', 'pass01', '남');
-- INSERT INTO user_unique VALUES(1, 'user02', 'pass02', '여'); -- Duplicate 에러
INSERT INTO user_unique VALUES(3, 'user03', 'pass01', '남');

SELECT * FROM user_unique;
```

### PRIMARY KEY

NOT NULL 과 UNIQE 의 성질을 가진 한 테이블에 하나만 가지는 고유 식별자

```sql
# PRIMARY KEY
-- 테이블 한 행의 정보를 찾기 위해 사용할 컬럼을 의미
-- 테이블에 대한 식별자 역할을 한다. (한 행씩 구분하는 역할)
-- NOT NULL + UNIQUE 제약 조건의 의미

CREATE TABLE IF NOT EXISTS user_primarykey (
	user_no INT,
    user_id VARCHAR(255) NOT NULL,
    user_pwd VARCHAR(255) NOT NULL,
    gender CHAR(1),
    PRIMARY KEY(user_no)
) ENGINE = INNODB;

DESCRIBE user_primarykey;

INSERT INTO user_primarykey VALUES(1, 'user01', 'pass01', '남');
-- INSERT INTO user_primarykey VALUES(1, 'user02', 'pass02', '여'); -- 마찬가지로 Duplicate 에러
INSERT INTO user_primarykey VALUES(2, 'user02', 'pass02', '여');

SELECT * FROM user_primarykey;
```

### FOREIGN KEY

외래키

자식 테이블의 값이 부모 테이블에 실제로 존재하도록 관계를 지키게 하기 위해 사용

`FOREIGN KEY (컬럼명) REFERENCES 부모테이블명(컬럼명)`

```
개인적 고려사항 : 회원이 주문을 한다 쳤을 경우, 나중에 회원이 탈퇴하더라도 주문 기록은 남아있어야 할 수 있다. 이럴 경우 외래키로 연결 되어 있으면 SET NULL 도 CASCADE 도 문제가 됨. 실제로 회원을 삭제 안하고 boolean 값으로 표기한다 치더라도 탈퇴한 회원 정보는 일정 시간이 지나면 삭제해야 함(보안 문제)
```

없는 컬럼을 넣을 경우 에러 코드 1452 발생

default 로 설정해놓은 값이 외래키쪽에서 사라졌을 경우 default 로 설정해놨지만 외래키가 없기에 해당 값 생략하고 insert 불가

```sql
# FOREIGN KEY
-- 두 테이블을 연결하고 관계를 맺어준다.
-- 참조된 다른 테이블에서 제공하는 값만 사용할 수 있다.
-- 제공되는 값 외에는 NULL 을 사용할 수 있다.

CREATE TABLE IF NOT EXISTS user_grade(
	grade_code INT PRIMARY KEY,
    grade_name VARCHAR(255) NOT NULL
) ENGINE=INNODB;

INSERT INTO user_grade VALUES(10, '일반 회원'), (20, '우수 회원'), (30, '특별 회원');

CREATE TABLE IF NOT EXISTS user_foreignkey1 (
	user_no INT PRIMARY KEY,
    grade_code INT NOT NULL DEFAULT(10),
    -- 이 테이블의 grade_code 는 user_grade 테이블의 grade_code 를 참조한다.
    FOREIGN KEY (grade_code) REFERENCES user_grade(grade_code)
) ENGINE=INNODB;

describe user_foreignkey1;

INSERT INTO user_foreignkey1 VALUES(1, 10);
-- INSERT INTO user_foreignkey1 VALUES(2, 50);	-- 외래키 제약 조건으로 에러 발생
INSERT INTO user_foreignkey1 VALUES(3, 30);

-- DELETE FROM user_grade WHERE grade_code = 10;  -- 별도의 설정이 없다면 사용중인 부모 컬럼의 수정/삭제가 허용되지 않는다.
DELETE FROM user_foreignkey1 WHERE grade_code = 10;
DELETE FROM user_grade WHERE grade_code = 10;

-- default 로 설정해놓은 값이 외래키쪽에서 사라졌을 경우 default 로 설정해놨지만 외래키가 없기에 insert 불가
INSERT INTO user_foreignkey1(user_no) VALUES(4);

INSERT INTO user_grade VALUES(10, '일반 회원');
INSERT INTO user_foreignkey1(user_no) VALUES(5);

select * from user_foreignkey1;
```

#### ON UPDATE / ON DELETE 옵션

- SET NULL : 부모(user_grade) 가 바뀌거나 사라지면, 해당 값을 NULL 로 바꾼다.
- CASCADE : 부모가 바뀌면 자식도 따라 바뀌고, 부모가 사라지면 자식도 함께 사라진다.

```sql
# ON UPDATE / ON DELETE 옵션
-- SET NULL : 부모(user_grade) 가 바뀌거나 사라지면, 해당 값을 NULL 로 바꾼다.
-- CASCADE : 부모가 바뀌면 자식도 따라 바뀌고, 부모가 사라지면 자식도 함께 사라진다.
CREATE TABLE IF NOT EXISTS user_foreignkey2 (
	user_no INT PRIMARY KEY,
    grade_code INT,
    FOREIGN KEY (grade_code) REFERENCES user_grade(grade_code)
    ON UPDATE CASCADE	-- 부모가 업데이트 됐을 때 함께 바뀌고 사라진다.
    ON DELETE SET NULL	-- 부모가 삭제 됐을 때 관련 값이 NULL 이 된다.
) ENGINE=INNODB;

INSERT INTO user_foreignkey2 VALUES(1, 10), (2, 20), (3, 30);

-- 부모 테이블의 grade_code 수정
UPDATE user_grade SET grade_code = 40 WHERE grade_code = 10;

-- 부모 테이블의 grade_code 삭제
DELETE FROM user_grade WHERE grade_code = 20;

SELECT a.*, b.grade_name FROM user_foreignkey2 a LEFT JOIN user_grade b ON a.grade_code = b.grade_code;
```

### CHECK

들어올 수 있는 값의 범위나 조건을 직접 지정 (INSERT / UPDATE 시 확인)

`CHECK (조건)`

제약조건 위반 시 에러 코드 3819 발생

```sql
# CHECK
-- 들어올 수 있느 값의 범위나 조건을 직접 지정
CREATE TABLE IF NOT EXISTS user_check (
	user_no INT AUTO_INCREMENT PRIMARY KEY,
    user_name VARCHAR(255) NOT NULL,
    gender VARCHAR(3) CHECK (gender IN ('남', '여', 'T-남', 'T-여')),
    age INT CHECK (age >= 19)
) ENGINE=INNODB;

INSERT INTO user_check VALUES(NULL, '홍길동', '남', 25);
-- INSERT INTO user_check VALUES(NULL, '판다', '여', 5);	-- user_check_chk_2 가 제약에 걸림 에러
-- INSERT INTO user_check VALUES(NULL, '코알라', '무', 20);	-- user_check_chk_1 가 제약에 걸림 에러

SELECT * FROM user_check;
```

### DEFAULT

INSERT 시 특정 컬럼에 값을 주지 않으면, 자동으로 채워질 기본값을 지정

`DEFAULT(기본값)` `DEFAULT 기본값`

- current_date : 현재 날짜
- current_time : 현재 시각

```sql
# DEFAULT
-- INSERT 시 특정 컬럼에 값을 주지 않으면, 자동으로 채워질 기본값을 지정한다.
CREATE TABLE IF NOT EXISTS tbl_country (
	country_code INT AUTO_INCREMENT PRIMARY KEY,
    country_name VARCHAR(255) DEFAULT '한국',
    add_day DATE DEFAULT(current_date),	-- 현재 날짜
    add_time TIME DEFAULT(current_time)	-- 현재 시각
) ENGINE=INNODB;

INSERT INTO tbl_country VALUES(NULL, DEFAULT, DEFAULT, DEFAULT);
INSERT INTO tbl_country(country_name) VALUES('미국');	-- 지정한 값 외에 디폴트 값 들어감
INSERT INTO tbl_country() VALUES();	-- 모든 값이 디폴트가 존재할 경우 아무것도 안 넣어도 성공적으로 들어감

SELECT * FROM tbl_country;
```

## CAST

조회 결과 타입을 변환(cast) 하는 함수

원본 데이터에는 영향을 주지 않고 조회할 때만 바꿈

### 명시적 형변환

- CAST : `CAST(값 AS 바꿀타입)`
- CONVERT : `CONVERT(값, 바꿀타입)`

CONCAT : 문자열 두 개 합치기 `CONCAT(문자열1, 문자열2)`

타입

- 부호가 있는 정수 : SIGNED INTEGER - INTEGER 는 생략 가능
- 날짜 : DATE
- 문자열 : CHAR(크기) - VARCHAR 은 안 됨

```sql
# 명시적 형변환
-- CAST
-- CONVERT

-- CAST(값 AS 바꿀타입)
SELECT AVG(menu_price) AS '변환 전 평균', cast(AVG(menu_price) AS SIGNED INTEGER) '메뉴 평균 가격' FROM tbl_menu;

-- CONVERT(값, 바꿀타입)
-- SIGNED INTEGER : 부호가 있는 정수, 즉 음수, 0, 양수를 표현. INTEGER 는 생략 가능
SELECT convert(AVG(menu_price), SIGNED) '메뉴 평균 가격' FROM tbl_menu;

-- 문자열을 날짜로 변경
SELECT CAST('2026-10-21' AS DATE);

-- 숫자를 문자열로 변경
SELECT menu_name, concat(cast(menu_price as CHAR(8)), '원') '가격' FROM tbl_menu;
```

### 암시적 형변환

```sql
# 암시적 형변환
SELECT '1' + '2';	-- 숫자로 계산해서 반환
SELECT 3 > 'MAY';	-- MAY 를 숫자로 처리하려하다 숫자로 변환할 수 없어서 0 으로 처리해서 3 > 0 으로 수행
```

## 내장 함수

수업용 깃헙 참고

## INDEX

- EXPLAIN : 실행 계획
    - rows : 확인할 거로 예상되는 행의 수
    - key : 비용 계산하여 정해진 사용할 인덱스 (내부 엔진이 판단하기에 사용하지 않는게 비용이 더 저렴할 것 같을 경우, 인덱스가 사용되지 않을 수도 있다.)
    - possible_keys : 사용될 수 있는 인덱스 후보
- `CREATE INDEX 인덱스명 ON 테이블명(컬럼명);` : 컬럼에 인덱스 추가
- `SHOW INDEX FROM 테이블명;` : 인덱스 확인
- `DROP INDEX 인덱스명 ON 테이블명;` : 인덱스 삭제

```sql
# INDEX

DROP TABLE IF EXISTS phone;

CREATE TABLE IF NOT EXISTS phone(
	phone_code INT PRIMARY KEY,
    phone_name VARCHAR(100) NOT NULL,
    phone_price DECIMAL(10, 2) NOT NULL	-- 고정 소수점 데이터 타입
);

INSERT INTO phone VALUES(1, 'iPhone', 2400000), (2, 'Galaxy', 2000000), (3, 'Xiaomi', 1700000);

-- 인덱스 추가
CREATE INDEX idx_phone_name ON phone(phone_name);

-- 인덱스 확인 (pk키 인덱스는 자동 생성 된다)
SHOW INDEX FROM phone;

-- EXPLAIN 은 SELECT 문을 MySQL 이 어떻게 실행할지 보여준다.
EXPLAIN SELECT * FROM phone WHERE phone_name = 'iPhone';

-- 인덱스 삭제
DROP INDEX idx_phone_name ON phone;
```

# Modeling
데이터 관점 모델링, 프로세스 모델링, 상관 모델링

1. 단순화 : 선택과 집중을 통해 목적에 부합하는 필요한 것만 선택
2. 추상화 : 일정한 형식의 표기법으로 관련 있는 것들을 묶어냄
3. 명확화 : 애매모호함을 제거하고 정확하게 기술

관계형 데이터 베이스 : 각 데이터를 관련 있는 것들로 나눠놓는다.

- 각 데이터들의 종류를 한 눈에 파악하기 쉬움
- 데이터 테이블 간의 관계를 통해 연관 있는 데이터 파악이 쉬움
- 중복 값을 제거

### 데이터 모델 표기법

- 피터첸 표기법 : ER Model 이라는 표기버
- **Barker 표기법** : 배타 관계나 슈퍼 서브 속성 표기를 더 구체적으로 표현 가능
    - DA# 으로 작성해볼 수 있음
- IE 표기법 : 범용적인 표기법

- 식별 관계
    - fk 가 pk 에 포함됐을 경우 : fk 가 pk 에 포함되어 있을 경우 (PFK)
- 비식별 관계
    - fk 가 pk 에 포함되지 않았을 경우 : fk 가 없어도 pk 로 알 수 있다.

### 관계형 모델 (Relation Model)

- 릴레이션 이름 : 테이블명
- Attribute : 칼럼
- Tuple : 행
- 식별자 : PK 키
- 어트리뷰트 값 : 칼럼 데이터
- Head / Body : 표의 헤드와 바디
- Degree(차수) : 어트리뷰트의 수
- Cardinality(카디널리티) : 튜플의 수

#### 무결성

- 엔티티 무결성 : 모든 인스턴스는 고유해야 하고 대표하는 속성은 널 값을 가지면 안된다
- 참조 무결성 : 외래 식별자 속성은 실제 존재하는 값을 사용해야 함
- 도메인 무결성 : 속성에 들어올 수 있는 값의 범위를 지켜야한다 (ex) 데이터 타입, 널 여부 등)
- 업무 무결성 : 기업에서 업무를 수행하는 방법이나 데이터를 처리하는 규칙

## ERD

### Entity

업무의 관심 대상 정보나 정보를 알아야 하는 유/무형의 개체

```
요구사항 : 회원이 도서를 대출한다.
-> 회원, 도서, 대출 엔티티 필요

엔티티 : 회원
속성(Attribute) : 이름, 전화번호, etc
인스턴스 : 실제 저장되는 회원 데이터
```

엔터티 도출 원칙

- 업무에서 사용되야 한다
- **두 개 이상의 인스턴스**를 소유해야 한다. (2개 이상의 데이터 행이 사용되야 함)
- 엔티티의 인스턴스를 식별할 수 있는 **식별자**가 존재해야 한다.
- 시스템에서 **지속해서 관리할 필요성**이 있어야 한다.

#### Attribute (속성)

- 단일 속성 : 고객명과 같이 하나의 의미로 사용
- 복합 속성 : 주소와 같이 시/분/구 등으로 세분 값으로 나눌 수 있는 속성
- 다가(다치) 속성 : 전화번호 처럼 한 고객이 여러 속성을 가질 수 있는 속성

- 기초 속성 : 고객명 같은 엔터티의 본질을 설명하는 속성
- 관계 속성 : 다른 엔터티와 연관성을 나타내는 속성 (외래키 등). 다른 엔터티에서는 식별자 역할을 한다.
- 추출 속성 : 다른 엔터티에서 가져와 (계산해) 저장한 값 (장바구니 상품별 금액 등) ← 빠르게 조회 가능하지만 기존 금액이 바뀔 시 수정해야 한다.

#### Identifier (식별자)

- `#` 주 식별자 : 고객 번호, 주문 번호 같은 PK 값
    - 유일성 : 중복 X
    - 최소성 : 유일성을 만족하는 최소한의 수
    - 불변성 : 주식별자가 한번 지정되면 그 식별자의 값은 변하지 않아야 한다.
    - 존재성 : NULL 허용 X
    - 복합키(하나가 아닌 여러 속성)일 수 있다.
        - 학생과 시험이 N 대 N 의 관계이기에 중간 엔터티로 점수 같은 것을 만들어서 N 대 N 관계를 해소시킨다. → 보통은 따로 단일 주 식별자를 만든다. (→ 코드가 복잡해지니까 + 재시험이라는 예외 사항 같은게 생길 수 있으니까? 근데 이러면 또다시 N 대 N 의 관계로 돌아옴…)
- 외래 식별자 `(FK)` : 외래키 (고객(부모)의 고객번호(PK)를 주문(자식)이 참조하여 고객번호(FK)로 가지고 있다.)
    - FK 가 PK 로도 쓰이면 식별 관계 (PFK 라고 표기하기도 한다)
    - FK 가 PK 에는 포함되지 않으면 비식별 관계

### 도메인

비슷한 속성들이 같은 도메인을 사용하게 만든다.

```
도메인 : 번호
속성명 : 카드번호, 전화번호, 핸드폰번호,주문번호
번호 도메인 타입 및 길이 : NUMBER

도메인 : 주소
속성명 : 회원 주소, 직장 주소
주소 타입 및 길이 : NVARCHAR(255)
```

### 관계

엔터티 간의 관계

- 종속 관계 : 부모가 존재해야만 자식이 존재 가능
- 참조 관계 : 엔터티 간의 연관성이 존재하긴 하지만 다른 엔터티가 없다고 존재할 수 없는 관계는 아님

#### 카디널리티

1 : 1 인지 1 : N 인지 N : N 관계인지 등.

다대다 관계일 경우 교차 엔터티를 통해 일대다 관계로 해소해야 한다.

#### 옵셔널리티

연관되는 값이 최소 1개 이상이면 Mandatory, 0개 이상이면 Optional 이다.

근데 고객한테는 주문이 0개 이상이어도 되는데 주문한테는 무조건 하나의 고객이 있어야 해서 DA# 에서 Mandatory 관계선으로 그어서 헷갈림.. 일단 한쪽은 반드시 Optional 이어서 그런가?

#### 일대일 관계

하나로 합쳐질 수 있는 관계

ex) 학과 엔터티와 학과장 엔터티

따로 사용할 가능성이 많을 경우 쪼개서 일대일 관계로 사용한다.

#### 배타 관계

```
회원 - 회원번호
		 - 이름
		 - 전화번호
		 - 주소
		 - 회원 구분    - 고객     - 국적구분
		 
									 - 직원     - 직급명
														  - 부서명
														  - 부서 전화번호
```

위의 상황에서 고객이자 직원인 경우는 존재할 수 없고 고객이나 직원 중 하나여야 할 경우 배타 관계이다.

#### 재귀 관계
자신이 자신의 관계를 참조하고 있을 경우  
ex. 카테고리 테이블의 상위 카테고리 컬럼


- [Database](#database)
  - [DDL](#ddl)
    - [CREATE](#create)
    - [ALTER](#alter)
    - [DROP](#drop)
    - [TRUNCATE](#truncate)
  - [CONSTRAINTS (제약 조건)](#constraints-제약-조건)
    - [NOT NULL](#not-null)
    - [UNIQUE](#unique)
    - [PRIMARY KEY](#primary-key)
    - [FOREIGN KEY](#foreign-key)
      - [ON UPDATE / ON DELETE 옵션](#on-update--on-delete-옵션)
    - [CHECK](#check)
    - [DEFAULT](#default)
  - [CAST](#cast)
    - [명시적 형변환](#명시적-형변환)
    - [암시적 형변환](#암시적-형변환)
  - [내장 함수](#내장-함수)
  - [INDEX](#index)
- [Modeling](#modeling)
    - [데이터 모델 표기법](#데이터-모델-표기법)
    - [관계형 모델 (Relation Model)](#관계형-모델-relation-model)
      - [무결성](#무결성)
  - [ERD](#erd)
    - [Entity](#entity)
      - [Attribute (속성)](#attribute-속성)
      - [Identifier (식별자)](#identifier-식별자)
    - [도메인](#도메인)
    - [관계](#관계)
      - [카디널리티](#카디널리티)
      - [옵셔널리티](#옵셔널리티)
      - [일대일 관계](#일대일-관계)
      - [배타 관계](#배타-관계)
      - [재귀 관계](#재귀-관계)
