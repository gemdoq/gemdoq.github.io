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

트랜잭션이 완료된 이후의 상태는 데이터베이스에 영구적으로 반영되어야 하며, 비록 시스템이 실패하여도 그 상태가 일관되게 유지되어야 함

<br>

## 트랜잭션 격리 수준

### 필요성

트랜잭션이 보장해야 되는 ACID 중 Isolation(격리성)에 대해 문제점들이 있었고, 그에 따라 트랜잭션을 동시에 실행시키면서 락보다 좀 더 완화된 방법으로 문제를 해결하기 위해 트랜잭션 격리 수준을 나눠야 할 필요성이 생김

#### Dirty Read

![dirty-read](/images/2022-12-12-about-transaction/dirty-read.png){: width="560"}

한 트랜잭션(Transaction A)이 데이터에 접근하여 값을 'A'에서 'B'로 변경했고 아직 커밋을 하지 않았을때, 다른 트랜잭션(Transaction B)이 해당 데이타를 Read 하면?
Transaction B가 읽은 데이터는 B가 될 것이다. 하지만 Transaction A이 최종 커밋을 하지 않고 롤백하였다면 Transaction B는 무결성이 깨진 데이터를 사용할 것이다.

#### Non-Repeatable Read

![non-repeatable-read](/images/2022-12-12-about-transaction/non-repeatable-read.png){: width="560"}

한 트랜잭션(Transaction A)이 데이타를 Read 하고 있다. 이때 다른 트랜잭션(Transaction B)가 데이터에 접근하여 값을 변경 또는, 데이터를 삭제하고 커밋을 실행한다면?
그 후 Transaction A는 다시 해당 데이터를 Read하고자 하면 변경된 데이터 혹은 사라진 데이터를 찾게 된다.

#### Phantom Read

![phantom-read](/images/2022-12-12-about-transaction/phantom-read.png){: width="560"}

트랜잭션(Transaction A) 중에 특정 조건으로 데이터를 검색하여 결과를 얻었다. 이때 다른 트랜잭션(Transaction B)가 접근해 해당 조건의 데이터 일부를 삭제 또는 추가했을때, 아직 끝나지 않은 Transaction A이 다시 한번 해당 조건으로 데이터를 조회 하면 Transaction B에서 추가/삭제된 데이터가 함께 조회/누락 된다.

### (Level 0) Read Uncommitted

![read-uncommitted](/images/2022-12-12-about-transaction/read-uncommitted.png){: width="560"}

한 트랜잭션에서 커밋하지 않은 데이터에 다른 트랜잭션이 접근 가능하다. 즉, 커밋하지 않은 데이터를 읽을 수 있다.
이 수준은 당연히 위에서 언급한 모든 문제에 대해 발생가능성이 존재한다. 대신, 동시 처리 성능은 가장 높다.

### (Level 1) Read Committed

![read-committed](/images/2022-12-12-about-transaction/read-committed.png){: width="560"}

커밋이 완료된 데이터만 읽을 수 있다.
Dirty Read가 발생할 여지는 없으나, Read Uncommitted 수준보다 동시 처리 성능은 떨어진다.  대신 Non-Repeatable Read 및 Phantom Read는 발생 가능하다.데이타베이스들은 보통 Read Committed를 디폴트 수준으로 지정한다.

### (Level 2) Repeatable Read

![repeatable-read](/images/2022-12-12-about-transaction/repeatable-read.png){: width="560"}

트랜잭션 내에서 한번 조회한 데이터를 반복해서 조회해도 같은 데이터가 조회 된다
이는 개별 데이타 이슈인 Dirty Read나 Non-Repeatable Read는 발생하지 않지만, 결과 집합 자체가 달라지는 Phantom Read는 발생가능

### (Level 3) Serializable

![serializable](/images/2022-12-12-about-transaction/serializable.png){: width="560"}

가장 엄격한 격리 수준
위 3가지 문제점을 모두 커버 가능하다. 하지만 동시 처리 성능은 급격히 떨어질 수 있다.

![transaction-isolation-level](/images/2022-12-12-about-transaction/transaction-isolation-level.png){: width="560"}

<br>

## Spring에서 트랜잭션

JDBC를 이용하는 개발자가 직접 여러 개의 작업을 하나의 트랜잭션으로 관리하려면 Connection 객체를 공유하는 등 상당히 불필요한 작업들이 많이 생길 것
그래서 Spring은 이러한 문제를 해결하고자 트랜잭션과 관련된 핵심 기술과 트랜잭션 세부 설정들을 제공

### Spring에서 사용할 수 있는 트랜잭션 핵심기술들 

#### 동기화

트랜잭션을 시작하기 위한 Connection 객체를 특별한 저장소에 보관해두고 필요할 때 꺼내쓸 수 있도록 하는 기술
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
하지만 개발자가 JDBC가 아닌 Hibernate와 같은 기술을 쓴다면 위의 JDBC 종속적인 트랜잭션 동기화 코드들은 문제를 유발하게 된다. 대표적으로 Hibernate에서는 Connection이 아닌 Session이라는 객체를 사용하기 때문이다. 이러한 기술 종속적인 문제를 해결하기 위해 Spring은 트랜잭션 관리 부분을 추상화한 기술을 제공하고 있다.

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
하지만 위와 같은 트랜잭션 관리 코드들이 비지니스 로직 코드와 결합되어 2가지 책임을 갖고 있기 때문에 Spring에서는 AOP를 이용해 이러한 트랜잭션 부분을 핵심 비지니스 로직과 분리

