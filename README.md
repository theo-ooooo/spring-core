# 🌱 스프링 핵심 원리 - 기본편 정리 (김영한 강의)

인프런 [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/스프링-핵심-원리-기본편) 강의를 들으면서 공부한 내용을 정리한 문서입니다.  
스프링의 핵심 개념을 실제 코드와 함께 배우고, 객체 지향 설계가 왜 중요한지 깨달을 수 있었던 좋은 강의였습니다.

---

## 📁 전체 패키지 구조 예시

```bash
src/main/java/hello/core
├── AppConfig.java
├── CoreApplication.java
├── member
│   ├── Member.java
│   ├── MemberService.java
│   ├── MemberServiceImpl.java
│   ├── MemberRepository.java
│   └── MemoryMemberRepository.java
├── order
│   ├── Order.java
│   ├── OrderService.java
│   ├── OrderServiceImpl.java
│   └── OrderApp.java
├── discount
│   ├── DiscountPolicy.java
│   ├── FixDiscountPolicy.java
│   └── RateDiscountPolicy.java
└── lifecycle
    ├── NetworkClient.java
    └── BeanLifeCycleTest.java
```

---

## 1️⃣ 객체 지향 설계의 중요성

### 💡 요약
- SRP, OCP, DIP 같은 객체지향 설계 원칙들을 개념이 아니라 **예제 코드로 체감**
- 역할과 구현을 분리하면 변경에 유연하고 확장 가능한 구조가 됨

### 🧪 예시 코드

```java
public interface MemberRepository {
    void save(Member member);
    Member findById(Long memberId);
}

public class MemoryMemberRepository implements MemberRepository {
    private Map<Long, Member> store = new HashMap<>();

    public void save(Member member) {
        store.put(member.getId(), member);
    }

    public Member findById(Long memberId) {
        return store.get(memberId);
    }
}
```

---

## 2️⃣ 순수 자바로 의존성 주입(DI) 구현하기

### 💡 요약
- 스프링 없이 AppConfig 클래스를 만들어서 직접 의존성 주입
- 관심사 분리(구성 vs 사용)의 중요성 체감

```java
public class AppConfig {
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy();
    }
}
```

---

## 3️⃣ 스프링 컨테이너 도입

### 💡 요약
- @Configuration과 @Bean을 이용한 설정
- 객체 관리를 스프링에게 위임하면서 코드가 더 깔끔해짐

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy(); // Fix → Rate로 변경
    }
}
```

---

## 4️⃣ 다양한 의존관계 주입 방법

### 💡 요약
- 생성자 주입(권장), 필드 주입, setter 주입 비교
- @Autowired 기본 동작 이해
- Lombok(@RequiredArgsConstructor) 활용

```java
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
}
```

---

## 5️⃣ 빈 생명주기 콜백

### 💡 요약
- 객체 초기화 및 종료 시점 제어
- @PostConstruct, @PreDestroy 사용 권장
- InitializingBean, DisposableBean은 옛 방식

```java
@Component
public class NetworkClient {

    @PostConstruct
    public void init() {
        connect();
        System.out.println("초기화 작업 완료");
    }

    @PreDestroy
    public void close() {
        disconnect();
        System.out.println("종료 정리 작업 완료");
    }
}
```

---

## 6️⃣ 설계 개선 & 확장 실습

### 💡 요약
- 고정 할인 정책 → 비율 할인 정책으로 변경
- OCP, DIP 원칙을 지켰기 때문에 설정만 변경하면 됨 (코드 수정 없이 확장 가능)

```java
@Bean
public DiscountPolicy discountPolicy() {
    return new RateDiscountPolicy();
}
```

---

## 🧠 느낀 점

- 단순히 코드를 작성하는 것보다 **설계를 먼저 고민하는 습관**이 중요하다는 걸 느낌
- 객체지향 설계 원칙과 스프링 DI가 자연스럽게 연결되어 있다는 걸 체감함
- 구성과 실행의 책임을 분리하는 것만으로 코드가 훨씬 유연해지고 테스트도 쉬워짐

---
