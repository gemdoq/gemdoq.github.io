---
title: "스프링부트로 이메일 발송 구현하기"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Email
last_modified_at: 2025-07-20
---

#### 📌 용어 한눈에
- JavaMailSender: 스프링에서 이메일 전송 처리 인터페이스  
- SMTP: 이메일 전송 프로토콜, Gmail, Naver 등 제공  
- application.yml: 스프링부트 설정 파일, SMTP 정보 관리  
- MimeMessage: HTML, 첨부파일 등 복잡한 이메일 형식  

---
#### 📌 이메일 발송이란
스프링부트에서 회원가입 인증, 비밀번호 재설정, 알림 전송 구현  
JavaMailSender로 간단히 처리  
외부 SMTP 서버(Gmail 등) 연동  

##### 실생활 비유
카페에서 손님에게 메시지 전달  
우체국(SMTP)을 통해 편지(이메일) 발송  
JavaMailSender는 편지 작성 도구  

---
#### 📌 이메일 설정

##### ✅ application.yml 설정
Gmail SMTP 연동 예시  
```yaml
spring:
  mail:
    host: smtp.gmail.com
    port: 587
    username: ${EMAIL_USERNAME}
    password: ${EMAIL_PASSWORD}
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
```

> 💡 팁  
> Gmail 앱 비밀번호 필요 (2단계 인증 후 생성)  
> 환경 변수로 `EMAIL_USERNAME`, `EMAIL_PASSWORD` 관리 추천  
> IntelliJ 환경 변수 설정으로 보안 강화  

---
#### 📌 코드 구현

##### ✅ 의존성 추가
build.gradle에 메일 의존성  
```groovy
implementation 'org.springframework.boot:spring-boot-starter-mail'
```

> 💡 팁  
> Gradle 빌드 후 IntelliJ로 의존성 자동 확인  

##### ✅ 이메일 서비스
텍스트와 HTML 이메일 처리  
```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import jakarta.mail.MessagingException;
import jakarta.mail.internet.MimeMessage;

@Service
@RequiredArgsConstructor
public class EmailService {
    private final JavaMailSender mailSender;

    public void sendSimpleEmail(String to, String subject, String text) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(text);
        message.setFrom("${spring.mail.username}");
        mailSender.send(message);
    }

    public void sendHtmlEmail(String to, String subject, String htmlContent) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(htmlContent, true);
        helper.setFrom("${spring.mail.username}");
        mailSender.send(message);
    }
}
```

> 💡 팁  
> Lombok `@RequiredArgsConstructor`로 생성자 간소화  
> application.yml에서 `spring.mail.username` 동적 참조  

##### ✅ 컨트롤러
이메일 발송 API  
```java
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/email")
@RequiredArgsConstructor
public class EmailController {
    private final EmailService emailService;

    @PostMapping("/send")
    public String sendEmail(@RequestParam String to, @RequestParam String subject, @RequestParam String text) {
        emailService.sendSimpleEmail(to, subject, text);
        return "Email sent successfully";
    }
}
```

> 💡 팁  
> IntelliJ로 `@RequestMapping` 자동완성  
> Postman으로 API 테스트 추천  

---
#### 📌 주의점
- Gmail 앱 비밀번호는 환경 변수로 관리  
- SMTP 포트 587, TLS 설정 확인  
- 대량 발송 시 SMTP 서버 제한 주의  
- HTML 이메일은 XSS 방지 위해 입력 검증  
- `@Slf4j` 로깅으로 디버깅 간편화  

---
#### 📌 이메일 기능의 가치
단순 전송 넘어 사용자 경험 향상  
회원가입, 알림 등 신뢰도 높이는 도구  
스프링부트로 설정 간소화, 핵심 로직 집중  

---
#### ✍ 느끼며
이메일 발송 처음엔 복잡해 보였음  
JavaMailSender와 application.yml로 쉽게 정리  
Gmail SMTP 연동하며 보안 중요성 깨달음  
IntelliJ 디버깅, Postman으로 테스트 편리  
환경 변수와 로깅으로 유지보수 쉬워짐  
사용자 경험까지 고려하는 설계의 묘미 느낌