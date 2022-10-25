# Spring
#TIL/Spring


## IoC(Inversion of Control)
### IoC란 ?
오브젝트 생성, 관계설정, 사용, 제거 등 오브젝트 전반에 걸친 모든 제어권을 애플리케이션이 아닌 프레임워크의 컨테이너에게 넘기는 것

### IoC가 필요한 이유
객체를 관리해주는 프레임워크와 개발자가 구현해야하는 부분으로 역할과 관심을 분리해 응집도를 높이고 결합도를 낮추며, 이에 따라 변경에 유연한 코드를 작성할 수 있는 구조가 됨 (객체지향 원칙을 잘 지킬 수 있음)

객체의 생명주기 전체에 대한 권한 및 관리를 프레임워크의 컨테이너에게 주어 개발자가 비즈니스 로직만 작성


## DI(Dependency Injection)
### DI란 ?
메모리에 올라가 있는 인스턴스의 레퍼런스를 인터페이스 타입의 파라미터로 외부로부터 의존관계를 설정하는 것
구체적인 의존 오브젝트와 그것을 사용할 주체, 클라이언트를 런타임 시에 연결해주는 작업


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

## Test Double(테스트 대역)
테스트 대상의 의존 객체를 대신해줄 수 있는 객체

#### Dummy
객체가 전달되지만 기능은 필요없는 경우 사용 (정상적인 동작을 보장하지 않음)

#### Fake
복잡한 로직이나 외부 객체의 동작을 단순화해서 구현 (동작하지만 정교하지 않음)

#### Stub
Dummy 객체가 실제 동작하는 것처럼 보이게 만듬
호출된 요청에 대해 미리 준비해둔 결과 제공 -> 테스트를 위해 의도된 결과만 반환

#### Spy
Stub의 역할을 가지면서 호출되었을 때의 정보를 기록
-> 특정 메서드가 제대로 호출되었는지 알기 위해 사용

#### Mock
호출에 대한 기대값을 작성한 뒤, 작성한 내용에 따라 동작하도록 프로그래밍된 객체


## 스프링이란
### 스프링의 특징
> 자바 엔터프라이즈 개발을 편하게 해주는 오픈소스 경량급 애플리케이션 프레임워크  

#### 애플리케이션 프레임워크
특정 계층이나 기술, 업무 분야에 국한되지 않고 애플리케이션 전 영역을 포괄하는 범용적인 프레임워크

#### 경량급(lightweight)
스프링 자체가 가볍다는 뜻이 아닌, 불필요하게 무겁지 않다는 의미
단순한 개발 툴과 기본적인 개발환경으로도 엔터프라이즈 개발을 할 수 있음

### POJO(Plain Old Java Object)
> 객체지향적인 원리에 충실하면서, 환경과 기술에 종속되지 않고 필요에 따라 재활용될 수 있는 방식으로 설계된 오브젝트  

기존 프레임워크(EJB, 스트럿츠)와 달리 자바 언어와 꼭 필요한 API 외에 종속되지 않음
또한, JNDI 등과 같은 특정 환경에 종속되지 않음 

#### POJO의 장점
자동화 및 유연한 방식과 레벨에서 빠르고 명확하게 테스트 가능

#### POJO 개발을 위한 세가지 기술 
##### 1. IoC/DI
스프링의 가장 기본이 되는 기술, 핵심 개발 원칙
DI 방식을 사용하는 이유 -> 유연한 확장이 가능 (OCP), 재사용이 가능
- 핵심기능의 변경
ex) DB 접근 기술에 따라 의존하는 DAO 구현체의 변경 (iBatis -> JPA -> ??)
- 핵심기능의 동적 변경
ex) DataSource를 선택적으로 변경 or 사용자별 독립적인 의존 오브젝트 유지
- 부가기능의 추가
트랜잭션, 인증, 로깅 등 비즈니스 로직 변경 없이도 외부에서 주입하는 DI 적용을 통한 부가기능 추가
- 인터페이스 변경
- 프록시
- 템플릿/콜백
- 싱글톤과 오브젝트 스코프
싱글톤 스코프 빈 관리 뿐만 아니라 임의의 생명주기를 갖는 오브젝트 제어
- 테스트

