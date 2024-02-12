# 04 엔티티 매핑   
   
JPA를 사용하는 데 가장 중요한 일은 엔티티와 테이블을 정확히 매핑하는 것   
∵ 매핑만 제대로 한다면 적절하게 SQL 생성해서 DB에 전달해줄 뿐만 아니라,   
   패러다임의 불일치 문제도 해결해주기 때문 (상속, 연관관계)   
   
- 대표 어노테이션 목록
  - 객체와 테이블 매핑: @Entity, @Table
  - 기본 키 매핑: @Id
  - 필드와 컬럼 매핑: @Column
  - 연관관계 매핑: @ManyToOne, @JoinColumn
   
＊매핑 정보 기술 방법
1. XML 기반 매핑   
- 장점: 여러 클래스의 매핑 정보를 한 곳에서 관리할 수 있어 유지보수가 상대적으로 쉽다.   
       (참고: https://velog.io/@yu-jin-song/JPA-XML%EC%9D%84-%ED%86%B5%ED%95%9C-%EB%A7%A4%ED%95%91)
   
- 단점: XML 파일을 작성하고 읽는 과정이 번거롭고, 가독성이 좋지 않을 수 있다.   
       XML 파일에 오타가 있거나 잘못된 설정이 들어가면 런타임에 오류가 발생할 수 있다.   
   
2. 어노테이션 기반 매핑   
- 장점: 소스 코드에 매핑 정보를 직접 기술하기 때문에 가독성이 좋고, 관련된 정보들이 한 곳에 모여 있어 코드를 이해하기 쉽다.   
       컴파일 시에 오류를 발견할 수 있어 타입 안정성을 보장한다.
   
- 단점: 설정 정보가 소스 코드에 섞여 있기 때문에 변경 시에는 코드를 수정하고 재컴파일해야 한다.   
       여러 클래스에 걸친 전체적인 설정 변경이 필요한 경우에는 일괄적으로 수정하기가 어려울 수 있다.   
   
두 방법 중, 어노테이션을 사용하는 방법을 많이 사용한다.   
   
<hr/>
   
## 4.1 @Entity   
JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 어노테이션을 필수로 붙여야 한다. (@Entity가 붙은 클래스는 JPA가 관리하는 것으로, 엔티티라고 부른다.)   
   
[@Entity 속성 정리]      
|속성|기능|기본값|
|---|--------|------|
|name|JPA에서 사용할 엔티티 이름을 지정한다. 보통 기본값인 클래스 이름을 사용한다. 만약 다른 패키지에 이름이 같은 엔티티 클래스가 있다면 이름을 지정해서 충돌하지 않도록 해야 한다.|설정하지 않으면 클래스 이름을 그대로 사용한다 (EX. Member)|

@Entity 적용 시 주의사항
1. 기본 생성자는 필수다. (파라미터가 없는 public or protected 생성자)
2. final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
3. 저장할 필드에 final을 사용하면 안 된다.
   
＋ JPA가 엔티티 객체를 생성할 때 기본 생성자를 사용하기 때문에 반드시 있어야된다.   
자바는 생성자가 없으면 기본 생성자를 대신 만들지만, 만약 하나 이상의 생성자가 있다면 기본 생성자를 만들지 않기 때문에 직접 만들어야 한다.   

```java
// 기본 생성자
public Member() {}

// 임의의 생성자
public Member(String name) {
  this.name = name;
}
```
   
＋ inner 클래스   
```java
public class OuterEntity {
    @Entity // 이 부분이 허용되지 않는다.
    public static class InnerEntity {
        // 내부 클래스의 멤버들
    }
}
```
   
## 4.2 @Table   
@Table은 엔티티와 매핑할 테이블을 지정한다. 생략하면 매핑한 엔티티 이름을 테이블 이름으로 사용한다.   
   
[@Table 속성 정리]   
|속성|기능|기본값|
|---|--------|------|
|name|매핑할 테이블 이름|엔티티 이름을 사용한다.|
|catalog|catalog 기능이 있는 데이터베이스에서 catalog를 매핑한다.||
|schema|schema 기능이 있는 데이터베이스에서 schema를 매핑한다.||
|uniqueConstraints(DDL)|DDL 생성 시에 유니크 제약조건을 만든다. 2개 이상의 복합 유니크 제약조건도 만들 수 있다. 참고로 이 기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용됨.|
   
＋ catalog: 데이터베이스 시스템 내에서 다양한 데이터베이스 및 스키마를 구분하고 관리하는 역할을 하는 개념
  ex. ORACLE: "ALL_" 또는 "DBA_" 접두사가 붙은 뷰들을 통해 다양한 메타데이터 정보를 제공. "DBA_TABLES" 뷰는 현재 사용자가 액세스할 수 있는 테이블에 대한 정보를 제공   
   
```java
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "employees", schema = "hr", catalog = "mydb")
public class Employee {

    @Id
    private Long id;
    ...
}
```
   
```SQL
SELECT * FROM mydb.hr.employees;
```

**카탈로그 > 스키마 > 테이블**   
   
## 4.3 다양한 매핑 사용   
02 JPA 시작하기 장에서 개발하던 회원 관리 프로그램에 추가된 요구사항
1. 회원은 일반 회원과 관리자로 구분해야 한다.
2. 회원 가입일과 수정일이 있어야 한다.
3. 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.
   
```java
package jpabook.start;

import javax.persistence.*;
import java.util.Date;

@Entity
@Table (name="MEMBER")
public class Member {
  @Id
  @Column (name = "ID") 
  private String id;
 
  @Column (name = "NAME")
  private String username;
  
  private Integer age;

  // == 추가 ==
  @Enumerated(EnumType.STRING)
  private RoleType roleType; // ★

  @Temporal(TemporalType.TIMESTAMP)
  private Date createdDate; 

  @Temporal(TemporalType.TIMESTAMP)
  private Date lastModifiedDate; // ★★

  @Lob
  private String description; // ★★★

  //Getter, Setter
}

package jpabook.start;

public enum RoleType {
  ADMIN, USER
 }
```
[↑ 요구사항에 맞춰 수정된 회원 엔티티]   
   
위 코드를 분석해보자.   
★ roleType: 자바의 enum을 사용해서 회원의 타입을 구분했다. 자바의 enum을 사용하려면 @Enumerated를 사용해서 매핑한다.   
★★ createDate, lastModifiedDate: 자바의 날짜 타입은 @Temporal을 사용해서 매핑한다.   
★★★ description: 길이 제한이 없는 컬럼은 CLOB 타입으로 저장해야된다. @Lob을 사용하면 CLOB (Character Large Object), BLOB (Binary Large Object)을 매핑할 수 있다.   
   
## 4.4 데이터베이스 스키마 자동 생성   
JPA는 데이터베이스 스키마를 자동으로 생성하는 기능을 지원한다. 클래스의 매핑 정보를 보면 어떤 테이블에 어떤 컬럼을 사용하는지 알 수 있다.   
JPA는 이 매핑정보와 DB 방언을 사용해서 데이터베이스 스키마를 생성한다. (EX. 가변문자타입 VARCHAR, VARCHAR2, ...)   

스키마 자동 생성을 사용하기 위해서는 META-INF/persistence.xml에 아래와 같은 속성을 추가   
```xml
<property name = "hibernate.hbm2ddl.auto" value = "create"/>
```
-> 애플리케이션 실행 시점에 데이터베이스 테이블을 자동으로 생성한다.   
   
＋ hibernate.show_sql 속성을 true로 생성하면 콘솔에 실행되는 테이블 생성 DDL을 출력할 수 있다.   
   
```
Hibernate:
  drop table MEMBER if exists 
Hibernate:
 create table MEMBER (
  ID varchar(255) not null, 
  NAME varchar(255), 
  age integer, 
  roleType varchar(255), 
  createdDate timestamp, 
  lastModifiedDate timestamp, 
  description clob, 
  primary key (ID)
}
```
[↑ DDL 콘솔 출력]   
-> 기존 테이블을 생성하고, 다시 생성한 것을 볼 수 있다.   
그리고 roleType은 VARCHAR 타입으로, createdDate, lastModifiedDate는 TIMESTAMP 타입으로, description은 CLOB으로 생성되었다.   
(만약 오라클용 DB방언을 적용했다면, varchar -> varchar2, integer -> number로 타입이 다르게 생성되었을 것.)   
   
**스키마 자동 생성 기능이 만든 DDL**   
장점: 편하다   
단점: 운영환경에서 사용하기엔 다소 위험하다. (∴ 개발환경에서 사용하거나, 매핑할 때 참고하는 정도로만 사용)
   
|옵션|설명|
|----|---------|
|create|기존 테이블을 생성하고 새로 생성한다. DROP + CREATE|
|create-drop|create 속성에 추가로 애플리케이션을 종료할 때 생성한 DDL을 제거한다. DROP + CREATE + DROP|
|update|데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정한다.|
|validate|데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다. 이 설정은 DDL을 수정하지 않는다.|
|none|자동 생성 기능을 사용하지 않으려면, hibernate.hbm2ddl.auto 속성 자체를 삭제하거나 유효하지 않은 옵션 값(none)을 주면 된다.|

<hr/>
   
＋ HBM2DDL 주의사항   
운영 서버에서 DDL을 수정하는 옵션은 절대 ! 사용하면 안 된다.   
개발 서버나 개발 단계에서만 사용하도록 하고, 개발 환경에 따른 추천 전략은 다음과 같다.   
- 개발 초기 단계일 경우에는 create or update
- 초기화 상태로 자동화된 테스트를 진행하는 개발자 환경과 CI 서버는 create or create-drop
- 테스트 서버는 update or validate
- 스테이징과 운영 서버는 validate or none

<hr/>
   
＋ 이름 매핑 전략 변경하기   
단어와 단어를 구분할 때 자바 언어는 관례상 roleType과 같이 카멜 표기법을 사용, DB는 ROLE_TYPE과 같이 언더스코어를 주로 사용함   
위에서 사용한 회원 엔티티를 매핑하기 위해서는 @Column_name 속성을 명시적으로 이용해야된다.
```
@Column(name = "role_type") // 언더스코어로 구분
String roleType // 카멜 표기법으로 구분
```
**hibernate.ejb.naming_strategy** 속성을 사용하면, org.hibernate.cfg.ImprovedNamingStarategy 클래스를 사용해, 테이블명이나 컬럼명이 생략되었을 경우 자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 매핑한다.
```
<property name = "hibernate.ejb.naming_strategy" value = "org.hibernate.cfg.ImprovedNamingStarategy"/>
```
위처럼 속성을 지정했을 때의 DDL은
```sql
create table member (
  id varchar(255 char) not null, 
  name varchar(255 char), 
  age integer,
  role_type varchar(255 char),
  created_date timestamp,
  last—modified一date timestairp,
  description clob, 
  primary key (id)
}
```
위와 같다.
   
## 4.5 DDL 생성 기능   
＋ 회원 이름은 필수로 입력되어야 하고, 10자를 초과하면 안 된다는 제약조건 추가
   
```java
@Entity
@Table (name="MEMBER") 
public class Member {
  @Id
  @Column (name = "ID") 
  private String id;

  @Column(name = "NAME", nullable = false, length = 10) // ** 추가
  private String username;
  ...
}
```
[↑ 추가된 코드]   
   
```sql
create table MEMBER (
  ID varchar(255) not null, 
  NAME varchar(10) not null,
  primary key (ID)
}
```
[↑ 생성된 DDL]   

~ 유니크 제약조건 추가   
```java
@Entity(name = "Member")
@Table (name="MEMBER" uniqueConstraints = {@UniqueConstraints name = "NAME_AGE_UNIQUE", columnNames = {"NAME}, "AGE"}) // 추가 
public class Member {
  @Id
  @Column (name = "ID") 
  private String id;

  @Column(name = "name")
  private String username;

  private Integer age;
  ...
}
```
[↑ 유니크 제약조건]
   
```sql
ALTER TABLE MEMBER ADD CONSTRAINT NAME_AGE_UNIQUE UNIQUE (NAME, AGE)
}
```
[↑ 생성된 DDL]   
   
위와 같은 속성들은 단지 DDL을 자동 생성할 때에만 사용되고, JPA의 실행 로직에는 영향을 주지 않는다.   
굳이 사용하지 않아도 될 것 같지만, 개발자가 DB 테이블을 까보지 않고, 엔티티만 보고도 손쉽게 다양한 제약조건을 파악할 수 있는 장점이 있다.   
   
## 4.6 기본 키 매핑   
```java
@Entity
public class Member {
   @1d
   @Column(name = "ID") 
   private String id;
   ...
}
```
[↑ 기존에 사용하던 기본 키 매핑]   
   
위 같은 형태는 저장할 때 이미 키 값이 있을 경우의 형태이지만, 저장할 때 DB에서 생성하는 값을 사용하는 방법을 알아보자. (EX. ORACLE: SEQUENCE, MySQL: AUTO_INCREMENT)   
   
JPA가 사용하는 DB 기본 키 생성 전략   
- 직접 할당: 기본 키를 애플리케이션에서 직접 할당한다.
- 자동 생성: 대리 키 사용 방식
   - IDENTITY: 기본 키 생성을 데이터베이스에 위임한다.
   - SEQUENCE: DB 시쿼느를 사용해서 기본 키를 할당한다.
   - TABLE: 키 생성 테이블을 사용한다.
↑ 자동 생성 전략이 다양한 이유는 DB 벤더마다 지원하는 방식이 다르기 때문
(EX. ORACLE: SEQUENCE 제공, MySQL: SEQUENCE 제공X / MySQL: AUTO_INCREMENT 제공)   

기본키를 직접 할당하려면 할당하고자 하는 컬럼에 @Id만 사용하면 되고, 자동생성전략을 사용하려면 @Id에 @GeneratedValue를 추가하고 원하는 키 생성 전략을 선택하면 된다.   

＋ 키 생성 전략을 사용하려면 persistence.xml에 **hibernate.id.new_generator_mapping = true** 속성을 반드시 ! 추가해야 한다.   
이 옵션을 true로 설정하면 키 생성 성능을 최적화하는 allocationSize 속성을 사용하는 방식이 달라진다.   
```xml
<property name = "hibernate.id.new_generator_mappings" value = "true"
```
   
### 4.6.1 기본 키 직접 할당 전략   
기본 키를 직접 할당하려면 다음 코드와 같이 @Id로 매핑하면 된다.
```java
@Id
@Column (name = "ID") 
private String id;
```

@Id 적용 가능 자바 타입은 다음과 같다.   
- 자바 기본형
- 자바 래퍼(Wrapper)형
  - String
  - java.util.Date
  - java.sql.Date
  - java.math.BigDecimal
  - java.math.BigInteger
  - ...

기본 키 직접 할당 전략은 em.persist()로 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 방법이다.   
```java
Board board = new Board(); 
board.setld("id1") //기본 키직접 할당 
em.persist(board);
```
   
＋ 기본 키 직접 할당 전략에서 식별자 값 없이 저장하면 예외가 발생하는데, 어떤 예외가 발생하는지 JPA 표준에는 정의되어 있지 않다.   
하이버네이트를 구현체로 사용하면 JPA 최상위 예외인 javax.persistence.PersistenceException 예외가 발생하는데, 내부에 하이버네이트 예외인 org.hibernate.id.identifireGenerationException 예외를 포함하고 있다.   
   
### 4.6.2 IDENTIFY 전략   
IDENTIFY는 기본 키 생성을 DB에 위임하는 전략이다. 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용한다.   

```SQL
CREATE TABLE BOARD (
   ID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
   DATA VARCHAR(255)
);

INSERT INTO BOARD (DATA) VALUES ('A');
INSERT INTO BOARD (DATA) VALUES ('B');
```
[↑ AUTO_INCREMENT를 사용]   
      
|ID|DATA|
|--|----|
|1|A|
|2|B|   

[↑ BOARD 테이블 결과]   
ID 컬럼에 자동으로 값이 입력된 것을 확인할 수 있다. DB에 값을 저장하고 나서야 기본 키 값을 구할 수 있을 때 사용한다.
   
```java
@Entity
   public class Board {
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   private Long id;
   ...
}
```
   
```java
private static void logic(EntityManager em) {
   Board board = new Board(); 
   em.persist(board);
   System.out.println("board.id = " + board.getId());
 }
 // 출력: board.id = 1 -> 저장 시점에 데이터베이스가 생성한 값을 jpa가 조회한 것이다.
```
   
<hr/>
   
＋ IDENTIFY 전략과 최적화   
IDENTIFY 전략은 DB에 INSERT를 한 후에 기본 키 값을 조회할 수 잆다.   
따라서 엔티티에 식별자 값을 할당하려면 JPA는 추가로 데이터베이스를 조회해야 한다.
JDBC3에서 추가된 Statement.getGeneratedKeys()를 사용하면 데이터를 저장하면서 동시에 생성된 기본 키 값도 얻어올 수 있다. 하이버네이트는 이 메서드를 사용해서 DB와 한 번만 통신한다.

   
<hr/>
   
＋ 엔티티가 영속 상태가 되려면 식별자가 반드시 필요하다. 그런데 **IDENTIFY 식별자 생성 전략은 엔티티를 DB에 저장해야 식별자를 구할 수 있으므로** em.persist()를 호출하는 즉시 INSERT SQL이 DB에 전달된다. 따라서 이 전략은 트랜잭션을 지원하는 **쓰기 지연이 동작하지 않는다.**   
   
### 4.6.3 SEQUENCE 전략   
데이터베이스 시퀀스: 유일한 값을 순서대로 생성하는 DB 오브젝트   
SEQUENCE 전략은 이 시퀀스를 사용해 기본 키를 생성하고, ORACLE, PostgreSQL, DB2, H2에서 사용할 수 있다.
   
시퀀스를 사용하기 위해서는 시퀀스를 먼저 생성해야 한다.   
```sql
CREATE TABLE BOARD (
   ID BIGINT NOT NULL PRIMARY KEY, 
   DATA VARCHAR(255)
)

// 시권스 생성
CREATE SEQUENCE B0ARD_SEQ START WITH 1 INCREMENT BY 1;
```
   
```java
@Entity
@SequenceGenerator(
 name = "BOARD_SEQ_GENERATOR",
 sequenceName = "BOARD_SEQ" // 매핑할 데이터베이스 시퀀스 이름
 initialValue = 1, allocationSize = 1) 
public class Board {
   @Id
   @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "BOARD__SEQ_GENERATOR")
   private Long id;
   ...
}
```
[↑ 시퀀스 매핑 코드]   
@SequenceGenerator를 사용해서 BOARD_SEQ_GENERATOR라는 이름의 시퀀스 생성기를 등록했다.   
그리고 sequenceName 속성에서 BOARD_SEQ를 지정했는데, JPA는 이 시퀀스 생성기를 실제 DB의 BOARD_SEQ 시퀀스와 매핑한다.   
   
그리고 키 생성 전략을 GenerationType.SEQUCNCE로 설정하고 generator = "BOARD_SEQ_GENERATOR"로 방금 등록한 시퀀스 생성기를 선택했다.   
   
```java
private static void logic(EntityManager em) {
   Board board = new Board(); 
   em.persist(board);
   System.out.println("board.id = " + board.getId());
 }
 // 출력: board.id = 1 -> em.persiste()를 호출할 때 먼저 DB 시퀀스를 사용해서 식별자를 조회한다.
 // 후에 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장한다.
 // 이후 트랜잭션을 커밋해서 플러시가 일어나면 엔티티를 DB에 저장한다.
```
IDENTITY, SEQUENCE의 전략은 같지만, 내부 동작 방식은 다르다.   
IDENTITY 전략은 먼저 엔티티를 DB에 저장한 후에 식별자를 조회하지만, SEQUENCE 전략은 저장 전에 조회해온 후에 담는 형식   
   
**@SequenceGenerator**   
[↓ @SequenceGenerator 속성 정리]   
|속성|기능|기본값|
|----|-------------|----|
|name|식별자 생성기 이름|필수|
|sequenceName|DB에 등록되어 있는 시퀀스 이름|hibernate_sequence|
|initialValue|DDL 생성 시에만 사용됨. 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정한다.|1|
|allocationSize|시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨)|50|
|catalog, schema|DB catalog, schema|
   
