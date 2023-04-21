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

## 사용

### 테스트 코드

```java
public class ThreadLocalTest {
	
	// 스레드 클래스
	static class MadThread extends Thread {
		private static final ThreadLocal<String> threadLocal = ThreadLocal.withInitial(() -> "defaultName");
		private final String name;

		public MadThread(String name) {
			this.name = name;
		}

		@Override
		public void run() {
			System.out.printf("%s Started,  ThreadLocal: %s%n", name, threadLocal.get());
			// 스레드 로컬에 값(현재 스레드 이름) 저장
			threadLocal.set(name);
			System.out.printf("%s Finished, ThreadLocal: %s%n", name, threadLocal.get());
		}
	}

	public void runTest() {
		for (int threadCount = 1; threadCount <= 5; threadCount++) {
			final MadThread thread = new MadThread("thread-" + threadCount);
			thread.start();
		}
	}

	public static void main(String[] args) {
		new ThreadLocalTest().runTest();
	}
}
```

### 실행 결과

```console
thread-1 Started,  ThreadLocal: defaultName
thread-1 Finished, ThreadLocal: thread-1
thread-5 Started,  ThreadLocal: defaultName
thread-5 Finished, ThreadLocal: thread-5
thread-4 Started,  ThreadLocal: defaultName
thread-4 Finished, ThreadLocal: thread-4
thread-3 Started,  ThreadLocal: defaultName
thread-2 Started,  ThreadLocal: defaultName
thread-3 Finished, ThreadLocal: thread-3
thread-2 Finished, ThreadLocal: thread-2
```

스레드가 동시에 실행되기 때문에 출력 순서는 실행 때마다 다를 수 있지만 스레드 간에 간섭 없이 값이 잘 저장된 것을 확인

<br>

## 주의사항

스레드가 재활용될 수 있기 때문에 사용이 끝났다면 스레드 로컬을 비워주는 과정이 필수
스레드 로컬을 비워주지 않으면 다음과 같은 상황이 발생할 수 있음(주의)
```java
package threadlocal;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ThreadLocalTest {
	static class MadThread extends Thread {
		private static final ThreadLocal<String> threadLocal = new ThreadLocal<>();
		private final String name;

		public MadThread(String name) {
			this.name = name;
		}

		@Override
		public void run() {
			System.out.printf("%s Started,  ThreadLocal: %s%n", name, threadLocal.get());
			threadLocal.set(name);
			System.out.printf("%s Finished, ThreadLocal: %s%n", name, threadLocal.get());
		}
	}

	// 스레드 풀 선언
	private final ExecutorService executorService = Executors.newFixedThreadPool(3);

	public void runTest() {
		for (int threadCount = 1; threadCount <= 5; threadCount++) {
			final String name = "thread-" + threadCount;
			final MadThread thread = new MadThread(name);
			executorService.execute(thread);
		}

		// 스레드 풀 종료
		executorService.shutdown();

		// 스레드 풀 종료 대기
		while (true) {
			try {
				if (executorService.awaitTermination(10, TimeUnit.SECONDS)) {
					break;
				}
			} catch (InterruptedException e) {
				System.err.println("Error: " + e);
				executorService.shutdownNow();
			}
		}
		System.out.println("All threads are finished");
	}

	public static void main(String[] args) {
		new ThreadLocalTest().runTest();
	}
}
```
출력 순서는 본인의 환경에 따라 실행할 때마다 다를 수 있지만 정상적인 상황이라면 스레드가 시작될 때 출력되는 스레드 로컬의 값은 "defaultName" 이어야 함

하지만 4번과 5번 스레드가 시작될 때를 보면 이미 스레드 로컬에 값이 들어있음
```console
thread-1 Started,  ThreadLocal: defaultName
thread-3 Started,  ThreadLocal: defaultName
thread-3 Finished, ThreadLocal: thread-3
thread-2 Started,  ThreadLocal: defaultName
thread-2 Finished, ThreadLocal: thread-2
thread-4 Started,  ThreadLocal: thread-3
thread-4 Finished, ThreadLocal: thread-4
thread-1 Finished, ThreadLocal: thread-1
thread-5 Started,  ThreadLocal: thread-2
thread-5 Finished, ThreadLocal: thread-5
All threads are finished
```
이러한 결과가 발생하는 이유는 스레드 풀을 통해서 스레드가 재사용되기 때문
방지하려면 사용이 끝난 스레드 로컬 정보는 제거될 수 있도록 remove 메서드를 마지막에 명시적으로 호출
```java
public void run() {
  System.out.printf("%s Started,  ThreadLocal: %s%n", name, threadLocal.get());
  threadLocal.set(name);
  System.out.printf("%s Finished, ThreadLocal: %s%n", name, threadLocal.get());
  threadLocal.remove(); // `remove` 메서드를 호출한다.
}
```

<br>

## 활용

<br>