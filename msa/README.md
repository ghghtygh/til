# 마이크로서비스 패턴
### 확장 큐브
X축 확장 : 다중 인스턴스에 요청 분산
Z축 확장 : 조건에 따라 요청 분산
Y축 확장 : 기능에 따른 애플리케이션 분해

SOA
서비스간 통신에 SOAP, WS 표준 등을 사용
서비스 통합 및 메시지 처리 로직에 ESB라는 스마트 파이프 활용
일정 개수 이하의 크고 복잡한 모놀리식 애플리케이션을 통합하는 용도

MSA
서비스간 통신에 메시지 브로커나 REST 또는 gRPC 같은 가벼운 프로토콜 위주 덤 파이프를 사용
규모가 작은 수십~수백개의 서비스로 구성

MSA 장점
크고 복잡한 애플리케이션을 지속적으로 전달 및 배포
서비스 규모가 작기 때문에 관리 용이
서비스 독립적으로 배포 및 확장 가능
결함 격리에 용이
새로운 기술 실험 및 도입이 용이

MSA 단점
모놀리식 아키텍처를 알맞게 분해하기가 어려움
복잡한 분산 시스템으로 개발, 테스트, 배포가 어려움
여러 서비스에 걸친 기능의 배포시 알맞은 조정이 필요
MSA 도입 시점 결정의 어려움

MSA 패턴 구분
- 인프라 패턴 : 개발 영역 밖 인프라 문제 해결
- 애플리케이션 인프라 : 개발에 영향을 미치는 인프라 문제 해결
- 애플리케이션 패턴 : 개발자가 맞닥뜨리는 문제 해결

레이어드 아키텍처
계층마다 명확히 정의된 역할을 분담
계층 간 디펜던시는 아키텍처로 제한
표현(프레젠테이션) 계층 : 사용자 인터페이스 또는 외부 API 구현
비즈니스 로직 계층
영속화(퍼시스턴스) 계층 : DB 상호 작용 로직이 구현된 계층

헥사고날 아키텍처
논리 뷰를 비즈니스 로직 중심으로 구성
인바운드 어댑터 : 표현 계층 대신 비즈니스 로직을 호출하여 외부 요청을 처리
아웃바인드 어댑터 : 영속화 계층 대신 비즈니스 로직에서 들어온 요청을 외부 애플리케이션/서비스를 호출해서 처리
포트 : 비즈니스 로직에 하나 이상 존재하며, 자신이 외부세계와 상호작용하는 방법이 정의(자바 인터페이스)
인바운드 포트 : 비즈니스 로직이 표출된 API (public 메서드 정의된 서비스 인터페이스)
아웃바운드 포트 : 비즈니스 로직이 외부 시스템을 호출하는 방법(리포지터리 인터페이스)

비즈니스 로직의 표현/데이터 접근 로직에 의존하지 않음(어댑터와 분리)
비즈니스 로직의 테스트가 용이
다양한 외부 시스템을 호출하거나 여러 형태의 UI로 구현된 현대 애플리케이션 아키텍처를 좀 더 정확하게 반영 가능

## 분해 전략

1단계 시스템 작업 식별
애플리케이션 요건을 핵심 요청으로 추출

2단계 서비스 식별
어떻게 서비스를 분해할지 결정

3단계 서비스 API 및 협동 정의

## 프로세스 간 통신
### 클라이언트/서비스 상호작용 스타일
#### 일대일/일대다 여부
일대일 : 각 클라이언트 요청을 한 서비스가 처리
일대다 : 각 클라이언트 요청을 여러 서비스가 협동해서 처리

#### 동기/비동기 여부
동기 : 클라이언트 제시간 응답을 기대하고 대기 중 블로킹
비동기 : 클라이언트가 블로킹하지 않음, 응답 즉시 전송 X

#### 일대일 상호작용 종류
요청/응답 : 클라이언트가 서비스에 요청 후 응답을 기다림 -> 서비스가 강하게 결합
비동기 요청/응답 : 클라이언트가 요청한 서비스가 비동기적으로 응답
단방향 알림 : 클라이언트가 서비스에 일방적으로 요청, 서비스 응답 X

