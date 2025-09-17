# Chapter4 엔티티 매핑 

<br>

# 목차

> **[1. @Entity](#Entity)**
> 
> **[2. @Table](#Table)**
> 
> **[3. 다양한 매핑 사용](#다양한-매핑-사용)**
> 
> **[4. 데이터베이스 스키마 자동 생성](#데이터베이스-스키마-자동-생성)**
> 
> **[5. DDL 생성 기능](#DDL-생성-기능)**
> 
> **[6. 기본 키 매핑](#기본-키-매핑)**
> 
> **[7. 필드와 컬럼 매핑: 레퍼런스](#필드와-컬럼-매핑-레퍼런스)**

## @Entity
JPA를 사용하여 `테이블과 매핑할 클래스`는 **@Entity 어노테이션을 필수로 붙여야한다.** 

➡️ @Entity가 붙으면 해당 클래스는 **JPA가 관리하는 엔티티**가 된다. 

- **기본 생성자는 필수**이다.  ➡️ `JPA는 기본 생성자로 엔티티 객체를 생성`한다. 
- final 클래스, enum, interface, inner 클래스에는 사용 ❌ 
- 저장할 필드에 final을 하면 안된다. 


<br>

## @Table

**매핑할 테이블을 지정**한다.

`name:` 테이블 이름 (생략 가능)

<br>

`catalog:` catalog 기능이 있는 DB에서 catalog 매핑 (PostgreSQL, Oracle에서 사용될 수 있는 개념이다.)

➡️  한 시스템에서 **여러개의 데이터 베이스를 사용하는 경우** <ins>**어느 데이터베이스에 속한 테이블인지 지정하는 것**</ins>을 의미한다. 
      
```java
@Table(catalog = "databaseA") //databaseA의 Member 테이블
public class Member {
    
}
```

<br>

`schema:` schema 기능이 있는 DB에서 schema 매핑

> **💡 MySQL에서는 데이터베이스라고 생각하면 된다.** 

<br>


`uniqueConstraints(DDL):` 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용한다. 

<br>

## 다양한 매핑 사용

### @Enumerated

**자바의 enum 타입 매핑**

- @Enumerated(EnumType.ORDINAL) : enum 순서를 저장
- @Enumerated(EnumType.STRING) : enum 이름을 저장

>  🚨 **ORDINAL사용 시** 이미 저장된 enum의 순서를 변경할 수 없기 때문에 <ins>**새로운 enum이 추가될 경우 순서가 꼬이는 문제가 발생할 수 있다.**</ins> 

<br>

### @Temporal

**자바의 날짜 타입 매핑**

- @Temporal(TemporalType.DATE) : 2025-03-23
- @Temporal(TemporalType.TIME) : 13:00:00
- @Temporal(TemporalType.TIMESTAMP) 2025-03-23 13:00:00


> MySQL ➡️ datetime
>
>H2, 오라클, PostgreSQL ➡️ timestamp

<br>

### @Lob

CLOB, BLOB 타입 매핑 

JPA에서 대용량 데이터(문자 또는 바이너리)를 저장할 때 사용하는 어노테이션이다. 

<br>

## 데이터베이스 스키마 자동 생성

JPA는 매핑 정보와 데이터베이스 방언을 통해 데이터베이스 스키마를 생성한다. 

![Image](https://github.com/user-attachments/assets/decc48e9-677a-452c-a80a-93535c60d0ed)


 ➡️ 개발 환경에서 참고하는 정도로만 사용하는 것이 좋다. 

<br>

### **ddl-auto 속성 종류**

**🟢 create [Drop + Create]**

기존 테이블 삭제하고 다시 생성한다. 

---

**🟢 create-drop [Drop + Create + Drop]**

애플리케이션이 종료할 때 생성한 DDL을 제거한다. 

---

**🟢 update**

변경 사항만 수정한다. 

---

**🟢 validate**

변경 사항이 있을 경우 경고를 남기고 애플리케이션 실행을 하지 않는다. (DDL 수정 ❌)

---

**🟢 none**

자동 생성 기능을 사용하고 싶지 않은 경우에 사용한다. 

<br>

### ⚠️ 속성 사용 시 주의점 
> 🚨 `create`, `create-drop`, `update`는 운영 서버에서 절대 사용하면 안된다. 

<br>

**개발 초기**  ➡️ `create` OR `update`

**테스트를 진행하는 개발자 환경**과 **CI 서버** ➡️  `create` OR `create-drop`

**테스트 서버** ➡️  `update` OR `validate`

**스테이징**과 **운영 서버** ➡️  `validate` OR `none`

## DDL 생성 기능

**DDL에 제약 조건 추가하기**

### @Column 

```java
@Column (nullable = false, length = 10)
private String userName;
```

`nullable = false`  설정 시 DDL에 not null 제약 조건을 추가할 수 있다. 

`length` 설정 시 DDL에 문자 크기를 지정할 수 있다. 

> 💡 **nullable 말고는 잘 사용되지 않는다.**

>  **💡 Java는 기본형일 경우 null을 허용하지 ❌** 
> 
> @Column은 기본적으로 nullable = true이기 때문에 기본형에 @Column 사용한다면 nullable = false로 지정하는 것이 좋다. 
<br>

### @Table의 uniqueConstraints 속성

```java
@ Table (uniqueConstraints  = {@UniqueConstraint(
        name = "NAME_AGE_UNIQUE", //제약 조건 컬럼 명
        columnNames = {"NAME", "AGE"} //유니크 제약이 적용될 컬럼 목록
)})
public class Member {
    
}
```

<br>

➡️ 이러한 기능들은 단지 **DDL을 자동으로 생성할 때만 사용**되고 <ins>**JPA의 실행 로직에는 영향을 주지 않는다.**</ins> 

그럼에도 이를 사용하는 이유는 **애플리케이션 개발자가 엔티티만 보고 쉽게 제약을 파악할 수 있기 때문**이다.

<br>

## 기본 키 매핑

JPA가 제공하는 데이터베이스 기본 키 생성 전략은 다음과 같다. 

### 직접 할당 : @Id 사용

엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당한다.

@Id 적용 가능한 자바 타입은 아래와 같다. 

- 기본형
- 래퍼형
- String
- util.Date
- sql.Date
- math.BigDecimal
- math.BigInteger



<br>


### 자동 생성 : @GeneratedValue

>  💡벤더마다 지원하는 방식이 다르기 때문에 다양한 전략이 존재한다. 

<br>

- #### IDENTITY ➡️ 기본 키 생성을 데이터베이스에 위임한다. 

MySQL, PostgreSQL, SQL Server, DB2에서 사용한다. **ex>** MySQL의 AUTO_INCREMENT


```java
import javax.annotation.processing.Generated;

@Id
@GenerateValue(strategy = GenerationType.IDENTITY)
private Long id;
```

>  🚨`IDENTITY 전략`은 **엔티티를 데이터베이스에 저장해야 식별자를 구할 수 있기 때문에** INSERT SQL이 **즉시** 데이터베이스에 **전달**된다. 
> 
> 따라서 이 전략은 <ins>**쓰기 지연이 동작하지 않는다.**</ins> 

<br>

- #### SEQUENCE ➡️ 데이터베이스 시퀸스를 사용해서 기본 키를 할당한다. 
시퀀스는 **유일한 값을 순서대로 생성**하는 데이터베이스 오브젝트이다.

오라클, PostgreSQL, DB2, H2 에서 사용할 수 있다.

```java
@Entity
@SequenceGenerator(
        name = "BOARD_SEQ_GENERATOR", //시퀀스 생성기 이름
        sequenceName = "BOARD_SEQ", // 데이터베이스에서 사용할 시퀸스 이름
        initialValue = 1, //시퀀스 초기값
        allocationSize = 1 //증가 단위 (기본 값: 50)
)
public class Board {
    @Id
    @GenerateValue(strategy = GenerationType.SEQUENCE, generator = "BOARD_SEQ_GENERATOR") // 시퀀스 생성기 선택
    private Long id;
}

```

>  💡 **allocationSize는 최적화 때문에 기본값으로 50이 할당된다.** 
> 
> 만약 데이터베이스 `시퀀스 값이 하나씩 증가하도록 설정`하였다면 **allocationSize는를 반드시 1**로 설정해야한다.


<br>

**SEQUENCE 전략은 persist() 호출 시** 

1. 데이터베이스 시퀀스를 사용해 <ins>**식별자를 조회 [통신1]**</ins>한다.
2. 조회한 식별자를 **엔티티에 할당 후에 엔티티를 영속성 컨텍스트에 저장**하고
3. 이후 `커밋` `플레시`가 일어나면 엔티티를 <ins>**데이터 베이스에 저장 [통신2]**</ins>한다. 

➡️ 데이터베이스와 2번의 통신 과정을 거친다.  

<br>

> ###  **💡 최적화 방법**
> 
> JPA는 시퀀스에 **접근하는 횟수를 줄이기 위해** 설정한 값만큼 시퀀스를 증가시키고 <ins>**메모리에 시퀀스 값을 할당한다.**</ins> 
> 
> 해당 방법은 **시퀀스 값을 선점**하기 때문에 `JVM이 동시에 동작해도 기본 키 값이 충돌하지 않는 장점`을 가지고 있다. 
> 
> 다만, 한번에 많은 값을 증가시키기 때문에 이런 상황을 피하고 싶다면 allocationSize를 1로 설정한다. 

<br>

- #### TABLE  ➡️ 키 생성 테이블을 사용한다. (마치 시퀀스처럼 사용하는 방법)

`키 생성 전용 테이블`을 하나 만들고 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스 흉내늘 내는 전략이다. 

모든 데이터베이스에 적용 가능한다. 

```java
@Entity
@TableGenerator(
        name = "BOARD_SEQ_GENERATOR", //테이블 키 생성기 등록 
        table = "MY_SEQUENCES", // 키 생성용 테이블 
        pkColumnValue = "BOARD_SEQ", allocationSize = 1 //엔티티의 키로 사용할 값 이름  
)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```

>  **💡 pkColumnValue**
> 
> **여러 엔티티가 table을 사용할 때 구분 지을 수 있는 키 이름을 설정한다.** 


> ### 💡 최적화 방법
> 
> table 전략은 squence보다 1번 더 데이터 베이스와 통신해야하는 단점이 있다. (SELECT, UPDATE, INSERT)
> 
> **최적화 방법은 시퀀스 전략과 동일하다.**

<br>

- #### AUTO  ➡️ IDENTITY, SEQUENCE, TABLE을 방언에 따라 자동 선택 

AUTO 전략을 사용한다면 코드를 수정할 필요가 없기 때문에 개발 초기 단계나 프로토타입 개발 시 편리하게 사용할 수 있다. 


> ### 📌 프로토타입 개발
> 
> **서비스의 초기버전**을 말한다.
> 
> 아이디어를 시각화하여 컨셉을 구체화하고 사용자의 피드백을 수집하는 목적으로 개발한다. 

<br>

## 필드와 컬럼 매핑: 레퍼런스

### @Transient

해당 어노테이션이 붙은 필드는 매핑하지 않는다. 

따라서 데이터베이스에 저장, 조회하지 않으며 **임시로 어떤 값을 보관하고 싶을 때 사용**한다. 


```java
@Transient
private boolean isOnline;
```

<br>

### @Access

JPA가 엔티티 **데이터에 접근하는 방식을 지정**한다.

- `필드 접근 :` 필드에 직접 접근한다. (private도 접근 가능)
- `프로퍼티 접근 :` 접근자(Getter)를 사용한다. 

```java
@Entity 
public class Member {
    @Id
    private String id;
    
    private String firstName;
    private String lastName;
    
    @Access(AccessType.PROPERTY)
    public String getFullName() {
        return firstName + lastName; //Member 테이블의 FULLNAME 컬럼에 연산 결과가 저장된다. 
    }
    
}
```