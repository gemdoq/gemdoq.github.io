---
layout: single
title: "메시지큐 알림수단 별 인터페이스화"
date: 2024-12-18 13:21:23 +0900
categories: springboot
tags: [message queue, spring integration, notification, interface]
typora-root-url: ../

---

# Spring Integration과 PostgreSQL로 구현한 Outbox 패턴 MessageQueue

## Spring Integration과 PostgreSQL로 구현한 Outbox 패턴 MessageQueue의 정의

> Spring Integration과 JdbcChannelMessageStore를 사용해 PostgreSQL로 메시지를 저장하고 꺼내는 Outbox 패턴을 만들고, 이메일·SMS 같은 알림수단을 자유롭게 선택하여 알림을 비동기적으로 전송할 수 있는 유연한 시스템을 구축하는 것을 목표로 함

Spring Integration과 PostgreSQL을 활용하여 Outbox 패턴을 구현, 비즈니스 로직과 알림 전송을 하나의 트랜잭션에서 처리하며 동일성과 원자성을 보장

---

### 사전 용어 정의

- 비동기: 작업을 기다리지 않고 동시에 여러 일을 처리하는 방식
- Polling: 주기적으로 데이터를 확인해서 처리하는 방법
- Transaction: 여러 작업을 하나로 묶어, 모두 성공하거나 모두 실패하게끔 보장하는 작업 단위
- 동일성: 시스템 안에서 데이터가 서로 맞아 떨어지는 상태를 유지되는 것을 의미
- 일관성: 시스템 전체가 정해진 조건이 계속 유지되는 것을 의미
- 원자성: 작업이 모두 완료되거나 아예 안 되는, 중간 상태가 없는 특성(예: true 혹은 false)

---

### MessageQueue에 대해

> 서로 다른 여러 시스템이 데이터를 비동기적으로 전송하기 위해 사용하는 중간 데이터 저장소. 저장되는 데이터는 주로 "메시지"라고 불리는 데이터 조각이며, 특정 작업이나 이벤트에 대한 정보를 담는 형태. 시스템 결합도를 낮추고, 메시지 손실을 방지함

#### MessageQueue 간략한 작동 방식

1. 애플리케이션에서 특정 이벤트가 발생하면, 그에 대한 정보를 담은 메시지를 생성
3. 생성된 메시지는 RabbitMQ, Kafka 등 메시지큐 시스템에 전송
5. 수신자가 준비될 때까지 메시지는 큐에 대기
7. 메시지 큐에 있는 메시지를 다른 서비스나 컴포넌트가 수신
9. 수신한 메시지를 기반으로 주문 처리, 알림 전송 등 필요한 작업을 실행
11. 작업이 성공적으로 완료되면, 해당 메시지는 큐에서 제거, 확인 처리

---

### Outbox 패턴에 대해

> 동일 트랜잭션 내에서 비즈니스 데이터(예: 주문 정보)와 이벤트 데이터(예: 알림 요청)를 함께 저장하고, 저장은 한 번에 안전하게 하고(transaction), 나중에 따로 꺼내서 처리(비동기). 메시지 손실을 방지하고, 문제 발생 시 재시도

#### Outbox 패턴 간략한 작동 방식

1. 비즈니스 로직 처리 시 이벤트 데이터를 담고 있는 메시지를 Outbox 테이블에 저장
    - 데이터 일관성을 유지하는 데 목적이 있음
3. Polling 또는 Trigger를 이용해 메시지를 읽고 브로커에 전달
    - 별도의 비동기 프로세스가 Outbox 테이블에서 읽어서 Kafka, RabbitMQ 등의 메시지 브로커에 전달
    - 처리된 메시지는 삭제하거나 처리 완료 상태로 업데이트

---

### Spring Integration 방식에 대하여

> 여러 시스템들이 메시지 기반으로 데이터를 전송하고 처리하는 방법을 제공하는 프레임워크. JdbcChannelMessageStore를 통해 메시지를 PostgreSQL에 저장하고, IntegrationFlow와 Polling으로 메시지를 비동기 처리. 이를 통해 더 확장성 있고 유연한 비즈니스 요구사항에 대한 구현 가능

#### 메시지 큐에서의 Spring Integration 사용 기대 효과

- 요청에 대해 비동기 처리를 통해 응답을 기다리지 않고 다른 작업이 가능
- 메시지 라우팅과, 변환 및 필터링 기능이 제공되어 유연한 아키텍쳐를 구성할 수 있음
- 여러 인스턴스가 동일한 큐에서 메시지를 분산 처리하므로 부하 분산
- 메시지 처리 중 발생하는 예외 관리

---

### 알림수단 별 전송방식

#### 보통 기존의 이메일 전송방식

> 이메일로 메시지를 보내는 기본 방식. 설정된 SMTP 메일 서버를 통해 수신자에게 메일 전송

```java
@Component
public class EmailSender {
    @Autowired
    private JavaMailSender sender;

    public void sendMessageToEmail(String to, String subject, String body, String from) throws MessagingException {
        MimeMessage message = sender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(body, true);
        helper.setFrom(from);
        sender.send(message);
    }
}
```

```java
@Service
public class EmailService {
    @Autowired
    private EmailSender emailSender;
    
    public void sendEmail(String recipient, String subject, String message, String sender) {
        try {
            emailSender.sendMessageToEmail(recipient, subject, message, sender);
        } catch (MessagingException e) {
            e.printStackTrace();
        }
    }
}
```

#### 보통 기존의 SOLAPI SMS API 이용 방식

> SMS(문자 메시지)를 보내기 위해 외부 API(SOLAPI)를 사용하는 방식. 간단히 전화번호로 SMS 문자 전송

```java
@Component
public class SolapiSmsSender {
    @Autowired
    private SolapiApiClient client;

    public void sendMsgToSms(String to, String body, String from) {
        SMSMessage.Message msg = new SMSMessage.Message(to, body, from);
        List<SMSMessage.Message> msglist = Collections.singletonList(msg);
        SMSMessage smsMsg = new SMSMessage(msglist);
        client.sendMessage(smsMsg);
    }
}
```

```java
@Service
public class SmsService {
    @Autowired
    private SolapiSmsSender smsSender;
    
    public void sendSms(String recipient, String subject, String message, String sender) {
        try {
            smsSender.sendMsgToSms(recipient, subject, message, sender);
        } catch (MessagingException e) {
            e.printStackTrace();
        }
    }
}
```

#### 보통 기존의 SOLAPI 카카오알림톡 API 이용 방식

