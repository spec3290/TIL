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
# 구성 요소

1. Aspect
	공통 관심사를 정의하는 모듈
2. Advice
	실제로 실행될 공통 기능 코드
3. Pointcut
	어떤 메서드가 어떤 advice를 받을지 결정하는 표현식
4. Join Point
	advice가 적용될 수 있는 위치
	메서드 실행 시점이든 예외 발생 시점이든..
5. target
	실제 비즈니스 로직을 가진 객체
	AOP는 target객체 사이에 advice를 끼워넣음

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
aop는 공통 관심사를 핵심 비즈니스 로직에서 분리하기 위한 프로그래밍 방식이다. 주로 프록시 기반으로 동작하며, @Transactional, @async도 aop 관점에서 이해하면 원리를 더 잘 이해할 수 있다.