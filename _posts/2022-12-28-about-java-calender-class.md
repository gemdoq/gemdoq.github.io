---
layout: single
title: "자바 Calendar 클래스"
date: 2022-12-28 11:14:31 +0900
categories: java
tags: [Calender]
typora-root-url: ../
---

## 자바 Calendar 클래스
> - 정의
> - 사용

<br>

## 정의

Calendar 클래스는 Date 클래스와 마찬가지로 날짜와 시간을 다루는 클래스

Calendar 클래스가 새로 추가되면서 Date 대부분의 메소드는 deprecated

경우에 따라, Date 메소드를 그대로 사용하거나 Calendar 클래스와 혼용

추상 클래스이기 때문에 직접 new 하여 객체 생성 불가

Calendar.getInstance() 메소드를 이용하거나 Calendar 클래스를 상속받는 GregorianCalendar 클래스를 이용하여 객체 생성

<br>

## 사용

### 주요 상수

|           상수           |        사용방법        |              설명              |
| :----------------------: | :--------------------: | :----------------------------: |
|     static int YEAR      |     Calendar.YEAR      |           현재 년도            |
|     static int MONTH     |     Calendar.MONTH     |        현재 월 (1월: 0)        |
|     static int DATE      |     Calendar.DATE      |         현재 월의 날짜         |
| static int WEEK_OF_YEAR  | Calendar.WEEK_OF_YEAR  |      현재 년도의 몇째 주       |
| static int WEEK_OF_MONTH | Calendar.WEEK_OF_MONTH |       현재 월의 몇째 주        |
|  static int DAY_OF_YEAR  |  Calendar.DAY_OF_YEAR  |        현재 년도의 날짜        |
| static int DAY_OF_MONTH  | Calendar.DAY_OF_MONTH  |         현재 월의 날짜         |
|  static int DAY_OF_WEEK  |  Calendar.DAY_OF_WEEK  | 현재 요일(일요일:1 ,토요일: 7) |
|     static int HOUR      |     Calendar.HOUR      |      현재 시간 (12시간제)      |
|  static int HOUR_OF_DAY  |  Calendar.HOUR_OF_DAY  |      현재 시간 (24시간제)      |
|    static int MINUTE     |    Calendar.MINUTE     |            현재 분             |
|    static int SECOND     |    Calendar.SECOND     |            현재 초             |

```java
public static void main(String[] args) {
  			
  			Calendar today = Calendar.getInstance();
        
  			int year = today.get(Calendar.YEAR);
        int month = today.get(Calendar.MONTH);
        int date = today.get(Calendar.DATE);
        
        int woy = today.get(Calendar.WEEK_OF_YEAR);
        int wom = today.get(Calendar.WEEK_OF_MONTH);
        
        int doy = today.get(Calendar.DAY_OF_YEAR);
        int dom = today.get(Calendar.DAY_OF_MONTH);
        int dow = today.get(Calendar.DAY_OF_WEEK);
        
        int hour12 = today.get(Calendar.HOUR);
        int hour24 = today.get(Calendar.HOUR_OF_DAY);
        int minute = today.get(Calendar.MINUTE);
        int second = today.get(Calendar.SECOND);
        
        int milliSecond = today.get(Calendar.MILLISECOND);
        int timeZone = today.get(Calendar.ZONE_OFFSET);
        int lastDate = today.getActualMaximum(Calendar.DATE);
        
        System.out.println("오늘은 " + year +"년 " + month+1 + "월" + date +"일");
        System.out.println("오늘은 올해의 " + woy +"째주, 이번달의 " + wom + "째주. " + date +"일");
        System.out.println("오늘은 이번 해의 " + doy +"일이자, 이번 달의 " + dom + "일. 요일은 " + dow +"일 (1:일요일)");
        System.out.println("현재 시각은 " + hour12 +":"+ minute + ":"+ second +", 24시간으로 표현하면 " + hour24+":"+ minute + ":"+ second);
        System.out.println("오늘은 " + year +"년 " + month+1 + "월" + date +"일");
        System.out.println("1000분의 1초 (0~999): " + milliSecond);
        System.out.println("timeZone (-12~+12): " + timeZone/(60*60*1000)); // 1000분의 1초를 시간으로 표시하기 위해 60*60*1000
        System.out.println("이 달의 마지막 날: " + lastDate);

}
```



### 주요 메소드

|                            메소드                            |                             설명                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                static Calendar getInstance()                 |      현재 날짜와 시간 정보를 가진 Calendar 객체를 생성       |
|                  boolean after(Object when)                  | when과 비교하여 현재 날짜 이후이면 true, 아니면 false를 반환 |
|                 boolean before(Object when)                  | when과 비교하여 현재 날짜 이전이면 true, 아니면 false를 반환 |
|                  boolean equals(Object obj)                  |         같은 날짜값인지 비교하여 true, false를 반환          |
|                      int get(int field)                      |    현재 객체의 주어진 값의 필드에 해당하는 상수 값을 반환    |
|                        Date getTime()                        |                현재의 객체를 Date 객체로 변환                |
|                    long getTimeInMills()                     |         객체의 시간을 1/1000초 단위로 변경하여 반환          |
|                void set(int field, int value)                |           현재 객체의 특정 필드를 다른 값으로 설정           |
|           void set(int year, int month, int date)            |         현재 객체의 년, 월, 일 값을 다른 값으로 설정         |
| void set(int year, int month, int date, int hour, int minute, int second) |   현재 객체의 년, 월, 일, 시, 분, 초 값을 다른 값으로 설정   |
|                   void setTime(Date date)                    |       date 객체의 날짜와 시간 정보를 현재 객체로 생성        |
|               void setTimeInMills(long mills)                |  현재 객체를 1/1000초 단위의 주어진 매개변수 시간으로 설정   |
|               int getActualMinimum(int field)                |            현재 객체의 특정 필드의 최소 값을 반환            |
|               int getActualMaximum(int field)                |            현재 객체의 특정 필드의 최대 값을 반환            |





<br>