> 카카오톡으로 알림 메시지를 보내는 방식으로, SOLAPI를 통해 템플릿 기반으로 전송. 알림톡 템플릿이 있어야 하며, 템플릿의 변수에 맞춰 variables를 작성해 카카오톡으로 알림 메시지 전송

```java
@Component
public class SolapiAlimtokSender {
    @Autowired
    private SolapiApiClient client;
    @Value("${solapi.api.kakao.pfid}")
    private final String pfId;  
    @Value("${solapi.api.kakao.templateId}")
    private final String templateId;

    public void sendMsgToAlt(String to) {
        Map<String, String> variables = new HashMap<>();  
        variables.put("#{홍길동}", "고객");  
        variables.put("#{url}", "example-url");
        
        SolApiKakaoMessage.Message.KakaoOptions kakaoOptions = new SolApiKakaoMessage.Message.KakaoOptions(pfId, templateId, variables);  
        SolApiKakaoMessage.Message kakaoMessage = new SolApiKakaoMessage.Message(message.recipient(), kakaoOptions);  
        SolApiKakaoMessage altMsg = new SolApiKakaoMessage(Collections.singletonList(kakaoMessage));
        client.sendMessage(altMsg);
    }
}
```

```java
@Service
public class AltService {
    @Autowired
    private SolapiAlimtokSender altSender;
    
    public void sendAlt(String recipient) {
        try {
            altSender.sendMsgToAlt(recipient);
        } catch (MessagingException e) {
            e.printStackTrace();
        }
    }
}
```

---

### 개발 방향 정리

> 알림 유형(이메일, SMS, 카카오알림톡 같은 수단)을 정의. 
>
> 알림 메시지와 받을 사람, 보낼 방법을 모두 하나로 묶어서 메시지화하여 저장. 
>
> 메시지를 지령처럼 주기적으로 확인(polling)해서 어떤 알림 수단으로 보낼지 선택하여 해당 알림 수단으로 알림이 전송되게끔 설계. 
>
> Outbox 패턴으로 비즈니스 로직(예: 주문 저장)과 알림 전송을 한 번에 처리(transaction)해서 동일성과 원자성을 보장

<img width="600" alt="스크린샷" src="https://github.com/user-attachments/assets/daca0655-5063-43db-84c3-69adab99aa85">

---

### 프로젝트 스펙

> 프로젝트에 사용된 기술 스택과 의존성을 정리

```
lang: java21
spring_boot: 3.4.2
dependency_manager: 1.1.7
build_tool: Gradle
database: PostgreSQL
dependency:
- Spring Web
- Spring Data JPA
- Spring Integration
- Spring Mail
- PostgreSQL Driver
- JDBC API
- OpenFeign
- Flyway
- Lombok
```

---

### 의존성 설정

> 빌드, 의존성, 플러그인 설정을 정의하는 설정 파일

build.gradle

```groovy
plugins {  
    id 'java'  
    id 'org.springframework.boot' version '3.4.2'  
    id 'io.spring.dependency-management' version '1.1.7'  
}  
  
ext {  
    springCloudVersion = "2024.0.0"  
}  
  
group = 'com.nicenomu'  
version = '0.0.1-SNAPSHOT'  
  
java {  
    toolchain {  
        languageVersion = JavaLanguageVersion.of(21)  
    }  
}  
  
repositories {  
    mavenCentral()  
}  
  
dependencies {  
    // Spring Boot 기본 의존성  
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'  
    implementation 'org.springframework.boot:spring-boot-starter-integration'  
    implementation 'org.springframework.boot:spring-boot-starter-mail'  
    implementation 'org.springframework.boot:spring-boot-starter-web'  
  
    // Flyway와 PostgreSQL    
    implementation 'org.flywaydb:flyway-core'  
    implementation 'org.flywaydb:flyway-database-postgresql'  
    runtimeOnly 'org.postgresql:postgresql'  
  
    // Spring Integration JDBC  
    implementation 'org.springframework.integration:spring-integration-jdbc'  
  
    // SOLAPI
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'  
    implementation 'net.nurigo:sdk:4.2.7'  
  
    // JSON 직렬화를 위한 Jackson    
    implementation 'com.fasterxml.jackson.core:jackson-databind'  

    // HmacSHA256 인증을 위한 commons-codec 
    implementation 'commons-codec:commons-codec:1.17.1'
  
    // Lombok  
    compileOnly 'org.projectlombok:lombok'  
    annotationProcessor 'org.projectlombok:lombok'  
  
    // 테스트  
    testImplementation 'org.springframework.boot:spring-boot-starter-test'  
    testImplementation 'org.springframework.integration:spring-integration-test'  
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'  
}  
  
dependencyManagement {  
    imports {  
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:$springCloudVersion"  
    }  
}  
  
tasks.named('test') {  
    useJUnitPlatform()  
}
```

---

### 속성 설정

> 스프링부트 어플리케이션의 포트, DB 정보, 로그 레벨 등의 환경 설정을 관리하는 파일

src/main/resources/application.yml

```yml
spring:  
  datasource:  
    url: ${DB_URL}/${DB_NAME}  
    username: ${DB_USERNAME}  
    password: ${DB_PASSWORD}  
    driver-class-name: org.postgresql.Driver  
  jpa:  
    hibernate:  
      ddl-auto: update  
    properties:  
      hibernate:  
        format_sql: true  
    show-sql: true  
    open-in-view: false  
  mail:  # Java Mail Sender  
    host: ${MAIL_HOST}  
    port: ${MAIL_PORT}  
    username: ${MAIL_USERNAME}  
    password: ${MAIL_PASSWORD}  
    properties:  
      mail:  
        smtp:  
          auth: true  
          starttls:  
            enable: true  
  flyway:  # Flyway  
    enabled: true  
    locations: classpath:db/migration  
  
logging:  
  level:  
    org.springframework: INFO  
    org.hibernate.SQL: DEBUG  
    org.flywaydb: DEBUG  
  
solapi:  # SOLAPI  
  api:  
    key: ${SOLAPI_API_KEY}  
    secret: ${SOLAPI_API_SECRET}  
    url: ${SOLAPI_API_URL}  
    from: ${SOLAPI_API_MESSAGE_FROM}  
    kakao:  
      pfid: ${SOLAPI_API_MESSAGE_KAKAO_OPTIONS_PFID}  
      templateId: ${SOLAPI_API_MESSAGE_KAKAO_OPTIONS_TEMPLATEID}
```

---

### Flyway 마이그레이션 설정