#### AOP를 이용한 분리

예를 들어 다음과 같이 트랜잭션 코드와 비지니스 로직 코드가 복잡하게 얽혀있는 코드가 있을 경우
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
위의 코드는 여러 책임을 가질 뿐만 아니라 서로 성격도 다르고 주고받는 것도 없으므로 분리하는 것이 적합하다.

하지만 위의 코드를 어떻게 분리할 것인지에 대한 고민을 해야 한다. 흔히 떠올릴 수 있는 방법으로는 내부 메소드로 추출하거나 DI로 합성을 이용해 해결하거나 상속을 이용할 수 있을 것이다.

하지만 위의 어떠한 방법을 이용하여도 트랜잭션을 담당하는 기술 코드를 완전히 분리시키는 것이 불가능하였다. 그래서 Spring에서는 마치 트랜잭션 코드와 같은 부가 기능 코드가 존재하지 않는 것 처럼 보이기 위해 해당 로직을 클래스 밖으로 빼내서 별도의 모듈로 만드는 AOP(Aspect Oriented Programming, 관점 지향 프로그래밍)를 고안 및 적용하게 되었고, 이를 적용한 트랜잭션 어노테이션(@Transactional)을 지원하게 되었다. 이를 적용하면 위와 같은 코드를 핵심 비지니스 로직만 다음과 같이 남길 수 있다.
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

#### 전파(Propagation)

트랜잭션 전파란 트랜잭션의 경계에서 이미 진행중인 트랜잭션이 있거나 없을 때 어떻게 동작할 것인가를 결정하는 방식을 의미한다. 예를 들어 어떤 A 작업에 대한 트랜잭션이 진행중이고 B 작업이 시작될 때 B 작업에 대한 트랜잭션을 어떻게 처리할까에 대한 부분이다.

1. A의 트랜잭션에 참여(PROPAGATION_REQUIRED)

B의 코드는 새로운 트랜잭션을 만들지 않고 A에서 진행중이 트랜잭션에 참여할 수 있다. 이 경우 B의 작업이 마무리 되고 나서, 남은 A의 작업(2)을 처리할 때 예외가 발생하면 A와 B의 작업이 모두 취소된다. 왜냐하면 A와 B의 트랜잭션이 하나로 묶여있기 때문이다.

2. 독립적인 트랜잭션 생성(PROPAGATION_REQUIRES_NEW)

반대로 B의 트랜잭션은 A의 트랜잭션과 무관하게 만들 수 있다. 이 경우 B의 트랜잭션 경계를 빠져나오는 순간 B의 트랜잭션은 독자적으로 커밋 또는 롤백되고, 이것은 A에 어떠한 영향도 주지 않는다. 즉, 이후 A가 (2)번 작업을 하면서 예외가 발생해 롤백되어도 B의 작업에는 영향을 주지 못한다.

3. 트랜잭션 없이 동작(PROPAGATION_NOT_SUPPORTED)

B의 작업에 대해 트랜잭션을 걸지 않을 수 있다. 만약 B의 작업이 단순 데이터 조회라면 굳이 트랜잭션이 필요 없을 것이다.

이렇듯 이미 진행중인 트랜잭션이 어떻게 영향을 미칠 수 있는가에 대한 부분이 트랜잭션 전파 속성이다. 트랜잭션을 시작하는 것처럼 보이는 getTransaction() 코드가 begin이 아닌 get으로 이름이 지어진 이유도 여기에 있다. 해당 트랜잭션은 다른 트랜잭션에 묶여있을 수 있기 때문이다. 위에서 설명한 대표적인 처리 방법 외에도 다양한 처리 방법을 지원하고 있다.

#### 격리 수준(Isolation Level)

모든 DB 트랜잭션은 격리수준을 가지고 있어야 한다. 서버에서는 여러 개의 트랜잭션이 동시에 진행될 수 있는데, 모든 트랜잭션을 독립적으로 만들고 순차 진행 한다면 안전하겠지만 성능이 크게 떨어질 수 밖에 없다. 따라서 적절하게 격리수준을 조정해서 가능한 많은 트랜잭션을 동시에 진행시키면서 문제가 발생하지 않도록 제어해야 한다. 이는 JDBC 드라이버나 DataSource 등에서 재설정할 수 있고, 트랜잭션 단위로 격리 수준을 조정할 수도 있다.

DefaultTransactionDefinition에 설정된 격리수준은 ISOLATION_DEFAULT로 DataSource에 정의된 격리 수준을 따른다는 것이다.

기본적으로는 DB나 DataSource에 설정된 기본 격리 수준을 따르는 것이 좋지만, 특별한 작업을 수행하는 메소드라면 독자적으로 지정해줄 필요가 있다.

#### 제한시간()

트랜잭션을 수행하는 제한시간을 설정할 수 있다. 제한시간의 설정은 트랜잭션을 직접 시작하는 PROPAGATION_REQUIRED나 PROPAGATION_REQUIRES_NEW의 경우에 사용해야만 의미가 있다.

#### 읽기전용(ReadOnly)

읽기전용으로 설정해두면 트랜잭션 내에서 데이터를 조작하는 시도를 막아줄 수 있을 뿐만 아니라 데이터 액세스 기술에 따라 성능이 향상될 수 있다.

<br>

## Spring에서 트랜잭션 사용

###

<br>
