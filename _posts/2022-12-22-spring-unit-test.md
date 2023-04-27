---
layout: single
title: "Spring 단위 테스트에 대하여"
date: 2022-12-22 10:15:22 +0900
categories: java
tags: [TDD, JUnit5, AssertJ, Mockito, GWT]
typora-root-url: ../
---


## Spring 단위 테스트에 대하여
> - 올바른 단위 테스트
> - 필요한 프레임워크와 테스트 패턴
> - 테스트용 모의 객체 프레임워크

<br>

## 올바른 단위 테스트

올바른 단위 테스트 코드란 `하나의 테스트 케이스는 하나의 관심사에 대해서만 테스트` 하도록 최대한 독립적으로 작성

### 예시

예를 들어 다음과 같이 sendMailIfRegisteredEmail 함수는 isEmailRegistered를 호출하여 사용하는 메일 전송 서비스 클래스가 있을 경우

```java
@Service
@RequiredArgsConstructor
public class MailService {

    private final MailRepository mailRepository;
    
    public boolean sendMailIfRegisteredEmail(final String email) {
        if(isEmailRegistered(email)) {
            return sendMail(email);
        }
        return false;
    }

    public boolean isEmailRegistered(final String email) {
        return mailRepository.existsByEmail(email);
    }

    public boolean sendMail(String email) {
    }

}
```

위 클래스에 대해 다음과 같은 테스트 코드를 작성

```java
@ExtendWith(MockitoExtension.class)
class MailServiceTest {

    @InjectMocks
    private MailService mailService;

    @Mock
    private MailRepository mailRepository;

    private final String email = "mangkyu@email.com";

    @DisplayName("Send mail successfully")
    @Test
    void sendMailIfRegisteredEmailSuccess() {
        // given
        doReturn(true).when(mailRepository).existsByEmail(email);

        // when
        final boolean result = mailService.sendMailIfRegisteredEmail(email);

        // then
        assertThat(result).isTrue();

        // verify
        verify(mailService, times(1)).sendMail(email);
    }

    @DisplayName("Failed to send mail - unregistered mail")
    @Test
    void sendMailIfRegisteredEmailFail_EmailNotRegistered() {
        // given
        doReturn(false).when(mailRepository).existsByEmail(email);

        // when
        final boolean result = mailService.sendMailIfRegisteredEmail(email);

        // then
        assertThat(result).isFalse();

        // verify
        verify(mailService, times(0)).sendMail(email);
    }

    @DisplayName("Verify that a mail address exists")
    @Test
    void isEmailRegisteredSuccess() {
        // given
        doReturn(true).when(mailRepository).existsByEmail(email);

        // when
        final boolean result = mailService.sendMailIfRegisteredEmail(email);

        // then
        assertThat(result).isFalse();
    }

}
```

위의 테스트 코드는 sendMailIfRegisteredEmail 에 대한 테스트 코드가 isEmailRegisteredSuccess와 많은 부분에서 중복됨

그 이유는 sendMailIfRegisteredEmail의 테스트가 isEmailRegistered에 대한 구현에 의존하기 때문

- 테스트에 대한 관심이 여러가지며, 독립적이지 못함
- 변경이 전파될 수 있음

sendMailIfRegisteredEmail의 테스트를 한가지 관심만 갖도록 isEmailRegistered를 stub하여 다음과 같이 코드 작성

```java
@ExtendWith(MockitoExtension.class)
class MailServiceTest {

    @InjectMocks
    private MailService mailService;

    @Mock
    private MailRepository mailRepository;

    private final String email = "mangkyu@email.com";

    @DisplayName("Send mail successfully")
    @Test
    void sendMailIfRegisteredEmailSuccess() {
        // given
        doReturn(true).when(mailService).isEmailRegistered(email);

        // when
        final boolean result = mailService.sendMailIfRegisteredEmail(email);

        // then
        assertThat(result).isTrue();

        // verify
        verify(mailService, times(1)).sendMail(email);
    }

    @DisplayName("Failed to send mail - unregistered mail")
    @Test
    void sendMailIfRegisteredEmailFail_EmailNotRegistered() {
        // given
        doReturn(false).when(mailService).isEmailRegistered(email);

        // when
        final boolean result = mailService.sendMailIfRegisteredEmail(email);

        // then
        assertThat(result).isFalse();

        // verify
        verify(mailService, times(0)).sendMail(email);
    }

}
```

