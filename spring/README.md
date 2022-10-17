# Spring
#TIL/Spring







## IoC(Inversion of Control)
### IoC란 ?
오브젝트 생성, 관계설정, 사용, 제거 등 오브젝트 전반에 걸친 모든 제어권을 애플리케이션이 아닌 프레임워크의 컨테이너에게 넘기는 것

### IoC가 필요한 이유
객체를 관리해주는 프레임워크와 개발자가 구현해야하는 부분으로 역할과 관심을 분리해 응집도를 높이고 결합도를 낮추며, 이에 따라 변경에 유연한 코드를 작성할 수 있는 구조가 됨 (객체지향 원칙을 잘 지킬 수 있음)

## DI(Dependency Injection)
### DI란 ?
외부로 부터 메모리에 올라가 있는 인스턴스의 레퍼런스를 인터페이스 타입의 파라미터로 의존관계를 설정하는 것

### 의존관계 주입 방법
#### 1. 필드를 이용한 의존관계 주입
```
@Service
public class AServiceImpl implements AService {
	@Autowired
	private ARepository aRepository;
}
```

#### 2. setter() 메서드를 이용한 의존관계 주입
```
@Service
public class AServiceImpl implements AService {

	private ARepository aRepository;
	
	@Autowired
	public void setARepository(ARepository aRepository) {
		this.aRepository = aRepository;	
	}
}
```

#### 3. 생성자를 이용한 의존관계 주입
```
@Service
public class AServiceImpl implements AService {

	private final ARepository aRepository;
	
	@Autowired
	public AServiceImpl(ARepository aRepository) {
		this.aRepository = aRepository;	
	}
}
```


### 생성자 주입을 사용해야하는 이유
	- NullPointerException 방지 -> 의존관계 설정이 되지 않으면 객체 생성 불가
	- 주입받을 필드 final 선언 가능 -> 불변성 보장
	- 순환참조 방지 -> 애플리케이션 구동시 실패
	- 테스트 코드 작성 용이 -> 원하는 구현체를 생성하여 넘겨줄 수 있음

### 용어
#### Bean
Spring 컨테이너에 등록되는 POJO 객체
Spring Application의 Component

#### Bean Factory
Spring IoC를 담당하는 핵심 컨테이너
Bean에 대한 등록/생성/조회/소멸 관리
구현체로 ApplicationContext 사용
getBean() 메서드 제공 -> Bean의 메타정보를 바탕으로 Bean 조회

#### Application Context
BeanFactory 인터페이스의 구현체

#### Configuration Meta Data
Application Context에서 IoC 설정을 위해 사용되는 메타정보
@Configuration 어노테이션을 통해 클래스로 설정 정보 구현 가능


## AOP(Aspect Oriented Programming)
### AOP란 ?
AOP란, 실제 핵심 로직을 수행하며 발생하는 횡단 관심사를 한데 모아 처리하는 것
Aspect Class를 별도로 지정하여 실행 메서드의 조인 포인트를 기반으로 포인트 컷을 설정함으로써 실제 모듈의 핵심 로직에 관여하지 않고도 횡단 로직을 처리할 수 있도록 해줌

### AOP가 필요한 이유
관심사의 분리(Separation of Concerns)
관심이 같은 것 끼리는 하나 또는 친한 객체로 모으고, 관심이 다른 것은 떨어져서 서로 영향을 주지 않도록 분리하는 것

객체지향의 5대 원칙 중 하나인 단일 책임 원칙(SRP) -> 한 클래스는 한가지 책임
하지만, 실제 핵심 로직(Core Concern)을 수행할 때 아무리 설계를 잘하더라도 분리하기 힘든 부분이 발생 -> 횡단 관심사(CrossCutting Concerns) 
ex) 트랜잭션, 로깅, 사용자 인증 등


### 사용 용어
#### Join Point
횡단관심사로 분리된 로직이 끼어들 수 있는 위치나 시점

#### Advice
횡단관심사로 분리된 로직에 대한 코드
Aspect로 분리된 뒤, 실행될 때 위빙되어 구체적인 처리를 하는 로직

#### Point cut
Advice 코드가 적용될 지점을 의미
공통으로 적용할 Join Point를 표현식 등의 기능을 사용하여 하나로 묶어냄

#### Weaving
Advice 코드를 핵심 로직의 Point cut에 적용하는 것을 의미
Advice로 분리된 코드를 핵심 로직에 다시 합치는 것
다음 두가지 방식으로 구분
	- 런타임시, 프레임워크를 통해 Porxy로 생성하여 코드를 합치는 방식
	- 컴파일 또는 클래스 로드시, 바이트 코드를 조작하여 실제 코드를 끼워 넣는 방식

#### Aspect
여러 객체에 공통으로 적용되는 관심 사항
Advice + Point cut

#### Target
핵심 로직을 구현하는 클래스
AOP가 적용되어 Advice를 받을 대상

#### Advisor
Advice 코드와 Point cut을 합친 것
공통 관심사 코드를 뽑아서 하나의 클래스에 담음


### Spring AOP
#### Spring에서 제공하는 AOP
Proxy를 이용하여 런타임 위빙(Runtime Weaving)으로 횡단 로직을 수행
	1. JDK Dynamic Proxy를 이용
	2. CGLIB Proxy를 이용 (CGLIB 라이브러리 추가 필요)

#### Proxy
자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 대신 받는 객체
실제 Target이 담당하는 역할을 대리 받아, Target에 대한 수정 없이 요청 전후에 로직을 추가하여 기능을 확장 시킬 수 있음 -> OCP(Open-Close Principal) 적용됨

#### JDK Dynamic Proxy
JDK 1.3이후 제공, Proxy Factory에 의해 런타임 시 동적으로 만들어지는 오브젝트
인터페이스에 대한 명세를 기준으로 Proxy 생성 (인터페이스 선언 강제)
InvocationHandler라는 인터페이스를 구현하여 invoke()를 통해 Proxy의 위임 기능을 수행
Reflection을 사용하기 때문에 퍼포먼스 하락의 원인이 되기도 함


#### CGLIB Proxy
CGLIB이라는 외부 라이브러리를 통해 사용
Enhancer라는 클래스를 바탕으로 Proxy를 생성하여 인터페이스 없이 Proxy 생성 가능
Target 클래스를 상속받아 생성 -> final, private 등 Override를 지원하지 않으면 Aspect 적용 X






