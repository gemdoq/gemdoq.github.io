---
layout: single
title: "Spring에서 Transaction에 대해"
date: 2022-12-12 11:43:51 +0900
categories: java
tags: [Transaction, ACID, Dirty Read, Non-Repeatable Read, Phantom Read, Transaction Isolation Level, Propagation]
typora-root-url: ../
---


## Transaction에 대하여
> - 트랜잭션이란
> - 보장해야 하는 ACID
> - 트랜잭션 격리 수준
> - Spring에서 트랜잭션
> - Spring에서 트랜잭션 사용

<br>

## 트랜잭션(Transaction)이란

### 정의

DBMS에서 데이터를 다루는 논리적인 작업의 단위

데이터베이스에 영향을 주는 일련의 작업들(삽입, 수정, 삭제 등)을 실행할 때 하나로 묶어 그룹화하는 것을 의미

트랜잭션은 전체가 수행되거나 또는 전혀 수행되지 않아야 한다.(All or Nothing)

![transaction-process](/images/2022-12-12-about-transaction/transaction-process.png){: width="560"}

- 트랜잭션 커밋 : 작업이 마무리되어 반영
- 트랜잭션 롤백 : 작업을 취소하고 이전의 상태를 유지

<br>

## 보장해야 하는 ACID

![transaction-specific](/images/2022-12-12-about-transaction/transaction-specific.png){: width="560"}

데이터베이스는 일반적인 프로그램과 다르게 4가지의 성질을 지니는데, 이를 ACID라 함

### A(Atomicity, 원자성)

트랜잭션은 모두 실행되거나 아예 실행되어서는 안됨

성공적인 트랜잭션은 commit 하고 실패한 트랜잭션은 rollback

### C(Consistency, 일관성)

트랜잭션이 실행을 성공적으로 완료하면 언제나 일관성 있는 데이터베이스 상태로 유지

각 트랜잭션은 일관성을 보장하도록 프로그램을 작성

### I(Isolation, 격리성)

하나의 트랜잭션이 데이터를 갱신하는 동안 이 트랜잭션이 완료되기 전에 갱신중인 데이터에 다른 트랜잭션에 영향을 주지 않아야 함

### D(Durability, 지속성)

트랜잭션이 완료된 이후의 상태는 데이터베이스에 영구적으로 반영되어야 하며, 비록 시스템이 실패하여도 그 상태가 일관되게 유지되야 함

<br>

## 트랜잭션 격리 수준

### 필요성

트랜잭션이 보장해야 되는 ACID 중 Isolation(격리성)에 대해 문제점들이 있었고, 그에 따라 트랜잭션을 동시에 실행시키면서 락보다 좀 더 완화된 방법으로 문제를 해결하기 위해 트랜잭션 격리 수준을 나눠야 할 필요성이 생김

#### Dirty Read

![dirty-read](/images/2022-12-12-about-transaction/dirty-read.png){: width="560"}

한 트랜잭션(Transaction A)이 데이터에 접근하여 값을 'A'에서 'B'로 변경했고 아직 커밋을 하지 않았을 때,

다른 트랜잭션(Transaction B)이 해당 데이타를 Read 하면?

Transaction B가 읽은 데이터는 B가 될 것이다. 

하지만 Transaction A이 최종 커밋을 하지 않고 롤백하였다면 Transaction B는 무결성이 깨진 데이터를 사용할 것이다.

#### Non-Repeatable Read

![non-repeatable-read](/images/2022-12-12-about-transaction/non-repeatable-read.png){: width="560"}

한 트랜잭션(Transaction A)이 데이타를 Read 하고 있다. 

이때 다른 트랜잭션(Transaction B)가 데이터에 접근하여 값을 변경 또는, 데이터를 삭제하고 커밋을 실행한다면?

그 후 Transaction A는 다시 해당 데이터를 Read하고자 하면 변경된 데이터 혹은 사라진 데이터를 찾게 된다.

#### Phantom Read

![phantom-read](/images/2022-12-12-about-transaction/phantom-read.png){: width="560"}

