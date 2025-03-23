# Chapter6 다양한 연관관계 매핑  

<br>

# 목차

> **[1. 일대일](#일대일)**
>
> **[2. 다대다](#다대다)**


<br>

>  **💡 다중성**
> 
> - **다대일 (@ManyToOne)**
> - **일대다 (@OneToMany)**
> - **일대일 (@OneToOne)**
> - **다대다 (@ManyToMany)** **실무에서 사용 ❌**

### 📌 다대일, 일대다는 생략하였다. 

<br>

## 일대일

**양쪽이 서로 하나의 관계만 가진다. [예시: 회원과 사물함]**

양쪽 테이블 중 **아무곳에서 외래키를 가질 수 있다.** 

<br>

### 주 테이블에서 외래 키 관리 ➡️ 객체지향 개발자들이 선호 

주 테이블에서 외래 키를 관리한다면 **외래 키를 객체의 참조와 비슷하게 사용**할 수 있어 <ins>**주 테이블만 확인해도 대상 테이블과의 연관관계를 확인할 수 있다.** </ins>

### 단방향

```java
public class Member {
    @OneToOne
    @JoinColumn(name = "locker_id") 
    private Locker locker;
}
```

<br>

### 양방향

```java
public class Locker {
    //Member가 외래키를 가지기 때문에 주인으로 선정 
    @OneToOne (mappedBy = "locker") //주인이 아님을 선언 
    private Member member;
}
```

<br>

### 대상 테이블에서 외래 키 관리  ➡️ 데이터베이스 개발자들이 선호

대상 테이블에 외래 키를 두는 것은 변경에 유리하다.

**[예시: 팀으로 사물함을 사용하는 경우 ➡️ 일대다로 변경될 수 있다.]**

<br>

### 단방향

**대상 테이블에 외래키가 있는데** `일대다를 단방향으로 설정하는 것`은 허용하지 않는다. 

<br>

### 양방향

주 엔티티인 Member를 대신해서 Locker를 주인으로 만들고 싶은 경우 아래와 같이 하면된다. 

```java
public class Member {
    @OneToOne (mappedBy = "member")
    private Locker locker;
}
```
```java
public class Locker {
    @OneToOne
    @JoinColumn(name = "member_id")
    private Member member;
}
```

> ⚠️ 프록시를 사용할 때 외래 키를 직접 관리하지 않는 일대일 관계는 지연 로딩으로 설정해도 즉시 로딩된다. ➡️ 이해안감 (프록시 공부하고 다시 알아봐야 할 것 같다.)

<br>



## 다대다

데이터베이스에서는 다대다인 경우 중간 테이블을 두어 관계를 나타낸다. 

![Image](https://github.com/user-attachments/assets/a94ad98e-8467-4923-b77a-c94450c0964a)


반면, 객체는 컬렉션을 사용하여 객체 2개로 다대다 관계를 만들 수 있다. 

<br>

### 단방향

```java
import java.util.ArrayList;

public class Member {
    @ManyToMany
    @JoinTable(
            name = "member_product", //중간 테이블 매핑 (따로 엔티티를 만들 필요 ❌)
            joinColumn(name = "member_id"), //회원과 매핑할 조인 컬럼 정보
            inverseJoinColumns = @JoinColumn(name = "product_id") //상품과 매핑할 조인 컬럼 정보 
    )
    private List<Product> products = new ArrayList<>();
}
```

<br>

### 양방향

```java
import java.lang.reflect.Member;
import java.util.ArrayList;

public class Product {
    @ManyToMany (mappedBy = "products")
    private List<Member> members = new ArrayList<>();
}
```

<br>

### 매핑의 한계와 극복, 연결 엔티티 사용 

위와 같은 방법을 사용하면 **연결 테이블을 자동으로 처리해주므로 도메인 모델이 단순해지고 여러가지로 편리**하다. 

하지만 **실무에서 사용하기에는 한계**가 있다.  ➡️ 보통 **연결 테이블**에 <ins>`주문 수량`, `날짜` 등의 더 많은 정보가 담기기 때문에</ins> 

<br>

**[연결 테이블]**

```java
//식별 관계 
@IdClass(MemberProductId.class) // 복합 기본 키 매핑 (식별자 클래스 지정)
public class MemberProduct {
    @Id
    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member; //기본 키이자 외래 키

    @Id
    @ManyToOne
    @JoinColumn(name = "product_id")
    private Product product; //기본 키이자 외래 키
    
    
}
```

### 복합키, 식별 관계, @IdClass, @EmbeddedId에 대한 내용은 Chapter7에 정리 

<br>

### 추천하는 기본 키 생성 전략 

데이터베이스에서 자동으로 생성해주는 대리 키를 Long값으로 사용하는 것이다. 

➡️ 간편하고 영구히 쓸 수 있으며 비즈니스에 의존하지 않는다. 

```java
// 비식별 관계
public class MemberProduct {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member; //기본 키이자 외래 키

    @ManyToOne
    @JoinColumn(name = "product_id")
    private Product product; //기본 키이자 외래 키
    
    
}
```