#### 일대다 상호작용 종류
발행/구독 : 클라이언트에서 알림 메시지 발행 후, 관심있는 서비스들이 메시지 소비
발행/비동기 응답 : 클라이언트가 요청 메시지 발행 후, 주어진 시간 동안 관련 서비스의 응답을 기다림

### 마이크로 API 정의
클라이언트가 호출 가능한 작업과 서비스가 발행되는 이벤트로 구성
작업 : 이름, 매개변수, 반환타입
이벤트 : 타입, 필드를 가지며 메시지 채널에 발행

### API 발전시키기
MSA에서 무중단 상태로 서비스 API를 변경하기 위해 전략이 필요

#### 시맨틱 버저닝 명세(Semvers, Semantic Versioning specification)
버전 번호를 사용하고 증가시키는 규칙 명시
버전 번호를 MAJOR, MINOR, PATH 파트로 구성
- MAJOR : 하휘 호환되지 않는 변경 API 적용
- MANOR : 하휘 호환되는 변경 API 적용
- PATCH : 하휘 호환되는 오류 수정

#### 하휘 호한되는 소규모 변경
- 옵션 속성을 요청에 추가
- 속성을 응답에 추가
- 새 작업 추가
클라이언트/서비스가 견고성 원칙을 뒷받침하는 요청/응답 포맷을 사용해야 함

#### 중대한 대규모 변경
기존 버전과 호환이 안되는 중요한 변경이 있는 경우, 일괄적으로 강제 업데이트하는 것이 불가하므로 일정 기간동안 서비스는 신구 버전 API 모두 지원해야함
REST API라면 URL에 메이저 버전 번호 삽입
HTTP content negotiation으로 버전 번호를 끼워넣는 방법도 존재
여러 버전의 API를 지원하기 위해서는 API 구현된 서비스 어댑터에 신구버전의 중계 로직이 있어야 함

### 메시지 포맷
데이터 포맷은 IPC 효율, API 사용성, 발전성에 많은 영향을 끼침
gRPC는 메시지 포맷이 정해져 있음
자바 직렬화는 자바에 국한되므로 사용되지 않음

#### 텍스트 메시지 포맷
JSON, XML 등 텍스트 기반 포맷은 사람이 읽을 수 있음
메시지 스키마가 바뀌어도 하위 호환성이 쉽게 보장
JSON에서도 XML 스키마 같은 JSON 스키마 표준이 제정 (메시지 프로퍼티명, 타입, 필수여부)
메시지가 다소 길고, 메시지 이외 속성명이 추가되거나 텍스트를 파싱하는 오버헤드 발생

#### 이진 메시지 포맷
메시지 구조 정의에 필요한 타입 IDL을 제공하며 컴파일러에서 메시지를 직렬화/역직렬화하는 코드를 생성
- 스리프트(Thrift)
데이터 직렬화 라이브러리 + RPC 프레임워크 (원격 호출)
단일 바이너리 인코딩을 표준으로 하는 프로토콜 버퍼와 아브로와 다르게 다양한 직렬화 포맷을 포함
- 프로토콜 버퍼
직렬화/역직렬화 속도가 빠르고, 직렬화 파일 크기를 월등히 줄임 -> 대용량 데이터 처리 성능 좋음
스키마를 변경할 때 필드이름은 바꿀 수 있지만(필드이름이 바이너리에 없어서) 태그 번호는 바꿀 수 없음
- 아브로
아브로의 컨슈머는 스키마를 알고 있어야 해석이 가능함
데이터의 쓸때와 읽을 때 정확히 같은 버전이 필요함
필드에 대한 지시자가 없어서 스키마에 보이는 순서로 차례대로 인코딩