??   
기본적으로 JPA에서 시퀀스를 사용할 때, 각 엔터티에 대한 새로운 식별자를 생성할 때마다 데이터베이스에 쿼리를 날려 시퀀스 값을 가져오게 됩니다. 이로 인해 매번 데이터베이스와의 통신이 필요하며, 성능에 영향을 미칠 수 있습니다.   
   
allocationSize를 사용하면 한 번에 할당되는 범위의 크기를 지정함으로써, 미리 할당된 범위 내에서 엔터티에 대한 식별자를 메모리 상에서 관리합니다. 이렇게 함으로써 데이터베이스와의 통신 횟수를 줄이고 성능을 최적화할 수 있습니다.   
??   
   
매핑할 DDL은 다음과 같다.   
```sql
CREATE SEQUENCE [sequenceName]
START WITH [initialValue] increment by [allocationSize]
```
JPA 표준 명세에서는 sequenceName의 기본값을 JPA 구현체가 정의하도록 했다.   
   
＋ SequenceGenerator.allocationSize의 기본 값이 50인 것에 주의하여야 한다.   
JPA가 기본으로 생성하는 DB 시퀀스는 호출할 때마다 시퀀스 값이 50씩 증가가 된다. 최적화 때문인데, DB 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 값을 반드시 1로 설정해야 한다.   
   