트랜잭션(Transaction A) 중에 특정 조건으로 데이터를 검색하여 결과를 얻었다. 

이때 다른 트랜잭션(Transaction B)가 접근해 해당 조건의 데이터 일부를 삭제 또는 추가했을 때, 

아직 끝나지 않은 Transaction A이 다시 한번 해당 조건으로 데이터를 조회 하면,

Transaction B에서 추가/삭제된 데이터가 함께 조회/누락된다.

### (Level 0) Read Uncommitted

![read-uncommitted](/images/2022-12-12-about-transaction/read-uncommitted.png){: width="560"}

한 트랜잭션에서 커밋하지 않은 데이터에 다른 트랜잭션이 접근 가능하다. 

즉, 커밋하지 않은 데이터를 읽을 수 있다.

이 수준은 당연히 위에서 언급한 모든 문제에 대해 발생가능성이 존재한다. 

대신, 동시 처리 성능은 가장 높다.

### (Level 1) Read Committed

![read-committed](/images/2022-12-12-about-transaction/read-committed.png){: width="560"}

커밋이 완료된 데이터만 읽을 수 있다.

Dirty Read가 발생할 여지는 없으나, Read Uncommitted 수준보다 동시 처리 성능은 떨어진다. 

대신 Non-Repeatable Read 및 Phantom Read는 발생가능하다.

데이타베이스들은 보통 Read Committed를 디폴트 수준으로 지정한다.

### (Level 2) Repeatable Read

![repeatable-read](/images/2022-12-12-about-transaction/repeatable-read.png){: width="560"}

트랜잭션 내에서 한번 조회한 데이터를 반복해서 조회해도 같은 데이터가 조회 된다.

이는 개별 데이터이슈인 Dirty Read나 Non-Repeatable Read는 발생하지 않지만, 

결과 집합 자체가 달라지는 Phantom Read는 발생 가능

### (Level 3) Serializable

![serializable](/images/2022-12-12-about-transaction/serializable.png){: width="560"}

가장 엄격한 격리 수준

위 3가지 문제점을 모두 커버 가능

하지만 동시 처리 성능은 급격히 떨어질 수 있음

![transaction-isolation-level](/images/2022-12-12-about-transaction/transaction-isolation-level.png){: width="560"}

<br>

## Spring에서 트랜잭션

JDBC를 이용하는 개발자가 직접 여러 개의 작업을 하나의 트랜잭션으로 관리하려면 Connection 객체를 공유하는 등 상당히 불필요한 작업들이 많이 생길 것

그래서 Spring은 이러한 문제를 해결하고자 트랜잭션과 관련된 핵심 기술과 트랜잭션 세부 설정들을 제공

### Spring에서 사용할 수 있는 트랜잭션 핵심기술들 

#### 동기화

트랜잭션을 시작하기 위한 Connection 객체를 특별한 저장소에 보관해두고 필요할 때 꺼내 쓸 수 있도록 하는 기술

트랜잭션 동기화 저장소는 작업 쓰레드마다 Connection 객체를 독립적으로 관리하기 때문에, 멀티쓰레드 환경에서도 충돌이 발생할 여지가 없음

```java
// 동기화 시작
TransactionSynchronizeManager.initSynchronization();
Connection c = DataSourceUtils.getConnection(dataSource);

// 작업 진행

// 동기화 종료
DataSourceUtils.releaseConnection(c, dataSource);
TransactionSynchronizeManager.unbindResource(dataSource);
TransactionSynchronizeManager.clearSynchronization();
```
하지만 개발자가 JDBC가 아닌 Hibernate와 같은 기술을 쓴다면 위의 JDBC 종속적인 트랜잭션 동기화 코드들은 문제를 유발

대표적으로 Hibernate에서는 Connection이 아닌 Session이라는 객체를 사용하기 때문

이러한 기술 종속적인 문제를 해결하기 위해 Spring은 트랜잭션 관리 부분을 추상화한 기술을 제공

#### 추상화