### 동기 RPI 패턴 응용 통신
RPI : 클라이언트가 서비스에 요청을 보냈을 때 서비스가 처리 후 응답을 회신하는 IPC
1. 클라이언트의 비즈니스 로직이 프록시 인스턴스 호출
2. 프록시 인터페이스는 RPI 프록시 어댑터 클래스로 구현
3. RPI 프록시가 서비스에 전달한 요청은 RPI 서버 어댑터 클래스가 접수
4. 비즈니스 로직을 마친 서비스는 RPI 프록시로 응답을 반환
5. 최종 결과 클라이언트 비즈니스 로직에 반환

프록시 인터페이스는 하부(underlying) 통신 프로토콜을 캡슐화

### 동기 RPI 패턴 : REST
HTTP로 소통하는 IPC
HTTP 동사를 사용해서 URL로 참조되는 리소스를 조작
REST API
스웨거라는 오픈소스 프로젝트를 발전시켜 Open API Specification를 REST IDL로 보급
스웨거
REST API를 문서화하는 도구
인터페이스 정의를 기반으로 클라이언트 스텁이나 서버 스켈레톤을 생성하는 툴 포함

요청 한번으로 많은 리소스를 가져오기 어려움
특정 주문과 소비자를 같이 가져오고 싶을 때 두번의 요청을 하게되는데
왕복 횟수가 많아지면서 지연이 늘어남
리소스 획득시 연관된 리소스를 조회하도록 할 수 있으나 효율이 떨어지며 GraphQL이나 넷플릭스 팔코 등 대체 API 기술 사용

GraphQL
클라이언트에서 자신에게 필요한 데이터만을 쿼리할 수 있도록 해줌
필요한 데이터에 대한 쿼리를 선언해 넘기면 쿼리를 해석해서 해당 데이터를 알맞은 형태로 반환 
네트워크 IO를 줄일 수 있지만 서버의 부담이 가중
클라이언트가 결정한 데이터가 잘못됬는지 알기 어려워 요청을 필터링하기 어려움
고정된 요청의 응답만 필요할 때는 쿼리로 인해 request 크기가 커짐
캐싱이 복잡함

Falcor
효율적으로 데이터를 가져오기 위한 자바스크립트 라이브러리
GraphQL과 용도가 비슷함
JSON Graph의 레퍼런스 타입을 이용할 수 있어 중복된 데이터를 조회하는 것과 그에 따른 오버페치를 제거
falcor model을 통해서 HttpDataSource를 호출하면 GET 방식으로 데이터를 호출하기 때문에 중복호출시 브라우저 캐싱을 이용할 수 있음

REST 장점
단순하고 익숙함
포스트맨이나 curl 등의 도구로 API를 간편하게 테스트 가능
중간 브로커가 필요하지 않음

REST 단점
요청/응답 통신만 지원
교환 일어나는 동안 양쪽이 모두 실행 중이어야 함 
서비스의 인스턴스를 클라이언트가 알고 있어야 함
요청 한번으로 여러 리소스를 가져오기 어려움
다중 업데이트 작업을 HTTP 동사에 매핑하기가 어려움

#### 동기 RPI 패턴 : gRPC
다양한 언어로 클라이언트/서버를 작성할 수 있는 프로토콜
바이너리 기반 프로토콜로 프로토콜 버퍼 기반의 IDL로 정의
하나 이상의 서비스와 요청/응답 메시지 데피니션으로 구성
서비스 데피니션은 자바 인터페이스와 비슷하게 정적 타입 메서드를 모아놓은것

프로토콜 버퍼
각 필드마다 번호가 매겨지고 타입 코드가 할당 
필요한 필드만 추출할 수 있기 때문에 하위 호환성 유지하며 API 발전 가능

gRPC는 REST를 대체할 방안이지만 REST처럼 동기 통신하는 매커니즘으로 부분 실패 문제가 존재

#### 회로 차단기 패턴
연속 실패 횟수가 임계치를 초과하면 일정 시간동안 호출을 거부하는 RPI 프록시

부분실패시 정해진 기본값이나 캐시된 응답 등의 대체 값 반환 방법도 가능

