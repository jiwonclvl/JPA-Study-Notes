# Chapter1 JPA 소개 


<br>

# 목차

> **[1. SQL을 다룰 때 발생하는 문제점](#SQL을-다룰-때-발생하는-문제점)**
> 
> **[2. 패러다임 불일치](#-패터다임-불일치-)**
> 
> **[3. JPA란 무엇인가?](#-jpajava-persistence-api란-)**


## JPA는 <ins>ORM(Object-Relational-Mapping) 기술 표준</ins>이다. 


> ### 🔎 **JPA의 장점**
> 
> - CRUD **SQL을 작성할 필요가 없다.** 
> - 조회된 객체의 <span style="color:orange;">**Mapping 작업도 자동으로 처리**</span>해준다. 
> -  SQL이 아닌 `객체 중심의 개발`로 **생산성 및 유지보수가 좋아지고 테스트 작성도 좋아진다.** 


<br>

### SQL을 다룰 때 발생하는 문제점 

> ### **🚨 SQL 직접 다룰 때 발생하는 문제점**  <br>
>
> - **진정한 의미의 계층 분활이 어렵다.**
>
> - **엔티티를 신뢰할 수 없다.**
>
> - **SQL에 의존적인 개발을 피하기 어렵다.**

<br>

**애플리케이션은 관계형 데이터베이스를 데이터 저장소로 사용**한다.

![Image](https://github.com/user-attachments/assets/e9ec9960-6446-4a43-974f-0ec566dfa43b)

<br>


**[JDBC를 활용한 회원 조회 및 등록]**  

```java
//조회용 SQL 작성 
SELECT MEMBER_ID, NAME FROM MEMBER M WHERE MEMBER_ID = ? 

// 작성한 SQL 실행 및 결과값 반환 
ResultSet rs = stmt.executeQuery(sql); 

//조회 결과를 객체로 Mapping
String memnerId = rs.getString("MENBER_ID");
String name = rs.getString("NAME");

Member member = new Member();
member.setMemberId(memnerId);
member.serName(name);

... 

//등록 SQL 작성

//Member 객체에서 값을 꺼내 SQL에 전달 및 실행 

```

현재는 CRUD의 C부분만 구현을 한 상황이지만 CRUD를 모두 구현하려고 한다면 <span style="color:#F05750;">**너무 많은 SQL과 JDBC API를 코드로 작성**</span>해야 한다. 


<br>

### SQL에 의존적인 개발 

**💭 위와 같이 애플리케이션의 기능을 모두 만든 상황에서 `추가적인 요구 사항이 생긴다면`?**

➡︎  `자바 컬렉션`에 저장된 객체라면 <span style="color:orange;">**로직만 수정**</span>하면 되지만, `관계형 데이터베이스`를 사용하면 <span style="color:orange;">**SQL을 수정해야 하고, 관련된 코드도 변경**</span>해야 한다.

<br>

**💭 위와 같이 애플리케이션의 기능을 모두 만든 상황에서 `연관된 객체가 있다면`?**

➡︎  연관된 객체의 사용 여부는 SQL에 달려 있다. 즉, <span style="color:#F05750;">**DAO를 열어서 어떤 SQL이 샐행되는지 확인**</span>해야 한다


<br>

## 🔎 JPA를 통한 문제 해결 


### JPA가 제공하는 API를 사용하면 된다. 

<br>

**[저장 기능]**


```java
jpa.persist() //저장 
```
➡︎ 메소드 호출 시 **JPA가 객체와 매핑 정보를 보고 적절한 INSERT SQL 생성해서 데이터 베이스에 저장**

>💡 `매핑정보:` **어떤 객체를 어떤 테이블에 관리할지 정의한 정보 (Id)**

<br>


**[조회 기능]**


```java
jpa.find(객체, 매핑 정보);
```

<br>

**[수정 기능]**

**별도의 수정 메서드를 제공하지 않는다.** 

➡︎ 객체를 조회해서 값을 변경만 하면 <span style="color:orange;">**트랜잭션 커밋 시**</span> **UPDATE SQL이 전달된다.** 

<br>

## ⚠️ 패터다임 불일치 

`객체`와 `관계형 데이터베이스`는 <ins>**지향하는 목적이 서로 다르므로 둘의 기능과 표현 방법도 다르다.**</ins>  ➡︎ <span style="color:#F05750;">**패러다임 불일치 문제**</span>


<br>

>  ### 💡  **객체는 속성(필드)과 기능(메소드)을 가진다.** 
> 
> ```java
> public class User () {
>   //속성
>   private final Long userId;
>   private final String userName;
> 
>   //기능 
>   public void updateUserName() {
>       ...
>   }
> }
> ```
> 
> ---
>
> ### 💭 만약 부모 객체를 상속받았거나 다른 객체를 참조하고 있다면?
> 
> ➡︎ **객체의 상태를 저장하는 것은 쉽지 않다.** 
> 
> 따라서, 자바에서는 이러한 문제를 고려하여 `객체를 파일`로 저장하는 <span style="color:orange;">**직렬화 기능**</span>과 저장된 `파일을 객체`로 변환하는 <span style="color:orange;">**역 직렬화 기능**</span>을 지원한다. 

<br>

### 🚨 객체 VS 테이블 차이점 1 (문제1)

`객체`는 <ins>상속이라는 기능을 가지고 있지만</ins> `테이블`은 <ins>상속이라는 기능을 가지고 있지 않다.</ins> 

데이터베이스 모델링에서 이야기하는 **슈퍼타입과 서브타입관계**를 통해 **상속과 유사한 형태로 테이블을 설계**할 수 있다. 


![Image](https://github.com/user-attachments/assets/7ff32747-9500-4d8a-87b2-766990e6ca0e)


> ITEM 테이블에는 모든 **서브 타입이 공통적**으로 가지는 `id`, `name`, `price`를 포함
> 
> ITEM 테이블의 `DTYPE 컬럼`은 **어떤 서브타입인지 구분하는 식별자**이다. 


<br>

상속과 유사한 형태를 만들기 위해 위와 같이 코드를 구성한다면 데이터를 조회 시 **테이블 JOIN 후** 그 결과 **객체를 변환하는 추가적인 작업이 필요**하다.   ➡︎ <span style="color:#F05750;">**추가적인 비용 발생** </span>


###  🔎 JPA는 `컬렉션에 객체를 저장`하듯이 `JPA에 객체를 저장`하여 위와 같은 문제를 해결한다. 


<br>

### 🚨 객체 VS 테이블 차이점 2 (문제2)

`객체`는 <span style="color:orange;">**참조를 사용해서 다른 객체와 연관관계**</span>를 가지고 참조에 접근하여 연관된 객체를 조회하지만 `테이블`은 <span style="color:orange;">**외래 키를 사용해서 다른 테이블과 연관관계**</span>를 가지고 조인을 사용해서 연관된 테이블을 조회한다. 

<br>


**[객체의 연관관계]**

![Image](https://github.com/user-attachments/assets/68190866-3e0c-4cbf-8c0d-950605eb9873)

`Member 필드에 Team 객체의 주소를 보관`하고 Team 객체와 관계를 맺는다. 따라서 **조회 시 참조 필드에 접근**하면 된다. 

<br>


**[테이블의 연관관계]**

![Image](https://github.com/user-attachments/assets/8f53cb04-81cb-4fcb-a416-c262b73b7529)

Member 테이블은 `TEAM_ID 외래 키`를 사용해서 Team 테이블과 연관관계를 맺는다. 따라서 **외래 키를 이용해 두 테이블을 Join하면 조회**할 수 있다.  


>  - **객체**는 <span style="color:#F05750;">**참조가 있는 방향으로만 조회 가능**</span>  (Team클래스에서는 Member가 참조되지 않는다.) 
>  - **테이블**은 외래 키 하나로 양쪽 접근 가능 
> 
>  ###  🚨 **아래와 테이블 중심으로 코드를 작성한다면** 객체지향적인 방법으로는 <span style="color:#F05750;">조회가 불가능</span>하다. 
> 
> ```java
>   class Member {
>       Long memberId;
>       Long teamId; // 참조 X 따라서, Team 객체 조회 불가능 
> }
> ```
> 
> ### 💡  따라서 **객체 중심으로 코드를 작성**해야한다. 
> 
> ```java
>   class Member {
>       Long memberId;
>       Team team; // 참조 O 따라서, Team 객체 조회 가능 
> }
> ```
> 다만, 객체 중심으로 코드를 작성하면 <span style="color:#F05750;">**저장 및 조회가 쉽지 않다.**</span> 그래서 결국 개발자가 변환 과정을 거쳐야 한다. 

> **[변환 과정]**
>
> **저장**
> 
> **Member 테이블에는 참조 객체가 아닌 TEAM_ID 외래 키가 저장**되어야 한다. 
> 
> ➡︎ 따라서, team 필드를 TEAM_ID 외래 키 값으로 변환 `member.getTeam().getId()`
> 
> **조회**
> 
> **TEAM_ID 외래 키 값을 Member 객체의 team 참조로 변환한 후 객체에 보관**해야한다. 

<br>

### 위의 모든 과정들은 <ins>패러다임 불일치를 해결하기 위해 소모하는 비용</ins>이다. 

###  🔎 JPA는 `컬렉션에 객체를 저장`하듯이 `JPA에 객체를 저장`하여 위와 같은 문제를 해결한다. 


<br>


### 객체 그래프 탐색 (문제3)

**참조를 사용하여 연관된 팀을 찾는 것**을 <span style="color:orange;">**객체 그래프 탐색**</span>이라고 한다.

![Image](https://github.com/user-attachments/assets/2cef256e-9467-442a-b43d-64b226009ad1)

**객체는 마음껏 객체 그래프를 탐색할 수 있어야 한다.**

<br>

### 💭 SQL을 통해 객체 탐색을 한다면? SQL에 따라 탐색 범위가 정해진다
```mysql
SELECT M.*, T.*
    FROM MEMBER M
    JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```
**Team과 Member 데이터만 조회 가능**하고 <span style="color:#F05750;">**다른 객체는 탐색이 불가능**</span> 하다. 

<br>

```java 
Member member = memberDao.find(memberId);
```

#### **위의 코드를 봤을 때 어떤 것을 탐색할 수 있을지 알 수 있을까?** 🚨NO🚨

결국, DAO를 열어봐야지만 어떤 것을 탐색하는지 확인할 수 있다. 
이는 <span style="color:#F05750;">**Entity가 SQL에 논리적으로 종속되어 발생하는 문제**</span>이다. 

<br>

###  🔎 JPA는 연관된 객체를 `사용하는 시점에 SQL을 실행`한다. 따라서 JPA를 사용하면 <ins>연관된 객체를 신뢰하고 마음껏 조회할 수 있다.</ins> 

--- 

위의 사진에서 Member 객체는 Team과 Order를 참조하는 필드를 가지고 있다. 

따라서 **find(memberId)로 Member 객체를 조회**해 두면 이후 `필요한 시점`에 해당 참조 필드를 통해 Order와 Team, 그리고 해당 필드가 참조하고 있는 객체까지 가져와서 자유롭게 사용할 수 있다.

즉, **JPA를 사용하면 연관된 객체를 손쉽게 조회**할 수 있다.



> ### 💡`실제 객체를 사용하는 시점`까지 데이터베이스 **조회를 미룬다**고 해서 <span style="color:orange;">**지연로딩**</span>이라고 한다. 
> 🔎 **JPA는 지연로딩을 Transparent하게 처리한다.**
> 
> ➡︎ JPA가 내부적으로 `프록시(proxy)를 사용하여 마치 실제 객체를 사용하는 것`처럼 보이게 하면서 **필요한 시점까지 데이터베이스 조회를 지연시키는 방식**을 의미한다. 



<br>

### 🚨 객체 VS 테이블 차이점 3 (문제4)

`객체`는 <ins>**동일성(Identity)비교와 동등성(equality)비교로 두 값을 비교**</ins>하고 `데이터베이스`는 **<ins>기본 키의 값으로 각 로우(Row)를 구분**</ins>한다. 


```java 
Member member1 = memberDao.getMember(memberId);
Member member2 = memberDao.getMember(memberId);

member1 == member2;
```

**같은 데이터베이스의 Row를 조회**했지만 `객체 측면`에서는 `memberDao.getMember(memberId)` 호출할 때마다 **새로운 인스턴스가 생성**되기 때문에 <span style="color:orange;">**== 결과가 false**</span>로 나온다. 

<br>

###  🔎 JPA는 <ins>같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장</ins>한다. 

> ### 💡`@Transactional`이 적용된 서비스 메서드에서 엔티티를 조회하면 해당 엔티티는 영속성 컨텍스트에 저장된다.
> 같은 트랜잭션 내에서 동일한 엔티티를 다시 조회하면 데이터베이스에서 가져오지 않고 영속성 컨텍스트에서 조회된 객체를 반환한다.  
> 
> 따라서 항상 같은 인스턴스를 보장함


<br>

## 💭 JPA(Java Persistence API)란? 

**ORM 기술 표준 [객체 ↔ 관계형 데이터 베이스 Mapping]** 

➡︎  이를 통해 **패러다임 불일치 문제를 개발자 대신 해결**해준다. 


### 🔎 객체를 마치 <ins>자바 컬렉션에 저장하듯이 ORM 프레임워크에 저장</ins>한다. 

![Image](https://github.com/user-attachments/assets/2d196f01-9917-465e-8be9-8004db3ceac4)

<br>

> 💡 `하이버네이트` **거의 대부분의 패러다임 불일치 문제를 해결해주는 성숙한 ORM 프레임워크**이다. 


<br>

### 💭 JPA를 사용해야하는 이유는? 

### **생산성**

<br>

### **유지보수**

  ➡︎ JPA가 패러다임 문제를 해결해주기 때문에 **객체지향 언어가 가지는 유연하고 유지보수하기 좋은 도메인 모델을 편리하게 사용**할 수 있다. 

<br>


### 패러다임 불일치 해결

<br>

### **성능**

  ➡ JPA를 사용한다면 `SELECT문을 한번만 사용`하여 **조회한 객체를 실행 시점에 재사용**할 수 있다.

<br>

### **데이터 접근 추상화 벤더(MySQL, ORCLE, ... 등) 독립성**
![Image](https://github.com/user-attachments/assets/79b7d54a-92e9-419f-9ff6-c3386063bd6a)

관계형 데이터베이스는 같은 기능도 벤더마다 사용법이 다르다. 
따라서 다른 것으로 벤더를 바꾼다면 변경하기가 매우어렵다. 

➡︎ 하지만 그림과 같이 JPA에서는 **애플리케이션이 특정 데이터베이스 기술에 종속되지 않도록 한다.** 따라서 추후 변경에 유연하다. 