＋ SEQUENCE 전략과 최적화   
SEQUCNCE 전략은 데이터베이스 시퀀스를 통해 식별자를 조회하는 추가 작업이 필요하다. -> DB와 두 번 통신   
1. 식별자를 구하려고 DB 시퀀스를 조회한다. (EX. SELECT BOARD_SEQ.NEXTVAL FROM DUAL)
2. 조회한 시퀀스를 기본 키 값으로 DB에 저장한다. (EX. INSERT INTO BOARD ...)
   
JPA는 시퀀스에 접근하는 횟수를 줄이기 위해 @SequenceGenerator.allocationSize를 사용한다.   
여기에 설정한 값이 한 번 시퀀스 값을 증가시키고 나서 그만큼 메모리에 시퀀스 값을 할당한다.   
ex. allocationSize 값이 50이면 시퀀스를 한 번에 50 증가시킨 다음에 1 ~ 50까지는 메모리에서 식별자를 할당한다. 그리고 51이 되면 시퀀스 값을 100으로 증가시킨 다음 51 ~ 100까지 메모리에서 식별자를 할당한다.   
이 최적화 방법은 시퀀스 값을 선점하므로 여러 JVM이 동시에 동작해도 기본 키 값이 충돌하지 않는 장점이 있다. 반면에 DB에 직접 접근해서 데이터를 등록할 때 시퀀스 값이 한 번에 많이 증가한다는 점을 염두해두어야 한다. 이런 상황이 부담스럽고 INSERT 성능이 중요하지 않으면 allocationSize의 값을 1로 설정하면 된다.
   
