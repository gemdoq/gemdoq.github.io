---
layout: single
title: "Spring Security에 대해"
date: 2022-12-13 10:17:29 +0900
categories: java
tags: [Spring Security]
typora-root-url: ../
---


## Spring Security에 대해
> - Spring Security란
> - 처리 과정
> - 주요 모듈
> - SecurityConfig

<br>

## Spring Security란

### 정의

Spring 기반의 애플리케이션의 보안(인증과 권한, 인가 등)을 담당하는 프레임워크
'인증'과 '인가'에 대한 부분을 Filter 흐름에 따라 처리

### 인증과 인가

- 인증(Authentication): 해당 사용자가 본인이 맞는지를 확인
- 인가(Authorization): 인증된 사용자가 요청한 자원에 접근 권한이 있는지 확인

이러한 인증과 인가를 위해 Principal을 아이디로, Credential을 비밀번호로 사용하는 Credential 기반의 인증 방식을 사용

- Principal(접근 주체): 보호받는 Resource에 접근하는 대상
- Credential(비밀번호): Resource에 접근하는 대상의 비밀번호

<br>

## 처리 과정



<br>

## 주요 모듈

### SecurityContextHolder

보안 주체의 정보를 포함, 응용프래그램의 현재 SecurityContext에 대한 세부 정보가 저장

### SecurityContext

Authentication 객체를 보관

### Authentication

주체의 정보와 권한을 담는 인터페이스
```java
public interface Authentication extends Principal, Serializable {
    // 현재 사용자의 권한 목록을 가져옴
    Collection<? extends GrantedAuthority> getAuthorities();
    
    // credentials(주로 비밀번호)을 가져옴
    Object getCredentials();
    
    Object getDetails();
    
    // Principal 객체를 가져옴.
    Object getPrincipal();
    
    // 인증 여부를 가져옴
    boolean isAuthenticated();
    
    // 인증 여부를 설정함
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

### UsernamePasswordAuthenticationToken

Authentication을 implements한 AbstractAuthenticationToken의 하위 클래스
User의 ID가 Principal, Password가 Credential
UsernamePasswordAuthenticationToken의 첫 번째 생성자는 인증 전 객체를 생성, 두번째 생성자는 인증 후 객체를 생성
```java
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
    // 주로 사용자의 ID에 해당함
    private final Object principal;
    // 주로 사용자의 PW에 해당함
    private Object credentials;
    
    // 인증 완료 전의 객체 생성
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
		super(null);
		this.principal = principal;
		this.credentials = credentials;
		setAuthenticated(false);
	}
    
    // 인증 완료 후의 객체 생성
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
			Collection<? extends GrantedAuthority> authorities) {
		super(authorities);
		this.principal = principal;
		this.credentials = credentials;
		super.setAuthenticated(true); // must use super, as we override
	}
}

public abstract class AbstractAuthenticationToken implements Authentication, CredentialsContainer {
}
```

### AuthenticationProvider

실제 인증에 대한 부분을 처리
인증 전의 Authentication객체를 받아서 인증이 완료된 객체를 반환하는 역할
아래와 같은 AuthenticationProvider 인터페이스를 구현해서 Custom한 AuthenticationProvider을 작성해서 AuthenticationManager에 등록
```java
public interface AuthenticationProvider {
	// 인증 전의 Authenticaion 객체를 받아서 인증된 Authentication 객체를 반환
    Authentication authenticate(Authentication var1) throws AuthenticationException;

    boolean supports(Class<?> var1); 
}
```

### AuthenticationManager

인증은 AuthenticationManager에 등록된 AuthenticationProvider에 의해 처리
인증이 성공하면, 2번째 생성자를 이용해 인증이 성공한 객체(isAuthenticated=true)를 생성, Security Context에 저장하고 인증 상태를 유지하기 위해 세션에 보관
인증이 실패한 경우, AuthenticationException 발생
```java
public interface AuthenticationManager {
	Authentication authenticate(Authentication authentication) 
		throws AuthenticationException;
}
```

### ProviderManager

AuthenticationManager를 implements한 ProviderManager는 실제 인증 과정에 대한 로직을 가지고 있는 AuthenticaionProvider를 List로 필드멤버로 가지며, ProividerManager는 for문을 통해 모든 provider를 조회하면서 authenticate 처리
```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware,
InitializingBean {
    public List<AuthenticationProvider> getProviders() {
		return providers;
	}
    public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		Authentication result = null;
		boolean debug = logger.isDebugEnabled();
        //for문으로 모든 provider를 순회하여 처리하고 result가 나올 때까지 반복한다.
		for (AuthenticationProvider provider : getProviders()) {
            ....
			try {
				result = provider.authenticate(authentication);

				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException e) {
				prepareException(e, authentication);
				throw e;
			}
		}
		throw lastException;
	}
}
```

### UserDetails

인증에 성공하면 생성되는 객체
Authentication객체를 구현한 UsernamePasswordAuthenticationToken을 생성하기 위해 사용
Entity에 UserDetails를 implements하여 처리 가능
```java
public interface UserDetails extends Serializable {

    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();

    String getUsername();

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();
}
```

### UserDetailsService

UserDetailsService 인터페이스는 UserDetails 객체를 반환하는 단 하나의 메소드를 가지고 있는데, 일반적으로 이를 구현한 클래스의 내부에 UserRepository를 주입받아 DB와 연결하여 처리
```java
public interface UserDetailsService {

    UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException;
}
```

### Password Encoding

AuthenticationManagerBuilder.userDetailsService().passwordEncoder() 를 통해 패스워드 암호화에 사용될 PasswordEncoder 구현체를 지정





<br>
