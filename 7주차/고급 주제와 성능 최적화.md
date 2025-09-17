# Chapter15 고급 주제와 성능 최적화 

<br>

# 목차

> **[1. 예외 처리](#예외-처리)**
>
> **[2. 엔티티 비교](#엔티티-비교)**
>
> **[3. 프록시 심화 주제](#프록시-심화-주제)**
> 
> **[4. 성능 최적화](#성능-최적화)**


<br>

## 예외 처리
### JPA 표준 예외 정리 

JPA 표준 예외들은 `PersistenceException`(RuntimeException의 자식)의 자식 클래스이다. 

![Image](https://github.com/user-attachments/assets/c1437583-0312-48a9-9e00-4a26dce78ae5)

`JPA 예외`는 모두 <ins>**언체크 예외**</ins>이다. 


> **💡언체크 예외란?**
> 
> RuntimeException의 하위 클래스들을 의미한다. 
> 
>  ➡️ **실행 중에 발생할 수 있는 예외**

<br>

**[표준 예외 2가지]**
- 트랜잭션 롤백을 **표시하는 예외** ➡️ 🚨 심각한 예외 🚨
  - **∴** <ins>복구하면 안된다.</ins> 
  - 트랜잭션 강제 커밋 시 RollbackException 예외 발생
  
    | 트랜잭션 콜백을 표시하는 예외 | 설명 |
    |:-------------------------------|:-----|
    | javax.persistence.**EntityExistsException** | EntityManager.persist(..) 호출 시 이미 같은 엔티티가 있으면 발생 |
    | javax.persistence.**EntityNotFoundException** | EntityManager.getReference(..)를 호출했는데 실제 사용 시 엔티티가 존재하지 않으면 발생. refresh(..), lock(..)에서도 발생 |
    | javax.persistence.**OptimisticLockException** | 낙관적 락 충돌 시 발생 |
    | javax.persistence.**PessimisticLockException** | 비관적 락 충돌 시 발생 |
    | javax.persistence.**RollbackException** | EntityTransaction.commit() 실패 시 발생. 콜백이 표시되어 있는 트랜잭션 커밋 시에도 발생 |
    | javax.persistence.**TransactionRequiredException** | 트랜잭션이 필요할 때 트랜잭션 없으면 발생. 트랜잭션 없이 엔티티 변경할 때 주로 발생 |


<br>

- 트랜잭션 롤백을 **표시하지 않는 예외** ➡️  심각한 예외 ❌

  | JPA 예외                                         | 스프링 변환 예외                                    |
  |:-----------------------------------------------|:---------------------------------------------|
  | javax.persistence.**QueryTimeoutException**    | 쿼리 실행 시간 초과 시 발생                             |
  | javax.persistence.**NoResultException**        | Query.getSingleResult() 호출 시 결과가 하나도 없을 때 발생 |
  | javax.persistence.**NonUniqueResultException** | Query.getSingleResult() 호출 시 결과가 둘 이상일 때 발생  |
  | javax.persistence.**LockTimeoutException**     | 비관적 락에서 시간 초과 시 발생                           |


<br>


### 스프링 프레임워크의 JPA 예외 변환

`서비스 계층`에서 **데이터 접근 계층의 구현 기술에 직접 의존**하는 것은 좋은 설계 ❌

`예외도 마찬가지`로 서비스 계층에서 JPA의 예외를 직접 사용하면 **JPA에 의존하게되어 좋은 설계가 되지 않는다.** 

**[642.page 참고]**

- 표 15.3 JPA 예외를 스프링 예외로 변경
- 표 15.4 JPA 예외를 스프링 예외로 변경 추가

<br>

### 트랜잭션 롤백 시 주의사항

- 트랜잭션을 롤백하는 것은 데이터베이스의 반영사항만 롤백하는 것

    ➡️ 수정한 자바 객체까지 원상태로 복구해주지 ❌

<br>

**🔖 예시** 

수정 중 문제 발생으로 트랜잭션을 롤백하면 <ins>**데이터베이스의 데이터는 원래대로 복구**</ins>되지만 `객체`는 <ins>**수정된 상태로 영속성 컨텍스트에 남아있다.**</ins> 

🚨 롤백된 영속성 컨텍스트를 그대로 사용하는 것은 위험 🚨

**[해결 방법]**

1. 새로운 영속성 컨텍스트 생성 
2. EntityManager.clear() 호출로 컨텍스트 초기화 


> 💡트랜잭션당 영속성 컨텍스트 전략 
> 
> 기본 전략은 문제 발생 시 트랜잭션 `AOP 종료 시점에 트랜잭션을 롤백`하면서 <ins>영속성 컨텍스트를 함께 종료</ins>하므로 문제되지 않는다.   
> 

<br>

 ➡️ `OSIV처럼 영속성 컨텍스트를 트랜잭션 범위보다 넓게 사용하는 경우`에는 위와 같은 문제를 고려해주어야 한다. 


> ⚠️ 해당 문제를 고려하지 않으면 **롤백 후 수정되지 않은 객체의 정보**를 <ins>다른 트랜잭션에서 사용하게 되는 문제가 발생</ins>할 수 있다. 

<br>

## 엔티티 비교

영속성 컨텍스트 내부에는 엔티티 인스턴스 보관을 위한 1차 캐시가 있다. 

> 💡 **1차 캐시는 영속성 컨텍스트와 생명주기를 같이 한다.** 


<br>

**[1차 캐시]**

- 영속성 컨텍스트를 통해 데이터 저장 및 조회 시 1차 캐시에 해당 엔티티가 저장된다.


- 변경 감지 기능을 제공한다. 


- 데이터 베이스르 통하지 않고 데이터 조회가 가능하다. 


- <ins>**애플리케이션 수준의 반복 가능한 읽기**</ins> (가장 큰 장점)
    
    ➡️ 엔티티 조회 시 **항상 같은 주소값**을 가지는 인스턴스를 반환하는 것을 의미

<br>

### 영속성 컨텍스트가 같을 때 엔티티 비교 

가정: 테스트 클래스에서 `@Transactional`를 붙일 경우

```java
@Transactinal 
public class MemberServiceTest {
    ...
}

@Transactinal
public class MemberService {
    
}
```

> 💬 위의 코드에서 테스트와 서비스 모두 @Transactional 어노테이션을 사용 
> 
>  ➡️  기본 전략으로 이미 시작된 트랜잭션이 **있다면** 해당 트랜잭션에 **참여** / **없다면** 새로운 트랜잭션 **생성**

테스트의 범위와 트랜잭션의 범위가 같다. 따라서, <ins>**테스트 전체는 동일한 영속성 컨텍스트에 접근**</ins>한다. 

<br>

> 💡 **영속성 컨텍스트가 같다면** 
>  
> 엔티티를 비교할 때 아래의 3가지 조건을 모두 만족한다. 
> 
> - 동일성(Identical): **`==` 비교가 같다.** 
> - 동등성(Equinalent): **`equals()` 비교가 같다.**
> - 데이터베이스 동등성: `@Id`인 **데이터베이스 식별자가 같다.** 


> ⚠️ **테스트 클래스에서 `@Transactional` 적용**
> 
> 테스트가 끝날 때 트랜잭션을 <ins>커밋하지 않고 강제로 롤백한다.</ins> 

<br>

### 영속성 컨텍스트가 다를 때 엔티티 비교


가정: 서비스 클래스 & Repository 클래스에만 `@Transactional`이 붙은 경우

```java
public class MemberServiceTest {
    ...
    
    @Test
    void 회원가입() {
        //given
        Member member =  new Member("회원1");
        
        //when
        Long id = memberService.join(member);
        
        //then
        Member findMember =  memberRepository.findOne(id);
        assertTrue(member==findMember); //false
        assertTrue(member.equals(findMember)); // true (단, equals() 구현이 필요)
        
    }
}

@Transactinal
public class MemberService {
    ...
    public Long join(Member member) {
        memberRepostitory.save(member); //레포에 맴버 저장
        return member.getId(); //해당 맴버의 ID 응답
    }
}

//에시를 위해 @Transactional 선언
@Transactinal
public class MemberRepository {
    ...
}
```

`Long id = memberService.join(member);`: 서비스 트랜잭션에 참여된 Repository **(영속성 컨텍스트1)**

`Member findMember =  memberRepository.findOne(id);`: 새로 만들어진 Repostitory 트랜잭션  **(영속성 컨텍스트2)**


<br>

>  💡 **데이터베이스 동등성**
> 
> @Id인 데이터베이스의 식별자는 같다.  
> 
> 따라서, 위와 같은 상황에서는 <ins>엔티티 비교</ins>보다 **<ins>데이터베이스 동등성 비교</ins> 방법을 사용한다.** 
> 
> ⚠️ 해당 비교는 **엔티티를 영속화해야 식별자를 얻을 수 있다는 문제**가 존재한다. 

---

> 💡 **equals()**
> 
> 비즈니스 키를 활용한 동등성 비교 권장 (ex> 주민번호, 회원 아이디 + 전번



<br>

## 프록시 심화 주제

### 영속성 컨텍스트와 프록시

#### [프록시 조회 후 원본 엔티티 조회]

```java
Member refMember = em.getReference(Member.class, "member1"); //프록시 반환
Member findMember = em.find(Member.class, "member1"); //프록시 반환
```

영속성 컨텍스트는 <ins>프록시로 조회된 엔티티에 대해 같은 엔티티를 찾는 요청</ins>이 오면 원본이 아닌 **처음 조회한 프록시를 반환**하다. 

➡️ 영속성 컨텍스트가 **영속 엔티티의 동일성을 보장**

<br>

#### [원본 엔티티 조회 후 프록시 조회]

```java
Member findMember = em.find(Member.class, "member1"); //원본 엔티티 반환
Member refMember = em.getReference(Member.class, "member1"); //원본 엔티티 반환
```

영속성 컨텍스트는 원본 엔티티를 이미 데이터베이스에서 조회하여 프록시 반환을 하지 않는다. 


➡️ 영속성 컨텍스트가 **영속 엔티티의 동일성을 보장**

<br>

### 프록시 타입 비교 

프록시의 경우 원본 엔티티를 상속 받아 만들어지므로 `==`이 아닌 `instanceof`를 사용해야한다.

```java
import java.lang.reflect.Member;

Member member = new Member("member1");

Member refMember = em.getReference(Member.class, "member1");

assertFalse(Member .class==refMember.getClass()); //false
assertTrue(refMember instanceof Member); //true
```

<br>

### 프록시 동등성 비교 

**IDE**나 **외부 라이브러리**를 사용하여 구현한 `equals()` 메소드로 <ins>엔티티를 비교할 때</ins> 비교 대상이 **프록시면 문제가 발생할 수 있다.** 

```java
Member member = new Member("member1");
Member refMember = em.getReference(Member.class, "member1");

//결과: false
if (member.name.equals(refMember.name)){
    log.info("두 엔티티의 name 필드는 같습니다.");   
}
```

`refMember.name`: 프록시를 통해 맴버변수에 직접 접근 

<br>

> 💡 equals 메서드 구현 시 
> 
> - 일반적으로 맴버변수를 직접 비교한다. 
> - private 맴버변수도 접근 가능하다.

➡️ 프록시의 경우 실제 데이터를 가지고 있지 않기 때문에 **프록시의 맴버변수에 접근하면 아무런 값도 조회할 수 없다.** 


따라서, 프록시의 필드를 조회하고 싶을 때는 <ins>**Getter를 사용**</ins>해야한다. 

![Image](https://github.com/user-attachments/assets/d52c329c-8e0e-4069-a321-4cefa749c19f)

<br>

### 상속관계와 프록시 

프록시를 부모 타입으로 조회하면 **부모 타입을 기반으로 프록시가 생성되는 문제 발생** 

- instanceof 연산 사용 불가
- 하위 타입으로 다운 캐스팅 불가

<br>

**[해결 방법]**
1. **JPQL로 대상 직접 조회** 


2. **프록시 벗기기 (하이버네이트가 제공하는 기능)**

    > ⚠️ **영속성 컨텍스트는 동일성 보장을 위해 프록시로 한번 노출된 엔티티는 계속 프록시로 노출한다.** 
    > 
    > ➡️ 위의 방법은 원본 엔티티를 직접 꺼내기 때문에 <ins>동일성 비교에 실패한다는 문제</ins>가 있다.  
    > 
    > 따라서, **필요한 곳에서만 잠깐 사용**해야한다.  

<br>

3. **기능을 위한 별도의 인터페이스 제공**
    
    ➡️ 공통 인터페이스를 만들어 **오버라이딩 진행**


4. **비지터 패턴 사용**

    **669 ~ 675 Page 참고하기** 

<br>

## 성능 최적화

### N + 1 문제 

**Chapter13 N + 1 부분** 

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
> 이와 같이 N + 1이 발생하면 SQL이 상당히 많이 노출될 수 있으므로 성능상의 치명적이다.  
> 

<br>

**[해결 방법]**

1. **페치 조인**


> 💡 만약 중복된 결과가 나타난다면 **DISTINCT를 사용하여 제거하는 것이 좋다.** 

<br>

2. **하이버네이트 @BatchSize**

@BatchSize 사용 시 연관된 엔티티를 조회할 때 **지정한 size만큼 SQL의 IN절을 사용해서 조회**한다. 

<br>

3. **하이버네이트 @Fetch(FetchMode.SUBSELECT)**


<br>

### 읽기 전용 쿼리의 성능 최적화

영속성 컨텍스트는 변경 감지를 위해 스냅샷 인스턴스를 보관하므로 **더 많은 메모리를 사용하는 단점이 있다.**

➡️ 따라서, 단순히 한번 조회하는 경우에는 읽기 전용으로 엔티티를 조회하여 메모리 사용량을 최적화할 수 있다. 


<br>

```mysql
select o from  Order o
```
---
#### 스칼라 타입으로 조회

엔티티가 아닌 **스칼라 타입으로 모든 필드 조회**

```mysql
select o.id, o.name, o.price from  Order o
```

 ➡️ 스칼라 타입은 영속성 컨텍스트가 결과를 관리하지 않는다. 

---
#### 읽기 전용 쿼리 힌트 사용

org.hibernate.readOnly 사용

`읽기 전용`이므로 영속성 컨텍스트는 **<ins>스냅샷을 보관하지 않아</ins> 메모리 사용량을 최적화할 수 있다.** 

---
#### 읽기 전용 트랜잭션 사용

@Transactional(readOnly = true)

- 하이버네이트가 세션의 **플러시 모드를 Manual로 설정**
- 강제로 플러시를 호출하지 않으면 플러시 발생 ❌

> **💡 하이버네이트 세션**
> 
> `JPA 엔티티 매니저`를 **하이버네이트로 구현한 구현체**를 의미한다. 


<br>

### 배치 처리 

일반적인 방식으로 **엔티티를 계속 조회**하면 `영속성 컨텍스트`에 **아주 많은 엔티티가 쌓이면서 메모리 부족 오류가 발생**한다. 

#### JPA 등록 배치 

수천, 수만 건 이상의 엔티티를 한번에 등록할 때에는 영속성 컨텍스트에 엔티티가 쌓이지 않도록 일정 단위마다 **영속성 컨텍스트의 엔티티를 데이터베이스에 플러시하고 영속성 컨텍스트를 초기화**해야한다. 

<br>

#### JPA 수정 배치  

1. **페이징 처리** 

    ➡️ 페이지 단위마다 영속성 컨텍스트 플러시 및 초기화 진행


<br>

2. Session (JPA는 JDBC 커서를 지원 ❌)

    하이버네이트를 scroll이라는 이름으로 JDBC 커서를 지원한다. 


3. 하이버네이트 무상태 세션 사용

    영속성 컨텍스트 없으 동작하는 것 

    ➡️ 영속성 컨텍스트가 없어 엔티티 수정을 원할 경우 `update() 메소드를 직접 호출`해야한다. 


<br>

### SQL 쿼리 힌트 사용 (688page 참고)

<br>

### 트랜잭션을 지원하는 쓰기 지연과 성능 최적화 
#### 트랜잭션을 지원하는 쓰기 지연과 JDBC 배치 

JDBC가 제공하는 SQL 배치 기능을 사용항 SQL을 모아서 데이터베이스에 한번에 보낼 수 있다. 

- SQL 배치 전략은 구현체마다 다르다. 
- batch_size 속성의 값 설정 
- SQL 배치는 같은 SQL일 때만 유효

<br>

#### 트랜잭션을 지원하는 쓰기 지연과 애플리케이션 확장성 

**[JPA를 사용하지 않고 SQL을 직접 다룰 경우]**

➡️ `update() 호출 `시 해당 SQL을 실행하면서 **데이터베이스 테이블 로우에 락을 건다.** 

해당 방법은 <ins>**트랜잭션이 커밋될 때까지 유지**</ins>된다.  

> 🗨️ **다른 트랜잭션이 해당 row를 수정하려고 한다면?**
> 
> 이전 **트랜잭션이 commit될 때까지 기다려야 한다.** 
> 
> **🚨DB 자원을 오래 점유🚨**

<br>

**[JPA 사용]**

JPA는 영속성 컨텍스트 안에 변경 내용을 저장해두고 `커밋 및 플러시를 하는 시점`에 **쿼리를 실행**한다. 

➡️ 쿼리를 보내고 바로 트랜잭션을 커밋하므로 <ins>**데이터베이스에 락이 걸리는 시간을 최소화**</ins>한다. 

| 항목             | SQL 직접 사용       | JPA 사용 (쓰기 지연)       |
|------------------|----------------------|-----------------------------|
| update 시점      | 즉시 실행            | flush 또는 commit 시점      |
| 락 걸리는 시점   | 즉시                 | 커밋 직전                   |
| 락 유지 시간     | 길다                 | 짧다                        |
| 확장성           | 상대적으로 낮음      | 상대적으로 높음            |