### 서비스 디스커버리
REST API 서비스를 호출하는 코드에서 서비스 인스턴스의 네트워크 위치(IP 주소 + 포트)를 알아야 하는데 이를 담당
어플리케이션 서비스 인스턴스의 네트워크 위치를 데이터베이스화한 서비스 레지스트리를 가지고 있어서 요청을 라우팅 가능

자가등록 패턴 : 서비스 인스턴스가 자신의 네트워크 위치를 서비스 레지스트리 등록 API를 호출해서 등록
클라이언트 디스커버리 패턴 : 서비스를 호출할 때 서비스 레지스트리의 서비스 인스턴스 목록을 요청해서 넘겨받음

### 비동기 메시징 패턴 응용 통신

#### 메시지 
헤더와 본문으로 구성
헤더 :  송신 데이터의 메타데이터 키/값, 메시지 아이디, 메시지 채널 반환 주소 등
본문 : 실제 송신한 텍스트 또는 바이너리 데이터

#### 메시지 종류
- 문서 : 제네릭
- 커맨드 : RPC 요청과 동등한 메시지
- 이벤트 : 송신자에게 사건이 발생했음을 알리는 메시지

#### 메시지 채널
메시지를 전달하는 통로
송신자 - 송신 포트 인터페이스
수신자 - 메시지 처리를 위한 메시지 핸들러 어댑터 클래스 호출 -> 수신 포트 인터페이스

- 점대점(point-to-point) 채널
채널을 읽는 컨슈머 중 하나만 지정하여 메시지 전달

- 발행-구독(publish-subscribe) 채널
같은 채널을 바라보는 모든 컨슈머에 메시지 전달

### 메시징 상호 작용 스타일 구현

#### 요청/응답 및 비동기 요청/응답
클라이언트가 요청을 보내면 서비스는 응답 반환
요청/응답 : 서비스가 즉시 응답할 것으로 기대
비동기 요청/응답 : 서비스 응답 기대 X

#### 단방향 알림(one-way notification)
클라이언트가 서비스가 소유한 점대점 채널로 메시지를 보내면, 서비스가 이 채널을 구독해서 메시지 처리, 응답 반환 X

#### 발행/구독
메시징은 발행/구독 스타일 상호작용을 기본 지원
클라이언트는 여러 컨슈머가 읽는 채널에 메시지 발행
서비스는 도메인 이벤트(도메인 객체의 변경) 발행
서비스는 자신이 관심 있는 도메인 객체의 이벤트 채널을 구독

#### 발행/비동기 응답
발행/구독 + 요청/응답
클라이언트는 응답 채널 헤더가 명시된 메시지를 발행/구독 채널에 발행
컨슈머는 CorrelationId+응답 메시지를 지정된 응답 채널에 작성
클라이언트는 CorrelationId로 응답을 취합하여 메시지와 요청을 맞춤

### 메시지 브로커
서비스 통신을 위한 인프라 서비스

#### 브로커리스 메시징
브로커리스 아키텍처 : 서비스가 서로 직접 통신
ZeroMQ : 브로커리스 메시징 기술 명세 및 라이브러리

브로커리스 메시징 장점
송신자에서 수신자로 직접 전달 -> 네트워크 트래픽 가볍고 지연시간 짧음
메시지 브로커가 병목점이나 SPOF(단일장애점) X
메시지 브로커의 설정 및 관리 필요성 X

브로커리스 메시징 단점
서로의 위치를 알기 위한 서비스 디스커버리 사용 필수
메시지 교환시 송신자/수신자 모두 실행 필요 -> 가용성 떨어짐
전달 보장 매커니즘 구현 어려움

#### 브로커 기반 메시징
메시지 브로커 : 모든 메시지가 지나가는 중간 지점
(송신자 -> 메시지 브로커 -> 수신자)
컨슈머의 네트워크 위치를 알지 않아도 되며 메시지 버퍼링 가능