<hr/>
   
＋ @SequcnceGenerator는 다음과 같이 @GeneratedValue 옆에 사용해도 된다.
```java
@Entity
public class Board {
   @Id
   @GeneratedValue(...)
   @SequenceGenerator(...) 
   private Long id;
```
      
### 4.6.4 TABLE 전략   
TABLE 전략은 키 생성 전용 테이블을 하나 만들고, 여기에 이름과 값으로 사용할 컬럼을 만들어 DB 시퀀스를 흉내내는 전략이다. => 테이블을 사용하므로 모든 DB에 적용할 수 있다.   
TABLE 전략을 사용하려면, 키 생성 용도로 사용할 테이블을 만들어야 한다.   
```SQL
create table MY_SEQUENCES (
   sequence_name varchar(255) not null ,
   next_val bigint,
   primary key ( sequence_name )
)
```
sequence_name 컬럼을 시퀀스 이름으로 사용하고 next_val 컬럼을 시퀀스 값으로 사용한다. (컬럼 이름 변경 가능하지만 위와 같이 사용하는 것이 기본 값)   
   
```java
@Entity
@TableGenerator(
 name = "BOARD_SEQ_GENERATOR", 
 table = "MY_SEQUENCES",
 pkColumnValue = "BOARD_SEQ", allocationSize = 1) 
public class Board {

   @Id
   @GeneratedValue(strategy = GenerationType.TABLE, 
                   generator = "BOARD__SEQ_GENERATOR")
   private Long id;
   ...
}
```
[↑ TABLE 전략 매핑 코드]   
@TableGenerator를 사용해서 테이블 키 생성기를 등록
   