> 어플리케이션이 실행될 때, Flyway에 의해 DB 스키마를 자동으로 생성 및 관리해주는 마이그레이션 파일

resources/db/migration/V1__init.sql

```sql
CREATE SEQUENCE int_channel_message_sequence_seq;  
  
CREATE TABLE int_channel_message (  
	group_key CHAR(36) NOT NULL,  
	message_id CHAR(36) NOT NULL,  
	region VARCHAR(100) NOT NULL,  
	created_date BIGINT NOT NULL,  
	message_priority INTEGER NOT NULL,  
	message_bytes BYTEA,  
	message_sequence BIGINT NOT NULL DEFAULT nextval('int_channel_message_sequence_seq'),  
	CONSTRAINT int_channel_message_pk PRIMARY KEY (group_key, message_id, region)  
);  
  
CREATE TABLE int_message (  
	message_id CHAR(36) NOT NULL,  
	region VARCHAR(100) NOT NULL,  
	created_date BIGINT NOT NULL,  
	message_bytes BYTEA,  
	CONSTRAINT int_message_pk PRIMARY KEY (message_id, region)  
);  
  
CREATE TABLE int_group_to_message (  
	group_key CHAR(36) NOT NULL,  
	message_id CHAR(36) NOT NULL,  
	region VARCHAR(100) NOT NULL,  
	CONSTRAINT int_group_to_message_pk PRIMARY KEY (group_key, message_id, region)  
);  
  
CREATE TABLE tb_contract (  
	id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,  
	company_name VARCHAR(255) NOT NULL,  
	representative VARCHAR(255) NOT NULL,  
	phone VARCHAR(20) NOT NULL,  
	email VARCHAR(255) NOT NULL,  
	content TEXT NOT NULL  
);
```

#### 초기 설정 후 bootRun 시, 에러

> 프로젝트 초기 설정 후 bootRun 시, 콘솔에 다음과 같은 에러 로그 출력

```plaintext
WARN 28205 --- [           main] o.f.core.internal.command.DbValidate     : No migrations found. Are your locations set up correctly?
WARN 28205 --- [           main] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaConfiguration.class]: Failed to initialize dependency 'flywayInitializer' of LoadTimeWeaverAware bean 'entityManagerFactory': Error creating bean with name 'flywayInitializer' defined in class path resource [org/springframework/boot/autoconfigure/flyway/FlywayAutoConfiguration$FlywayConfiguration.class]: Found non-empty schema(s) "public" but no schema history table. Use baseline() or set baselineOnMigrate to true to initialize the schema history table.
```

> 예외 내용을 요약하면 Flyway가 데이터베이스 스키마를 초기화하려고 시도했지만, 스키마 히스토리 테이블이 없어서 오류 발생. 이를 해결하는 방법은 Flyway를 초기화 하는 두 가지 방법이 있음

##### Flyway 초기화 방법 1: baseline() 사용

> flyway baseline() 메서드를 호출하여 스키마를 초기화하는 방법

global/configuration/flyway/FlywayConfig.java

```java
@Configuration
public class FlywayConfig {

    private final Flyway flyway;

    public FlywayConfig(Flyway flyway) {
        this.flyway = flyway;
    }

    @PostConstruct
    public void init() {
        // 기준점 설정
        flyway.baseline();
    }
}
```

##### Flyway 초기화 방법 2: baseline-on-migrate 속성 설정

> 다음 내용을 application.yml에 속성으로 추가

resources/application.yml

```yml
spring:
  flyway:  # Flyway 설정  
    enabled: true  
    locations: classpath:db/migration  
    baseline-on-migrate: true # 마이그레이션 시 기본 설정 활성화
```

> 위와 같이 속성으로 설정해주면 Flyway가 데이터베이스를 처음 봤을 때, 기존 데이터베이스 상태를 수용하고 flyway_schema_history 테이블에 Flyway Baseline을 생성. 그 이후 정의된 마이그레이션 SQL파일을 적용하면서 테이블을 추가하거나 수정

<img width="1400" alt="스크린샷" src="https://github.com/user-attachment/assets/b677ee4c-6fda-4bb6-a74c-a7fd79dded67">

> 만약, 스프링부트가 사용하는 Datasource인 데이터베이스에 다른 테이블이 존재한다면, 위 방법으로 flyway를 초기화할 때, 해당 데이터베이스의 flyway_schema_history 테이블에 다음과 같이 <Flyway Baseline>이라고 생성됨

<img width="1400" alt="스크린샷" src="https://github.com/user-attachment/assets/8b397142-2718-4dc5-854c-7d7e79daf383">

> 또한 데이터베이스에도 SQL에 의한 테이블이 생성되지 않음

<img width="400" alt="스크린샷" src="https://github.com/user-attachment/assets/d70d6132-1269-4dd2-8d62-217a98365ffd">

> 만약, 스프링부트가 사용하는 Datasource인 데이터베이스에 다른 테이블이 존재하지 않는다면, 다음과 같이 flyway에 의해 초기화가 이루어지고, migration되어 데이터베이스에 테이블이 생성됨

<img width="400" alt="스크린샷" src="https://github.com/user-attachment/assets/9b609c28-244b-409a-8738-7f5d1bf4b013">

> 그리고 다음과 같이 flyway_schema_history 테이블에 다음과 같이 해당 테이블에 대한 히스토리 레코드가 생성됨

<img width="1400" alt="스크린샷" src="https://github.com/user-attachment/assets/1c2a3d25-0c38-4860-b529-40768ebfd798">

---

### 알림 정의 및 생성

> 알림에 대한 내용을 메시지 형태로 메시지큐에 저장을 해놓아야 함. 그래야 그 메시지큐에 저장된 메시지를 가지고 다시 알림메시지로 변환해서 알림을 보낼 때 사용할 수 있음. 그렇기 때문에 메시지는 알림에 대한 항목들을 포함하고 있어야 함. 알림에 대해 수신자, 내용, 알림수단 항목들이 필요함

#### 알림수단에 대한 정보를 담고 있는 열거형 클래스

domain/notification/model/enums/NotificationType.java

```java
public enum NotificationType {  
    EMAIL("email"),  
    SMS("sms"),  
    KAKAO_ALIMTALK("kakao");  
  
    private final String value;  
  
    NotificationType(String value) {  
       this.value = value;  
    }  
  
    public String getValue() {  
       return value;  
    }  
}
```

#### 각각의 알림 메시지를 생성할 불변 레코드

domain/notification/model/record/NotificationMessage.java

