---
layout: single
title: "Dirty Checking에 대하여"
date: 2022-12-02 08:58:27 +0900
categories: java
tags: [Dirty Checking, DynamicUpdate]
typora-root-url: ../
---


## Dirty Checking에 대하여
> - Dirty Checking이란
> - @DynamicUpdate

<br>

## Dirty Checking이란

### 정의

Dirty는 `상태 변화가 생긴 것`을 의미

즉, Dirty Checking은 `상태 변경 검사`를 의미

JPA에서는 `트랜잭션이 끝나는 시점`에,

`영속성 컨텍스트가 관리하는 Entity들`의

스냅샷으로 만들어 둔 `최초 조회 상태`를 기준으로,

`상태 변화(dirty)`가 생긴 Entity 객체의 `모든 필드`를

데이터베이스에 `자동으로 반영`

### 예시

```java
@Slf4j
@RequiredArgsConstructor
@Service
public class PayService {

    public void updateNative(Long id, String tradeNo) {
        EntityManager em = entityManagerFactory.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin(); //트랜잭션 시작
        Pay pay = em.find(Pay.class, id);
        pay.changeTradeNo(tradeNo); // 엔티티만 변경
        tx.commit(); //트랜잭션 커밋
    }
}
```
데이터베이스에 update 쿼리를 전송하는 코드 없음

![dirty-checking](/images/2022-12-02-dirty-checking/dirty-checking.png){: width="560"}

변경 사항을 저장하지 않았음에도 update 쿼리가 실행

<br>

## @DynamicUpdate

### 정의

필드가 늘어날 수록 전체 필드 Update 쿼리는 비효율적

→ @DynamicUpdate를 엔티티에 사용하여 `dirty 필드만 Update 반영`

### 예시

```java
@Getter
@NoArgsConstructor
@Entity
@DynamicUpdate // 변경한 필드만 대응
public class Pay {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String tradeNo;
    private long amount;
}
```

![dynamic-update](/images/2022-12-02-dirty-checking/dynamic-update.png){: width="560"}

변경분 (trade_no)만 Update 쿼리에 반영된 것을 확인

<br>