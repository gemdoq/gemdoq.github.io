---
layout: single
title: "자바 throws와 throw"
date: 2023-01-14 11:22:18 +0900
categories: java
tags: [Throws, Throw]
typora-root-url: ../
---

## 자바 throws와 throw
> - Throws
> - Throw

<br>

## Throws

메소드를 정의할 때 잠재적으로 어떤 Exception이 발생할 수 있는지 명시

다음은 예제로 만든 InputEvent 클래스

```java
public class InputEvent {
    private int x;
    private int y;

    public InputEvent(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public void moveTo(int desX, int desY) throws NumberFormatException {
        if (desX < 0 || desY < 0) {
            throw new NumberFormatException("It's not allowed a number under 0 : (x: "
                    + desX + ", y: " + desY + ")");
        }
        this.x = desX;
        this.y = desY;
    }
}
```

moveTo() 메소드의 정의에 throws NumberFormatException라고 정의

사용할 때 다음과 같이 catch와 함께 사용

```java
InputEvent event = new InputEvent(10, 20);
try {
    event.moveTo(-1, -2);
} catch (NumberFormatException e) {
    // TODO
}
```

<br>

## Throw

고의로 Exception을 발생시킬 때 사용

다음은 인자로 전달된 문자를 Integer로 변환하는 Integer.parseInt()

```java
public static int parseInt(String s, int radix) throws NumberFormatException
{
    if (s == null) {
        throw new NumberFormatException("null");
    }
    if (radix < Character.MIN_RADIX) {
        throw new NumberFormatException("radix " + radix +
                " less than Character.MIN_RADIX");
    }
}
```

숫자가 아닌 문자열을 인자로 전달받으면 NumberFormatException을 throw하고, 이를 어플리케이션에서 적절히 처리하지 못하면 종료

만약 Integer.parseInt() 사용 시, Exception 발생으로 인해 어플리케이션이 종료되지 않게 하려면 다음과 같이 catch로 예외 처리

```java
String numStr = "11a";
int sum = 0;
int num = 0;
try {
    int num = Integer.parseInt(numStr);
    sum = sum + num;
} catch (NumberFormatException e) {
    // catch exception
    throw new IllegalArgumentException("This string is not a number format");
} catch (IllegalArgumentException e) {
    // catch exception
}
```

Java는 IllegalArgumentException, NumberFormatException 등의 Exception을 기본적으로 제공하지만 다음과 같이 직접 Exception을 정의할 수도 있음

```java
public class InvalidNumberException extends Exception {
    public InvalidNumberException(String message) {
        super(message);
    }
}
```

<br>