```java
public record NotificationMessage(  
       String content,  
       Set<NotificationType> channels,  
       String recipient  
) implements Serializable {  
    private static final long serialVersionUID = 1L;  
}
```

#### SOLAPI를 통해 SMS를 전송하기 위한 DTO

domain/notification/model/dto/SolApiSmsMessage.java

```java
@Getter  
@Setter  
@NoArgsConstructor  
@AllArgsConstructor  
public class SolApiSmsMessage {  
    private List<Message> messages;  
  
    @Getter  
    @Setter    
    @NoArgsConstructor    
    @AllArgsConstructor    
    public static class Message {  
       private String to;  
       private String from;  
       private String text;  
    }  
}
```

#### SOLAPI를 통해 카카오알림톡을 전송하기 위한 DTO

domain/notification/model/dto/SolApiKakaoMessage.java

```java
@Getter  
@Setter  
@NoArgsConstructor  
@AllArgsConstructor  
public class SolApiKakaoMessage {  
    private List<Message> messages;  
  
    @Getter  
    @Setter    
    @NoArgsConstructor    
    @AllArgsConstructor    
    public static class Message {  
       private String to;  
       private KakaoOptions kakaoOptions;  
  
       @Getter  
       @Setter       
       @NoArgsConstructor       
       @AllArgsConstructor       
       public static class KakaoOptions {  
          private String pfId;  
          private String templateId;  
          private Map<String, String> variables;  
       }  
    }  
}
```

---

### 인프라 정의 및 생성

#### 알림 수단을 추상화한 인터페이스

```java
public interface NotificationSender {  
    void send(NotificationMessage message);  
    boolean supports(NotificationType channel);  
}
```

#### 알림 메시지를 전송하기 위한 Gateway

global/configuration/NotificationGateway.java

```java
@MessagingGateway  
public interface NotificationGateway {    
    @Gateway(requestChannel = "notificationInput")  
    void sendNotification(NotificationMessage message);  
}
```

---

### 모듈에 필요한 설정 및 정의

#### SOLAPI 인증을 위한 유틸

global/util/notification/SolApiAuthUtil.java

```java
public class SolApiAuthUtil {  
  
    public static String generateSignature(String apiSecret, String dateTime, String salt) {  
       try {  
          String data = dateTime + salt;  
          Mac mac = Mac.getInstance("HmacSHA256");  
          mac.init(new SecretKeySpec(apiSecret.getBytes(StandardCharsets.UTF_8), "HmacSHA256"));  
          String signature = new String(Hex.encodeHex(mac.doFinal(data.getBytes(StandardCharsets.UTF_8))));  
          return signature;  
       } catch (Exception e) {  
          throw new RuntimeException("Signature 생성 에러", e);  
       }  
    }  
  
    public static String generateSalt() {  
       return UUID.randomUUID().toString().replaceAll("-", "");  
    }  
  
    public static String getCurrentDateTime() {  
       return DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss'Z'")  
             .withZone(ZoneOffset.UTC)  
             .format(Instant.now());  
    }  
}
```

#### SOLAPI 연동을 위한 FeignClient 설정

global/configuration/SolApiFeignConfig.java

```java
@Configuration  
@EnableFeignClients(basePackages = "com.nicenomu.mqwithspringintegration.global.api")  
public class SolApiFeignConfig {  
  
    @Value("${solapi.api.key}")  
    private String apiKey;  
  
    @Value("${solapi.api.secret}")  
    private String apiSecret;  
  
    @Bean  
    public RequestInterceptor requestInterceptor() {  
       return requestTemplate -> {  
          String dateTime = SolApiAuthUtil.getCurrentDateTime();  
          String salt = SolApiAuthUtil.generateSalt();  
          String signature = SolApiAuthUtil.generateSignature(apiSecret, dateTime, salt);  
           
          String authorizationHeader = String.format(  
                "HMAC-SHA256 apiKey=%s, date=%s, salt=%s, signature=%s",  
                apiKey, dateTime, salt, signature  
          );  
  
          requestTemplate.header("Authorization", authorizationHeader);  
       };  
    }  
}
```

#### SOLAPI를 이용한 알림 전송을 위한 인터페이스

global/api/SolApiFeignClient.java

```java
@FeignClient(name = "solapi", url = "${solapi.api.url}", configuration = SolApiFeignConfig.class)  
public interface SolApiFeignClient {  
    @PostMapping(value = "/messages/v4/send-many/detail", consumes = "application/json")  
    String sendMessage(@RequestBody SolApiSmsMessage message);  
  
    @PostMapping(value = "/messages/v4/send-many/detail", consumes = "application/json")  
    String sendMessage(@RequestBody SolApiKakaoMessage message);  
}
```

#### 이메일을 통해 알림을 전송할 때 발생하는 예외를 처리하기 위한 예외 클래스

global/exception/MailSendException.java

```java
public class MailSendException extends RuntimeException {  
    public MailSendException(String message) {  
       super(message);  
    }  
  
    public MailSendException(String message, Throwable cause) {  
       super(message, cause);  
    }  
}
```

---

### 알림 기능 로직 정의 및 생성

#### 이메일을 통해 알림을 전송하는 구현체

global/util/notification/CustomMailSender.java

```java
@Component  
public class CustomMailSender implements NotificationSender {  
    private static final Logger log = LoggerFactory.getLogger(CustomMailSender.class);  
    private final JavaMailSender mailSender;  
  
    private String NOTIFICATION_MAIL_TITLE = "나이스 알람";  
  
    @Value("${spring.mail.username}")  
    private String NOTIFICATION_MAIL_FROM;  
  
    public CustomMailSender(JavaMailSender mailSender) {  
       this.mailSender = mailSender;  
    }  
  
    @Override  
    public void send(NotificationMessage message) throws MailSendException {  
       log.debug("이메일 전송 시도: {}", message.recipient());  
       try {  
          MimeMessage mimeMessage = mailSender.createMimeMessage();  
          MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, false, "UTF-8");  
  
          helper.setTo(message.recipient());  
          helper.setSubject(NOTIFICATION_MAIL_TITLE);  
          helper.setText(message.content());  
          helper.setFrom(NOTIFICATION_MAIL_FROM);  
          mailSender.send(mimeMessage);  
          log.info("이메일이 성공적으로 전송되었습니다: {}", message.recipient());  
       } catch (Exception e) {  
          log.error("이메일 전송 실패: {}", message.recipient(), e);  
          throw new MailSendException("메일 전송 실패: " + e.getMessage(), e);  
       }  
    }  
  
    @Override  
    public boolean supports(NotificationType channel) {  
       return channel == NotificationType.EMAIL;  
    }  
}
```

