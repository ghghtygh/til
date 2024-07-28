# Spring 환경에 바로 적용하는 테스트의 모든 것 초격차 패키지 Online

## Ch 1. 테스트의 개념과 중요성 이해하기

**Unit Test(단위 테스트)**
- 가장 작은 단위로 기능을 나누어 테스트
- 하나의 모듈을 기준으로 독립적으로 진행
- 모듈 == 애플리케이션을 구성하는 객체의 기능(메서드)

장점
- 회귀성 보장 -> 신규 기능/리팩토링 불안감 해소
- 코드 자체가 API의 기능을 설명하는 문서로 제공 가능

단점
- 객체 간의 의존성 해결 필요

**Integeration Test(통합 테스트)**
- 서로 다른 모듈 or 객체 간 상호작용의 유효성 검증


**UI Test(인수 테스트)**
- 실제 화면에 대한 테스트

**TDD(Test-Driven-Development) 테스트 주도 개발**
실패하는 테스트 작성 -> 테스트 성공하는 코드 -> 리팩토링과 같은 일련의 스프린트를 여러번 거쳐 점진적으로 코드를 완성

장점
- 디버깅 시간 단축
- 불안정성 개선에 따른 생산성 증대
- 재설계 시간 단축 (테스트 시나리오 작성을 통한 요구사항 이해 및 에외사황 먼저 고려 가능)
- 추가 구현이 용이
- 각 테스트가 요구사항을 식별하므로 코드와 요구사항이 긴밀히 연결됨

단점
- 시간이 너무 많이 소요됨

TDD 흐름
1. 작은 단위 Test Case 요구사항은 만족, 실행은 안되는 코드 작성
2. 정상 동작하면서 Test Case 통과하는 테스트 코드로 수정, 테스트 통과 여부 체크
3. 개선할 코드에 대한 리팩토링 수행, 테스트 통과하는지 확인
4. 위 1~3 과정을 반복하면서 기능을 점진적으로 완성
