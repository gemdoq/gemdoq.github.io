---
layout: single
title: "ThreadLocal에 대해"
date: 2022-12-16 10:07:17 +0900
categories: java
tags: [ThreadLocal]
typora-root-url: ../
---


## ThreadLocal에 대해
> - ThreadLocal이란?
> - 사용법
> - 주의사항
> - 활용

<br>

## ThreadLocal이란?

ThreadLocal은 JDK 1.2부터 제공된 오래된 클래스
스레드 단위로 로컬 변수를 마치 전역변수처럼 여러 메서드에서 사용 가능

### 관련 클래스

#### ThreadLocalMap

ThreadLocalMap은 ThreadLocal 클래스의 정적 내부 클래스
ThreadLocal 객체를 키로 사용하여 WeakReference를 상속받는 Entry 클래스를 보유
```java
public class ThreadLocal<T> {
	static class ThreadLocalMap {
		static class Entry extends WeakReference<ThreadLocal<?>{}
	}
}
```

#### Thread

ThreadLocalMap 값이 속하는 클래스
특정 스레드의 정보를 ThreadLocal에서 직접 호출
```java
public class Thread implements Runnable {
	/* ThreadLocal values pertaining to this thread. This map is maintained by the ThreadLocal class. */
	ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

#### ThreadLocal

외부에 공개되는 몇 가지 public 메소드를 가지는 클래스

##### set과 get 메서드

값을 저장하는 set메서드, 값을 가져오는 get
```java
public void set(T value) {
  Thread t = Thread.currentThread();
  ThreadLocalMap map = getMap(t);
  if (map != null) {
    map.set(this, value);
  } else {
    createMap(t, value); 
  }
}

public T get() {
  Thread t = Thread.currentThread();
  ThreadLocalMap map = getMap(t);
  if (map != null) {
    ThreadLocalMap.Entry e = map.getEntry(this);
    if (e != null) {
      @SuppressWarnings("unchecked")
      T result = (T)e.value;
      return result;
    }
  }
  return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
  return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
  t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

##### withInitial 메서드

변수를 생성하면서 특정 값으로 초기화하는 메서드

```java
public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {
  return new SuppliedThreadLocal<>(supplier);
}
```

##### remove 메서드

변수 값을 삭제하는 메서드

```java
public void remove() {
  ThreadLocalMap m = getMap(Thread.currentThread());
  if (m != null)
    m.remove(this);
}
```

<br>

## 사용법

<br>

## 주의사항

<br>

## 활용

<br>