#### SOLAPI를 통해 SMS를 전송하는 구현체

global/util/notification/SolApiSmsSender.java

```java
@Component  
public class SolApiSmsSender implements NotificationSender {  
    private final SolApiFeignClient solApiFeignClient;  
    private final String from;  
  
    public SolApiSmsSender(SolApiFeignClient solApiFeignClient, @Value("${solapi.api.from}") String from) {  
       this.solApiFeignClient = solApiFeignClient;  
       this.from = from;  
    }  
  
    @Override  
    public void send(NotificationMessage message) {  
       SolApiSmsMessage.Message sms = new SolApiSmsMessage.Message(message.recipient(), from, message.content());  
       SolApiSmsMessage smsMessage = new SolApiSmsMessage(Collections.singletonList(sms));  
       solApiFeignClient.sendMessage(smsMessage);  
    }  
  
    @Override  
    public boolean supports(NotificationType channel) {  
       return channel == NotificationType.SMS;  
    }  
}
```

#### SOLAPI를 통해 카카오알림톡을 전송하는 구현체

global/util/notification/SolApiKakaoSender.java

```java
@Component  
public class SolApiKakaoSender implements NotificationSender {  
    private final SolApiFeignClient solApiFeignClient;  
    private final String from;  
    private final String pfId;  
    private final String templateId;  
  
    public SolApiKakaoSender(  
          SolApiFeignClient solApiFeignClient,  
          @Value("${solapi.api.from}") String from,  
          @Value("${solapi.api.kakao.pfid}") String pfId,  
          @Value("${solapi.api.kakao.templateId}") String templateId) {  
       this.solApiFeignClient = solApiFeignClient;  
       this.from = from;  
       this.pfId = pfId;  
       this.templateId = templateId;  
    }  
  
    @Override  
    public void send(NotificationMessage message) {  
       Map<String, String> variables = new HashMap<>();  
       variables.put("#{홍길동}", "고객");  
       variables.put("#{url}", "example-url");  
  
       SolApiKakaoMessage.Message.KakaoOptions kakaoOptions = new SolApiKakaoMessage.Message.KakaoOptions(pfId, templateId, variables);  
       SolApiKakaoMessage.Message kakaoMessage = new SolApiKakaoMessage.Message(message.recipient(), kakaoOptions);  
       SolApiKakaoMessage kakaoMsg = new SolApiKakaoMessage(Collections.singletonList(kakaoMessage));  
  
       solApiFeignClient.sendMessage(kakaoMsg);  
    }  
  
    @Override  
    public boolean supports(NotificationType channel) {  
       return channel == NotificationType.KAKAO_ALIMTALK;  
    }  
}
```

---

### 통합 흐름 정의 및 생성

#### Spring Integration에서 사용하는 MessageStore를 설정

global/configuration/MessageStoreConfig.java

```java
@Configuration  
public class MessageStoreConfig {  
    @Bean  
    public JdbcChannelMessageStore jdbcChannelMessageStore(DataSource dataSource) {  
       JdbcChannelMessageStore store = new JdbcChannelMessageStore(dataSource);  
       store.setTablePrefix("int_");  
       store.setChannelMessageStoreQueryProvider(new PostgresChannelMessageStoreQueryProvider());  
       return store;  
    }  
}
```

#### 알림 메시지를 처리하는 IntegrationFlow를 정의하고 각 채널을 등록하는 설정

global/configuration/IntegrationConfig.java

```java
@Configuration  
@EnableIntegration  
public class IntegrationConfig {  
    private static final Logger log = LoggerFactory.getLogger(IntegrationConfig.class);  
  
    @Bean(name = "notificationInput")  
    public DirectChannel notificationInput() {  
       DirectChannel channel = new DirectChannel();  
       log.info("'notificationInput' 이름으로 입력 채널 생성 완료");  
       return channel;  
    }  
  
    @Bean(name = "notificationOutbox")  
    public QueueChannel notificationOutbox(JdbcChannelMessageStore messageStore) {  
       QueueChannel channel = MessageChannels.queue(messageStore, "notification-outbox").getObject();  
       channel.addInterceptor(new ChannelInterceptor() {  
          @Override  
          public Message<?> preSend(Message<?> message, MessageChannel channel) {  
             log.debug("notificationOutbox로 메시지 전송 전: 헤더={}, 페이로드={}", message.getHeaders(), message.getPayload());  
             return message;  
          }  
  
          @Override  
          public void postSend(Message<?> message, MessageChannel channel, boolean sent) {  
             if (sent) {  
                log.debug("notificationOutbox로 메시지 전송 성공: {}", message.getPayload());  
             } else {  
                log.error("notificationOutbox로 메시지 전송 실패: {}", message.getPayload());  
             }  
          }  
  
          @Override  
          public void afterSendCompletion(Message<?> message, MessageChannel channel, boolean sent, Exception ex) {  
             if (ex != null) {  
                log.error("notificationOutbox 전송 중 예외 발생: 페이로드={}, 예외={}", message.getPayload(), ex);  
             }  
          }  
       });  
       log.info("'notificationOutbox' 이름으로 아웃박스 채널 생성 완료");  
       return channel;  
    }  
  
    @Bean  
    public IntegrationFlow notificationFlow(JdbcChannelMessageStore messageStore, List<NotificationSender> senders) {  
       return IntegrationFlow.from(notificationInput())  
             .log(LoggingHandler.Level.DEBUG, "com.matchhub.mqwithspringintegration", m -> "notificationInput에서 메시지 수신: " + m.getPayload())  
             .channel(notificationOutbox(messageStore))  
             .handle(message -> {  
                NotificationMessage msg = (NotificationMessage) message.getPayload();  
                log.debug("아웃박스에서 알림 처리 중: {}", msg);  
                senders.stream()  
                      .filter(sender -> msg.channels().stream().anyMatch(sender::supports))  
                      .forEach(sender -> {  
                         try {  
                            sender.send(msg);  
                            log.info("알림 전송 성공: {}", msg);  
                         } catch (Exception e) {  
                            log.error("알림 전송 실패: {}, 예외={}", msg, e);  
                            throw e;  
                         }  
                      });  
             }, e -> e.poller(Pollers.fixedDelay(Duration.ofSeconds(1)).maxMessagesPerPoll(1)))  
             .get();  
    }  
}
```

---

### 엔티티 모델 정의