공통점을 담은 트랜잭션 추상화 기술을 제공하여 애플리케이션에 각 기술마다(JDBC, JPA, Hibernate 등) 종속적인 코드를 이용하지 않고도 일관되게 트랜잭션을 처리 가능

![transaction-abstract](/images/2022-12-12-about-transaction/transaction-abstract.png){: width="560"}

Spring이 제공하는 트랜잭션 경계 설정을 위한 추상 인터페이스는 PlatformTransactionManager

사용하는 기술과 무관하게 PlatformTransactionManager를 통해 다음의 코드와 같이 트랜잭션을 공유, 커밋, 롤백 가능
```java
public Object invoke(MethodInvoation invoation) throws Throwable {
	TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
	
	try {
		Object ret = invoation.proceed();
		this.transactionManager.commit(status);
		return ret;
	} catch (Exception e) {
		this.transactionManager.rollback(status);
		throw e;
	}
}
```
위와 같은 트랜잭션 관리 코드는 비지니스 로직 코드와 결합되어 1개 초과 책임

그래서 Spring에서는 AOP를 이용해 이러한 트랜잭션을 핵심 비지니스 로직과 분리

#### AOP를 이용한 분리

예를 들어 다음과 같이 트랜잭션 코드와 비지니스 로직 코드가 복잡하게 얽혀 있는 경우
```java
public void addUsers(List<User> userList) {
	TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
	
	try {
		for (User user: userList) {
			if(isEmailNotDuplicated(user.getEmail())){
				userRepository.save(user);
			}
		}

		this.transactionManager.commit(status);
	} catch (Exception e) {
		this.transactionManager.rollback(status);
		throw e
	}
}
```
위의 코드는 여러 책임을 가질 뿐만 아니라 서로 성격도 다르고 주고받는 것도 없으므로 분리하는 것이 적합

Spring에서는 AOP(Aspect Oriented Programming, 관점 지향 프로그래밍) 기능으로써 트랜잭션 어노테이션(@Transactional)을 지원(트랜잭션과 같은 부가 기능에 대한 로직을 클래스 밖으로 빼내서 별도의 모듈로 만듦)

이를 적용하면 위와 같은 코드를 아래 코드와 같이 핵심 비지니스 로직만 남길 수 있음

```java
@Service
@RequiredArgsConstructor
@Transactional
public class UserService {

    private final UserRepository userRepository;

    public void addUsers(List<User> userList) {
        for (User user : userList) {
            if (isEmailNotDuplicated(user.getEmail())) {
                userRepository.save(user);
            }
        }
    }
}
```

### Spring 트랜잭션의 세부 설정

#### 전파속성(Propagation)

트랜잭션의 경계에서 이미 진행중인 트랜잭션이 있거나 없을 때 어떻게 동작할 것인가를 결정하는 방식을 의미

예를 들어 어떤 A 작업에 대한 트랜잭션이 진행중이고 B 작업이 시작될 때 B 작업에 대한 트랜잭션을 어떻게 처리할지 결정

##### 1. REQUIRED

디폴트 속성으로 모든 트랜잭션 매니저가 지원

이미 시작된 트랜잭션이 있으면 참여하고 없으면 새로 시작

하나의 트랜잭션이 시작된 후 다른 트랜잭션 경계가 설정된 메소드를 호출하면 같은 트랜잭션으로 묶임

##### 2. SUPPORTS

이미 시작된 트랜잭션이 있으면 참여하고, 그렇지 않으면 트랜잭션 없이 진행함

트랜잭션이 없어도 해당 경계 안에서 Connection 객체나 하이버네이트의 Session 등은 공유 할 수 있음

##### 3. MANDATORY

이미 시작된 트랜잭션이 있으면 참여하고, 없으면 새로 시작하는 대신 없으면 예외를 발생시킴

MANDATORY는 혼자서 독립적으로 트랜잭션을 진행하면 안되는 경우에 사용

##### 4. REQUIRES_NEW

항상 새로운 트랜잭션을 시작해야 하는 경우에 사용