```java
private static void logic(EntityManager em) {
   Board board = new Board(); 
   em.persist(board);
   System.out.println("board.id = " + board.getId());
 }
 // 출력: board.id = 1 
```
[↑ TABLE 전략 매핑 사용 코드]   
테이블을 사용한다는 것만 제외하면 시퀀스와 내부 동작 방식이 같다. (조회 후 키 값 할당, 커밋할 때 플러시)   
키 생성기를 사용할 때마다 next_val 컬럼 값이 증가한다.   
! 참고로 MY_SEQUENCE 테이블에 값이 없으면 jpa가 값을 insert하면서 초기화 하므로 값을 미리 넣어둘 필요는 없다.   
   
|sequence_name|next_val|
|-------------|--------|
|BOARD_SEQ|2|
|MEMBER_SEQ|10|
|PRDUCT_SEQ|50|
   
**@TableGenerator**   
[↓@TableGenerator 속성 정리]   
|속성|기능|기본값|
|-----|----------|--------|
|name|식별자 생성기 이름|필수|
|table|키생성 테이블명|hibernate_sequences|
|pkColumnName|시퀀스 컬럼명|sequence_name|
|valueColumnName|시퀀스 값 컬럼명|next_val|
|pkColumnValue|키로 사용할 값 이름|엔티티 이름|
|initialValue|초기 값, 마지막으로 생성된 값이 기준이다.|0|
|allocationSize|시퀀스 한 번 호출에 증가하는 수(성능최적화에 사용됨)|50|
|catalog, schema|데이터베이스 catalog, schema 이름||
|uniqueConstraints(DDL)|유니크 제약 조건을 지정할 수 있다.||
   
