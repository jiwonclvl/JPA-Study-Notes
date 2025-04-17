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




<br>  

## OSIV

<br>

## 너무 엄격한 계층

