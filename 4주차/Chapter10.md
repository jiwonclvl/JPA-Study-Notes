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
select m //m만 조회해도 연관 객체를 조회할 수 있다.
from Member m join fetch m.team
```


**💬 회원과 팀을 지연로딩으로 설정했다고 가정**

회원을 페치 조인으로 조회 시 연관된 팀도 함께 조회되면 `Team엔티티`는 프록시가 아닌 <ins>실제 엔티티</ins>이다. 따라서 
**연관된 팀을 사용해도 지연 로딩이 일어나지 않는다.** 

**➡️ 즉, 이미 조회되었기 때문에 추가의 쿼리가 나가지 않아도 된다.** 


#### 💡 하나의 쿼리로 모든 데이터를 조회할 수 있다. 



실제 엔티티이기 때문에 **회원 엔티티가 영속성 컨텍스트에서 분리되어 준영속 상태가 되어도 연관된 팀을 조회할 수 있다.** 

<br>


#### 컬렉션 페치 조인 

일대다 관계에서 팀을 통해 회원을 조회할 수도 있다. 

![Image](https://github.com/user-attachments/assets/1c64ce16-ed62-474f-be7c-b681c6a09c97)

Team에서는 하나의 데이터만 있지만 Member는 FK로 여러개의 값을 가지기 때문에 **일대다의 경우에는 결과가 증가되어 조회될 수 있다.** 

<br>

#### 패치 조인과 DISTINCT

일대다 처럼 여러개가 조회되는 경우에는 DISTINCT를 통해 중복되는 Team 데이터를 제거할 수 있다. 


<br>

### 페치 조인과 일반 조인의 차이 

`일반 조인`을 사용한다면 JPQL은 결과 반환 시 연관관계까지 고려하지 않는다. **단지 SELECT 절에 지정한 엔티티만 조회**한다. 

또한, `일대다의 컬렉션`을 <ins>지연 로딩</ins>으로 조회할 경우에는 **프록시**나 **아직 초기화되지 않은  컬렉션 래퍼**를 반환한다. ➡️ 호출되기를 기다리는 상태

<ins>즉시 로딩</ins>의 경우에는 ⚠️**쿼리문을 하나 더 실행**하게 된다. **(N + 1 문제와 이어진다.)**


<br>

### 페치 조인의 특징과 한계 

페치 조인을 사용하면 연관된 객체를 한번에 조회할 수 있어서 SQL 호출 횟수를 줄여 성능 최적화를 할 수 있다. 

```java
@ManyToONe(fetch = FetchType.LAZY) //전체적으로 영향을 미치는 글로벌 로딩 전략이다. 
```

`페치 조인`은` 글로벌 로딩 전략`보다 **우선시되기 때문에 글로벌로 지연로딩을 설정해도 페치 조인을 할 경우 연관된 객체는 지연로딩이 적용되지 않고 조회**된다.  

> **⚠️ 즉시 로딩**
> 
> `즉시 로딩`은 <ins>사용하지 않는 엔티티를 자주 로딩</ins>하기 때문에 **성능에 악영향을 미칠 수 있다.** 
> 
> ➡️ **따라서 될 수 있으면 지연로딩을 사용하는 것이 좋다.**  


<br>

**[한계]**

- 페치 조인 대상에는 별칭을 줄 수 없다. 
- 둘 이상의 컬렉션을 페치할 수 없다. 
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다. 

<br>

### 경로 표현식

#### 경로 표현식 용어 정리

- 상태 필드 (State Field): 값을 저장하기 위한 필드 (필드 or 프로퍼티)
- 연관 필드 (Association Field): 연관관계를 위한 필드, 임베디드 타입 포함 (필드 or 프로퍼티)
  - 단일 값 연관 필드: @ManyToOne, @OneToOne, 대상이 엔티티 
  - 컬렉션 값 연관 필드 : @OneToMany, @ManyToMany, 대상이 컬렉션

<br>

#### 경로 표현식과 특징 

- 상태 필드 경로: 경로 탐색의 끝으로 **더 이상 탐색 할 수 없다.** 
- 단일 값 연관 경로: 묵시적으로 내부 조인이 일어난다. **계속 탐색 가능**
- 컬렉션 값 연관 경로: 묵시적으로 내부 조인이 일어난다. 더는 **탐색 불가능 (별칭으로 탐색은 가능하다.)**

> **💡 묵시적 조인** 
> 
> 명시적 조인은 직접 JOIN을 적어주는 것을 의미하며, `묵시적 조인`은 **경로 표현식에 의해 묵시적으로 조인이 일어나는 것을 의미**한다. 

> **⚠️ 묵시적 조인의 주의 사항** 
> 
> - 항상 내부 조인이다. 
> - 컬렉션에서 경로 탐색을 하려면 명시적으로 조인을 해주어야한다. 
> - SQL의 FROM절에 영향을 준다. 
> 
> ➡️ 묵시적 조인은 조인이 일어나는 상황을 정확히 파악할 수 없기 때문에 **가능하면 명시적 조인을 사용하는 것이 좋다.** 

<br>

### 서브 쿼리 

#### 서브 쿼리 함수 

`exists (서브 쿼리)`: 서브 쿼리 결과가 존재하면 참이다.


`ALL`: 조건을 모두 만족하면 참
`ANY or SOME`: 조건을 하나라도 만족하면 참
`IN`: 서브쿼리 결과 중 하나라도 있으면 참

<br>


## 388 ~ 402 page 문법 내용 참고하기 

<br>

### Named쿼리

JPQL은 동적 쿼리와 정적 쿼리로 나뉜다. 

- 동적 쿼리: JPQL을 문자로 완성해서 직접 넘기는 것을 의미하며, 런타임에 특정 조건에 따라 JPQL을 동적으로 구성
- 정적 쿼리: 한번 정의하면 변경할 수 없는 쿼리를 의미한다. [Named 쿼리]

➡️ Named 쿼리는 @NamedQuery 어노테이션을 사용한다. 