JPA 표준 명세에서는 table, pkColumnName, valueColumnName의 기본값을 JPA 구현체가 정의하도록 했다.   
[매핑할 DDL, 테이블명 {table}]????   
|||
|----|----|
|{pkColumnName}|{valueColumnName}|
|pkColumnValue}|{initialValue}|
   
＋TABLE 전략과 최적화   
TABLE 전략은 값을 조회하면서 SELECT 쿼리를 사용하고 다음 값으로 증가시키기 위해 UPDATE 쿼리를 사용한다. 이 전략은 SEQUENCE 전략과 비교해서 DB와 한 번 더 통신하는 단점이 있다. TABLE 전략을 최적화하려면 @TableGenerator.allocationSize를 사용하면 된다. 이 값을 사용해서 최적화하는 방법은 SEQUENCE 전략과 같다.
   
### 4.6.5 AUTO 전략   
GenerationType.AUTO는 선택한 DB 방언에 따라 IDENTIFY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다.
   
```java
@Entity
public class Board {

   @Id
   @GeneratedValue(strategy = GenerationType.AUTO) 
   private Long id;
   ...
}
```
[↑ AUTO 전략 매핑 코드]   

위처럼 GeneratedValue의 속성을 지정해도 되고, 아래처럼 속성 지정 없이 @GeneratedValue 호출만 해도 된다.
   
