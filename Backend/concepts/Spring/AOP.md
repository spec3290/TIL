# AOP가 뭘까?
*Aspect Oriented Programming* 즉, 관점 지향 프로그래밍이라고 불리는 개념이다.
관점 지향 프로그래밍이 조금 추상적이지만, 객체 지향 프로그래밍이 객체를 중점으로 둔다면, 관점 지향 프로그래밍은 Aspect를 중점으로 보는 프로그래밍 기법이라고 보면 된다.
![[Pasted image 20260625030011.png]]
처음에는 이해가 안 갈수도 있는데 조금 설명을 보태자면
주황, 파랑, 빨강이 공통 관심사이고 class에 핵심 관심사 기능이 있다고 보면 된다. 즉 aop의 가치는 공통 관심사를 클래스에서 분류해 내어 코드의 복잡성을 해결하고 유지보수에 편리함을 줄 수 있다.
## 공통 관심사
여러 기능에 반복적으로 필요한 부가 기능
## 핵심 관심사
해당 서비스가 실제로 해야 하는 비즈니스 기능
# AOP 구성 요소

1. **Aspect**  
    공통 관심사를 모듈화한 단위이다.  
    하나의 Aspect 안에는 보통 Pointcut과 Advice가 함께 정의된다.  
    예를 들어 실행 시간 측정, 로깅, 트랜잭션 처리 같은 기능을 Aspect로 분리할 수 있다.
	스프링에서 advice 코드 모아서 한 클래스로 만드는데 그게 Aspect
	
2. **Advice**  
    실제로 실행될 공통 기능 코드이다.  
    메서드 실행 전, 실행 후, 예외 발생 후, 메서드 실행 전후 등 어느 시점에 실행될지를 정의한다.  
    예를 들어 `@Before`, `@After`, `@AfterThrowing`, `@Around` 등이 있다.
    
3. **Pointcut**  
	Advice를 어떤 메서드에 적용할지 결정하는 조건식이다.  
    예를 들어 `execution(* com.example.demo.service..*(..))`는 `service` 패키지 아래의 메서드들을 대상으로 Advice를 적용하겠다는 의미이다.
    
4. **Join Point**  
    Advice가 적용될 수 있는 실행 지점이다.  
    Spring AOP에서는 주로 **메서드 실행 시점**을 의미한다.  
    예를 들어 `UserService.signup()` 메서드가 실행되는 순간이 Join Point가 될 수 있다.
    
5. **Target**  
    실제 비즈니스 로직을 가지고 있는 원본 객체이다.  
    예를 들어 `UserService`, `OrderService` 같은 서비스 객체가 Target이 될 수 있다.
    
6. **Proxy**  
    Spring AOP에서 중요한 개념으로, Target 객체 앞에서 호출을 대신 받아주는 대리 객체이다.  
    외부에서 메서드를 호출하면 실제 Target 객체를 바로 호출하는 것이 아니라 Proxy를 먼저 거치고, Proxy가 Advice를 실행한 뒤 Target 메서드를 호출한다.
	쓰는 이유는 비즈니스 로직을 건드리지 않고 공통 로직을 수행하기 위해서라고 한다.
	안쓰면 트랜잭션 처리, 로깅 같은 지저분한거 서비스코드에 다 넣어야됨 -> 복잡하고 읽기 싫어짐

# 예시
```
@Aspect
@Component
class LoggingAspect {

    @Around("execution(* com.example.demo.service..*(..))")//demo.service //pointcut
    // advice
    fun logExecutionTime(joinPoint: ProceedingJoinPoint): Any? {
        val start = System.currentTimeMillis()

        val result = joinPoint.proceed()

        val end = System.currentTimeMillis()
        println("${joinPoint.signature.name} 실행 시간: ${end - start}ms")

        return result
    }
} 
``` 
# 어떨 때 사용해야 좋은가?
반복되는 공통 기능을 분류하여 사용할 때 좋다.
예시로 ```
서비스 메서드 실행 시간 측정
- API 요청/응답 로깅
- 예외 로깅
- 트랜잭션 처리
- 권한 검사
- 감사 로그 저장
- 캐싱 처리
등이 있다.

다만 그렇다고 해서 몇 줄 없는 메서드에다가 아무렇게나 떡칠하면 되려 복잡해질 수도 있으므로 여러 곳에서 사용되는 공통 기능에만 적용하는 게 좋을 것 같다

## 정리
aop는 공통 관심사를 핵심 비즈니스 로직에서 분리하기 위한 프로그래밍 방식이다. oop의 단점을 보완하는 프로그램이라고 보면 된다.