브로커 기반 메시징 장점
느슨한 결합 : 서비스 인스턴스 위치 알려주는 디스커버리 매커니즘 필요 X
메시지 버퍼링 : 메시지 브로커는 처리 가능한 시점까지 메시지 버퍼링
유연한 통신 : 여러 상호 작용 스타일 지원
명시적 IPC : 원격 서비스가 자신의 로컬 서비스인 것 처럼 호출 시도

브로커 기반 메시징 단점 
성능 병목 가능성 : 메시지 브로커가 성능 병목점이 될 가능성이 있음
단일 장애점 가능성 : 시스템의 신뢰성을 위한 고가용성 보장 필요
운영 복잡도 부가 : 설치/구성/운영 필요한 시스템 컴포넌트 추가되는 것

### 수신자 경합과 메시지 순서 유지
메시지 수신자를 스케일 아웃 하면서 메시지 순서를 유지하려면 ? 
메시지 브로커는 샤딩된 채널 이용하며 다음 부분으로 구성
1. 샤딩된 채널은 복수의 샤드로 구성되며, 각 샤드는 채널처럼 작동
2. 메시지 송신시 헤더에 지정된 샤드 키로 각 샤드/파티션에 배정
3. 여러 수신 인스턴스를 묶어 동일한 논리 수신자(컨슈머 그룹)로 취급
주문 이벤트에서 주문별로 동일한 샤드에 발행 + 한 컨슈머 인스턴스만 메시지를 읽어 처리 순서를 보장

### 중복 메시지 처리
멱등한 메시지 핸들러 작성 + 메시지 추적 및 중복 방지
1. 순서를 유지한다는 전제하에 동일한 입력 값을 반복 호출해도 부수 효과가 없는 메시지 처리 로직 작성
2. 컨슈머가 처리한 메시지 ID를 DB 테이블에 저장하여 메시지 처리여부 추적 및 중복 방지

### 트랜잭셔널 메시징
DB 업데이트와 메시지 전송을 동일한 트랜잭션으로 수행하는 방법 필요

#### DB 테이블을 메시지 큐로 활용
트랜잭셔널 아웃박스 패턴 : 이벤트나 메시지를 DB 저장 -> DB 트랜잭션의 일부로 발행
아웃박스 테이블 : DB 테이블을 임시 메시지 큐로 사용
메시지 릴레이 : 아웃박스 테이블을 읽어 메시지 브로커에 메시지를 발행

DB에 저장된 메시지를 메시지 브로커로 옮기는 방법 필요

#### 폴링 발행기 패턴 
메시지 릴레이로 DB의 아웃박스를 폴링하여 메시지 발행
DB 미발행 메시지 조회 -> 메시지 브로커 발행 -> 테이블 메시지 삭제

규모가 작은경우 단순하게 사용
폴링 주기에 따른 추가적인 비용 발생
NoSQL 쿼리 성능에 따른 사용 여부 결정

#### 트랜잭션 로그 테일링 패턴
메시지 릴레이로 DB 트랜잭션 로그를 테일링하여 변경분을 발행
커밋된 업데이트는 각 DB 트랜잭션 로그 항목으로 남음
트랜잭션 로그마이너로 트랜잭션 로그를 읽어 변경분을 메시지 브로커로 발행
디비지움, 링크드인 데이터버스, 다이나모디비 스트림즈, 이벤추에이트 트램

### 메시징 라이브러리/프레임워크 - 이벤추에이트 트램
저수준 메시징 API 추상화
고수준 상호 작용 스타일 필요

#### 기초 메시징
MessageProducer 인터페이스 : 프로듀서 서비스 메시지를 채널에 발행시 사용
MessageConsumer 인터페이스 : 컨슈머 서비스 메시지 구독시 사용

#### 도메인 이벤트 발행
DomainEventPublisher 인터페이스 : 비즈니스 객체 생성/수정/삭제시 발생시킨 도메인 이벤트 발행/구독 API 제공