위와 같이 테스트 코드 작성 시, 다음과 같은 에러 발생

```console
Argument passed to when() is not a mock!
Example of correct stubbing:
    doThrow(new RuntimeException()).when(mock).someMethod();
org.mockito.exceptions.misusing.NotAMockException: 
Argument passed to when() is not a mock!
```

즉, 테스트 대상으로써 Mock 객체를 주입받는 MailService는 Mock 객체가 아니라 실제 객체이기 때문이 stub이 안됨

그래서 이에 대한 해결책으로 @Spy 어노테이션을 활용

여기에서 @Spy는 Stub하지 않은 메소드들은 원본 메소드 그대로 사용하는 어노테이션

MailService에 @InjectMocks와 함께 @Spy를 붙여주면 isEmailRegistered를 stubbing하여 1개의 책임을 갖는 단위 테스트로 수정

최종적으로 수정한 테스트 코드

```java
@ExtendWith(MockitoExtension.class)
class MailServiceTest {

    @Spy
    @InjectMocks
    private MailService mailService;

    @Mock
    private MailRepository mailRepository;

    private final String email = "mangkyu@email.com";

    @DisplayName("Send mail successfully")
    @Test
    void sendMailIfRegisteredEmailSuccess() {
        // given
        doReturn(true).when(mailService).isEmailRegistered(email);

        // when
        final boolean result = mailService.sendMailIfRegisteredEmail(email);

        // then
        assertThat(result).isTrue();

        // verify
        verify(mailService, times(1)).sendMail(email);
    }

    @DisplayName("Failed to send mail - unregistered mail")
    @Test
    void sendMailIfRegisteredEmailFail_EmailNotRegistered() {
        // given
        doReturn(false).when(mailService).isEmailRegistered(email);

        // when
        final boolean result = mailService.sendMailIfRegisteredEmail(email);

        // then
        assertThat(result).isFalse();

        // verify
        verify(mailService, times(0)).sendMail(email);
    }

}
```

단위 테스트는 최대한 독립적으로 작성하여, 1가지 기능에 대해서만 테스트하는 것이 좋음

<br>

## 필요한 프레임워크와 테스트 패턴

### 필요한 프레임워크

- JUnit5: 자바 단위 테스트를 위한 테스팅 프레임워크
- AssertJ: 자바 테스트를 돕기 위해 다양한 문법을 지원하는 라이브러리

JUnit 만으로도 단위 테스트를 충분히 작성할 수 있지만 JUnit에서 제공하는 assertEquals()와 같은 메소드는 AssertJ가 주는 메소드에 비해 가독성이 떨어지기 때문에 순수 Java 애플리케이션에서 단위 테스트를 위해 JUnit5와 AssertJ 조합을 많이 사용

### 테스트 패턴

테스트의 패턴은 크게 AAA(Arrange-Act-Assert, 준비-실행-검증)의 GWT(Given-When-Then) 패턴으로 작성

- Given(준비): 어떠한 데이터가 준비되었을 때
- When(실행): 어떠한 함수를 실행하면
- Then(검증): 어떠한 결과가 나와야 한다

추가적으로 어떤 메소드가 몇번 호출되었는지를 검사하기 위한 verify도 사용

```java
@DisplayName("로또 번호 갯수 테스트")
@Test
void lottoNumberSizeTest() {
    // given

    // when

    // then
}
```
JUnit은 테스트 패키지 하위의 @Test 어노테이션이 붙은 메소드를 단위 테스트로 인식

@DisplayName 어노테이션을 사용하여 다른 테스트 명을 부여할 수 있음

### 예시

다음과 같이 1000원을 주면 1개의 로또를 생성해주는 클래스

```java
public class LottoNumberGenerator {

    public List<Integer> generate(final int money) {
        if (!isValidMoney(money)) {
            throw new RuntimeException("올바른 금액이 아닙니다.");
        }
        return generate();
    }

    private boolean isValidMoney(final int money) {
        return money == 1000;
    }

    private List<Integer> generate() {
        return new Random()
                .ints(1, 45 + 1)
                .distinct()
                .limit(6)
                .boxed()
                .collect(Collectors.toList());
    }

}
```