> 알림 인터페이스를 테스트하기 위해 선택한 도메인인 "계약"에 대한 엔티티

domain/contract/model/entity/ContractEntity.java

```java
@Entity  
@Table(name = "tb_contract")  
@Getter  
@NoArgsConstructor  
@AllArgsConstructor  
public class ContractEntity {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
    private String companyName;  
    private String representative;  
    private String phone;  
    private String email;  
    private String content;  
  
    public ContractEntity(String companyName, String representative, String phone, String email, String content) {  
       this.companyName = companyName;  
       this.representative = representative;  
       this.phone = phone;  
       this.email = email;  
       this.content = content;  
    }  
}
```

> 계약 정보를 생성하고 그에 대한 알림을 전송하기 위한 요청 정보 DTO

domain/contract/model/dto/ContractRequest.java

```java
public record ContractRequest(  
       String companyName,  
       String representative,  
       String phone,  
       String email,  
       String content,  
       String recipient,  
       Set<NotificationType> channels  
) {}
```

---

### 리포지토리 생성

domain/contract/repository/ContractRepository.java

```java
public interface ContractRepository extends JpaRepository<ContractEntity, Long> {  
}
```

---

### 서비스 로직

#### 수신자에 관한 정보가 이메일 주소인지 아니면, 전화번호인지 검증하기 위한 유틸리티

> 하이픈과 같은 문자가 포함된 전화번호인 경우, 전화번호를 숫자만 남겨서 일반화

global/util/notification/RecipientValidator.java

```java
public class RecipientValidator {  
    private static final Pattern EMAIL_PATTERN = Pattern.compile("^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$");  
    private static final Pattern PHONE_PATTERN = Pattern.compile("^(\\d{2,3}-\\d{3,4}-\\d{3,4}|\\d{9,11})$");  
  
    public static boolean isEmail(String recipient) {  
       return EMAIL_PATTERN.matcher(recipient).matches();  
    }  
  
    public static boolean isPhone(String recipient) {  
       return PHONE_PATTERN.matcher(recipient).matches();  
    }  
  
    public static String normalizePhone(String recipient) {  
       if (isPhone(recipient)) {  
          return recipient.replaceAll("[\\s-]", "");  
       }  
       return recipient;  
    }  
}
```

#### 계약 생성 비즈니스 로직을 처리하고 관련 알림을 전송하는 서비스

domain/contract/service/ContractService.java

```java
@Service  
public class ContractService {  
    private static final Logger log = LoggerFactory.getLogger(ContractService.class);  
    private final ContractRepository contractRepository;  
    private final NotificationGateway notificationGateway;  
  
    public ContractService(ContractRepository contractRepository, NotificationGateway notificationGateway) {  
       this.contractRepository = contractRepository;  
       this.notificationGateway = notificationGateway;  
    }  
  
    // 비즈니스 로직과 알림 전송을 트랜잭션으로 보장하는 메서드(아웃박스 패턴)  
    @Transactional  
    public void execute(String companyName, String representative, String phone, String email, String content, String recipient, Set<NotificationType> channels) {  
       log.info("회사 {}에 대한 계약 생성 및 알림 전송 시작", companyName);  
       try {  
          ContractEntity contract = createContract(companyName, representative, phone, email, content);  
          sendNotification(contract, recipient, channels);  
          log.info("회사 {}에 대한 계약 생성 및 알림 전송 완료", companyName);  
       } catch (Exception e) {  
          log.error("회사 {}에 대한 계약 생성 및 알림 전송 실패: {}", companyName, e);  
          throw e;  
       }  
    }  
  
    // 계약 엔티티를 생성하고 저장하는 메서드  
    public ContractEntity createContract(String companyName, String representative, String phone, String email, String content) {  
       ContractEntity contract = new ContractEntity(companyName, representative, phone, email, content);  
       log.debug("계약 생성 중: {}", contract);  
       ContractEntity savedContract = contractRepository.save(contract);  
       log.debug("계약 저장 완료: {}", savedContract);  
       return savedContract;  
    }  
  
    // 계약 정보를 기반으로 알림을 전송하는 메서드  
    public void sendNotification(ContractEntity contract, String recipient, Set<NotificationType> channels) {  
       String normalizedRecipient = RecipientValidator.isPhone(recipient)  // 수신자가 전화번호인지 확인  
             ? RecipientValidator.normalizePhone(recipient)  
             : recipient;  
  
       validateRecipient(channels, normalizedRecipient);  
  
       String messageContent = String.format("계약 회사: %s\n계약 내용: %s", contract.getCompanyName(), contract.getContent());  
       NotificationMessage message = new NotificationMessage(  
             messageContent,  // 메시지 내용  
             channels,  // 알림 채널 목록  
             normalizedRecipient  // 정규화된 수신자  
       );  
       log.debug("알림 전송 중: {}", message);  
       notificationGateway.sendNotification(message);  // 게이트웨이를 통해 알림 전송  
       log.debug("알림이 게이트웨이에 전송됨: {}", message);  
    }  
  
    // 수신자와 채널의 유효성을 검사하는 메서드  
    private void validateRecipient(Set<NotificationType> channels, String recipient) {  
       if (channels.contains(NotificationType.EMAIL) && !RecipientValidator.isEmail(recipient)) {  
          throw new IllegalArgumentException("이메일 채널을 사용하려면 수신자가 이메일 주소 형식이어야 합니다.");  
       }  
       if ((channels.contains(NotificationType.SMS) || channels.contains(NotificationType.KAKAO_ALIMTALK))  
             && !RecipientValidator.isPhone(recipient)) {  
          throw new IllegalArgumentException("SMS 또는 카카오 알림톡 채널을 사용하려면 수신자가 전화번호 형식이어야 합니다.");  
       }  
    }  
}
```

---

### 컨트롤러 생성

domain/contract/controller/ContractController.java

```java
@RestController  
@RequestMapping("/api/contract")  
public class ContractController {  
    private final ContractService contractService;  
  
    public ContractController(ContractService contractService) {  
       this.contractService = contractService;  
    }  
  
    @PostMapping("/notify")  
    public ResponseEntity<String> sendNotification(@RequestBody ContractRequest request) {  
       contractService.execute(  
             request.companyName(),  
             request.representative(),  
             request.phone(),  
             request.email(),  
             request.content(),  
             request.recipient(),  
             request.channels()  
       );  
       return ResponseEntity.ok("알림 전송 요청이 완료되었습니다.");  
    }   
}
```

---

### API 테스트 결과 에러

#### 현상