#### 커맨드/응답 메시지
CommandProducer 인터페이스 : 클라이언트가 커맨드 메시지를 서비스에 보낼 때 사용
CommandDispatcher 클래스 : 서비스가 커맨드 메시지를 소비할 때 사용

## 트랜잭션 관리
MSA 환경에서는 서비스마다 별도의 DB를 사용하기 때문에 여러 DB에 걸친 데이터 일관성 유지 수단 필요
이전에는 분산 트랜잭션을 이용하여 2PC를 이용한 전체 트랜잭션의 원자성을 보장
NoSQL 또는 메시지 브로커(RabbitMQ, 카프카)는 분산 트랜잭션을 지원하지 않음
동기 IPC 형태기 때문에 서비스가 모두 가동 중이어야 함

### 사가 패턴
비동기 메시징을 이용한 로컬 트랜잭션 -> 격리성이 없음, 보상 트랜잭션을 걸어 롤백
ACID 트랜잭션은 DB 롤백을 이용한 변경 내용 복구가 쉬움
사가는 단계마다 로컬 DB의 변경분을 커밋하므로 자동 롤백은 불가 -> 명시적으로 언두 필요 (보상 트랜잭션 미리 작성)

#### 코레오그래피(choreography) 사가
의사 결정과 순서를 참여자에게 맡김
이벤트 교환 방식으로 통신
사가 참여자가 서로 이벤트를 구독하여 그에 따른 반응

장점
단순함 : 비즈니스 객체를 생성, 수정, 삭제할 때 서비스가 이벤트를 발행
느슨한 결합 : 이벤트만 구독하여 서로를 알지 못함

단점
서비스 구현 로직이 흩어져 이해하기 어려움
서로 이벤트를 구독하기 때문에 순환 의존성의 발생이 쉬움(잠재적인 설계 취약점)
자신에게 영향을 미치는 이벤트를 모두 구독해야 하므로 결합도가 높아질 위험성

#### 오케스트레이션(orchestration) 사가
사가 편성 로직을 사가 오케스트레이터에 중앙화
사가 참여자에게 커맨드 메시지를 보내 수행할 작업을 지시
사가 참여자가 로컬 트랜잭션을 완료하는 시점에 로컬 트랜잭션의 상태와 결과에 따라 상태 전이를 어떻게할지, 어떤 액션을 취할지 결정

장점
오케스트레이터만 참여자를 호출하므로 순환 의존성이 발생하지 않음
각 서비스는 오케스트레이터가 호출하는 API만을 구현하기 때문에 결합도가 낮음
사가 편성 로직이 오케스트레이터에만 있으므로 도메인 객체가 단순해짐

단점
오케스트레이터에 중앙화가 과해지면 깡통 서비스를 호출하게 될 수 있음
오케스트레이터가 순서화만 담당하도록 설계 필요

### 비격리 문제 처리
1. 소실된 업데이트
한 사가의 변경을 다른 사가가 덮어씀
2. 더티 읽기
업데이트 중인 데이터를 다른 사가가 읽음
3. 반복 불가능한 읽기

사가 구조
- 보상가능 트랜잭션 - 보상 트랜잭션으로 롤백 가능
- 피봇 트랜잭션 - 사가의 진행/중단 시점
- 재시도 가능 트랜잭션 - 반드시 성공

#### 시맨틱락
보상 가능 트랜잭션이 생성/수정하는 레코드에 플래그를 셋팅

교환적 업데이트
업데이트를 어떤 순서로도 실행가능하도록 설계

비관적 관점
사가 단계의 순서를 재조정하여 더티 읽기의 문제를 최소화

값 다시 읽기
업데이트 전에 다시 읽어 변경되었는지를 확인

버전 파일
레코드 수행 작업을 하나씩 기록

값에 의한
요청을 보고 사가를 쓸지 분산 트랜잭션을 쓸지 판단

## 비즈니스 로직 설계
### 트랜잭션 스크립트 패턴(transaction script pattern)
비즈니스 로직을 요청 타입별 매핑된 절차적 트랜잭션 스크립트 뭉치로 구성
단순한 비즈니스 로직에 적합
#### 장점
단순하게 작성할 수 있음

