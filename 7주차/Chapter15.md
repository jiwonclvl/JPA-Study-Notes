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

## 프록시 심화 주제

## 성능 최적화



