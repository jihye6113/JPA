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