> 스프링부트 기동에 성공하고, Flyway로 데이터베이스도 잘 설정되는 것을 확인(테스트 중이라 매번 새로 시작). 문제는 API테스트를 하면 Outbox 패턴에 의해 데이터베이스에 이벤트 메시지가 들어가야 하는데, 실제로 저장되지 않는 현상 발생

#### 문제 파악

> 문제를 파악하기 위해 가장 먼저 application.yml에 로깅 수준을 debug로 설정 후 로그 확인

application.yml

```yml
logging:  
  level:  
    org.springframework: INFO  
    org.hibernate.SQL: DEBUG  
    org.flywaydb: DEBUG  
    org.springframework.transaction: DEBUG  
    com.matchhub.mqwithspringintegration: DEBUG  
    org.springframework.integration.jdbc: DEBUG  
    org.springframework.jdbc: DEBUG
```

#### 로그 확인 및 문제 분석

```
INFO 7385 --- [main] o.s.i.e.SourcePollingChannelAdapter : started bean 'testFlow.org.springframework.integration.config.SourcePollingChannelAdapterFactoryBean#0'
```

> 애플리케이션 초기화 및 testFlow 시작: testFlow가 시작됨 → 주기적으로 "Test Message"를 생성하여 notificationOutbox로 전달 시도

```
DEBUG 7385 --- [scheduling-1] c.n.m.g.configuration.IntegrationConfig : notificationOutbox로 메시지 전송 전: 헤더={id=cfde9305-1a47-f5b7-fa7d-bb3ac15d5f13, timestamp=1740547048927}, 페이로드=Test Message
DEBUG 7385 --- [scheduling-1] o.s.jdbc.core.JdbcTemplate : Executing prepared SQL statement [INSERT into int_CHANNEL_MESSAGE(MESSAGE_ID, GROUP_KEY, REGION, CREATED_DATE, MESSAGE_PRIORITY, MESSAGE_BYTES) values (?, ?, ?, ?, ?, ?)]
DEBUG 7385 --- [scheduling-1] o.s.j.s.SQLStateSQLExceptionTranslator : Extracted SQL state class '23' from value '23502'
DEBUG 7385 --- [scheduling-1] o.s.i.j.store.JdbcChannelMessageStore : The Message with id [cfde9305-1a47-f5b7-fa7d-bb3ac15d5f13] already exists. Ignoring INSERT...
DEBUG 7385 --- [scheduling-1] c.n.m.g.configuration.IntegrationConfig : notificationOutbox로 메시지 전송 성공: Test Message
```

