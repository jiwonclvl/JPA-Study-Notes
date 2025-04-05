# Chapter10 객체 지향 쿼리 언어

<br>

# 목차

> **[1. 객체지향 쿼리 소개](#객체지향-쿼리-소개)**
>
> **[2. JPQL](#JPQL)**
>
> **[3. Criteria](#Criteria)**
>
> **[4. QueryDSL](#QueryDSL)**
>
> **[5. 네이티브 SQL](#네이티브-SQL)**
> 
> **[6. 객체지향 쿼리 심화](#네이티브-SQL)**

<br>

## 객체지향 쿼리 소개


실제 애플리케이션을 개발할 때 복잡한 조건을 통해 엔티티를 조회하는 경우가 많이 생긴다.

해당 엔티티를 조회하기 위해서는 **데이터베이스에 있는 데이터를 SQL로 적절히 필터링하여 조회**해야한다. 

하지만, `ORM`을 사용하면 `데이터베이스 테이블이 아닌 엔티티 객체를 대상으로 개발`하기 때문에 <ins>검색도 테이블이 아닌 엔티티를 대상으로 하는 방법이 필요</ins>하다. 

**JPA는 이러한 점들을 고려하여 다양한 검색 방법을 제공한다.** 

<br>

### **JPQL (Java Persistence Query Language)**

     엔티티 객체를 조회하는 객체지향 쿼리이다. 

- JPQL은 **SQL을 추상화해서 특정 데이터베이스에 의존하지 않는다.** 


- 방언만 변경하면 <ins>JPQL을 수정하지 않아도 자연스럽게 데이터베이스를 변경</ins>할 수 있다. 
    
    ➡️ 표준화된 함수를 사용하면 **선택한 방언에 따라 적절한 SQL 함수가 실행**된다.  


<br>

```java
//회원 이름이 "kim"인 사람만 조회 
String jpql = "select m from Member as m where m.username = 'kim'"; 

List<Member> resultList = em.createQuery(jpal, Member.class).getResultList();
```

`m.username:` 객체의 필드명

`em.createQuery(jpal, Member.class):` 실행할 jpql과 반환 객체 

`.getResultList():` JPQL을 SQL로 변환해서 데이터베이스 조회 

<br>


### **Criteria 쿼리** 

     JPQL을 편하게 작성하도록 도와주는 API (빌더 클래스 모음)

- 문자가 아닌 **프로그래밍 코드로 JPQL을 작성할 수 있다.** 

    ➡️ `JPQL`은 <ins>문자열</ins>로 값을 받기 때문에 **오타가 있더라도 컴파일 시점에 확인할 수 없지만** `Criteria`를 사용하면 **컴파일 시점에 오류를 발견할 수 있다.** 

**[장점]**

- 컴파일 시점에 오류를 발견할 수 있다. 


- IDE를 사용하면 코드 자동완성을 지원한다. 


- 동적 쿼리를 작성하기 편하다.

<br>

**[단점]**

- 가진 장점은 많지만 모든 것을 상쇄할 정도로 복잡하고 장황하다. 
- 코드가 한눈에 들어오지 않는다. 

<br>


```java
//준비 
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cd.createQuery(Member.class);

//조회를 시작할 클래스
Root<Member> m = query.from(Member.class);

//쿼리 생성
CriteriaQuery<Member> cq = query.select(m)
                                .where(cb.equal(m.get("username"), "kim"));
```

`m.get("username"):` "username"도 문자열로 쓰고 싶지 않다면 **MetaModel**을 사용한다. MetaModel

<br>


### **네이티브 SQL**

     JPA에서 JPQL 대신 직접 SQL을 사용할 수 있다. 

- JPQL에서 지원하지 않는 기능을 사용해야할 때 사용한다. 


-  **특정 데이터베이스에 의존하는 SQL을 작성해야 한다는 단점**이 있다. 

<br>


### QueryDSL [Criteria보다 선호]

    Criteria와 같은 기능을 하며 비표준 오픈소스 프레임워크이다. 

> **💡 비표준 오픈소스 프레임워크**
> 
>   누구나 자유롭게 소스 코드를 보고 **수정, 배포할 수 있는 소프트웨어**

➡️ QueryDSL도 어노테이션 프로세서를 사용해 쿼리 전용 클래스를 만들어야 한다. Q클래스 

<br>

```java
List<Member> members = 
        query.from(member)
            .where(member.username.eq("kim"))
            .list(member);
```

### **JDBC**

    MyBatis 같은 SQL 매퍼 프래임워크를 사용한다. (필요하면 JDBC를 직접 사용할 수 있다.)

- `JDBC`나 `마이바티스`를 **JPA와 함께 사용하면 영속성 컨텍스트를 적절한 시점에 강제로 플러시** 해야한다.



<br>

## JPQL

어떤 방법을 사용하든 JPQL에서 모든 것이 시작한다. 


### 기본 문법과 쿼리 API

SQL과 비슷하기 SELECT, UPDATE, DELETE문을 사용할 수 있다. (persist()를 사용하면 되기 때문에 INTSERT는 존재하지 않는다.)

<br>

#### SELECT문

```jpql
SELECT m FROM Member AS m where m.username = 'Hello'
```

- 엔티티와 속성은 대소문자를 구분한다.  `SELECT m FROM Member AS m...`
- 엔티티명을 사용한다. `Member`
- 별칭은 필수이다. `m`


<br>

#### TypeQuery, Query

JPQL을 실행하려면 쿼리 객체를 만들어야 한다. 

- `반환할 타입을 명확하게 지정`할 수 있으면 ➡️ **TypeQuery** 객체 사용


- `반환 타입을 명확하게 지정`할 수 없으면 ➡️ **Query** 객체 사용


<br>

#### 결과 조회 

- `query.getResultList();` ➡️ 결과가 있으면 **List 반환**, 없으면 **빈 배열 반환**


- `query.getSingleResult();` ➡️ 결과가 정확히 하나일 때 

<br>

### 파라미터 바인딩 [필수]

#### 이름 기준 파라미터 

```java
TypeQuery<Member> query = em.createQuery("SELECT m FROM Member AS m where m.username = :username", Member.class);

query.setPArameter("username", usernameParam); //파라미터 바인딩 
```
---
**[체인 방식]**
```java
TypeQuery<Member> query = em.createQuery("SELECT m FROM Member AS m where m.username = :username", Member.class)
                .setParameter("username", usernameParam)
                .getResultList();
```
<br>

#### 위치 기준 파라미터

```java
TypeQuery<Member> query = em.createQuery("SELECT m FROM Member AS m where m.username = ?1", Member.class)
                .setParameter("username", usernameParam)
                .getResultList();
```


➡️ 위치 기준 방식보다는 **이름 기준 파라미터 바인딩 방식을 사용하는 것이 더 명확**하다.



> **🚨 인젝션 공격**
> 
> 파라미터 바인딩을 쓰지않고 **집접 문자를 입력할 경우** 악의적인 사용자에 의해 <ins>인젝션 공격을 당할 수 있다.</ins> 

<br>

### 프로젝션

SELECT 절에 조회할 대상을 지정하는 것을 의미한다. 

- **엔티티 프로젝션**

```java
SELECT m FROM Member //엔티티 조회
SELECT m.team FROM Member // 연관된 엔티티 조회
```

 ➡️ 조회한 엔티티는 영속성 컨텍스트에서 관리된다. 

<br>


- **임베디드 타입 프로젝션**

    임베디드 타입은 조회의 시작점이 될 수 없다. (엔티티를 통해서 조회할 수 있다.)

```java
SELECT a FROM Address ❌

SELECT o.address FROM ORDER ⭕
```

➡️ `임베디드`는 엔티티 타입이 아닌 <ins>값 타입</ins>이기 때문에 위와 같이 **직접 조회한 임베디드 타입은 영속성 컨텍스트에서 관리되지 않는다.** 


<br>

- **스칼라 타입 프로젝션**

    숫자, 문자, 날짜와 같은 기본 데이터 타입

<br>

- **여러 값 조회** 

```java
SELECT m.name, m.age FROM Member 
```


#### new 명령어 

SELECT 다음에 NEW 명령어를 사용하면 반환 받을 클래스를 지정할 수 있다. 

- 패키지 명을 포함한 전체 **클래스 명을 입력**해야 한다. 
- 순서와 타입이 일치하는 **생성자가 필요**하다. 


<br>

### 페이징 API 

- `setFirstResult (int startPosition):` 조회 시작 위치(0부터 시작)
- `setMaxResult (int maxResult):` 조회할 데이터 수 


➡️ 데이터베이스마다 다른 페이징 처리를 같은 API로 처리할 수 있는 것은 데이터베이스 방언 덕분이다. 


> 💡 페이징 쿼리 결과는 책 364 ~ 366 page를 참고하면 된다. 


<br>

> ## 📌 사용 문법은 추후 사용할 때 찾아볼 예정이다. (정리를 상세히 하지는 않음)


### 집합과 정렬 

#### 집합 함수 

![Image](https://github.com/user-attachments/assets/fedbdeef-ffea-42c5-906e-e7b0e0568b09)

**[집합 함수 사용시 참고사항]**

- NULL 값은 무시
- 값이 없는데 사용하면 NULL이 된다. 
- DISTINCT를 집합 함수 안에 사용해서 중복된 값을 제거할 수 있따. 
- DISTINCT를 COUNT에서 사용할 때 임베디드 타입은 지원하지 않는다. 


<br>

#### GROUP BY, HAVING, ORDER BY

368 page 참고 


<br>

### JPQL 조인 


#### 내부 조인 (INNER JOIN)

 - INNER는 생략 가능 
 - JPQL에서는 연관된 필드를 사용하여 조인을 한다. 

```java
SELECT m FROM Member m JOIN m.team t
```

<br>

#### 외부 조인 (LEFT OUTER JOIN)

- OUTER는 생략 가능 
- SQL의 외부 조인과 같다. 

<br>

#### 컬렉션 조인 

- 컬렉션을 사용하는 곳에 조인하는 것을 의미한다. 


```java
SELECT t, m FROM Team t JOIN t.member m
```


➡️ Team은 컬렉션의 Member를 가지고 있다. 


<br>

#### 세타 조인

- 세타 조인은 내부 조인만 지원한다. 


<br>

### 페치 조인 

- 성능 최적화를 위해 제공하는 기능이다. 
- 연관된 엔티티나 컬렉션을 한 번에 같이 조회한다. 
- join fetch 명령어를 사용한다.  
- 🚨 **페치 조인은 별칭을 사용할 수 없다. (하이버네이트는 허용)**


#### 엔티티 페치 조인 

```java
select m
from Member m join fetch m.team
```

#### 컬렉션 페치 조인 

#### 패치 조인과 DISTINCT

