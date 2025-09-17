# Chapter7 고급 매핑

<br>

# 목차

> **[1. 상속 관계 매핑](#상속-관계-매핑)**
>
> **[2. @MappedSuperclass](#MappedSuperclass)**
>
> **[3. 복합 키와 식별 관계 매핑](#복합-키와-식별-관계-매핑)**
>
> **[4. 조인 테이블](#조인-테이블)**
>
> **[5. 엔티티 하나에 여러 테이블 매핑](#엔티티-하나에-여러-테이블-매핑)**



<br>

## 상속 관계 매핑

데이터베이스에서는 상속이라는 개념이 존재하지 않고 **`슈퍼타입`과 `서브타입` 관계라는 모델링 기법**(**Chapter1 참고하기**) 이 사용된다. 


이러한 슈퍼타입과 서브타입 논리 모델을 실제 테이블로 구현할 때는 **3가지 방법을 선택할 수 있다.** 

1. **각각의 테이블로 변환 :** 모두 테이블을 만들고 Join으로 조회 ➡️ `Join 전략`
2. **통합 테이블로 변환 :** 테이블을 하나만 만들어서 통합  ➡️ `단일 테이블 전략`
3. **서브타입 테이블로 변환 :** 서브 타입마다 하나의 테이블을 만든다.  ➡️ `테이블 전략` 

<br>

### Join 전략 

엔티티 **모두 테이블로 만들고 자식 테이블**이 부모 테이블의 기본 키를 받아서 `기본 키(자신을 나타냄) + 외래 키(부모의 기본 키)`로 사용하는 전략


>  ⚠️ 테이블은 타입의 개념이 없기 때문에 **타입을 구분하는 컬럼을 추가**해야한다. 


```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED) // 부모 클래스에 사용하는 어노테이션 JOINED 매핑 전략 선택 
@DiscriminatorColumn(name = "DTYPE") // 구분 테이블을 지정한다.
public abstract class Item {

    @GeneratedValue @Id
    private Long id;
		...
}

@Entity
@DiscriminatorValue("A") //자신의 테이블 값 지정
@PrimaryKeyJoinColumn(name = "ALBUM_ID") // 자신의 키본 키 컬럼 명 지정
public class Album extends Item {
    private String artist;
		...
}
```

> **만약 Album 객체를 저장할 경우 Item 테이블의 DTYPE 컬럼의 값이 "A"로 저장된다.**  


<br>

**[Join 전략의 장점]**

- 테이블 정규화
- 외래 키 참조 무결성 제약 조건 활용 가능 
- 저장 공간을 효율적으로 사용

**[Join 전략의 단점]**
- 조회 시 조인이 많이 사용되어 성능이 저하될 수 있다.
- 조회 쿼리가 복잡하다.
- 데이터 등록할 때 INSERT SQL 2번 실행

<br>

### 단일 테이블 전략

테이블을 하나만 사용하기 때문에 조인을 사용하지 않아 일반적으로 가장 빠르다.

![Image](https://github.com/user-attachments/assets/5d56a715-c822-470b-9e8f-78f5d518324c)


하나로 모두 통합되기 때문에 구분 컬럼을 필수로 사용해야한다.


<br>

**[단일 테이블 전략의 장점]**

- 조회 성능이 빠르고 쿼리가 단순하다. 

**[단일 테이블 전략의 단점]**
- 정규화 되지 않고 테이블이 커질 수 있다. ➡️  오히려 성능이 느려질 수 있다. 
- 매핑한 컬럼은 모두 null을 허용해야 한다. 

<br>


### 테이블 전략 

자식 엔티티마다 테이블을 만들고 자식 테이블에는 가가에 필요한 컬럼이 모두 존재한다. 

`@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`를 선택한다. 

 ➡️ **일반적으로 추천하지 않는 방식**이다. 

<br>

## @MappedSuperclass


부모 클래스는 테이블과 매핑하지 않고 **부모 클래스를 상속 받는 자식 클래스에게 매핑 정보만 제공하고 싶을 때 사용**한다. 

```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class) 
public abstract class Timestamped {

    @CreatedDate
    @Column(updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime createdAt; //공통 매핑 정보

    @LastModifiedDate
    @Column
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime modifiedAt; //공통 매핑 정보
}

@Entity
@Table(name = albums)
public class Album extends Timestamped {
    private String artist;
		...
}
```
<br>

> **💡 물려받은 매핑 정보 재정의** **@AttributeOverride**
> 
>  **💡 연관관계 제정의** **@AssociationOverrides**

<br>


## 복합 키와 식별 관계 매핑

### 식별 관계 VS 비식별 관계

데이터베이스 테이블 사이에 관계는 **외래키가 기본 키에 포함되는지 여부**에 따라 <ins>**식별 관계와 비식별 관계로 구분한다.** </ins>


### 식별 관계 (Identifying Relationship)

부모 테이블의 기본 키를 내려 받아 자식 테이블의 기본 키 + 외래 키로 사용하는 관계 

![Image](https://github.com/user-attachments/assets/e2a17b5b-454d-4c8d-ba3f-a7dc8ff0946b)

<br>

### 비식별 관계 (non-Identifying Relationship)

부모 테이블의 기본 키를 받아서 **자식 테이블의 외래 키로만 사용하는 관계**

![Image](https://github.com/user-attachments/assets/74146b9e-2d22-49f3-8794-64cafa370a4f)

비식별자 관계는 외래 키에 NULL을 허용하는지에 따라 `필수적 비식별 관계(NULL 허용 ❌)`와 `선택적 비식별 관계(NULL 허용 ⭕)`로 나눈다. 

<br>

### 비식별 관계로 복합 키 매핑하기 

JPA에서 **식별자를 둘 이상 사용하려면 별도의 식별자 클래스를 만들어야 한다.** 

영속성 컨텍스트에서 식별자를 구분하기 위해 equals와 hashCode를 사용하여 동등성 비교를 해야한다. 

#### @IdClass [방법1]

사용 시 만족해야 하는 조건 

- 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 한다. 
- Serializable를 구현해야 한다. 
- 기본 생성자가 있어야 한다. 
- 식별자 클래스는 public이어야 한다. 

<br>

🟢 부모 클래스
```java
@Entity
@IdClass(ParentId.class)
public class Parent {
		@Id
		@Column(name = "PARENT_ID1")
		private String id1; //ParentId.id1과 연결

		@Id
		@Column(name = "PARENT_ID2") 
		private String id2; //ParentId.id2과 연결
		...
}
```
```java
public class ParentId implements Serializable {
		private String id1; //Parent.id1 매핑 
		private String id2; //Parent.id2 매핑
		
		public ParentId() {
		}
        
		public ParentId(String id1, String id2) {
				this.id1 = id1;
				this.id2 = id2;
		}

		@Override
		public boolean equals(Object o) {...}
		@Override
		public int hashCode() {...}
}
```

<br>

🟢 자식 클래스 
```java
@Entity
public class Child {
		@Id
		private String id;
		
		@ManyToOne
		@JoinColumns({
				@JoinColumn(name = "PARENT_ID1", referencedColumnName = "PARENT_ID1"),
				@JoinColumn(name = "PARENT_ID2", referencedColumnName = "PARENT_ID2")
		})
		private Parent parent;
}
```

<br>

#### @EmbeddedId [방법2]

좀 더 객체 지향적인 방법이다. 

**조건** 

- @Embeddable을 붙여야한다.
- Serializable를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- 식별자 클래스는 public이어야 한다.
- equals와 hashCode를 구현해야 한다. 


<br>

🟢 부모 클래스
```java
@Entity
public class Parent {
		@EmbeddedId
		private ParentId id;
		
		...
}
```

```java
@Embeddable
public class ParentId implements Serializable {
		@Column(name = "PARENT_ID1")
		private String id1;
		@Column(name = "PARENT_ID2")
		private String id2;		
		
		//equals and hashCode 구현
		...
}
```
<br>

저장 시 parentId를 직접 생성해서 사용한다. 

```java
ParentId id1 = new ParentId(); // id1와 인스턴스가 다르다.
id1.setId1("myId1");
id1.setId2("myId2");

ParentId id2 = new ParentId(); // id2와 인스턴스 다르다.
id2.setId1("myId1");
id2.setId2("myId2");
```

따라서 반드시 equals와 hashCode를 구현해야한다. 

<br>

### 식별 관계로 복합 키 매핑하기

**262 ~ 266 page** 


### ORM에서 추천하는 방법은 `비식별 관계`를 사용하고 기본 키는` Long 타입`의 대리 키를 사용하는 것이다. 

<br>

## 조인 테이블

연관관계를 설계하는 방법에는 2가지가 존재한다. 

1. 조인 컬럼 사용 (외래 키)
2. 조인 테이블 사용 (테이블)

<br>

## 엔티티 하나에 여러 테이블 매핑

@SecondaryTable을 사용하면 엔티티 하나에 여러 테이블을 매핑할 수 있다. 

➡️ **테이블당 엔티티를 각각 만들어 일대일 매핑하는 것을 권장한다.** 