이미 진행중인 트랜잭션이 있으면 이를 보류시키고 새로운 트랜잭션을 만들어 시작함

##### 5. NOT_SUPPORTED

이미 진행중인 트랜잭션이 있으면 이를 보류시키고, 트랜잭션을 사용하지 않도록 함

##### 6. NEVER

이미 진행중인 트랜잭션이 있으면 예외를 발생시키며, 트랜잭션을 사용하지 않도록 강제함

##### 7. NESTED

이미 진행중인 트랜잭션이 있으면 중첩(자식) 트랜잭션을 시작함

중첩 트랜잭션은 트랜잭션 안에 다시 트랜잭션을 만드는 것으로, 독립적인 트랜잭션을 만드는 REQUIRES_NEW와는 다름

#### 격리 수준(Isolation Level)

동시에 여러 트랜잭션이 진행될 때, 트랜잭션의 작업 결과를 여타 트랜잭션에게 어떻게 노출할 것인지 결정

##### 1. DEFAULT

사용하는 데이터 액세스 기술 또는 DB 드라이버의 디폴트 설정을 따름

일반적으로 드라이버의 격리 수준은 DB의 격리 수준을 따르며, 대부분의 DB는 READ_COMMITED를 기본 격리수준으로 가짐

##### 2. READ_UNCOMMITTED

가장 낮은 격리수준

하나의 트랜잭션이 커밋되기 전에 그 변화가 다른 트랜잭션에 그대로 노출

하지만 가장 빠르기 때문에 데이터의 일관성이 조금 떨어지더라도 성능을 극대화할 때 의도적으로 사용

##### 3. READ_COMMITTED

Spring은 기본 속성이 DEFAULT → DB는 일반적으로 READ_COMMITED(가장 많이 사용)

READ_UNCOMMITTED와 달리 다른 트랜잭션이 커밋하지 않은 정보는 읽을 수 없음

대신 하나의 트랜잭션이 읽은 로우를 다른 트랜잭션이 수정할 수 있음 

→ 처음 트랜잭션이 같은 로우를 다시 읽을 때 다른 내용이 발견될 수 있음

##### 4. REPEATABLE_READ

하나의 트랜잭션이 읽은 로우를 다른 트랜잭션이 수정할 수 없도록 막음

하지만 새로운 로우를 추가하는 것은 막지 않음

따라서 SELECT로 조건에 맞는 로우를 전부 가져오는 경우 트랜잭션이 끝나기 전에 추가된 로우가 발견될 수 있음

##### 5. SERIALIZABLE

가장 강력한 트랜잭션 격리 수준으로, 이름 그대로 트랜잭션을 순차적으로 진행

SERIALIZABLE은 여러 트랜잭션이 동시에 같은 테이블의 정보를 액세스할 수 없음

#### 제한시간(timeout)

트랜잭션에 제한시간을 int 타입의 초 단위로 지정

문자열로 지정하기를 원하면 timeoutString을 사용

#### 읽기전용(readOnly)

INSERT, UPDATE, DELETE의 작업이 진행되면 예외 발생
- 읽기 전용으로 설정함으로써 성능을 최적화함
- 쓰기 작업이 일어나는 것을 의도적으로 방지함

<br>

## Spring에서 트랜잭션 사용

### 선언적 트랜잭션(declarative transaction)

@Transactional이라는 어노테이션을 통해 Spring AOP 포인트 컷에 등록하여 트랜잭션의 기능을 부여하는 방식
@Transactional 어노테이션을 이용하면 트랜잭션 속성을 메소드 단위로 다르게 지정할 수 있어 매우 세밀한 트랜잭션 속성 제어가 가능할 뿐만 아니라 직관적

반대로 프로그램에 의한 트랜잭션(programmatic transaction)은 TransactionTemplate이나 개별 데이터 기술의 트랜잭션 API를 이용해 직접 코드안에서 기능을 구현하는 방식

### 롤백 처리

