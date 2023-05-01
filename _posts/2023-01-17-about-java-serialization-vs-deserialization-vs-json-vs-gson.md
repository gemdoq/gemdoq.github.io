---
layout: single
title: "자바 직렬화와 역직렬화, JSON, GSON"
date: 2023-01-17 10:19:27 +0900
categories: java
tags: [직렬화, 역직렬화, JSON, GSON]
typora-root-url: ../
---

## 자바 직렬화와 역직렬화, JSON, GSON
> - 직렬화와 역직렬화
> - JSON
> - GSON

<br>

## 직렬화와 역직렬화

자바 직렬화(Serialization)란 자바 시스템 내부에서 사용되는 `객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 스트림(stream) 형태로 데이터 변환하는 기술`을 말하고, 반대로 자바 역직렬화(Deserialization)란 자바 시스템 내부에서 사용하기 위해 `바이트(byte) 스트림(stream) 형태의 데이터를 다시 객체로 변환하여 JVM의 메모리(heap 또는 stack)에 상주시키는 기술`을 의미

Serializable 인터페이스를 구현한 클래스들을 보면 serialVersionUID라는 값을 지정

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    private static final long serialVersionUID = 362498820763181265L;
}
```

예를 들어 HashMap 클래스의 경우, 위와 같이 Serializable 인터페이스를 구현 시, 위와 같이 serialVersionUID 값을 지정(만약 별도로 지정하지 않으면, 자바 소스가 컴파일될 때 자동으로 생성)

반드시 static final long으로 선언해야 하며, 변수명도 serialVersionUID로 선언해 주어야 자바에서 인식

### 객체 저장

다음과 같은 UserDTO 클래스가 있을 경우 

```java
import java.io.Serializable;

public class UserDTO implements Serializable {
    private String username;
    private String password;

    public UserDTO(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public String toString() {
        // override
    }
}
```

다음과 같이 객체를 저장

```java
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;

public class ManageObject {
    public static void main(String[] args) {
        ManageObject manage = new ManageObject();
        String fullPath = "/Users/Documents/userDTO.md";

        UserDTO dto = new UserDTO("gemdoq", "0000");
        manage.saveObject(fullPath, dto);
    }

    public void saveObject(String fullPath, UserDTO dto) {
        FileOutputStream fos = null;
        ObjectOutputStream oos = null;
        try {
            fos = new FileOutputStream(fullPath);
            oos = new ObjectOutputStream(fos);
            oos.writeObject(dto);
            System.out.println("Write Success");
        } catch (Exception e) { 
            e.printStackTrace();
        } finally {
            if (oos != null) {
                try {
                    oos.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            if (fos != null) {
                try {
                    fos.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

위와 같이 ObjectOutputStream를 사용하여 객체를 저장하고, ObjectInputStream을 사용하여 저장해놓은 객체를 읽을 수 있음

writeObject()를 통해서 매개변수로 넘어온 객체를 저장

### 객체 읽기

다음과 같이 저장해놓은 객체 읽기

```java
import java.io.*;

public class ManageObject {
    public static void main(String[] args) {
        ManageObject manage = new ManageObject();
        String fullPath = "/Users/Documents/userDTO.md";
        manage.loadObject(fullPath);
    }

    public void loadObject(String fullPath) {
        FileInputStream fis = null;
        ObjectInputStream ois = null;
        try {
            fis = new FileInputStream(fullPath);
            ois = new ObjectInputStream(fis);
            Object obj = ois.readObject();
            UserDTO dto = (UserDTO)obj;
            System.out.println(dto);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (ois != null) {
                try {
                    ois.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        if (fis != null) {
            try {
                fis.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```
```console
UserDTO{username='gemdoq', password='0000'}
```

위와 같이 파일에 저장된 객체 정보 확인 가능

### transient

보안상 노출에 민감하거나 저장할 필요가 없는 변수 앞에 사용하면 객체를 저장하거나 다른 JVM으로 보낼 때 해당 변수는 Serializable에서 제외시킬 수 있는 예약어

```java
transient private String password;
```

<br>

## JSON

JSON(JavaScript Object Notation)은 클라이언트가 사용하는 언어에 관계없이 통일된 데이터를 주고 받을 수 있도록 serialization 가능한 값(serializable value) 또는 "키-값 쌍"으로 표현되는 개방형 표준 문자열 형식

<br>

## GSON

구글의 GSON은 JSON 데이터를 Java 객체로 생성해주는 라이브러리

### GSON 직렬화

다음과 같은 Person 클래스가 있을 경우 

```java
public class Person {
    private String name;
    private String sex;

    public Person(String name, String sex) {    
        this.name = name;
        this.sex = sex;
    }

    @Override
    public String toString() {
        // override
    }
}
```

다음과 같이 객체를 JSON 문자열 형태로 직렬화

```java
public class GsonUtil {
    public static void main(String[] args) {
        Person person = new Person("gemdoq", "male");
        Gson gson = new GsonBuilder().create();
        String json = gson.toJson(person);
        System.out.println(json);
    }
}
```
```console
{"name": "gemdoq","sex": "male"}
```

위와 같이 toJson()으로 객체를 JSON 형태로 데이터 변환

### GSON 역직렬화

다음과 같이 JSON 문자열을 객체로 역직렬화

```java
public class GsonUtil {
    public static void main(String[] args) {
        String json = "{"name": "gemdoq","sex": "male"}";
        Gson gson = new GsonBuilder().create();
        Person person = gson.fromJson(json, Person.class);
        System.out.println(person);
    }
}
```
```console
[gemdoq, male]
```

위와 같이 fromJson()으로 JSON 문자열을 객체로 데이터 변환

<br>