위와 같은 로또 번호 생성 코드에 대한 테스트 코드들을 작성

1. 로또 번호 갯수 테스트

```java
@DisplayName("로또 번호 갯수 테스트")
@Test
void lottoNumberSizeTest() {
    // given
    final LottoNumberGenerator lottoNumberGenerator = new LottoNumberGenerator();
    final int price = 1000;

    // when
    final List<Integer> lottoNumber = lottoNumberGenerator.generate(price);

    // then
    assertThat(lotto.size()).isEqualTo(6);
}
```
2. 로또 번호 범위 테스트

```java
@DisplayName("로또 번호 범위 테스트")
@Test
void lottoNumberRangeTest() {
    // given
    final LottoNumberGenerator lottoNumberGenerator = new LottoNumberGenerator();
    final int price = 1000;

    // when
    final List<Integer> lotto = lottoNumberGenerator.generate(price);

    // then
    assertThat(lotto.stream().allMatch(v -> v >= 1 && v <= 45)).isTrue();
}
```
3. 잘못된 로또 금액 테스트

```java
@DisplayName("잘못된 로또 금액 테스트")
@Test
void lottoNumberInvalidMoneyTest() {
    // given
    final LottoNumberGenerator lottoNumberGenerator = new LottoNumberGenerator();
    final int price = 2000;

    // when
    final RuntimeException exception = assertThrows(RuntimeException.class, () -> lottoNumberGenerator.generate(price));

    // then
    assertThat(exception.getMessage()).isEqualTo("올바른 금액이 아닙니다.");
}
```
이와 같이 다른 목적의 테스트에는 그에 맞는 메소드를 사용하여 단위 테스트를 작성

예시에서는 외부 객체와 데이터를 주고 받는 상호작용이나 의존성이 없었지만, 애플리케이션에서는 여러 객체들 간 의존성이 생기고, 그에 따라 모의 객체를 주입시켜 테스트를 작성할 필요 생김

그런 경우 사용하는 것이 Mockito(테스트용 모의 객체 프레임워크)

<br>

## 테스트용 모의 객체 프레임워크

### Mockito란?

개발자가 동작을 직접 제어할 수 있는 모의 객체를 지원하는 테스트 프레임워크

일반적으로 Spring으로 웹 애플리케이션을 개발하면, 여러 객체들 간의 의존성이 생김

이러한 의존성은 단위 테스트를 작성을 어렵게 하는데, 이를 해결하기 위해 가짜 객체를 주입시켜주는 Mockito 프레임워크를 활용

Mockito를 활용하면 가짜 객체에 원하는 결과를 Stub하여 단위 테스트 진행 가능

### 사용법

#### 1. Mock 객체 의존성 주입

Mockito에서 가짜 객체의 의존성 주입을 위해서는 크게 3가지 어노테이션이 사용

- @Mock: 가짜 객체를 만들어 반환해주는 어노테이션
- @Spy: Stub하지 않은 메소드들은 원본 메소드 그대로 사용하는 어노테이션
- @InjectMocks: @Mock 또는 @Spy로 생성된 가짜 객체를 자동으로 주입시켜주는 어노테이션

예를 들어 UserController에 대한 단위 테스트를 작성하고자 할 때, UserService를 사용하고 있다면 @Mock 어노테이션을 통해 가짜 UserService를 만들고, @InjectMocks를 통해 UserController에 이를 주입

#### 2. Stub로 결과 처리

Mockito에서는 다음과 같은 stub 메소드를 통해 정해진 결과를 반환하게 할 수 있음

- doReturn(): 가짜 객체가 특정한 값을 반환해야 하는 경우
- doNothing(): 가짜 객체가 아무 것도 반환하지 않는 경우(void)
- doThrow(): 가짜 객체가 예외를 발생시키는 경우

#### 3. Mockito와 Junit의 결합

Mockito도 테스팅 프레임워크이기 때문에 JUnit과 결합되기 위해서 @ExtendWith(MockitoExtension.class)를 사용해서 결합

### 예시

#### 컨트롤러 단위 테스트

다음과 같은 회원 가입 API와 사용자 목록 조회 API가 있고, 이에 대한 단위 테스트를 작성해주어야 하는 경우

