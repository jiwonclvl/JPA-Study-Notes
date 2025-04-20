# Chapter13 웹 애플리케이션과 영속성 관리 

<br>

# 목차

> **[1. 트랜잭션 범위의 영속성 컨텍스트](#트랜잭션-범위의-영속성-컨텍스트)**
>
> **[2. 준영속 상태와 지연 로딩](#준영속-상태와-지연-로딩)**
>
> **[3. OSIV](#OSIV)**
>
> **[4. 너무 엄격한 계층](#너무-엄격한-계층)**


<br>

## 트랜잭션 범위의 영속성 컨텍스트

**J2SE 환경**에서 JPA 사용 시 <ins>개발자가 직접 엔티티 매니저 생성 및 트랜잭션 관리 수행</ins>

**J2EE or Spring 환경**에서 JPA 사용 시 <ins>컨테이너가 제공하는 전략을 따라야 한다.<ins> 

<br>

### 스프링 컨테이너의 기본 전략 

트랜잭션 범위의 영속성 컨텍스트 전략을 기본으로 사용 

➡️ **트랜잭션의 범위**와 **영속성 컨텍스트**의 <ins>**생명주기가 같다**</ins>는 의미

  | **개념**       | **생성 시점**       | **소멸 시점**                 | **생존 범위**           |
  |----------|-----------------|---------------------------|---------------------|
  | **트랜잭션**     | `@Transactional` 실행 시 | 메서드 종료 시            | 메서드 단위 (기본)  |
  | **엔티티 매니저**  | 트랜잭션 시작 시       | 트랜잭션 종료 시          | 트랜잭션 단위       |
  | **영속성 컨텍스트** | EntityManager 생성 시 | EntityManager 종료 시     | 트랜잭션 단위       |

> ✅ `같은 트랜잭션 안`에서는 항상 <ins>같은 영속성 컨텍스트에 접근<ins> 

---

<br>

#### @Transactional과 AOP의 관계

스프링 프레임워크를 사용하면 **Service에 @Transactional 어노테이션을 선언해서 트랜잭션을 시작**한다. 

> ⚠️ Service의 메소드를 호출하는 것 처럼 보이지만 **실제는 `메소드 실행 직전`에 스프링 트랜잭션 <ins>AOP가 먼저 동작**</ins>한다. 

![Image](https://github.com/user-attachments/assets/c039b720-cad7-45a6-b2f9-1ce71449eeb6)


➡️ **메서드 실행 전·후로 AOP가 "횡단 관심사"인 트랜잭션을 관리**

> 💡 AOP는 프록시 기반이다.

**[흐름]**

1. 메서드를 호출하기 직전에 트랜잭션 시작 
2. 메소드가 정상 종료되면 트랜잭션 커밋을 하면서 종료


> - **트랜잭션 내에서 예외 발생** ❌ ➡️ JPA는 영속성 컨텍스트를 플러시해서 <ins>변경 내용을 데이터베이스 반영하고 트랜잭션 커밋
> 
> - **트랜잭션 내에서 예외 발생** ⭕ ➡️ 트랜잭션을 <ins>롤백하고 종료 (플러시는 호출 ❌)

 
<br>

#### 🟠 Controller
```java
@RestController
@RequiredArgsConstructor
public class Controller {
    private final Service service;
    
    public void controller() {
        Member member = service.logic(); //반환된 엔티티는 준영속 상태
    }
}
```

 ➡️ **트랜잭션이 종료된 후 영속성 컨텍스트도 종료되기 때문에 준영속 상태가 된다.** 

<br>

#### 🟠 Service
```java
@Service
@RequiredArgsConstructor
public class Service {
    private final Repository repository1;
    private final Repository repository2;

    /**
     * 트랜잭션 시작 시 엔티티 매니저, 영속성 컨텍스트도 생성
     */
    @Transactional
    public Member logic() {
        repository1.logic();
        
        //영속 상태
        Member member = repository2.findMember();
        return member;
    }
}
```

위에서 언급했지만 **트랜잭션 안에서는 항상 동일한 영속성 컨텍스트를 사용**한다. 

➡️ 따라서, `repository1`, `repository2`는 모두 <ins>동일한 영속성 컨텍스트 사용<ins>하게 된다. 

> **🗨️ 만약 Repoitory마다 엔티티 매니저를 주입한다면?**
> 
> ![Image](https://github.com/user-attachments/assets/f44d1ebb-2b43-4d6a-82d3-0fec60201806)


<br>

#### 🟠 Repository
```java
public interface Repository1 extends JpaRepository<Entity, Long> {
    void logic();
}

public interface Repository2 extends JpaRepository<Member, Long> {
  Member findMember();
}
```

➡️  책에서는 @PersistenceContext를 통해 엔티티 매니저를 주입하였지만 위와 같이 **주입을 하지 않는다면** 한 트랜잭션 내에서 <ins>2개의 Repository는 같은 엔티티 매니저를 사용</ins>하게 된다. 

<br>

> **💡 트랜잭션이 다르면 다른 영속성 컨텍스트를 사용한다.**
> 
> ![Image](https://github.com/user-attachments/assets/84021042-21f0-495e-a2c1-ef8d557ae4ca)


> **🤔 헷갈렸던 점** 
> 
> 원래 알고 있었던 엔티티 매니저의 개념은 아래와 같다.  
> 
> **[Chapter3 부분 참고]**
>
> ![Image](https://github.com/user-attachments/assets/d2d6c18c-12ef-4626-af92-86d5fbd77a4e)
> 
> <br>
>
> **[의문 1]**
> 
> 위의 설명에서는 **<ins>같은 엔티티 매니저를 사용해도 트랜잭션이 다르기 때문에 멀티스레드 환경에서 안전**</ins>하다고 설명한다.
> 
> ➡️  이 설명만 보면, **하나의 엔티티 매니저가 여러 스레드에서 공유되는 것처럼** 보였다. 
> 
> <br>
>
> **[의문 2]**
> 
> 영속성 컨텍스트가 만들어지는 시기는 아래와 같다고 생각하였다.
> > **트랜잭션 시작 ➡️  엔티티 매니저 할당(커넥션 풀) ➡️ 영속성 컨텍스트 생성**
> 
> ➡️ **여러 스레드가 하나의 엔티티 매니저를 사용하면 결국 하나의 영속성 컨텍스트를 공유**하게 되는 건 아닐까?하는  생각이 들었다.  
> 
> ---
> **[결론]**
> 
> 👉🏻  `위 상황에서의 엔티티 매니저`는 **어노테이션을 통해 주입받은 프록시 객체**라는 점이고 `실제`로는 **트랙잭션마다 별도의 엔티티 매니저를 사용**한다고 한다. 
> 

<br>

## 준영속 상태와 지연 로딩

트랜잭션은 보통 Service 계층에서 시작되고 끝나기 때문에 컨트롤러나 뷰와 같은 프레젠테이션 계층에서는 준영속 상태가 된다. 

따라서, 변경 감지와 지연로딩이 동작하지 않아 **컨트롤러에서 `getMember()`와 같은 로직을 추가한다면 해당 로직이 호출되는 시점에서 예외가 발생**한다. 

➡️ **initializationException 발생**

> **💡 단순히 데이터를 보여주기만 하는 프레젠테이션 계층에서 수정할 일은 없다.** 

> ⚠️ 프레젠테이션 계층에서 데이터를 수정한다는 것 자체가 <ins>책임이 모호</ins>해지고 너무 <ins>많은 계층에서 데이터를 관리해야하기 때문에 유지보수에도 좋지 않다. </ins>

 
<br>

**[해결 방법]**

- 뷰가 필요한 엔티티를 미리 로딩해두는 방법
- OSIV를 사용해서 엔티티를 항상 영속 상태로 유지하는 방법 

<br>

### 미리 로딩해두는 방법

엔티티가 준영속 상태로 변해도 연관된 엔티티를 이미 다 로딩해 두어서 지연로딩이 발생하지 않는다. 


**[방법]**

- 글로벌 패치 전략
- JPQL페치 조인
- 강제로 초기화

<br>

#### 글로벌 패치 전략 

페치 전략을 지연로딩에서 즉시 로딩으로 변경하는 방법

`fetch = FetchType.Lazy` ➡️ `fetch = FetchType.EAGER`

**[단점]**

- 사용하지 않는 엔티티를 로딩
- 🚨 N + 1문제 발생 (가장 조심해야 하는 문제)

>  **⚠️ N + 1**
> 
> 즉시 로딩 시 Member를 조회할 떄 연관된 객체인 Team도 바로 로딩한다. 
> 
> 이때, 문제는 **JPA가 JPQL을 분석해서 SQL을 생성할 때**는 글로벌 페치 전략을 참고하지 않고 <ins>오직 JPQL 자체만 사용</ins>한다는 것이다. 
> 
> `SELECT m FROM Member` ➡️ Team까지 조회 
> 
> 만약 조회한 Member 엔티티가 10개라면 Team을 조회하는 SQL도 10번 실행된다. 
> 
> 이와 같이 N + 1이 발생하면 SQL이 상당히 많이 노출될 수 있으므로 성능상의 치명적이다. 


<br>  

## OSIV

<br>

## 너무 엄격한 계층