#### 단점
상호 연관된 클래스가 많아져 복잡한 비즈니스 로직 개발이 까다로움
특정 트랜잭션 관리 제약 조건하에서도 작동되는 비즈니스 로직 설계가 힘듬


### 도메인 모델 패턴
비즈니스 로직을 상태와 동작을 가진 클래스인 객체 모델로 구성
비즈니스 로직을 영속화 도메인 객체에 위임 - 단순한 서비스 메서드

#### 장점
설계를 이해하거나 관리하기 쉬움 -> 소수의 책임만 맡은 작은 여러 클래스로 구성되기 때문
테스트하기 쉬움 -> 각 클래스가 독립적으로 테스트 가능

객체를 참조하지 않고 PK를 이용하여 애그리거트가 서로 참조
한 트랜잭션으로 한 애그리거트만 생성 또는 수정 가능
ACID 트랜잭션이 하나의 서비스 내부에만 걸리게 됨

#### 단점
하나의 도메인 모델 구축하는데 많은 노력 필요


## 비즈니스 로직 개발: 이벤트 소싱
상태 변화를 나타내는 여러 이벤트로 애그리거트를 저장하는 것
애플리케이션은 이벤트를 재연하여 애그리거트의 현재 상태를 재생성
이벤트를 통해 도메인 객체의 이력 로그를 정확히 남길 수 있고 도메인 이벤트를 확실하게 발행할 수 있음

데이터를 직접 업데이트하지 않아도 되기 때문에 동시 업데이트로 인한 충돌을 방지

이벤트 소싱에는 긍정적인 요소와 부정적인 요소가 있다. 긍정적인 점에는 과거 기록을 관리할 수 있는 반면, 부정적인 점에는 데이터베이스의 데이터가 기하급수적으로 증가할 수 있다는 점이다. 이러한 이유로 이벤트 소싱이 CQRS와 함께 사용되는 것이 일반적이다. 정확히는 동일한 레코드의 수가 많기 때문에 데이터베이스에서 검색을 방해하지 않기 위해 독립적인 Query를 수행할 수 있는 환경을 제공하는 것이다.


JPA, MyBatis 애플리케이션에서 저장할 때 문제
- 객체-관계 임피던스 부정합
-> 테이블 형태의 관계형 스키마와 복잡한 도메인 모델의 그래프 구조가 맞지 않는 경우가 많음
- 애그리거트 이력 X
- 감사 로깅 구현 번거롭고 에러가 발생
- 이벤트 발행 로직이 비즈니스 로직에 추가

#### 장점 
애그리거트 이력 보존 -> 감사/통제 용도
도메인 이벤트를 확실하게 발행

#### 단점
이벤트 저장소를 커리하기 쉽지 않아 CQRS 패턴 적용 필요

스냅샷
데이터 중간에 스냅샷으로 저장하여 빠르게 데이터를 찾아갈 수 있음

사가 오케스트레이터
코레오그래피 사가
데이터 삭제하기 까다로운 이유

## 마이크로서비스 쿼리 구현
API 조합 패턴
- 오버헤드 증가 : 여러 서비스 호출 및 DB 쿼리로 인한 오버헤드
- 가용성 저하 우려 : 여러 서비스가 개입되어야 하기 떄문
- 데이터 일관성 결여 : 모놀리식에 비해 여러 DB를 사용하기 떄문

CQRS
여러 API를 조합하여 데이터를 조회하려면 비효율적인 인메모리 조인이 필요
이벤트 소싱 애플리케이션의 쿼리가 가능
다양한 쿼리 효율적으로 구현 가능
-> 각 쿼리가 효율적으로 구현된 하나 이상의 뷰가 정의되어 단일 데이터 저장소의 한계 극복

한 서비스가 다른 서비스에서 소유한 데이터를 반환하는 쿼리의 구현도 가능하므로 관심사 분리 관점에서 유리
CQRS 뷰의 최종 일관성 처리가 필요