```java
@Entity
public class Board {

   @Id @GeneratedValue 
   private Long id;
   ...
}
```
AUTO 전략의 장점: DB를 변경해도 코드 수정 필요가 없다. 특히 키 생성 전략이 확정되지 않은 개발 초기 단계나 프로토타입 개발 시 편리하게 사용할 수 있다.   
주의할 점은 미리 시퀀스나 키 생성용 테이블을 만들어 두어야 한다는 것.   
   
### 4.6.6 기본 키 매핑 정리   
영속성 컨텍스트는 엔티티를 식별자 값으로 구분하므로 엔티티를 영속 상태로 만드려면 식별자 값이 반드시 있어야 한다.
   
- em.persist()를 호출한 직후에 발생하는 일
   - 직접 할당: em.persist() 호출 전, 애플리케이션에서 직접 식별자 값을 할당해야 한다. 만약 식별자 값이 없으면 예외가 발생한다.
   - SEQUENCE: 데이터베이스 시퀀스에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다.
   - TABLE: 데이터베이스 시퀀스 생성용 테이블에서 식별자 갓을 획득한 후 영속성 컨텍스트에 저장한다.
   - IDENTITY: 데이터베이스에 엔티티를 저장해서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다. (IDENTITY 전략은 테이블에 데이터를 저장해야 식별자 값을 획득할 수 있다.)
   
＋ 참고   
＊ 권장하는 식별자 선택 전략   
1. null 값은 허용하지 않는다.
2. 유일해야 한다.
3. 변해선 안 된다.
   
＊ 테이블의 기본 키를 선택하는 전략   
- 자연키: 비즈니스에 의미가 있는 키(주민번호, 이메일, 전화번호)
- 대리키: 비즈니스와 관련 없는 임의로 만들어진 키, 대체키로도 불림(오라클 시퀀스, auto_increment 사용)
   
**자연 키보다는 대리 키를 권장한다.** -> 전화번호, 주민번호가 바뀔 경우에 키 값이 모두 뒤틀린다. 특히 주민번호의 경우에는 법적으로 보호가 되도록 변화되었기 때문에 이런 변화가 있을 경우 데이터의 수정이 굉장히 많아진다.   
대리키는 비즈니스와 무관한 임의의 값이기 때문에 요구사항이 변경되어도 기본 키가 변경되는 일은 드물다.   
   
＋ 주의   
기본 키는 변하면 안 된다는 기본 원칙으로 인해, 저장된 엔티티의 기본 키 값은 절대 변경하면 안 된다.   
setId()와 같이 식별자를 수정하는 메서드를 외부에 공개하지 않는 것도 문제를 예방하는 하나의 방법이 될 수 있다.   
   
<hr>
   
## 4.7 필드와 컬럼 매핑: 레퍼런스   
[↓ 필드와 컬럼 매핑 분류]   
|분류|매핑 어노테이션|설명|
|-----|-------|------------|
|필드와 컬럼 매핑|@Column|컬럼을 매핑한다|
||@Enumerated|자바의 enum 타입을 매핑한다|
||@Temporal|날짜 타입을 매핑한다|
||@Lob|BLOB, CLOB 타입을 매핑한다|
||@Transient|특정필드를 DB에 매핑하지 않는다|
|기타|@Access|JPA가 엔티티에 접근하는 방식을 지정한다|
   