```java
@RestController
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping("/users/signUp")
    public ResponseEntity<UserResponse> signUp(@RequestBody SignUpRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(userService.signUp(request));
    }

    @GetMapping("/users")
    public ResponseEntity<List<UserResponse>> findAll() {
        return ResponseEntity.ok(userService.findAll());
    }
}
```

다음과 같이 MockMvc를 생성

```java
@ExtendWith(MockitoExtension.class)
class UserControllerTest {

    @InjectMocks
    private UserController userController;

    @Mock
    private UserService userService;

    private MockMvc mockMvc;

    @BeforeEach
    public void init() {
        mockMvc = MockMvcBuilders.standaloneSetup(userController).build();
    }

}
```

이제 회원가입 성공과 사용자 목록 조회 케이스에 대한 테스트 코드 작성

```java
@DisplayName("회원 가입 성공")
@Test
void signUpSuccess() throws Exception {
    // given
    SignUpRequest request = signUpRequest();
    UserResponse response = userResponse();
    
    doReturn(response).when(userService)
        .signUp(any(SignUpRequest.class));

    // when
    ResultActions resultActions = mockMvc.perform(
        MockMvcRequestBuilders.post("/users/signUp")
                .contentType(MediaType.APPLICATION_JSON)
                .content(new Gson().toJson(request))
    );

    // then
    MvcResult mvcResult = resultActions.andExpect(status().isOk())
        .andExpect(jsonPath("email", response.getEmail()).exists())
        .andExpect(jsonPath("pw", response.getPw()).exists())
        .andExpect(jsonPath("role", response.getRole()).exists())
}

@DisplayName("사용자 목록 조회")
@Test
void getUserList() throws Exception {
    // given
    doReturn(userList()).when(userService)
        .findAll();

    // when
    ResultActions resultActions = mockMvc.perform(
        MockMvcRequestBuilders.get("/user/list")
    );

    // then
    MvcResult mvcResult = resultActions.andExpect(status().isOk()).andReturn();
    
    UserListResponseDTO response = new Gson().fromJson(mvcResult.getResponse().getContentAsString(), UserListResponseDTO.class);
    assertThat(response.getUserList().size()).isEqualTo(5);
}

private List<UserResponse> userList() {
    List<UserResponse> userList = new ArrayList<>();
    for (int i = 0; i < 5; i++) {
        userList.add(new UserResponse("test@test.test", "test", UserRole.ROLE_USER));
    }
    return userList;
}
```

스프링 부트는 컨트롤러 테스트를 위한 @WebMvcTest 어노테이션을 제공

- MockMvc 객체 자동 생성
- 웹 계층 테스트에 필요한 요소들(ControllerAdvice나 Filter, Interceptor 등)을 모두 bean 등록(=스프링 컨텍스트 환경 구성)

@WebMvcTest는 스프링 부트가 제공하는 테스트 환경이므로, @Mock과 @Spy 대신 각각 @MockBean과 @SpyBean을 사용해야 함

@WebMvcTest는 스프링 부트에서 제공하는 테스트 환경이므로, 보다 빠른 테스트를 원한다면 직접 MockMvc를 생성했던 처음 방법을 사용하는 것이 좋음(굳이 특정 컨트롤러를 bean으로 만들고 새로운 컨텍스트를 생성해서 mocking할 필요 없음)

```java
@WebMVcTest(UserController.class)
class UserControllerTest {

    @MockBean
    private UserService userService;

    @Autowired
    private MockMvc mockMvc;
}
```

#### 서비스 단위 테스트

다음과 같은 비지니스 로직 레이어(Service Layer)가 있고, 이에 대한 단위 테스트를 작성해주어야 하는 경우

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserServiceImpl {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder passwordEncoder;

    @Transactional
    public UserResponse signUp(final SignUpRequest request) {
        final User user = User.builder()
                .email(request.getEmail())
                .pw(passwordEncoder.encode(request.getPw()))
                .role(UserRole.ROLE_USER)
                .build();

        return UserResponse.of(userRepository.save(user));
    }

    public List<User> findAll() {
        return userRepository.findAll().stream()
            .map(UserResponse::of)
            .collect(Collectors.toList()));
    }
}
```

다음과 같이 @ExtendWith(MockitoExtension.class)와 모의 객체 주입

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @InjectMocks
    private UserService userService;

    @Mock
    private UserRepository userRepository;

    @Spy
    private BCryptPasswordEncoder passwordEncoder;

}
```

