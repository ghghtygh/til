# 단위 테스트

### 단위 테스트의 목표
- 소프트웨어 프로젝트의 지속 가능한 성장을 가능하게 하는 것
테스트가 없는 코드는 개발을 빠르게 시작할 수는 있지만 프로젝트가 진행됨에 따라 개발속도가 현저히 느려짐
(= 소프트웨어 엔트로피)

- 테스트 코드도 애플리케이션의 정확성을 보장하는 코드베이스의 일부
가능한 적은 코드(최소 유지비)로 문제를 해결해야 하며, 리팩토링 또한 필요

- 좋은 테스트와 좋지 않은 테스트의 구분 필요
대부분의 애플리케이션의 가장 중요한 부분은 비즈니스 로직(도메인 모델)이며, 초점이 해당 부분에 머물러 있어야 함


### 커버리지 지표
테스트 스위트가 소스 코드를 얼마나 실행하는지를 백분율로 나타낸 것

#### 코드 커버리지(code coverage, test coverage)

테스트 스위트가 실행한 코드 라인 수와 제품 코드베이스의 전체 라인 수의 비율
 
#### 분기 커버리지(branch coverage)

원시 코드 라인 수 보다는 if, switch 문과 같은 제어 구조에 중점을 둠 

테스트 스위트가 수행하는 코드 분기 수와 제품 인드베이스의 전체 분기 수의 비율

#### 커버리지 지표의 문제
- 테스트 대상 시스템의 모든 가능한 결과의 검증이 보장되지 않음
- 외부 라이브러리의 코드 경로를 고려할 수 없음

### 단위 테스트란
작은 단위의 코드 조각을 검증하고, 빠르게 수행되며, 격리된 방식으로 처리하는 자동화된 테스트

여기서 격리된 방식에 대한 견해가 갈림

1. 런던파
하나의 클래스가 다른 클래스에 의존할 때 모든 의존성을 테스트 대역으로 대체
- 한 번에 한 클래스만을 확인 -> 서로 연결된 클래스 그래프가 커져도 테스트하기 쉬움
- 테스트 실패시 어떤 기능이 실패했는지 파악 용이 

2. 고전파
단위가 아닌 단위 테스트 자체를 서로 격리된 상태로 실행(공유 의존성만을 테스트 대역으로 대체)
- 코드 단위의 테스트인 런던파와 달리 동작 단위의 단위 테스트 가능