### 4.7.1 @Column   
@Column은 객체 필드를 테이블 컬럼에 매핑한다.   
속성 중 주로 name, nullable을 사용한다. insertable, updatable 속성은 DB에 저장되어 있는 정보를 읽기만 하고 실수로 변경하는 것을 방지하고 싶을 때 사용한다.   
   
[↓ @Column 속성 정리]   
|속성|기능|기본값|
|----|----------------|-----|
|name|필드와 매핑할 테이블의 컬럼 이름|객체의 필드 이름|
|insertable|엔티티 저장 시 이 필드도 같이 저장한다는 뜻. false로 설정하면 이 필드는 DB에 저장하지 않는다. false 옵션은 읽기 전용일 때 사용한다.|true|
|updatable|엔티티 수정 시 이 필드도 같이 수정한다. false로 설정하면 DB에 수정하지 않는다. false는 읽기 전용일 때 사용한다.|true|
|table|하나의 엔티티를 두 개 이상의 테이블에 매핑할 때 사용한다. 지정된 필드를 다른 테이블에 매핑할 수 있다.|현재 클래스가 매핑된 테이블|
|nullable(DDL)|null 값의 허용 여부를 설정한다. false로 설정하면 DDL 생성 시에 NOT NULL 제약조건이 붙는다.|true|
|unique(DDL)|@Table의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약 조건을 걸 때 사용한다. 만약 두 컬럼 이상을 사용해서 유니크 제약조건을 사용하려면 클래스 레벨에서 @Table.uniqueConstraints를 사용해야 한다.||
|columnDefinition(DDL)|데이터베이스 컬럼 정보를 직접 줄 수 있다.|필드의 자바 타입과 방언 정보를 사용해 적절한 컬럼 타입을 생성한다.|
|length(DDL)|문자 길이 제약조건. String 타입에만 사용한다.|255|
|precision, scale(DDL)|BigDecimal 타입에서 사용한다(BigInteger도 사용 가능). precision은 소수점을 포함한 전체 자릿수를, scale은 소수의 자릿수다. 참고로 double, float 타입에는 적용되지 않는다. 아주 큰 숫자나 정밀한 소수를 다루어야 할 때만 사용한다.|precision = 19, scale = 2|
   
<hr>
   
▼ nullable(DDL 생성 기능)   
@Column(nullable = false)   
private String data;
   
//생성된 DDL   
data varchar(255) not null   
   
▼ unique(DDL 생성 기능)   
@Column(unique = true)   
private String username;   
   
//생성된 DDL   
alter table Tablename   
add constraint UK_Xxx unique (username)   

▼ columnDefinition(DDL 생성 기능)   
@Colulumn (columnDefinition = "varchar (100) default 'EMPTY'")   
private String data;   
   
//생성된 DDL   
data varchar(100) default 'EMPTY'   
   
▼ length(DDL 생성 기능)   
@Column(length = 400)   
private String data;   
   
//생성된 DDL   
data varchar(400)   

▼ precision, scale(DDL 생성 기능)   
@Column(precision = 10, scale = 2)   
private BigDecimal cal;   
   
//생성된 DDL   
cal numeric(10,2) // H2, PostgreSQL   
cal number (10,2) // 오라클   
cal decimal(10,2) // MySQL   
   
<hr/>
   
＋ 참고   
@Column 생략   
@Column을 생략하면 어떻게 될까? 대부분 @Column 속성의 기본값이 적용되는데, 자바 기본 타입일 때는 nullable 속성에 예외가 있다.   
```
int data1; // @Column 생략, 자바 기본 타입
data1 integer not null // 생성된 DDL

Integer data2; // @Column 생략, 객체 타입
data2 integer // 생성된 DDL

@Column
int data3; // @Column 사용, 자바 기본 타ㅣㅇㅂ
data3 integer // 생성된 DDL
```
int data1 같은 자바 기본 타입에는 null 값을 입력할 수 없다.   
Integer data2처럼 객체 타입일 때에만 null 값이 허용된다.   
=> 자바 기본 타입인 data1을 DDL로 생성할 때에는 NOT NULL 제약 조건을 추가하는 것이 안전하기에 JPA에서는 기본 타입일 때 NOT NULL 제약 조건을 추가한다.   
=> 그런데 int data3처럼 @Column을 사용하면 @Column은 nullable = true가 기본 값이므로, not null 제약 조건을 설정하지 않는다. (@Column 우선) =>> 자바 기본 타입에 @Column을 사용하면, nullable = false로 지정하는 것이 안전.   
   
### 4.7.2 @Enumerated   
