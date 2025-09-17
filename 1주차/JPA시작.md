# Chapter2 JPA 시작 

<br>

# 목차 

> **[1. 객체 매핑 시작](#객체-매핑-시작-)**
> 
> **[2. 애플리케이션 개발](#애플리케이션-개발-)**

<br>

## 객체 매핑 시작 

### @Entity & @Table

<span style="color:#EEDC82;">@Entity</span>: 해당 클래스를 `테이블과 매핑`한다고 알려준다.  
<span style="color:#EEDC82;">@Table</span> : `매핑할 테이블 정보`를 알려준다. 

```java
@Entity
@Table(name = "members")
public class Member() {
    ...
}
```
> 💡 `@Table 어노테이션`을 생략하면 **클래스 이름을 테이블 이름으로 매핑**한다. 


<br>

### @Id
**엔티티 클래스의 필드**를 `테이블의 기본 키(Primary Key)`에 매핑한다. <span style="color:orange;">**(식별자 필드)**</span>
```java
@Entity
@Table(name = "members")
public class Member() {
    @Id
    private Long id;
}
```




<br>

### @Column
필드를 컬럼에 매핑한다.
```java
@Entity
@Table(name = "members")
public class Member() {
    @Id
    private Long id;

    @Column (name = "name") //name속성을 통해 테이블 name 컬럼에 매핑
    private String memberName;
}
```
> 💡 `@Column 어노테이션` 생략 시 필드명으로 컬럼명 매핑 


<br>

### 데이터베이스 방언 

SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능을 JPA에서는 방언이라고 한다.

> 💡**하이버네이트 속성**
>
> - **hibernate.dialect: 데이터베이스 방언(Dialect) 설정**


**데이터 타입:** MySQL은 VARCHAR, 오라클 VARCHAR2

**다른 함수명:** MySQL 표준 SUBSTRING(), 오라클 SUBSTR() 

**페이징 처리:** MySQL은 LIMIT, 오라클 ROWNUM


<br>

## 애플리케이션 개발 

- 엔티티 매니저 설정
- 트랜잭션
- 비즈니스 로직 

### 엔티티 매니저 설정 

![Image](https://github.com/user-attachments/assets/6202395e-75bb-4cc9-8e45-93c7ebff3031)


### 1. 엔티티 매니저 팩토리 생성

**Persistence 클래스**를 사용해서 <span style="color:orange;">**엔티티 매니저 팩토리 생성**</span>해서 JPA 사용 준비 

**[흐름]**

- 설정 정보를 읽어서
- JPA 동작을 위한 `기반 객체`를 만든다.

➡︎ JPA 구현체에 따라서는 **데이터베이스 커넥션 풀도 생성**하므로 <ins>**엔티티 매니지 팩토리를 생성하는 비용은 아주 크다.**</ins> 

따라서, 엔티티 매니저 팩토리는 애플리케이션 전체에서 <span style="color:#F05750;">**딱 한번 생성 후 공유해서 사용**</span>한다. 

<br>

### 2. 엔티티 매니저 생성 

`JPA의 기능 대부분(CURD)`은 <span style="color:orange;">**엔티티 매니저가 제공**</span>한다.

엔티티 매니저는 내부 **데이터소스(데이터베이스 커넥션)** 를 유지하면서 데이터베이스와 통신한다. 

>  ⚠️ 엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드간에 <span style="color:#F05750;">**공유하거나 재사용하면 안된다!**</span>

엔티티 매니저는 **반드시 종료**해야한다. 

종료하지 않으면 커넥션 풀에 리소스가 계속 쌓이면서 서버 성능저하 및 부하를 일으킨다. 


<br>

### 3. 트랜잭션 관리 

**JPA 사용시** 항상 <span style="color:orange;">**트랜잭션 안에서 데이터를 변경**</span>해야한다. 

비즈니스 로직이 **정상 동작**하면 트랜잭션을 <ins>**커밋(Commit)**</ins>하고 **예외 발생**하면 트랜잭션 <ins>**롤백(rollback)**</ins>한다.

<br>

### 4. 비즈니스 로직 

CRUD 구현


<br>

### 5. JPQL

