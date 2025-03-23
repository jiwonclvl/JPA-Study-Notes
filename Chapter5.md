# Chapter5 연관관계 매핑 기초 

<br>

# 목차

> **[1. 단방향 연관관계](#단방향-연관관계)**
>
> **[2. 연관관계 사용](#연관관계-사용)**
>
> **[3. 양방향 연관관계](#양방향-연관관계)**
>
> **[4. 연관관계의 주인](#연관관계의-주인)**
>
> **[5. 양방향 연관관계 저장](#양방향-연관관계-저장)**
>
> **[6. 양방향 연관관계의 주의점](#양방향-연관관계의-주의점)**


<br>

>  💡 방향 (Direction)
> 
> <ins>한쪽만 참조하는 </ins> **단방향**과 <ins>양쪽 모두 서로 참조하는</ins> **양방향**이 있다. 
> 
> 🔎 **객체관계에만 존재**하고 테이블 관계는 항상 양방향이다. 

<br>

>  💡 다중성 (Multiplicity)
> 
> **`1:1`, `N:1`, `1:N`, `N:N`**

<br>

>  💡연관관계의 주인 (Owner)
> 
> 객체를 양방향 관계로 만들면 연관관계의 주인을 정해야한다. 


<br>


## 단방향 연관관계

### 객체의 연관관계
![Image](https://github.com/user-attachments/assets/d43fa506-9822-4e22-96db-706e2e157733)

`Member`는 **team 필드를 통해 Team을 알 수 있지만** `Team`은 **Member를 알 수 없다.** 

➡️ 참조를 통한 연관관계는 언제나 단방향이기 때문에 객체간의 연관관계에서 양방향을 만들고 싶으면 **반대쪽에도 필드를 추가해서 참조를 보관해야한다.** 

⚠️ 양방향 관계가 아니라 <ins>**서로 다른 단방향 관계 2개인 점**</ins>을 명심해야한다. 

<br>

### 테이블의 연관관계

![Image](https://github.com/user-attachments/assets/783c1b98-dcf3-4643-9ad6-d5b382e19ba3)

**테이블의 연관관계에서는 외래키를 통해 관계를 맺기 때문에** 회원과 팀은 조인해서 조회할 수 있다. 


<br>

### JPA를 통한 매핑

```java
public class Member {
    
    ...
    
    @ManyToOne // 다대일 
    @JoinColums(name = "team_id") // 외래키를 매핑할 때 사용한다. (키 이름을 지정)
    private Team team; 
}
```
> **💡 @JoinColums 생략 가능하다.**
>
> 생략 시 ➡️ **필드명 + _ + 테이블 컬럼명**


<br>

> **@ManyToOne (FetchType.EAGER) 즉시 로딩**
> 
> **@ManyToOne (FetchType.LAZY) 지연 로딩**

<br>

```java
import java.util.ArrayList;

public class Team {
    
    ...

    @OneToMany //일대다
    private List<Member> members = new ArrayList<>(); // Team은 여러 회원과 관계를 맺을 수 있다. 
}
```
>  **⚠️ @OneToMany에서 단방향 매핑을 할 경우**
> 
>   @JoinColumn을 명시해야한다.
> 
> ➡️  명시하지 않으면 JPA는 연관관계를 관리하는 **조인 테이블 전략으로 매핑**한다. 
> 
> 🚨 OneToMany 단방향 방법은 다른 테이블의 외래 키를 관리해야 한다. 
 
<br>

## 연관관계 사용

### 저장

JPA는 **참조한 식별자(예시: team_id)를 외래 키로 사용**해서 적절한 등록 쿼리를 생성한다. 

<br>

### 조회

**객체 그래프 탐색 [방법1]**

`member.getTeam()`을 통해 member와 연관된 team 엔티티를 조회할 수 있다. 

<br>

**객체 지향 쿼리 사용(JPQL) [방법2]**

회원이 팀과 관계를 가지고 있기 때문에 필드를 통해 Member와 Team을 조인한다. 

```jpaql
select m from Member m join m.team t 
where t.name=:teamName
```

`:`로 시작하는 것은 **파라미터를 바인딩받는 문법**이다. 

> **💡 객체(엔티티)를 대상으로 하고 SQL보다 간결하다.** 

<br>

### 수정

수정의 경우 Dirty Checking을 통해 JPA가 알아서 처리해준다. 


<br>

### 삭제

#### 연관관계 제거 

참조이기 때문에 주소를 <ins>**null로 설정 시 관계가 제거된다.**</ins> 

`member.deleteTeam(null)`를 통해 연관관계를 제거한다. 

<br>

#### 연관된 엔티티 삭제 

기존의 연관관계를 위와 같이 제거하고 삭제해야한다.  

➡️ **외래 키 제약 조건으로 인해 데이터베이스에서 오류가 발생**한다. 

`em.remove(team)`를 통해 삭제한다. 


> ### **💡 외래 키 제약 조건**
> 
> 데이터베이스에서는 **참조 무결성을 보장하기 위해** <ins>**자식 테이블(Member)이 부모 테이블(Team)의 데이터를 참조하고 있는 경우, 부모 데이터를 먼저 삭제할 수 없다.**</ins> 
> 
> 연관관계를 제거하지 않고 team을 삭제할 경우 Member 입장에서는 존재하지 않는 부모의 외래 키를 참조하는 것이기 때문에 제약 조건에 위배된다.  


<br>


## 양방향 연관관계

```java
public class Member {
    
    ...
    
    @ManyToOne 
    @JoinColums(name = "team_id")
    private Team team; 
}
```

```java
import java.util.ArrayList;

public class Team {
    
    ...

    @OneToMany (mappedBy = "team") //반대쪽 매핑의 필드 이름을 값으로 주면된다. 
    private List<Member> members = new ArrayList<>();
}
```
<br>

## 연관관계의 주인

객체에는 양방향 연관관계라는 것은 없다. 

**서로 다른 단방향 연관관계 2개**를 애플리케이션 로직으로 묶어서 양방향인 것처럼 보이게 할 뿐이다. 

따라서 객체의 연관관께는 **관리해야하는 포인트가 2곳**이 된다. 

➡️ 두 객체 연관관계 중 하나를 정해서 <ins>**테이블의 외래 키를 관리하는 연관관계 주인으로 설정**</ins>해야한다. 

<br>

### 연관관계의 주인 


`연관관계의 주인`만이 **데이터베이스 연관관계와 매핑되고 외래 키를 관리**할 수 있다. 반면 <ins>**주인이 아닌 쪽은 읽기만 가능하다.**</ins> 

### **🗨️ 주인은 어떻게 결정해야할까?**

**항상 다 쪽이 외래 키를 가진다. (@ManyToOne)**

![Image](https://github.com/user-attachments/assets/f6b3246c-a24d-4e0e-b122-3b593e0f8037)

<br>

## 양방향 연관관계 저장

생성된 Team을 Member에 전달해주기만 하면 값을 따로 설정하지 않아도 **데이터베이스에 외래 키 값이 정상 입력**된다. 

 ⚠️ **연관관계의 주인이 아닌 곳에서 입력된 값은 외래 키에 영향을 주지 못한다.** 

## 양방향 연관관계의 주의점

주인에는 값을 입력하지 않고 주인이 아닌 곳에만 값을 입력하면 주인에는 아무런 값도 저장되지 않는다. 

>  ### **🗨️ 그렇다면 주인에만 값을 저장하고 주인이 아닌 곳에는 값을 저장하지 않아도 될까?**
> 
> 객체 관점에서는 **양쪽 방향에 모두 값을 입력해주는 것이 가장 안전**하다. 


### 연관관계 편의 메소드

주인 엔티티에서 양방향 설정을 모두 한다. 

```java
public class Member {
    @ManyToOne
    @JoinColums(name = "team_id")
    private Team team;
}
public void saveTeam(Team team) {
    this.team = team; // Team 객체 저장
    team.getMembers.add(this); //Team의 members 필드에 Member 객체 추가 
}
```

<br>

> **⚠️ team을 변경할 경우**
> 
> 
> 
> ```java
> member.saveTeam(teamA);
> member.saveTeam(teamB);
> Member findMember = teamA.getMember(); // 조회된다. 
> ```
> 
> Member에서 편의 메소드를 통해 양방향 설정을 모두 해주기 때문에 `값이 변경될 경우` **Team에서는 변경 이전에 저장해 두었던 teamA가 그대로 저장**되어 있게 된다. 
> 
> ![Image](https://github.com/user-attachments/assets/f8a69ba5-9ecc-4eab-9f47-d77e7a918e4c)
> 
> 따라서 삭제하는 코드를 추가해주어야 한다. 
> 
> 💡 `영속성 컨텍스트가 다시 생성된다면` 위의 **관계가 문제되지 않지만**, `영속성 컨텍스트가 계속 유지된 상태`라면 **관계가 끊겼음에도 teamA를 조회하면 member가 조회**되어 <ins>**영속성 컨텍스트와 DB 상태가 일치하지 않는 문제가 발생한다.** </ins> 

<br>


### 양방향은 객체 그래프 탐색 기능이 필요할 때 사용하도록 하는 것이 좋다. 

### 양방향 매핑 시에는 무한 루프에 빠지지 않도록 조심해야한다. (예시 찾아보기)