> 메시지 전달 및 저장 시도:
>
> - notificationOutbox로 메시지 전송 전 → notificationOutbox로 메시지 전달 시도
> - INSERT 쿼리 실행 → JdbcChannelMessageStore가 int_channel_message에 저장 시도
> - PostgreSQL 오류 코드 23502[🔗PostgreSQL 9.5.4 문서](https://postgresql.kr/docs/9.5/errcodes-appendix.html) → NOT NULL 제약 조건 위반
> - Ignoring INSERT → DataIntegrityViolationException 발생 → 저장 실패로 처리
> - notificationOutbox로 메시지 전송 성공 → QueueChannel은 성공으로 간주
> - 데이터베이스 int_channel_message 테이블에는 레코드가 생성되지 않음

#### 분석 결과

1. 메시지 전달 성공: notificationOutbox로 메시지 전송 전 로그 출력 → testFlow가 notificationOutbox로 메시지 전달 성공
3. 저장 실패:
    - SQLException 23502: NOT NULL 제약 조건 위반 → int_channel_message 테이블의 필수 컬럼 중 하나가 NULL로 전달됨
    - 알고 보니, JdbcChannelMessageStore클래스의 addMessageToGroup() 메서드 내의 ChannelMessageStorePreparedStatementSetter.setValues()가 필수 필드 중 하나를 NULL로 설정
    - Ignoring INSERT: JdbcChannelMessageStore가 예외를 잡고 무시 → 실제 저장 안 됨

#### 떠오른 해결 방안 3가지

1. 스키마 조정: MESSAGE_PRIORITY를 NULL 허용으로 변경 → 우선순위 없는 경우 NULL 허용
3. MESSAGE_PRIORITY 기본값 설정: 테이블에서 MESSAGE_PRIORITY에 기본값 추가 (예: DEFAULT 0). MessageStoreConfig(JdbcChannelMessageStore)에 setPriorityEnabled(true) 설정 → 우선순위 명시
5. 로그 강화: SQLException 상세 메시지 확인 → 정확한 컬럼 파악

#### 해결

```sql
CREATE TABLE int_channel_message (
    group_key CHAR(36) NOT NULL,
    message_id CHAR(36) NOT NULL,
    region VARCHAR(100) NOT NULL,
    created_date BIGINT NOT NULL,
    message_priority INTEGER NOT NULL,
    message_bytes BYTEA,
    message_sequence BIGINT NOT NULL DEFAULT nextval('int_channel_message_sequence_seq'),
    CONSTRAINT int_channel_message_pk PRIMARY KEY (group_key, message_id, region)
);
```

> message_priority INTEGER NOT NULL:  NOT NULL 제약 조건 → 값 필수 → 기본값 없음

> JdbcChannelMessageStore클래스의 addMessageToGroup() 메서드 내의 setValues()에 의해 실행되는 라이브러리 모듈 중 ChannelMessageStorePreparedStatementSetter에 아래 코드가 있음

```java
if (priorityEnabled) { 
    Object priority = message.getHeaders().get("priority"); 
    int messagePriority = priority instanceof Number ? ((Number) priority).intValue() : 0; 
    ps.setInt(pos++, messagePriority);  
}
```

> 메시지의 헤더에서 priority가 존재한다면 삼항 연산하여 priorityEnabled=false일 때 MESSAGE_PRIORITY 설정 생략 → 쿼리에 포함 안 됨 → NULL 전달 → PostgreSQL 오류 코드 `23502` → NOT NULL 제약 조건 위반

> Spring Integration의 JdbcChannelMessageStore과 그 내부 모듈, 그리고 그 라이브러리들의 기본 동작에 맞춰, `해결 방안 1번인 스키마 조정`을 통해 아래와 같이 Flyway에서 사용하는 sql 변경

```sql
CREATE TABLE int_channel_message (
    group_key CHAR(36) NOT NULL,
    message_id CHAR(36) NOT NULL,
    region VARCHAR(100) NOT NULL,
    created_date BIGINT NOT NULL,
    message_priority INTEGER, -- NOT NULL 제거
    message_bytes BYTEA,
    message_sequence BIGINT NOT NULL DEFAULT nextval('int_channel_message_sequence_seq'),
    CONSTRAINT int_channel_message_pk PRIMARY KEY (group_key, message_id, region)
);
```

---

### Spring Integration 정상 동작 확인

#### 현상

resources/application.yml

```yml
logging:  
  level:  
    org.springframework: INFO  
    org.hibernate.SQL: DEBUG  
    org.flywaydb: DEBUG  
    org.springframework.transaction: DEBUG  
    com.matchhub.mqwithspringintegration: DEBUG  
    org.springframework.integration.jdbc: DEBUG  
    org.springframework.jdbc: DEBUG
```

> 스프링부트 설정 중 org.springframework.jdbc를 DEBUG로 해놓고 스프링부트를 실행

<img width="1400" alt="스크린샷" src="https://github.com/user-attachment/assets/7e6a26a1-6f61-49b3-beaf-af1a02d82488">

> 위와 같은 로그를 볼 수 있음. 요약하면 Spring Integration에 스레드 scheduling-1에서 스케쥴링된 폴링작업에 의해 JdbcChannelMessageStore가 아웃박스 메시지 테이블에서 메시지를 폴링하기 위해 DELETE 쿼리 실행

#### 현상 세부 설명

- where절: GROUP_KEY와 REGION으로 특정 그룹의 메시지 검색
- order by절: 오래된 메시지부터 순서대로 처리
- limit 1 for update skip locked: 충돌 방지를 위해 한 번에 한 메시지만 처리
- returning MESSAGE_ID, MESSAGE_BYTES: 삭제된 메시지 데이터 반환 후 사용

#### 왜 DELETE인가?

> Outbox 패턴에서 메시지를 처리한 후 중복 처리를 방지하고 테이블을 비우기 위해, Spring Integration의 JdbcChannelMessageStore는 메시지를 폴링할 때 기본적으로 DELETE 쿼리를 사용. 
>
> Outbox 패턴의 핵심은 메시지를 한 번만 처리하고 제거하는 것이기 때문에, Outbox 패턴에서는 Spring Integration의 Flow에서 스케쥴링된 Polling 프로세스가 데이터베이스 테이블(int_channel_message)에 저장된 메시지를 읽어 Flow에 정의된 대로 처리하고, 처리가 완료되면 동시에 해당 메시지를 테이블에서 제거하는 식으로 중복 처리를 방지하여 원자성을 보장하는 원리. 
>
> 그리고 테이블에서 메시지를 삭제하지 않으면, 레코드가 지속적으로 늘어남에 따라 성능이 저하되고, Polling의 효율성이 따라서 낮아짐

---

### API 테스트

#### API 테스트를 위한 json 예시

```json
{  
  "dev": {  
    "host": "http://127.0.0.1:8080",  
    "representative": 문자열 계약자이름,  
    "phone": 문자열 계약자전화번호,  
    "email": 문자열 계약자이메일주소,  
    "content": 문자열 계약내용본문,  
    "recipient": [문자열 수신자이메일주소, 문자열 수신자핸드폰번호],  
    "channels": ["EMAIL", "SMS", "KAKAO_ALIMTALK"]  
  }  
}
```

#### 계약 생성 이메일 알림 테스트

```http
POST {{host}}/api/contract/notify  
Content-Type: application/json  
  
{  
  "companyName": "matchhub",  
  "representative": "{{representative}}",  
  "phone": "{{phone}}",  
  "email": "{{email}}",  
  "content": "{{content}}",  
  "recipient": "{{recipient[0]}}",  
  "channels": ["{{channels[0]}}"]  
}
```

##### 계약 생성 이메일 알림 테스트 결과

<img width="1400" alt="스크린샷" src="https://github.com/user-attachment/assets/73dea81a-f439-4503-bb05-b4fc483d6fed">  <img width="600" alt="스크린샷" src="https://github.com/user-attachment/assets/5519b05d-deec-4e4d-9b17-d7737ed78a8e">

#### 계약 생성 SMS 알림 테스트

```http
POST {{host}}/api/contract/notify  
Content-Type: application/json  
  
{  
  "companyName": "matchhub",  
  "representative": "{{representative}}",  
  "phone": "{{phone}}",  
  "email": "{{email}}",  
  "content": "{{content}}",  
  "recipient": "{{recipient[1]}}",  
  "channels": ["{{channels[1]}}"]  
}
```

##### 계약 생성 SMS 알림 테스트 결과

<img width="1400" alt="스크린샷" src="https://github.com/user-attachment/assets/e5548998-fa62-4e81-92b5-fc1b345883e6">  <img width="400" alt="스크린샷" src="https://github.com/user-attachment/assets/16326aa6-83ad-4e3e-aa2c-83cf882e531b">

---

### 개선할 점

> 아래 두 항목은 테스트 혹은 개발 간 임시로 설정된 값이므로 정상동작을 확인 후 추후 개선 필요(TODO)

1. 로그 레벨 조정: 너무 많은 쿼리 로그가 반복적으로 출력 → application.yml에서 로그 레벨 INFO 조정
3. 폴링 주기 조정: 현 주기 1초 → 빈 테이블에서도 쿼리 반복(상단한 부하) → IntegrationConfig.java#notificationFlow polling 주기 조정

---

### 결론

> 기존의 알림을 보내는 방식을 통합하여, 이제 알림에 대한 공통화된 수신자와 본문과 알림 수단을 정의한 메시지를 인터페이스를 사용해 메시지큐에 넣고, Spring Integration의 flow를 정의하여 비동기적(polling)으로 알림 전송 가능. 
>
> 결과적으로 비지니스 로직과 알림 전송을 하나의 트랜잭션에서 구성하여, 동일성과 원자성을 보장하는 아웃박스 패턴으로 인터페이스를 활용한 비동기적 알림 전송 기능 구현

---

## Spring Integration과 PostgreSQL로 구현한 Outbox 패턴 MessageQueue의 중요성

Spring Integration과 PostgreSQL을 활용한 Outbox 패턴은 비즈니스 로직과 알림 전송을 하나의 트랜잭션에서 처리하여 데이터의 동일성과 원자성을 보장하며, 비동기 처리를 통해 시스템의 유연성과 확장성을 높이는 데 기여. 

이를 통해 메시지 손실을 방지하고, 시스템 간 결합도를 낮추며, 부하 분산과 예외 관리를 효과적으로 수행할 수 있음

<br>