기존의 RDB에서의 동시성은 같은 시간에 조회하는 데이터는
항상 동일한 데이터임을 보증
NoSQL 또는 분산 노드에 의해 빠른 데이터 처리가 주목적인 경우 동시성을 보장하긴 힘들지만 데이터 변경이 발생했을 때 시간이 지남에 따라 여러 노드에 전파되면서 당장은 아니지만 최종적(결과적)으로는 일관성을 유지하는 것



## 외부 API 패턴
### 게이트웨이
한 네트워크에서 다른 네트워크로 이동하기 위해 거치는 지점
전송 계층에 해당하며 서로 다른 프로토콜끼리도 네트워크 통신이 가능하도록 연결해주는 기기

### API 게이트웨이
모든 서버로의 요청을 단일 지점을 거쳐서 처리하도록 하는 것
로드밸런싱, 클라이언트별 엔드포인트를 제공해줌
메시지 또는 헤더 기반 필터 및 라우팅
인증 및 권한 부여
서비스 검색 통합
응답 캐싱
부하 분산
로깅, 추적
트래픽 관리 가능

- API 게이트웨이 장점
애플리케이션 내부 구조를 캡슐화할 수 있고, API 최적화를 통해 왕복 횟수를 줄일 수 있음 -> 클라이언트 코드 단순화

- API 게이트웨이 단점
고가용 컴포넌트가 하나 늘어남, API 게이트웨이 자체가 병목지점이 될 수 있음

### BFF(Backend for Frontend) 패턴
각 클라이언트 마다 API 게이트웨이를 따로 구현

API 게이트웨이와 같은 진입점을 하나로 두지 않고 프론트엔드의 유형에 따라 데이터를 가공하거나 통합하는 역할을 하는 중간 서버를 별도로 두는 것

실제 비즈니스 로직의 구현과 응답 데이터를 클라이언트에서 요구되는 데이터로 파싱하는 
두가지 관점으로 분리하여 복잡도를 낮추고 필요한 작업에 집중하기가 쉬워짐

복잡한 구현 로직을 감추고 추상화된 인터페이스 사용
프론트엔드의 관심사를 분리하여 본연의 비즈니스 로직에 집중할 수 있음
불필요한 데이터를 숨길 수 있음

한 view를 완성하기 위해 여러 도메인의 API 응답 값이 필요한 경우
데이터 전송시 불필요한 데이터를 숨길 경우
프론트엔드에서 많은 양의 연산을 요구하는 작업이 진행되어야 하는 경우
OpenAPI 연동시 제공하는 API 조합으로 특정 기능을 완성해야 하는 경우


### API 게이트웨이 설계
- 성능과 확장성
API 게이트웨이 요청 처리 로직의 성격에 따라 동기 I/O 사용할지 비동기 I/O 사용할지 결정 ( I/O 집약적 로직인지, CPU 집약적 로직인지)

- 리액티브 프로그래밍 추상체
서비스를 순차적으로 호출하게 되는 경우 각 서비스의 응답 시간을 합한 시간만큼 대기하게 됨
서비스를 동시 호출하여 응답시간을 줄일 수 있지만 기존 비동기 콜백 방식을 이용하면 병렬/순차 요청이 혼용될 때 관리가 힘듬
-> 리액티브하게 선언형 스타일로 작성
ex) 자바 8 CompletableFutures, RxJava의 observable

- 부분 실패 처리
한정된 리소스를 이용해 안정적으로 운영하기 위해 실패하거나 길게 지연되는 요청의 처리가 필요 -> 회로 차단기 패턴

- 아키텍처의 선량한 시민 되기
서비스 디스커버리 패턴을 이용하면 API 게이트웨이 같은 서비스 클라이언트가 호출할 서비스 인스턴스의 네트워키 위치를 파악 가능
observability pattern을 이용하여 개발자가 애플리케이션 동작 상태를 모니터링하고 문제를 진단


