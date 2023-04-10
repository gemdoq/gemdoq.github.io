---
layout: single
title: "Comparable과 Comparator의 차이와 사용법"
date: 2022-11-02 09:33:41 +0900
categories: knowledge
tags: [Java, Comparable, Comparator, 정렬, 인터페이스, Overflow, Underflow]
typora-root-url: ../
---


## Comparable과 Comparator
> - 인터페이스
> - Comparable을 이용한 정렬
> - Comparator을 이용한 정렬
> - Overflow vs Underflow

<br>

## 인터페이스

Comparable과 Comparator는 모두 인터페이스(interface)다.

그래서 사용하려면 인터페이스 내에 선언된 메소드를 반드시 구현해야 한다.

인터페이스는 클래스의 `청사진(blue print)`과 같다.

예를 들어 자동차를 설계한다고 가정할 때, '자동차'라는 것 자체는 추상적인 개념이다.

대개 '자동차'라고 하면 바퀴가 4개 있고, 핸들과 기어가 있는 

동력의 전달로 이동하는 기계라는 추상적 개념이 존재한다.

이러한 `'바퀴 4개, 핸들과 기어, 동력의 전달로 이동'`이 바로 `'자동차 인터페이스'`다.

이러한 추상적인 개념을 `'클래스'`에 따라 구체화하여 구현한 제품이 바로 `'인스턴스'`다.

즉, 인터페이스에는 추상적인 개념만 있지만, 

이걸 구체화하고 구현해서 '인스턴스'를 만들어 낼 수 있는 것들이 바로 '클래스'에 담겨 있다.

그래서 인터페이스의 경우 메소드(함수)를 선언만 해놓을 뿐 

함수의 내용은 구체화 해놓지 않고, 이를 '추상 메소드'라 부른다.

```java
interface Car { 

	void setHandle();
	void setWheel();
 
	int gear();

}
```
Car 인터페이스의 추상 메소드들을 구현하면 다음과 같다.
```java
class GT350 implements Car {
  
	@Override
	void setHandle(int size) {
		handleSize = size;
	}
 
	@Override
	void setWheel() {
		WheelCount = 4;
	}
 
	@Override
	int gear() {
		return nowGear;
	}

}
```
GT350은 Car라는 인터페이스를 구현할 수 있는 구체적인 설계도, 

즉 '클래스'이며, 이를 통해 '인스턴스'를 만듦으로써 제품화가 가능하다.

Comparable과 Comparator는 모두 '인터페이스'며, 

둘 다 객체를 비교하기 위한 목적을 가진다.

<br>

## Comparable을 이용한 정렬

Comparable은 lang 패키지에 있으므로 import를 할 필요가 없다.

자기 자신과 파라미터로 들어오는 매개변수 객체를 비교한다.

Comparator 내부의 compareTo 메소드는 int값을 반환하도록 되어있다.

### 구현 방법

- 정렬할 객체에 Comparable interface를 implements 후, compareTo() 메서드를 오버라이드하여 구현한다.
- compareTo() 메서드 작성법
  - 현재 객체 < 파라미터로 넘어온 객체: 음수 리턴
  - 현재 객체 == 파라미터로 넘어온 객체: 0 리턴
  - 현재 객체 > 파라미터로 넘어온 객체: 양수 리턴
  - 음수 또는 0이면 객체의 자리가 그대로 유지되며, 양수인 경우에는 두 객체의 자리가 바뀐다.

### 구현 예시

```java
// x좌표가 증가하는 순, x좌표가 같으면 y좌표가 감소하는 순으로 정렬하라.
class Point implements Comparable<Point> {
    int x, y;

    @Override
    public int compareTo(Point p) {
        if(this.x > p.x) {
            return 1; // x에 대해서는 오름차순
        }
        else if(this.x == p.x) {
            if(this.y < p.y) { // y에 대해서는 내림차순
                return 1;
            }
        }
        return -1;
    }
}

// main에서 사용법
List<Point> pointList = new ArrayList<>();
pointList.add(new Point(x, y));
Collections.sort(pointList);
```

<br>

## Comparator를 이용한 정렬

Comparator는 util 패키지에 있으므로 import를 해줘야 한다.

파라미터로 들어오는 두 매개변수 객체를 비교한다.

Comparator 내부의 compare 메소드는 int값을 반환하도록 되어있다.

### 구현 방법

- Comparator interface를 implements 후 compare() 메서드를 오버라이드한 myComparator class를 작성한다.
- compare() 메서드 작성법
  - 첫 번째 파라미터로 넘어온 객체 < 두 번째 파라미터로 넘어온 객체: 음수 리턴
  - 첫 번째 파라미터로 넘어온 객체 == 두 번째 파라미터로 넘어온 객체: 0 리턴
  - 첫 번째 파라미터로 넘어온 객체 > 두 번째 파라미터로 넘어온 객체: 양수 리턴
- 음수 또는 0이면 객체의 자리가 그대로 유지되며, 양수인 경우에는 두 객체의 자리가 변경된다.
  - 즉, `Integer.compare(x, y)`(오름차순 정렬)와 동일한 개념이다.
    - `return (x < y) ? -1 : ((x == y) ? 0 : 1);`
  - 내림차순 정렬의 경우 두 파라미터의 위치를 바꿔준다.
    - `Integer.compare(y, x)`(내림차순 정렬)

### 구현 예시

```java
// x좌표가 증가하는 순, x좌표가 같으면 y좌표가 감소하는 순으로 정렬하라.
class MyComparator implements Comparator<Point> {
  @Override
  public int compare(Point p1, Point p2) {
    if (p1.x > p2.x) {
      return 1; // x에 대해서는 오름차순
    }
    else if (p1.x == p2.x) {
      if (p1.y < p2.y) { // y에 대해서는 내림차순
        return 1;
      }
    }
    return -1;
  }
}

// main에서 사용법
List<Point> pointList = new ArrayList<>();
pointList.add(new Point(x, y));
MyComparator myComparator = new MyComparator();
Collections.sort(pointList, myComparator);
```

<br>

## Overflow vs Underflow

Comparable과 Comparator를 활용하여 비교하고 정렬할 수 있지만,

뺄셈과정에서 자료형의 범위를 넘어버리는 경우가 발생할 수 있다.

Int자료형의 범위는 32비트(4바이트) 자료형이며, 

표현 범위가 -2,147,483,648 ~ 2,147,483,647이다.

해당 범위 밖을 넘게 되면 반대편의 값으로 넘어가게 된다.

이런 식으로 상한선(2,147,483,647) 위로 넘어가면 `Overflow`,

하한선(-2,147,483,648) 밑으로 내려가면 `Underflow`라 한다.

<br>