이번에는 BCryptPasswordEncoder에 @Spy를 사용

앞서 설명하였듯 Spy는 Stub하지 않은 메소드는 실제 메소드로 동작하게 하는데, 위의 예제에서 실제로 사용자 비밀번호를 암호화해야 하므로, BCryptPasswordEncoder에 @Spy를 사용

이제 회원가입 성공과 사용자 목록 조회 케이스에 대한 테스트 코드 작성

```java
@DisplayName("회원 가입")
@Test
void signUp() {
    // given
    BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
    SignUpRequest request = signUpRequest();
    String encryptedPw = encoder.encode(request.getPw());

    doReturn(new User(request.getEmail(), encryptedPw, UserRole.ROLE_USER)).when(userRepository)
        .save(any(User.class));
        
    // when
    UserResponse user = userService.signUp(request);

    // then
    assertThat(user.getEmail()).isEqualTo(request.getEmail());
    assertThat(encoder.matches(signUpDTO.getPw(), user.getPw())).isTrue();

    // verify
    verify(userRepository, times(1)).save(any(User.class));
    verify(passwordEncoder, times(1)).encode(any(String.class));
}

@DisplayName("사용자 목록 조회")
@Test
void findAll() {
    // given
    doReturn(userList()).when(userRepository)
        .findAll();

    // when
    final List<UserResponse> userList = userService.findAll();

    // then
    assertThat(userList.size()).isEqualTo(5);
}

private List<User> userList() {
    List<User> userList = new ArrayList<>();
    for (int i = 0; i < 5; i++) {
        userList.add(new User("test@test.test", "test", UserRole.ROLE_USER));
    }
    return userList;
}
```

이번에는 추가적으로 mockito의 verify()를 사용

verify는 Mockito 라이브러리를 통해 만들어진가짜 객체의 특정 메소드가 호출된 횟수 검증 가능

위에서는 passwordEncoder의 encode 메소드와 userRepository의 save 메소드가 각각 1번만 호출되었는지를 검증하기 위해 사용

#### 레포지토리 단위 테스트

다음과 같은 사용자 회원가입과 목록 조회를 위한 JPA 레포지토리가 있고, 이에 대한 단위 테스트를 작성해주어야 하는 경우

```java
public interface UserRepository extends JpaRepository <User, Long> {}
```

이제 사용자 추가와 사용자 목록 조회 케이스에 대한 테스트 코드 작성

스프링 부트는 JPA 레포지토리를 손쉽게 테스트할 수 있는 @DataJpaTest 어노테이션을 제공

@DataJpaTest를 사용하면 기본적으로 인메모리 데이터베이스인 H2를 기반으로 테스트용 데이터베이스를 구축하며, 테스트가 끝나면 트랜잭션 롤백을 해줌

레포지토리 계층은 실제 DB와 통신없이 단순 모킹하는 것은 의미가 없으므로 직접 데이터베이스와 통신하는 @DataJpaTest를 사용

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @DisplayName("사용자 추가")
    @Test
    void addUser() {
        // given
        User user = user();
        
        // when
        User savedUser = userRepository.save(user);

        // then
        assertThat(savedUser.getEmail()).isEqualTo(user.getEmail());
        assertThat(savedUser.getPw()).isEqualTo(user.getPw());
        assertThat(savedUser.getRole()).isEqualTo(user.getRole());
    }

    @DisplayName("사용자 목록 조회")
    @Test
    void addUser() {
        // given
        userRepository.save(user());
        
        // when
        List<User> userList = userRepository.findAll();

        // then        
        assertThat(userList.size()).isEqualTo(1);
    }

    private User user() {
        return User.builder()
                .email("email")
                .pw("pw")
                .role(UserRole.ROLE_USER).build();
    }
}
```

성공하는 케이스에 대해서만 간단히 예시로 정리했지만, 사실 실패하는 케이스에 대한 테스트 코드가 더 중요하기 때문에 실패하는 케이스에 대해서도 테스트 코드를 작성해야 함

<br>