##### 2. AOP
- 다이나믹 프록시, CGLib
프록시 AOP 방식
데코레이터 패턴의 응용
조인포인트가 메서드 호출 시점으로 한정

- AspectJ
프록시 방식의 AOP에서는 불가능한 다양한 조인포인트 제공
메서드 호출, 인스턴스 생성, 필드 액세스, 특정 호출 경로의 메서드 호출 등에 부가기능 제공
-> 별도의 AOP 컴파일러를 이용한 빌드 또는 클래스 로딩시 바이트 코드를 조작하는 위빙
@Configurable 등 스프링 내부 구현에서도 반드시 필요

##### 3. 포터블 서비스 추상화(PSA)

JTA 같은 트랜잭션 서비스 추상화 제공
OXM/JavaMail 이용시 스프링이 정의한 추상 API를 이용한 코드 작성

JTA(Java Transaction APIs)
플랫폼마다 상이한 트랜잭션 매니저와 어플리케이션이 상호작용할 수 있는 인터페이스 정의

XA(eXtended Architecture)
동일한 전역 트랜잭션 내에서 몇 개의 백엔드 데이터 저장소에 접근하기 위한 X/Open 그룹 표준 중 하나 (2 Phase Commit, 수행중인 작업 통보, 지연 중인 트랜잭션 회복)

Atomikos : JTA와 XA 제공, JTA 구현체 오픈소스 제공
IBM : JTA 구현체를 어플리케이션의 한 부분으로 제공

OXM(Object Xml Mapping)
- JAXB : Java Object를 XML로 직렬화 <-> 역직렬화, 어노테이션 제공





### 애플리케이션 아키텍처
아키텍처 : 내부 구성요소들이 어떤 책임을 갖고 어떤 방식으로 서로 관계를 맺고 동작하는지에 대한 규정

#### 계층형 아키텍처(layered architecture)
책임과 성격이 다른 것을 크게 그룹으로 만들어 분리해두는 것
각 계층의 응집도가 높으면서 다른 계층과는 낮은 결합도가 유지되어야 함



#### 3-tier or 3-layer 아키텍처
데이터 액세스(DataAccess) 계층
서비스 계층
프레젠테이션 계층


RIA (Rich Internet Application)
SOFEA(Service Oriented Front End Architecture) 아키텍처
프레젠테이션 계층의 코드가 서버에서 클라이언트로 다운로드되어 클라이언트 장치에서 동작하며 서비스 계층 또는 부분 프레젠테이션 계층과 통신하는 구조

DDD
헥사고날 아키텍처

AsjectJ 정책/표준 강제화(policy/standards enfourcement)
도메인 주도 개발(?)에서 뷰 쪽에서 도메인의 비즈니스 로직을 호출하지 못하도록 강력한 개발 컨벤션 적용, 강제화


JNDI(Java Naming and Directory Interface)
디렉터리 서비스에서 제공하는 데이터 및 객체를 발견(discover)하고 참고(lookup) 하기위한 API
용도
자바 애플리케이션을 외부 디렉터리 서비스에 연결(DB, LDAP 서버)
호스팅 웹 컨테이너가 제공하는 구성정보를 참고

JCA(Java Cryptography Architecture)
프로바이더 구조를 사용하면서 보안과 관련된 API 제공
인증서, 전자서명, 유효성 검사 등 제공

JAX-WS(Java API for XML Web Services)
웹 서비스를 생성하는 자바 API

JMS(Java Message Service)
Java EE 애플리케이션에서 메시지를 작성, 전송, 수신, 조회할 수 있게 해주는 API

JSF(JavaServer Faces)
서블릿 API를 기반으로 하는 서블렛 프로그램으로 웹 컨테이너에서 동작
UI 개발을 쉽게 할 수 있도록 해주는 자바 기반의 웹 애플리케이션 프레임워크

하이버네이트(Hibernate)
JPA의 구현체
내부적으로 JDBC API 사용

Spring Data JPA는 Repository 인터페이스 (JPA를 한단계 더 추상화)
JPA(하이버네이트)는 DB쪽에 가까움






























