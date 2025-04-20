# Chapter13 웹 애플리케이션과 영속성 관리 

<br>

# 목차

> **[1. 트랜잭션 범위의 영속성 컨텍스트](#트랜잭션-범위의-영속성-컨텍스트)**
>
> **[2. 준영속 상태와 지연 로딩](#준영속-상태와-지연-로딩)**
>
> **[3. OSIV](#OSIV)**


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
> > `SELECT m FROM Member` 
> > 
> > ➡️ JPQL에는 Team에 대한 명시가 없으므로, 우선 Member만 조회하는 SQL이 실행 
> 
> <br>
>
> > `SELECT m FROM Member WHERE id = ?` 
> > 
> > ➡️ 여러 맴버를 조회한다면 Team 조회 쿼리도 그에 맞춰 추가로 실행되어 N + 1발생
> > 
> > - Member 전체 조회 쿼리 1번
> > - 각 Member가 소속된 Team을 조회하는 쿼리 N번
> 
> 
> 이와 같이 N + 1이 발생하면 SQL이 상당히 많이 노출될 수 있으므로 성능상의 치명적이다.  ➡️ **페치 조인으로 해결 가능** 

<br>  

#### JPQL 페치 조인 

JPQL을 호출하는 시점에 함께 로딩할 엔티티를 선택하는 방법 

>  💡 연관된 **엔티티를 이미 로딩**했으므로 <ins>글로벌 페치 전략이 적용되지 않는다.</ins> 

<br>

**[단점]**

- 무분별하게 사용하면 레포지토리 메소드가 증가할 수 있다. 

**EX** 

1. member만 조회하는 경우
2. member와 연관된 team을 조회하는 경우 

2가지 경우를 모두 만족하기 위해서는 2개의 repository를 만들어야 한다. 

➡️ 최적화는 할 수 있지만 **뷰와 레포지토리 간에 논리적인 의존관계가 발생** 

<br>

> **🤔 뷰와 레포지토리 간에 논리적인 의존관계가 발생**
> 
> 어떤 페이지에서는 Member만 조회 → findMembers()
> 
> 다른 페이지에서는 Member와 연관된 Team도 함께 조회 → findMembersWithTeam()
> 
> 이렇게 되면 Repository에 뷰(혹은 서비스)가 요구하는 형태로 메서드가 점점 늘어나게 된다. 


<br>

#### 강제로 초기화


`영속성 컨텍스트가 살아있을 때` **프레젠테이션 계층이 필요한 엔티티를 강제로 초기화해서 반환**하는 방법

**설정: 지연 로딩을 전략으로 선택**

<br>

**[기본적인 지연 로딩 흐름]** 

```java
@Transactional
public Member findMember(Long memberId) {
    Member member = repository.findMemberById(memberId);
    
    Team team = member.team(); //프록시 객체만 반환(프록시 객체의 주소만 지니는 상태)
  
    ...

    team.getName(); // 프록시 객체 초기화 (실제 엔티티 값으로 초기화됨) 
}
```
<br>

**[강제 초기화]**

```java
@Transactional
public Member findMember(Long memberId) {
    Member member = repository.findMemberById(memberId);
    
    member.team().getName(); //프록시 강제 초기화
    
    return member;
}
```

➡️ 프리젠테이션 객체에서 필요한 객체를 영속성 컨텍스트가 살아 있을 때 강제로 초기화 해주면 준영속 상태가 되어도 사용할 수 있다. 

![Image](https://github.com/user-attachments/assets/1aac0eca-2651-400c-b21b-9885aeaf83ad)

<br>

> ⚠️ 프리젠테이션 계층을 위해 서비스 계층에서 로직을 구현하는 것은 좋지 않다. 
>
> ➡️ 각 계층의 역할 분리가 되어야 한다. 

따라서, 이를 해결하기 위해 `FACADE 계층`을 추가하여 **프리젠테이션 계층을 위한 프록시 초기화 역할을 분리**해야한다.  

<br>

### FACADE 계층 추가 

![Image](https://github.com/user-attachments/assets/208d6ccd-fd07-4181-9d16-aad267c689f0)

**프리젠테이션 계층**과 **도메인 모델 계층** 간의 논리적 의존성을 분리 

1. 프리젠테이션 계층에서 필요한 프록시 객체를 초기화한다. 
2. 서비스 계층을 호출해서 비즈니스 로직을 실행한다. 
3. 레포지토리를 직접 호출해서 뷰가 요구하는 엔티티를 찾는다. 

```java
@RequiredArgsConstructor
class MemberFacade {
    private final Service service;
    
    public Member findMember(Long memberId) {
      Member member = MemberService.findMemberById(memberId);

      member.team().getName(); //프록시 강제 초기화

      return member;
    }
}

class Service {
    public Member findMember(Long memberId) {
        return MemberRepository.findMember(memberId);
    }
}
```

➡️위 코드의 문제점은 <ins>하나의 계층이 더 생긴다는 것</ins>과 FACADE는 단순히 <ins>서비스 계층을 호출만 하는 위임 코드가 많을 것</ins>이라는 것이다.  


<br>

위와 같이 준영속 상태와 지연 로딩 문제를 극복하기 위해 많은 전략들을 해당 방법들은 모두 번거롭고 많은 단점들이 존재한다. 

**📢이 모든 문제는 엔티티가 프리젠테이션 계층에서 준영속 상태이기 때문에 발생** ➡️ **OSIV로 해결**

<br>

## OSIV(Open Session In View)

OSIV는 **영속성 컨텍스트를 View까지 열어둔다는 의미**이다. 

<br>

### 과거 OSIV: 요청 당 트랜잭션

클라이언트 요청이 들어오자마자 `서블릿 필터`나 `스프링 인터셉터`에서 <ins>트랜잭션 시작 후</ins>  **요청이 끝날 때** <ins>트랜잭션 끝</ins> 


➡️ 이를 **요청 당 트랜잭션 방식의 OSIV**라고 한다.

<br>

#### 문제점

**컨트롤러나 뷰 같은 프리젠테이션 계층이 엔티티를 변경할 수 있다는 점**이다. 

**[해결 방법]** 

- 엔티티를 읽기 전용 인터페이스로 제공
- 엔티티 레핑
- DTO 반환 


<br>

#### 엔티티를 읽기 전용 인터페이스로 제공 

읽기 전용 메서드만 제공하는 인터페이스를 제공

<br>

#### 엔티티 레핑


엔티티의 읽기 전용 메소드만 가지고 있는 엔티티를 감싼 객체를 만들고 이를 반환하는 방법 

```java
public MemberWrapper (member) {
    this.member = member;
}
```
<br>

#### DTO 반환

가장 전통적인 방법으로 단순히 데이터만 전달하는 객체인 DTO를 생성해서 반환한다. 

>  해당 방법은 **OSIV를 사용하는 장점을 살릴 수 없고** 엔티티를 거의 복사한 듯한 **DTO 클래스도 하나 더 만들어야 한다.** 

---

➡️ 위의 모든 방법들은 코드량이 상당히 증가한다는 단점이 있다. 따라서 최근에는 거의 사용 ❌

<br>

### 스프링 OSIV 

위의 문제를 보안하기 위한 스프링 프레임워크에서 제공하는 OSIV이다. 

<br>


#### 스프링 OSIV분석 

스프링 프레임워크가 제공하는 OSIV는 <ins>**비즈니스 계층에서 트랜잭션을 사용하는 OSIV**</ins>이다. 

<br>

#### 동작 과정

1. 클라이언트 요청
2. 영속성 컨텍스트를 생성 (트랜잭션은 시작 ❌)
3. 서비스에서 앞에서 생성한 영속성 컨텍스트에 트랜잭션 시작 
4. 트랜잭션 커밋 시 영속성 컨텍스트 플러시 
5. **트랜잭션만 종료**하고 <ins>**영속성 컨텍스트는 살려둔다.** 
6. 요청이 끝나면 영속성 컨텍스트 종료 

> **📢 트랜잭션 없이 읽기**
> 
> - 영속성 컨텍스트는 트랜잭션 내에서 엔티티 조회 및 수정이 가능하다. 
> - 영속성 컨텍스트는 트랜잭션 외에서는 오직 읽기만 가능하다. ➡️ 이를 트랜잭션 없이 읽기(NonTransactional reads)
> 

위의 특징을 이용하여 프리젠테이션 계층에서 엔티티 수정이 가능했던 **기존 OSIV의 단점을 보안**한다. 

또한, 트랜잭션 없이 읽기를 사용해서 프리젠테이션 계층에서도 지연로딩이 가능해진다. 

<br>


#### 스프링 OSIV 주의사항

**프리젠테이션 계층에서 엔티티를 수정한 직후에 트랜잭션을 시작하는 서비스 계층을 호출하면 문제가 발생**한다. 

```java
class Controller {
    public String viewMember(Long id) {
        Member member = memberService.findMember(id);
        member.setName("이름 수정");
        
        memberService.biz(); // 이름 수정 후 서비스 호출 
    }
}
```
스프링 OSIV는 같은 **영속성 컨텍스트를 여러 트랜잭션이 공유할 수 있기 때문에** 위와 같은 상황에서 <ins>biz()가 수행되고 플러시</ins>를 하면 이전에 🚨**프리젠테이션 계층에서 변경되었던 
엔티티도 데이터베이스에 반영되는 문제가 발생**한다.🚨


**➡️ 트랜잭션이 있는 비즈니스 로직을 모두 호출하고 나서 변경하면 된다.** 


<br>

> **💡 OSIV는 같은 JVM을 벗어난 원격 상황에서 사용 불가하다.** 
> 
> 원격지인 클라이언트에서 엔티티를 지연로딩하는 것이 불가능하다. 
> 
> **[JSON 응답]**
> 
> ```json
> {
> "userId": 1,
> "name": "Alice",
> "orders": null
> }
> ```
> 
> **JSON 응답 안에 포함되지 않은 데이터를 클라이언트가 나중에 lazy loading 식으로 자동으로 불러올 수는 없다.**
> 
> ➡️ 따라서 클라이언트가 필요한 데이터를 모두 JSON으로 생성해서 반환해주어야 한다. 
> 