선언적 트랜잭션(@Transactional) 안에서 예외 발생 시, 언체크 예외(런타임 예외)라면 자동적으로 롤백
@Transactional의 rollbackFor 속성에 명시되지 않은 체크 예외라면 롤백하지 않음
Spring 트랜잭션에서는 transactionAttribute의 rollbackOn에 의해 검사되어 롤백 여부 rollback-only를 True로 변경
```java
@Override
public boolean rollbackOn(Throwable ex) {
    return (ex instanceof RuntimeException || ex instanceof Error);
}
```
그리고 만약 언체크 예외라면 rollback 처리를 진행
```java
protected void completeTransactionAfterThrowing(@Nullable TransactionInfo txInfo, Throwable ex) {
    if (logger.isTraceEnabled()) {
        logger.trace("Completing transaction for [" + txInfo.getJoinpointIdentification() + "] after exception: " + ex);
    }
    if (txInfo.transactionAttribute != null && txInfo.transactionAttribute.rollbackOn(ex)) {
        try {
            txInfo.getTransactionManager().rollback(txInfo.getTransactionStatus());
        }
    }
}
```

### 대체 정책(Fallback Policy)

Spring이 @Transactional 어노테이션을 적용할 때, 아래와 같은 순으로 확인해서 먼저 발견되는 트랜잭션 속성을 적용하는 정책
1. 타깃 메소드
2. 타깃 클래스
3. 선언 메소드
4. 선언 타입(클래스나 인터페이스) 

### 활용법

#### 비지니스 로직과의 결합

DBMS로부터 읽어온 데이터를 사용하고 변경하는 등의 작업을 하는 곳이 서비스 계층
→ 비지니스 로직이 있는 서비스 계층과 결합
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder passwordEncoder;

    public List<User> getUserList() {
        return userRepository.findAll();
    }
}
```
테스트와 같은 특별한 이유가 아니고는 다른 계층이나 모듈에서 DAO에 직접 접근을 차단
DAO에 접근할 때는 서비스 계층을 거치도록 하는 것이 바람직

#### 읽기 전용 트랜잭션의 공통화

클래스 레벨에는 공통적으로 적용되는 읽기전용 트랜잭션 어노테이션을 선언
추가나 삭제 또는 수정이 있는 작업에는 메소드에 별도로 @Transacional 어노테이션을 따로 선언
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder passwordEncoder;

    public List<User> getUserList() {
        return userRepository.findAll();
    }

    @Transactional
    public User signUp(final SignUpDTO signUpDTO) {
        final User user = User.builder()
                .email(signUpDTO.getEmail())
                .pw(passwordEncoder.encode(signUpDTO.getPw()))
                .role(UserRole.ROLE_USER)
                .build();

        return userRepository.save(user);
    }
}
```

#### 테스트의 롤백(Rollback)

트랜잭션 어노테이션을 테스트에 활용하면 테스트를 진행하는 동안에 조작한 데이터를 모두 롤백
어떠한 경우에도 커밋을 하지 않기 때문에 테스트가 성공하거나 실패해도 상관이 없음
```java
@Transactional
@ExtendWith(SpringExtension.class)
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void findByEmailAndPw() {
        final User user = User.builder()
                .email("email")
                .pw("pw")
                .role(UserRole.ROLE_USER).build();
        userRepository.save(user);

        assertThat(userRepository.findAll().size()).isEqualTo(1);
    }
}
```
테스트 메소드 안에서 진행되는 작업을 하나의 트랜잭션으로 묶고는 싶지만 DB에 반영이 되게 하고 싶다면, @Rollback(false) 이용
@Rollback은 메소드에만 적용 가능하므로, 만약 테스트의 클래스 레벨에 Rollback 설정을 적용하고 싶다면, 클래스 레벨에 @TransactionConfiguration(defaultRollback=false)를 설정하고, 롤백을 원하는 메소드에만 @Rollback(true) 적용
auto_increment나 sequence 등에 의해 증가된 값은 롤백이 되지 않기 때문에, 테스트를 위해서는 별도의 데이터베이스로 연결을 하거나 또는 H2와 같은 휘발성 데이터베이스(인메모리)를 사용

<br>