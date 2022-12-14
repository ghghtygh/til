# MySQL
#TIL/MySQL

## MySQL 엔진 아키텍처
MySQL 서버 = MySQL 엔진 + 스토리지 엔진

### MySQL 엔진
클라이언트 접속 및 쿼리 요청 처리하는 커넥션 핸들러
SQL 파서 및 전처리기
쿼리의 최적화된 실행을 위한 옵티마이저

### 스토리지 엔진
실제 데이터를 디스크 스토리지에 저장하거나 데이터를 읽어오는 부분 전담
MySQL 엔진은 하나를 사용하고 스토리지 엔진은 여러 개를 동시에 사용 가능
MyISAM 스토리지 엔진 (키 캐시)
InnoDB 스토리지 엔진 (InnoDB 버퍼 풀)

### 핸들러(Handler) API
핸들러 요청에 사용되는 API

핸들러(Handler) 요청 : MySQL 엔진의 쿼리 실행기에서 데이터를 쓰거나 읽어야할 때 스토리지 엔진에 보내는 요청

``` 
# 핸들러 API 조회
mysql> SHOW GLABAL STATUS LIKE ‘Handler%’; 
```


## MySQL 스레딩 구조

![](README/C414F661-7644-46D5-B044-0AB023FA29AB.png)

### 포그라운드(Foreground) 스레드
최소 MySQL 서버에 접속된 클라이언트 수만큼 존재
각 클라이언트가 요청하는 쿼리 문장 처리
커넥션 종료시 스레드 캐시(Thread cahce)로 돌아가는데 일정 개수 이상 스레드가 있으면 종료(thread_cache_size)
MyISAM 테이블은 디스크 쓰기 작업까지 포그라운드 스레드가 처리

### 백그라운드(Background) 스레드
인서트 버퍼를 병합
로그를 디스크로 기록
InnoDB 버퍼 풀 데이터를 디스크에 기록
데이터를 버퍼로 읽어옴
잠금이나 데드락 모니터링


## 메모리 구조
### 글로벌 메모리 영역	
	- 테이블 캐시
	- InnoDB 버퍼 풀
	- InnoDB 어댑티브 해시 인덱스
	- InnoDB 리두 로그 버퍼
클라이언트 스레드 수와 무관
모든 스레드에 의해 공유


### 로컬 메모리 영역(세션 메모리 영역)
	- 정렬 버퍼(Sort buffer)
	- 조인 버퍼
	- 바이너리 로그 캐시
	- 네트워크 버퍼
MySQL 서버 상에 존재하는 클라이언트 스레드가 쿼리를 처리하는데 사용
각 클라이언트 스레드별 독립적 할당, 공유되지 않음
각 쿼리의 용도별로 필요할 때만 공간이 할당될 수도 있음

### 플러그인 스토리지 엔진 모델
MySQL의 독특한 구조 중 대표적인 것
전문 검색 엔진을 위한 검색어 파서, 사용자 인증을 위한 Native Authentication 등이 플러그인으로 구현되어 제공
사용자가 직접 스토리지 엔진을 개발할 수 있음

### 컴포넌트
MySQL 8.0 이후 제공되는 아키텍처
플러그인의 단점을 개선
	- MySQL 서버만 인터페이스 가능, 플러그인끼리 통신 불가
	- MySQL 서버의 변수나 함수를 직접 호출 (캡슐화 안됨)
	- 상호 의존 관계 설정할 수 없어 초기화가 어려움


## 쿼리 실행 구조

![](README/DA27BE1F-81D4-4047-91B0-BFB82BEB849F.png)

### 쿼리 파서
사용자 요청으로 들어온 문장을 토큰으로 분리해 트리 구조로 만드는 작업
토큰 : MySQL이 인식할 수 있는 최소 단위의 어휘나 기호

### 전처리기
파서 과정에서 만들어진 파서 트리를 기반으로 쿼리 문장의 문제점 여부 확인
테이블, 컬럼, 함수명 등을 매핑하여 객체 존재여부 및 접근권한 확인

### 옵티마이저
쿼리 문장을 어떻게 적은 비용으로 빠른 처리할지 결정
옵티마이저가 더 나은 선택을 하도록 유도하는 것이 필요

### 실행 엔진
만들어진 계획대로 각 핸들러에 요청한 뒤, 요청 결과를 다른 핸들러 요청의 입력으로 연결

### 핸들러(스토리지 엔진)
데이터를 디스크로 저장하고 디스크로부터 읽어오는 역할(=스토리지 엔진)

## 쿼리 캐시(Query Cache)
SQL 실행 결과를 메모리에 캐시하고 동일한 SQL 쿼리가 실행되면 즉시 결과를 반환
데이터가 변경될 경우 캐시 결과 중 관련 데이터를 모두 삭제하는데, 이 때 동시 처리 성능 저하가 유발되었고, 성능 개선되는 과정에서 많은 버그의 원인이 되면서 MySQL 8.0에서는 제거됨

## 스레드 풀(Thread Pool)
엔터프라이즈 에디션에서는 제공하지만 커뮤니티 에디션은 지원하지 않음

### Percona Server에서 제공하는 스레드 풀
플러그인 형태로 작동
사용자 요청을 처리하는 스레드 개수를 줄여서 CPU가 제한된 개수의 스레드 처리에만 집중할 수 있도록 자원 소모를 줄임


## InnoDB 스토리지 엔진 아키텍처
스토리지 엔진 중 유일하게 레코드 기반 잠금을 제공 -> 높은 동시성 처
리, 안정적이며 성능이 뛰어남

### 프라이머리 키에 의한 클러스터링
InnoDB의 모든 테이블은 기본적으로 PK 키 기준으로 클러스터링되어 저장
PK 값 순서대로 디스크에 저장, 세컨더리 인덱스는 레코드의 주소가 아닌 PK 값을 논리적인 주소로 사용

### MVCC(Multi Version Concurrency Control)
잠금을 사용하지 않는 일관된 읽기를 제공하기 위해, 레코드 레벨의 트랜잭션을 지원하는 DBMS가 제공하는 기능
InnoDB는 Undo log를 이용
멀티 버전이란 하나의 레코드에 대해 여러개의 버전이 동시에 관리된다는 의미

#### MVCC 동작 과정
메모리
	- InnoDB 버퍼 풀
	- 언두 로그
디스크
	- 데이터파일
1. INSERT 직후 레코드는 데이터파일, InnoDB 버퍼 풀에 저장
2. UPDATE 후 커밋 여부와 상관없이 InnoDB 버퍼 풀의 데이터가 변경
3. UPDATE 후 레코드에서 변경된 컬럼의 변경 전 값만 언두 로그로 복사
4. COMMIT 전 다른 트랜잭션이 해당 레코드를 SELECT 하는 경우
	1. READ_UNCOMMITTED : InnoDB 버퍼 풀의 변경된 데이터 조회
	2. READ_COMMITTED 이상 : 커밋 전이기 때문에 언두 영역의 데이터 반환
5. COMMIT 하는 경우, 현재 상태를 영구적인 데이터로 만든 뒤, 언두 영역의 내용이 필요한 트랜잭션이 없을 때 삭제
6. ROLLBACK 하는 경우, 언두 영역의 백업 데이터를 InnoDB 버퍼 풀로 복구한 뒤 언두 영역의 내용을 삭제

### 잠금 없는 일관된 읽기(Non-Locking Consistent Read)
InnoDB 스토리지 엔진은 MVCC를 이용해 잠금을 걸지 않고 읽기 작업을 수행
변경 전 데이터를 읽기 위해 언두 로그를 사용

### 자동 데드락 감지
교착 상태 체크를 위해 잠금 대기 목록을 그래프(Wait-for List) 형태로 관리
데드락 감지 스레드가 주기적으로 잠금 대기 그래프를 검사 -> 교착 상태의 트랜잭션 중 언두 로그 레코드가 적은 트랜잭션을 강제 종료(롤백)

동시 처리 스레드가 많아지거나 각 트랜잭션의 잠금 수가 많아지면 데드락 감지 스레드가 느려지기 때문에 데드락 감지 스레드를 끄기도 함(innodb_deadlock_detect를 OFF)

그러면 잠금 획득을 설정 시간이상 획득하지 못했을 경우 에러를 반환하도록 innodb_lock_wait_timeout 옵션을 낮은 시간으로 변경해서 사용할 것을 권장(기본값 50초)

### 자동화된 장애 복구
MySQL 서버가 시작할 때 InnoDB 데이터 파일은 자동 복구를 수행하는데, 복구될 수 없는 손상이 있으면 MySQL 서버가 종료되버림
이 경우 MySQL 설정 파일에 innodb_force_recovery 시스템 변수를 설정해서 시작
설정 가능한 값은 1 부터 6 까지 인데 값이 커질수록 데이터 손실 가능성이 커지고 복구 가능성이 적어짐

1(SRV_FORCE_IGNORE_CORRUPT)
테이블스페이스의 데이터나 인덱스 페이지의 손상된 부분을 무시

2(SRV_FORCE_NO_BACKGROUND)
메인 스레드를 시작하지 않고 MySQL 서버 시작(메인 스레드가 언두 데이터를 삭제하는 과정에서 장애가 발생한 경우)

3(SRV_FORCE_NO_TRX_UNDO)
커밋되지 않은 트랜잭션 작업을 롤백하지 않고 그대로 둔채 시작

4(SRV_FORCE_NO_IBUF_MERGE)
인서트 버퍼의 기록된 내용을 무시하고 강제로 시작(인서트 버퍼는 실제 데이터가 아니라 데이터 손실 없을 수도 있음)

5(SRV_FORCE_NO_UNDO_LOG_SCAN)
언두 로그를 모두 무시하고 시작 (커밋되지 않은 작업도 커밋된 것처럼 처리됨)

6(SRV_FORCE_NO_LOG_REDO)
리두 로그를 모두 무시한 채로 MySQL 서버 시작 (데이터 파일에 기록되지 않은 데이터 모두 무시)

### InnoDB 버퍼 풀
InnoDB 스토리지 엔진의 가장 핵심적인 부분
디스크의 데이터 파일이나 인덱스 정보를 메모리에 캐시
쓰기 작업을 지연시켜 일괄 작업으로 처리할 수 있게 해줌

1. 버퍼 풀 크기 설정
OS와 각 클라이언트 스레드가 사용할 메모리를 충분히 고려
MySQL 5.7부터는 동적으로 조절이 가능하므로, 전체 메모리의 50%에서 시작해서 조금씩 올려가며 최적점을 찾음(innodb_buffer_pool_size)

내부 잠금(세마포어) 경합을 줄이기 위해 여러개로 쪼개어 관리할 수 있게 개선(버퍼 풀 인스턴스)
기본적으로 8개로 초기화되는데 메모리가 크다면 버퍼 풀 인스턴스 당 5GB 크기를 가질 수 있도록 인스턴스 개수를 설정(innodb_buffer_pool_instances)

2. 버퍼 풀 구조
InnoDB 스토리지 엔진은 버퍼 풀을 페이지 크기(innodb_page_size)의 조각으로 쪼개어 관리

버퍼 풀 페이지 크기 조각 관리
	- LRU(Least Recently Used) 리스트
	LRU + MRU(Most Recently Used) 결합된 형태 
	한 번 읽어온 페이지를 최대한 오랫동안 버퍼풀의 메모리에 유지해서 디스크 읽기를 최소화
	자주 사용되면 MRU 영역에서 살아남고, 사용 안되면 LRU의 끝으로 밀려나 결국 InnoDB 버퍼 풀에서 제거
	- 플러시(Flush) 리스트
	디스크로 동기화되지 않은 데이터를 가진 데이터 페이지(더티 페이지)의 변경 시점 기준의 페이지 목록을 관리
	데이터 변경이 가해진 데이터 페이지는 플러시 리스트에 관리되고 특정 시점에 디스크에 기록
	- 프리(Free) 리스트

3. 리두 로그
InnoDB 버퍼 풀은 DB 서버의 성능 향상을 위해 데이터 캐시와 쓰기 버퍼링 용도로 사용
버퍼 풀의 메모리 공간만 단순하게 늘리는 것은 데이터 캐시 기능만 향상시키는 것

여기서 버퍼링 기능을 향상시키려면 ?

버퍼 풀은 클린 페이지(변경사항 없는 데이터)와 더티 페이지(변경된 데이터)를 가짐
더티 페이지는 언젠가 디스크로 기록되야 함
InnoDB 스토리지 엔진에서 리두 로그는 1개 이상의 고정 크기 파일을 연결해서 순환 고리처럼 사용
버퍼 풀이 크더라도 리두 로그 파일 크기가 작다면 자주 디스크에 저장해야 함 -> 버퍼링 효과를 거의 못보는 것

### Double Write Buffer
InnoDB 스토리지 엔진의 리두 로그는 페이지에서 변경된 내용만 기록
-> 하드웨어 오작동 또는 시스템 비정상 종료 등으로 인해 일부만 기록된다면(Partital-page, Torn-page), 페이지의 내용 복구가 어려움

이를 막기 위해 Double-Write 기법을 이용 
InnoDB 버퍼 풀의 변경 내용을 데이터 파일에 기록하기 전에, DoubleWrite 버퍼에 변경된 데이터 페이지를 모아서 한번에 기록


### 언두 로그 
InnoDB 스토리지 엔진은 트랜잭션과 격리 수준을 보장하기 위해 DML로 변경되기 전 버전 데이터를 별도로 기록(Undo Log)

- 트랜잭션 보장
트랜잭션 롤백시 언두 로그에 백업해둔 이전 버전 데이터를 이용해 복구

- 격리 수준 보장
다른 커넥션에서 데이터를 조회할 때 트랜잭션 격리 수준에 맞게, 레코드가 아닌 언두 로그의 데이터를 읽어서 반환

롤백 세그먼트는 1개 이상의 언두 슬롯을 가짐
1개의 롤백 세그먼트는 InnoDB 페이지 크기를 16바이트로 나눈 값의 개수만큼 언두 슬롯을 가짐
하나의 트랜잭션은 최대 4개의 언두 슬롯을 사용

최대 동시 트랜잭션 수 = InnoDB 페이지 크기 / 16 * 롤백 세그먼트 개수 * 언두 테이블 스페이스 개수

가장 일반적인 설정인 16KB InnoDB에서 기본 설정인 innodb_undo_tablespaces=2, innodb_rollback_segments=128을 가정하면 16*1024/16*128*2/2 = 131,072개의 트랜잭션 동시 처리가 가능

언두 슬롯은 부족하면 트랜잭션이 시작할 수 없기 때문에 동시 트랜잭션 개수에 맞게 언두 테이블 스페이스와 롤백 세그먼트 개수 설정 필요

### 체인지 버퍼
쓰기 작업을 할 때 데이터 파일 변경 뿐 아니라 테이블의 인덱스도 업데이트 해야하는데, 이 작업은 랜덤하게 디스크를 읽기 때문에 인덱스가 많다면 자원이 많이 소모됨

그래서 변경할 인덱스 페이지가 버퍼 풀에 있으면 바로 업데이트하지만, 없는 경우 임시 공간에 저장해두고 결과를 바로 반환하는데, 이 때 사용하는 임시 메모리 공간을 체인지 버퍼(Change Buffer)라고 함

유니크 인덱스는 결과 반환 전에 중복 여부를 체크해야 해서 체인지 버퍼를 사용할 수 없음
체인지 버퍼에 임시로 저장된 인덱스 레코드 조각은 체인지 버퍼 머지 스레드에 의해 병합

### 리두 로그 및 로그 버퍼
리두 로그(Redo Log)는 ACID 중 D에 관련, 서버가 비정상적으로 종료되었을 경우 데이터 파일에 기록되지 않은 데이터를 잃지 않게 해주는 안전장치

데이터 파일은 쓰기보다 읽기 성능이 더 고려됨
쓰기 작업에서의 성능 저하를 막기 위해 쓰기 비용이 낮은 자료구조인 리두 로그를 갖게되고, 비정상 종료가 되었을 때 리두 로그의 내용으로 데이터 파일을 복구
리두 로그의 버퍼링을 위한 InnoDB 버퍼 풀과 로그 버퍼와 같은 자료 구조를 갖고 있음

커밋되었지만 데이터 파일에 기록되지 않은 데이터는 리두 로그에 저장된 데이터를 데이터 파일에 복사

롤백되었지만 데이터 파일에 이미 기록된 데이터는 리두 로그로 트랜잭션의 상태를 확인한 뒤, 언두 로그의 내용을 가져와 데이터 파일에 복사

innodb_flush_log_at_trx_commit : 어느 주기로 리두 로그를 디스크에 동기화할지
0 : 1초에 한번씩 리두 로그를 디스크에 기록하고 동기화
1 : 트랜잭션 커밋될 때마다 기록하고 동기화
2 : 커밋될때마다 기록하고 1초에 한번씩 동기화

### 어댑티브 해시 인덱스(Adaptive Hash Index)
InnoDB 스토리지 엔진에서 사용자가 자주 요청하는 데이터에 대해 자동으로 생성
B-Tree 인덱스의 검색 시간을 줄여주기 위해 도입

성능 향상에 크게 도움이 되지 않는 경우
	- 디스크 읽기가 많은 경우
	- 특정 패턴의 쿼리가 많은 경우
	- 매우 큰 데이터를 가진 테이블의 레코드를 넓게 읽는 경우

성능 향상에 많은 도움이 되는 경우
	- 디스크의 데이터가 InnoDB 버퍼 풀 크기와 비슷한 경우(디스크 읽기가 적은 경우)
	- 동등 조건 검색이 많은 경우(동등 비교, IN 연산자)
	- 쿼리가 일부 데이터에 집중되는 경우

### MySQL 로그파일

#### 에러 로그 파일
MySQL 실행 도중 발생하는 에러나 경고 메시지가 출력
my.cnf의 log_error 이름의 파라미터로 정의된 경로

#### 제너럴 쿼리 로그
실행되는 쿼리의 전체 목록을 뽑기 위한 로그
시간 단위로 실행됐던 쿼리의 내용이 모두 기록
에러가 발생되도 기록

#### 슬로우 쿼리 로그
long_query_time 시스템 변수에 설정한 시간 이상 소요된 쿼리 기록
실제 소요된 시간이기 때문에 쿼리가 정상적으로 실행이 완료되어야 기록이 가능
log_output 옵션이 TABLE인 경우에 제너럴 로그 또는 슬로우 쿼리 로그를 테이블로 저장(general_log, slow_log), FILE인 경우 디스크의 파일로 저장

## 트랜잭션
트랜잭션(Transaction) : 데이터의 정합성을 보장
잠금(Lock) : 동시성 제어



### MySQL 엔진 잠금
모든 스토리지 엔진에 영향을 미치는 잠금
테이블 락(Table Lock) : 테이블 데이터 동기화를 위한 잠금
메타데이터 락(Metadata Lock) : 테이블의 구조를 잠금
네임드 락(Named Lock) : 사용자의 필요에 맞게 사용

#### 글로벌 락(Global Lock)
`FLUSH TABLES WITH READ LOCK` 명령으로 획득
MySQL 제공하는 잠금 중 가장 범위가 큼
한 세션에서 글로벌 락을 획득한 경우 다른 세션에서 SELECT를 제외한 대부분의 DDL, DML 문이 대기 상태
MyISAM 또는 MEMORY 테이블에 대해 mysqldump 로 일관된 백업을 받아야할 때 사용

MySQL 8.0 부터는 InnoDB가 기본 스토리지 엔진이고, 백업 툴들을 위해 백업 락이 도입되어 잘 사용하지 않음

#### 테이블 락(Table Lock)
`LOCK TABLES table_name [READ|WRITE]`  명령으로 특정 테이블의 락을 획득
개별 테이블 단위로 설정되는 잠금
InnoDB 테이블의 경우 스토리지 엔진 차원에서 레코드 기반의 잠금을 제공하기 때문에 DDL 에 영향을 미침

#### 네임드락(Named Lock)
`GET_LOCK()` 함수를 이용하여 임의 문자열에 대한 잠금 설정
사용자가 지정한 문자열(String)에 대해 획득하고 반납하는 잠금

여러 클라이언트가 특정 정보를  상호 동기화 처리해야할 때 사용
배치 프로그램처럼 많은 레코드를 변경하는 경우에 데드락의 원인이 되기도 하는데 네임드 락을 걸고 참조하는 프로그램끼리 분류해서 쿼리를 실행하는 것이 해결책이 될 수 있음

#### 메타데이터 락(Metadata Lock)

데이터베이스 객체의 이름이나 구조를 변경하는 경우에 획득하는 잠금
`RENAME TABLE` 등의 명령을 사용하는 경우 자동으로 획득됨

### InnoDB 스토리지 엔진 잠금
#### 레코드 락(Record Lock)
레코드 자체만을 잠그는 것으로, InnoDB 스토리지 엔진은 레코드 자체가 아닌 인덱스의 레코드를 잠금
인덱스가 없더라도 내부적으로 자동 생성된 클러스터 인덱스를 이용해 잠금을 설정

#### 갭 락(Gap Lock)
레코드 자체가 아니라 레코드와 인접한 레코드 사이 간격만을 잠금
레코드와 레코드 사이에 새로운 레코드가 생성되는 것을 제어

#### 넥스트 키 락(Next Key Lock)
레코드 락 + 갭 락 형태
STATEMENT 포맷의 바이너리 로그를 사용하는 MySQL 서버에서는 REPEATABLE READ 격리 수준을 사용해야 함
innodb_locks_unsafe_for_binlog 시스템 변수가 비활성화되면(0으로 설정) 변경을 위해 검색하는 레코드에 넥스트 키 락 방식으로 잠금이 걸림

바이너리 로그에 기록되는 쿼리가 레플리카 서버에서 실행될 때 소스 서버에서 만들어낸 결과와 동일한 결과를 만들도록 보장하는 것이 주 목적

데드락이 발생하거나 다른 트랜잭션을 대기하게 만드는 일이 자주 발생하기 때문에 바이너리 로그 포맷을 ROW 형태로 바꿔서 넥스트 키 락 또는 갭 락을 줄이는 것이 좋음 (MySQL 8.0 기본 설정)

#### 자동 증가 락(Auto Increment Lock)
AUTO_INCREMENT 칼럼이 사용된 테이블에 동시에 여러 레코드가 INSERT되는 경우, 중복되지 않게 하기 위해 사용하는 테이블 수준의 잠금

새로운 레코드를 저장하는 쿼리(INSERT 또는 REPLACE 문)에만 걸림
트랜잭션과 관계없이 AUTO_INCREMENT 값을 가져오는 순간만 락이 걸렸다가 즉시 해제됨
innodb_autoinc_lock_mode=0
모든 INSERT 문에서 자동 증가 락 사용
innodb_autoinc_lock_mode=1
INSERT 되는 레코드의 건수를 정확히 예측이 가능할 경우 더 빠른 래치(뮤텍스)를 이용
하지만 INSERT … SELECT 문에서는 (예측이 불가하므로) 자동 증가 락을 사용 

innodb_autoinc_lock_mode=2
무조건 래치(뮤텍스)를 사용
동시 처리 성능이 높아지지만 연속된 자동 증가 값을 보장하지 않음
STATEMENT 포맷의 바이너리 로그를 사용하는 복제에서는 소스와 레플리카 서버의 자동 증가 값이 달라질 수 있음

STATEMENT 포맷의 바이너리 로그 사용시 innodb_autoinc_lock_mode 1로 변경해서 사용할 것을 권장

### 인덱스와 잠금
InnoDB 잠금은 레코드를 잠그는 것이 아니라 인덱스를 잠금
UPDATE 문의 WHERE 조건에 여러 조건이 있더라도 인덱스 조건의 레코드를 잠그게 됨

예시로
 `UPDATE … WHERE A = ‘1’ AND B = ‘2’` 
다음 업데이트 문에서 A에만 인덱스가 있다면 A=‘1’인 레코드를 모두 잠금
만약 테이블의 인덱스가 없다면 테이블을 풀 스캔하며 테이블의 모든 레코드를 잠그게 됨

### 격리 수준
여러 트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있게 허용할지 말지를 결정하는 것

#### READ UNCOMMITTED
각 트랜잭션의 변경 내용이 COMMIT/ROLLBACK 여부에 상관없이 다른 트랜잭션에 보이는 것
Dirty read 발생 (트랜잭션의 작업완료 전 다른 트랜잭션에서 조회할 수 있는 현상)
정합성에 문제가 많은 격리 수준

#### READ COMMITTED
오라클 DBMS에서 기본적으로 사용하는 격리 수준
어떤 트랜잭션에서 데이터를 변경했더라도 COMMIT이 완료되어야 조회할 수 있음

REPEATABLE READ 문제  발생 : 같은 트랜잭션에서 같은 조회 쿼리를 사용했을 때 다른 결과가 조회되는 현상 (조회 쿼리 사이에 다른 트랜잭션에서 해당 데이터를 변경한 뒤 커밋)
입출금이 실시간으로 이루어지는 테이블에서 입금 총합을 계산할 때 문제가 발생할 수 있음

#### REPEATABLE READ
MySQL InnoDB 스토리지 엔진에서 기본으로 사용되는 격리 수준
바이너리 로그를 가진 MySQL 서버에서는 최소 이 격리 수준 이상을 사용해야 함
왜?
READ-COMMITTED 방식은 하나의 트랜잭션에서도 실행 시점에 따라 사용하는 스냅샷이 다른데 바이너리 로그를 남기게되면 어떤 시점의 데이터로 처리되었는지 알 수 없을 것 같다

InnoDB 스토리지 엔진은 트랜잭션의 롤백 가능성에 대비해 변경 전 데이터를 Undo 공간에 백업해두고 실제 InnoDB 버퍼 풀의 데이터를 변경함(MVCC)
READ COMMITTED도 마찬가지로 Undo 공간의 변경 전 데이터를 조회하긴 함
REPEATABLE READ는 트랜잭션 고유 번호로 자신의 트랜잭션 번호보다 작은 트랜잭션에서 변경된 것만 보여진다는 차이

다른 트랜잭션에서 수행한 변경 작업에 의해 레코드가 보였다 안보였다 하는 PHANTOM READ 현상이 일어날 수 있음 -> InnoDB 스토리지 엔진에서는 갭 락과 넥스트 키 락 떄문에 발생하지 않음

#### SERIALIZABLE
읽기 작업 조차 공유 잠금을 획득해야 함 -> 동시성이 너무 떨어짐, 잘 사용 X

## 데이터 압축
### 페이지 압축(Transparent Page Compression)
MySQL 서버가 디스크에 저장하는 시점에 데이터 페이지가 압축되어 저장
MySQL 서버가 디스크에 데이터 페이지를 읽어올 때 압축 해제
데이터 페이지를 압축한 결과의 용량이 얼마나 될지 예측이 불가한 채, 최소 하나의 테이블은 동일한 크기의 페이지(블록)로 통일되어야 함 -> 펀치 홀(Punch hole) 사용
펀치 홀 기능은 OS와 하드웨어가 둘다 해당 기능을 지원해야 사용 가능
따라서 실제로 페이지 압축은 많이 사용되지 않음

### 테이블 압축
OS나 하드웨어의 제약이 없음
단, 버퍼 풀 공간 활용률이 낮고, 쿼리 처리 성능이 낮음
빈번한 데이터 변경시 압축률이 떨어짐

#### 압축 테이블 생성
innodb_file_per_table = ON으로 설정하고, ROW_FORMAT=COMPRESSED 옵션 명시

TODO: 추후 보충



## 인덱스
### 디스크 읽기 방식
#### HDD와 SSD
SSD는 HDD의 데이터 저장용 플래터를 제거하고 플래시 메모리를 장착
순차 I/O에서는 SSD가 조금 더 빠르거나 거의 비슷한 성능
랜덤 I/O에서는 SSD가 훨씬 빠름
DB에서 순차 I/O는 비중이 크지 않고 대부분 랜덤 I/O를 통해 데이터를 읽고 씀

#### 랜덤 I/O와 순차 I/O
순차 I/O는 3개의 페이지를 디스크에 기록할 때 1번의 시스템 콜을 요청
랜덤 I/O는 3개의 페이지를 디스크에 기록할 때 3번의 시스템 콜을 요청
디스크에 기록할 위치를 찾기 위해 헤드를 몇번 움직였는지에 차이
디스크의 성능은 디스크 헤더의 위치 이동 없이 얼마나 많은 데이터를 한번에 기록하느냐에 의해 결정

인덱스 레인지 스캔은 랜덤 I/O 풀 테이블 스캔은 순차 I/O 
큰 테이블의 레코드 중 대부분을 읽을 떄는 풀 테이블 스캔이 더 빠를 수 있음

### 인덱스
책 내용을 빠르게 찾기 위한 책 마지막 페이지의 찾아보기(색인) 처럼 데이터 파일을 더 빠르게 조회하기 위해 레코드의 주소를 값으로 저장한 키와 값의 쌍

인덱스는 SortedList 처럼 정렬해서 저장 데이터 파일은 ArrayList 처럼 저장되는 순서 그대로 유지한 채 저장

SortedList는 항상 값을 정렬해야 하므로 저장이 느리지만 빠르게 값을 찾아올 수 있음

### B-Tree 인덱스

컬럼 원래의 값을 변형시키지 않고 인덱스 구조체 내에서 항상 정렬된 상태로 유지

#### B-Tree 구조
트리 구조의 최상위에 하나의 루트 노드(Root node)가 존재하고 최하위에 여러 리프 노드(Leaf node)가 존재
루트 노드와 리프 노드가 아닌 중간의 노드들을 브랜치 노드(Branch node)라고 함 
인덱스의 리프 노드는 실제 데이터 레코드를 찾아가기 위한 주소 값을 가짐

InnoDB 스토리지 엔진에서는 PK가 ROWID 역할을 하고, 모든 세컨더리 인덱스 검색에서 데이터 레코드를 읽기위해 PK 키를 저장하는 B-Tree를 다시 한번 검색

MyISAM 테이블은 세컨더리 인덱스가 물리적인 주소를 갖고, 한번 검색으로 실제 데이터의 주소 값을 찾아 갈 수 있음

#### B-Tree 인덱스 추가
새로운 키 값이 B-Tree에 저장될 때 저장될 키 값으로 B-Tree의 저장할 위치를 검색한 뒤,  레코드의 키 값과 레코드의 주소 정보를 B-Tree의 리프 노드에 저장
리프 노드가 꽉 차서 저장할 수 없는 경우 리프 노드가 분리(Split) -> 비용이 많이 소요

#### B-Tree 인덱스 삭제
키 값이 저장된 B-Tree의 리프 노드를 찾아 삭제 마크
삭제 마킹된 인덱스 키 공간은 방치되거나 재활용

#### B-Tree 인덱스 키 변경
키 값에 따라 저장될 리프 노드의 위치가 결정되므로, 키 값을 삭제한 뒤, 새로운 값을 추가

#### B-Tree 인덱스 키 검색
루트 노드, 브랜치 노드를 거쳐 최종 리프 노드까지 이동하며 비교 작업 수행(트리 탐색)
100% 일치 또는 값의 앞부분만 일치에 사용
뒷부분만 일치 또는 변형된 값(함수, 연산)으로는 B-Tree의 빠른 검색 기능을 사용할 수 없음
InnoDB 스토리지 엔진에서는 레코드 잠금이나 넥스트 키락이 검색을 수행한 인덱스를 잠그기 때문에 UPDATE나 DELETE문에서 적절한 인덱스가 없는 경우, 불필요하게 많은 레코드를 잠그게됨

#### B-Tree 인덱스 키 값의 크기
InnoDB 스토리지 엔진에서 디스크에 데이터를 저장하는 가장 기본 단위를 페이지(Page), 블록(Block)이라고 하며 디스크 I/O 작업의 최소 작업 단위가 됨
페이지는 버퍼 풀에서 데이터를 버퍼링하는 기본 단위기도 함

페이지 당 저장할 수 있는 인덱스의 개수는 인덱스 키 값에 크기에 따라 다른데, 키가 커질 수록 같은 레코드 범위를 읽더라도 디스크를 읽어야 하는 횟수가 늘어나 느려지게 됨

인덱스를 캐시해 두는 InnoDB 버퍼 풀 또는 MyISAM의 키 캐시 영역은 크기가 제한적이기 때문에 한 레코드를 위한 인덱스 크기가 커질수록 메모리에 캐시해 둘 수 있는 레코드 수는 줄어들어 메모리 효율이 떨어짐

인덱스 키 값의 크기가 커질수록 같은 레코드 건수 당 B-Tree의 깊이도 깊어져 디스크 읽기가 많이 필요하게 됨 (대용량 DB여도 Depth가 5단계 이상 깊어지는 경우가 흔치않음)

선택도(Selectivity), 기수성(Cardinality)
모든 인덱스 키 값 가운데 유니크한 값의 수
전체 인덱스 100개 중 유니크한 값이 10개라면 기수성은 10
중복된 값이 많아질 수록 기수성은 낮아지고, 선택도가 떨어짐

일반적인 DBMS의 옵티마이저에서 1건의 조회에서 인덱스를 통해 읽는 것은 테이블을 직접 읽는 것보다 4-5배 정도 비용이 많이 드는 작업으로 예측하므로, 인덱스를 통해서 읽을 때 전체 테이블 레코드의 20-25%를 넘어서게 되면 테이블을 모두 읽어서 필터링하는 방식으로 처리하는 것이 효율적으로 판단하게 됨

따라서 전체 건수 대비 유니크한 값이 적을수록 옵티마이저에서 인덱스가 쿼리의 효율성에 도움이 되지 않는다고 판단해 사용하지 않게될 가능성이 높고, 힌트로 사용한다고 해도 성능상의 도움이 안될 수 있음

그래도 인덱스를 만드는 것이 정렬이나 그루핑에 도움이 될 수 있기 때문에 적절히 인덱스를 설계할 필요가 있음

#### B-Tree 인덱스를 통한 데이터 읽기
1. 인덱스 레인지 스캔

가장 대표적인 접근 방식, 2번과 3번 접근 방식보다는 빠름
검색해야 할 인덱스의 범우가 결정되었을 때 사용하는 방식

1) 인덱스에서 조건이 만족하는 값이 저장된 위치를 찾음 (index seek)
2) 1번에서 탐색된 위치에서 필요한 만큼 인덱스를 읽음(index scan)
3) 2번에서 읽어 들인 인덱스 키와 레코드 주소를 이용해 레코드가 저장된 페이지를 가져오고 최종 레코드를 읽어옴

3번 과정이 필요없는 경우(커버링 인덱스) 그만큼 랜덤 읽기가 줄어들기 때문에 성능이 빨라짐

2. 인덱스 풀 스캔

인덱스 리프 노드의 제일 앞이나 뒤에서 리프 노드끼리 연결된 링크드 리스트를 따라서 처음부터 끝까지 스캔하는 방식

조건 절의 컬럼이 인덱스의 첫 번째 컬럼이 아닌 경우, 인덱스의 명시된 컬럼만으로 조건을 처리할 수 있는 경우 인덱스 풀 스캔 방식을 사용

일반적으로 인덱스 크기가 테이블 크기보다 작으므로 인덱스만 읽는 것이 효율적(적은 디스크 I/O로 처리 가능)

3. 루스(Loose) 인덱스 스캔

Oracle의 인덱스 스킵 스캔 기능과 작동 방식이 비슷, 앞의 두 가지 방식은 타이트(Tight) 인덱스 스캔으로 분류

조건에 만족하지 않는 레코드를 무시하고 듬성듬성하게 인덱스를 읽는 것을 의미

GROUP BY 같은 집합 함수에서 MAX(), MIN() 함수의 최적화에 사용

4. 인덱스 스킵 스캔
MySQL 8.0 버전 부터 도입된 것으로, 인덱스의 두번째 컬럼의 조건으로도 인덱스 검색이 가능하게 함
루스 인덱스 스캔과 비슷하지만 GROUP BY에서만 사용하는 루스 인덱스 스캔과 다르게 WHERE 조건절의 검색을 위해 사용할 수 있도록 범위가 넓어짐

인덱스 스킵 스캔 단점
- WHERE 조건절에 조건이 없는 인덱스의 선행 컬럼에서 유니크한 값의 개수가 적어야함
- 쿼리가 인덱스에 존재하는 컬럼만으로 처리 가능해야 함(커버링 인덱스)
 
추후 옵티마이저가 개선되면 해결될 부분으로 보임

#### B-Tree 인덱스 정렬
각 컬럼의 정렬을 오름차순이나 내림차순으로 설정
MySQL 5.7까지는 혼합이 불가
MySQL 8.0부터는 혼합 순서 인덱스도 생성 가능 (A ASC, B DESC)

InnoDB에서 다음 이유로 Forward index scan이 더 빠름
- 페이지 잠금이 Forward index scan에 적합한 구조
- 페이지 내에서 인덱스 레코드가 단방향으로만 연결된 구조

#### B-Tree 인덱스의 효율성
다중 컬럼 인덱스에서 컬럼 순서와 컬럼에 사용 조건에 따라 인덱스의 효율이 달라짐

다음 쿼리에서 
`SELECT … WHERE A = ‘test’ AND B > 10;`

INDEX(A,B)
INDEX(B,A)

두가지 인덱스 중 첫번째 인덱스는 5건의 레코드에 대해 5번의 비교작업을 수행(효율적)
두번째 인덱스는 5건에 레코드에 대해 5+a(필터링된 레코드 수) 만큼의 비교작업을 수행(비효율적)
인덱스 후행 컬럼에서의 동등조건 비교라서 단순 필터링 역할만 수행하기 떄문(작업 범위를 줄이지 못함) 

작업 범위를 결정하는 조건은 많을수록 쿼리 처리 성능을 높이지만, 체크조건(필터링조건)은 처리 성능을 높이지 못함(오히려 쿼리 실행이 느려질 수 있음)

#### B-Tree 인덱스의 가용성

B-Tree 인덱스는 왼쪽 값에 기준해서(Left-most) 오른쪽 값이 정렬되어 있기 때문에 값의 왼쪽 부분을 모르면 인덱스 레인지 스캔 방식으로 검색이 불가능

다음 조건에서는 작업 범위 결정 조건으로 인덱스 사용이 불가
- NOT-EQUAL 비교 (<>, NOT IN, NOT BETWEEN, IS NOT NULL)
- Like ‘%검색문자열’
- 스토어드 함수나 다른 연산자로 컬럼이 변형된 뒤 비교
- NOT-DETERMINISTIC 속성의 스토어드 함수가 비교조건에 사용
- 데이터 타입이 서로 다른 비교
- 문자열 데이터 타입의 콜레이션이 다른 경우

### R-Tree 인덱스
공간 인덱스(Spatial Index)로 2차원의 데이터를 인덱싱하고 검색하는 목적
기본적인 내부 매커니즘은 B-Tree와 흡사하지만 1차원의 스칼라 값인 B-Tree와 달리 2차원의 공간 개념 값으로 구성

MySQL에서 공간 정보 저장 및 검색을 위한 데이터 타입
POINT
LINE
POLYGON
GEOMETRY

MBR(Minimum Bounding Rectangle) : 해당 도형을 감싸는 최소 크기의 사각형
R-Tree 인덱스는 MBR의 포함 관계를 B-Tree 형태로 구현한 인덱스

WGS84(GPS) 기존의 위도, 경도 좌표 저장에 주로 사용
ST_Contains(), ST_Within() 등의 포함 관계 비교 함수로 검색을 수행하는 경우에 인덱스를 이용
ex) 현재 위치로 부터 반경 5km 이내의 음식점 검색

### 전문(Full Text) 검색 인덱스

MySQL의 B-Tree 인덱스는 실제 컬럼이 1MB라도 MyISAM 기준 1,000 바이트,
InnoDB 기준 3,072 바이트 까지만 잘라서 인덱스 키로 사용

전문 검색은 문서 본문의 내용에서 사용자가 검색할 키워드를 분석하고,  해당 키워드로 인덱스를 구축
문서의 키워드를 인덱싱 하는 방식에 따라 어근 분석, n-gram 분석으로 구분

#### 어근 분석 알고리즘
불용어(Stop Word) 처리 -> 어근 분석(Stemming)
어근 분석은 검색어로 선정된 단어의 뿌리인 원형을 찾는 작업
형태소 분석을 통해 명사와 조사를 구분(MeCab을 MySQL 서버에 적용)

#### n-gram 알고리즘
형태소 분석은 결과를 내기위해 많은 시간과 노력이 필요
n-gram은 단순히 키워드를 검색해내기 위한 인덱싱 알고리즘

n에 해당하는 글자 수(2 이상)로 잘라서 불용어를 제거한 뒤 인덱스에 등록

MATCH (…) AGAINST (…) 구문으로 쿼리를 작성하여 사용

### 함수 기반 인덱스
MySQL 8.0 부터 컬럼의 값을 변형하여 만들어진 값에 대해 구축한 인덱스
인덱싱할 값을 계산하는 과정의 차이만 있고 나머지 구조는 B-Tree와 동일

#### 가상 칼럼을 이용한 인덱스
first_name 컬럼과 last_name 컬럼이 있을 때 두 컬럼을 합쳐서 검색해야 하는 경우 MySQL 8.0 부터는 full_name 가상 컬럼을 추가하고 가상 컬럼에 인덱스를 생성할 수 있음

하지만 추가할 때 실제 테이블 구조가 변경됨

#### 함수를 이용한 인덱스
직접 인덱스에 함수식을 넣어서 계산된 결과값의 검색을 빠르게 만들어줌
함수 생성 시 명시된 표현식과 쿼리 WHERE 조건절에서 사용된 표현식이 다르면 함수 기반 인덱스를 사용하지 못함(옵티마이저가 다른 표현식으로 간주)

### 멀티 밸류 인덱스
전문 검색 인덱스를 제외하고는 레코드 1건이 1개의 인덱스 키 값을 가짐
멀티 밸류 인덱스는 하나의 데이터 레코드가 여러 개의 키 값을 가질 수 있는 형태의 인덱스(JSON 타입 컬럼)

인덱스 활용에 다음 함수의 이용이 필수
MEMBER OF()
JSON_CONTAINS()
JSON_OVERLAPS()

### 클러스터링 인덱스
InnoDB 스토리지 엔진만 지원
PK 값이 비슷한 레코드끼리 묶어서 저장하는 것
PK 값에 의해 레코드의 저장 위치가 결정(PK 변경시 물리적인 저장위치도 변경)

인덱스 알고리즘이라기 보다는 테이블 레코드의 저장 방식으로, 클러스터링 인덱스와 클러스터링 테이블이 동의어로 사용되기도 함 

PK 기반 검색이 매우 빠르며, 레코드 저장이나 PK의 변경이 상대적으로 느림

클러스터링 인덱스의 리프 노드에는 레코드의 모든 칼럼이 저장됨
클러스터링 테이블 자체가 하나의 거대한 인덱스 구조로 관리

InnoDB 스토리지 엔진에서 클러스터링 키 선택 순서
1) PK (있을시)
2) NOT NULL 옵션의 유니크 인덱스 중 첫번째 인덱스
3) 자동으로 유니크한 값을 가지도록 증가하는 컬럼(내부적으로 추가)

MyISAM이나 MEMORY 테이블 (클러스터링 X)은 테이블이 INSERT될 때 저장된 공간에서 이동 X, 주소는 내부적인 레코드 아이디(ROWID) 역할
따라서 PK나 세컨더리 인덱스의 각 키는 ROWID를 이용해 실제 데이터 레코드를 찾아옴
-> PK와 세컨더리 인덱스의 구조적 차이가 없음

실제 주소를 저장하고 있기 때문에 데이터 레코드의 주소가 변경되면 모든 인덱스에 저장된 실제 주소 값을 변경하는 오버헤드가 생김

그래서 InnoDB 테이블(클러스터링)은 모든 세컨더리 인덱스가 데이터 저장 주소가 아닌 PK 값을 저장

#### 클러스터링 인덱스의 장점
- 클러스터링 키(PK)로 검색할 때 성능이 매우 빠름(특히 범위검색)
- 모든 세컨더리 인덱스가 PK를 가지고 있기 때문에 커버링 인덱스되는 경우가 많아짐

#### 클러스터링 인덱스의 단점
- 모든 세컨더리 인덱스가 클러스터링 키를 갖기 때문에 그 크기가 클 경우 전체적으로 인덱스 크기가 커짐
- 세컨더리 인덱스 검색에서 PK로 다시 검색해야 하므로 처리 성능이 느림
- INSERT할 때 PK에 의해 레코드의 저장위치가 결정되므로 처리 성능이 느림
- PK 변경시 레코드를 DELETE하고 INSERT하기 때문에 처리 성능이 느림

#### PK는 AUTO-INCREMENT 보다는 업무적인 칼럼이 좋음

#### 프라이머리 키는 반드시 명시
내부적으로 AUTO-INCREMENT되는 일련번호 컬럼이 어차피 추가되며, 이는 사용이 불가하므로 생성하는 것이 좋음
ROW 기반 복제나 InnoDB Cluster에서는 모든 테이블이 PK를 가져야 정상적인 복제 성능을 보장

### 유니크 인덱스
인덱스 보다는 제약 조건에 가까움
NULL은 특정 값이 아니기 때문에 2개 이상 저장 가능

다른 세컨더리 인덱스에 비해 조회 성능이 특별히 더 빠르지는 않음

인덱스 쓰기 작업에 중복 체크 과정이 있어 세컨더리 인덱스보다 쓰기가 느림
중복 값 체크에서 사용하는 읽기 잠금과 쓰기할 때 사용하는 쓰기 잠금이 데드락을 빈번히 일으키기도 함
InnoDB 스토리지 엔진에서 인덱스 키의 저장을 버퍼링하는 체인지 버퍼의 사용이 불가(중복 체크 때문)

유일성이 꼭 보장되어야 하는 컬럼이 아니라면 세컨더리 인덱스를 생성

### 외래키
InnoDB 스토리지 엔진에서만 생성 가능

- 테이블의 변경이 발생하는 경우에만 잠금 경합이 발생
- 외래키와 연결되지 않은 컬럼의 변경은 최대한 잠금 경합을 발생시키지 않음

#### 자식 테이블의 변경이 대기하는 경우
자식 테이블의 외래키 칼럼이 변경할 때 부모 테이블 해당 레코드의 쓰기 잠금을 확인

#### 부모 테이블의 변경이 대기하는 경우
부모 테이블의 레코드를 삭제할 때 자식 테이블 레코드의 쓰기 잠금을 확인


## 옵티마이저
### 쿼리 실행 절차
1. 요청된 SQL 문장을 쪼개 MySQL 서버가 이해할 수 있는 수준으로 분리 
(SQL 파서: SQL Parsing 수행 -> SQL 파스 트리 생성)
2. SQL 파싱정보를 확인하며 어떤 테이블부터 어떤 인덱스로 테이블을 읽을지 선택
(옵티마이저: 최적화 및 실행 계획 수립 -> 실행 계획 수립)
3. 2단계에서 결정된 읽기 순서 또는 선택된 인덱스로 스토리지 엔진에서 데이터를 가져옴

### 옵티마이저 종류

1. 비용 기반 최적화(Cost-based optimizer, CBO)
쿼리를 처리하기 위한 여러 가능한 방법 생성 
-> 각 단위 작업의 비용 정보와 테이블 예측 통계로 실행 게획별 비용을 산출
-> 비용이 최소로 소요되는 처리 방식으로 쿼리 실행

2. 규칙 기반 최적화(Rule-based optimizer, RBO)
옵티마이저에 내장된 우선 순위에 따라 실행 게획을 수립
테이블의 레코드 건수나 컬럼값의 분포도 조사하지 않고 수립되어, 같은 쿼리는 거의 항상 같은 실행 방법을 만듬

현재 대부분의 RDBS가 비용 기반의 옵티마이저를 채택 (MySQL 포함)

#### 기본 데이터 처리
##### 풀 테이블 스캔

## 실행 계획
### 통계정보
MySQL 5.7 까지는 테이블과 인덱스에 대한 개괄적인 정보로 실행 계획을 수립
MySQL 8.0 부터는 인덱스되지 않은 칼럼에 대해서도 데이터 분포도를 수집해서 저장(히스토그램)

#### 테이블 및 인덱스 통계 정보
5.6 버전 부터 InnoDB 스토리지 엔진 사용하는 테이블에 대한 통계정보를 별도의 테이블로 영구적으로 관리 (이전 까지는 메모리로만 관리)

#### 히스토그램(Histogram)
옵티마이저의 실행 계획 최적화를 위해 컬럼의 데이터 분포도를 참조할 수 있도록 MySQL 8.0 부터 지원
버킷 단위로 구분하여 레코드 건수나 컬럼값 범위를 관리

	- Singleton(싱글톤) 히스토그램
	컬럼값 개별로 레코드 건수를 관리(도수 분포)
	컬럼이 가질 수 있는 값에 대한 누적된 레코드 건수 비율
	코드 값 등 유니크한 값의 개수가 상대적으로 적은 경우에 사용

	- Equi-Height(높이 균형) 히스토그램
	컬럼값의 범위를 균등한 개수로 구분하여 관리
	레코드 건수 비율이 누적으로 표시

히스토그램이 없는 경우 옵티마이저는 데이터가 균등하게 분포되어 있다고 예측
하지만 실제 데이터는 항상 균등한 분포도를 가지지 않음
컬럼의 히스토그램 정보로 어느 테이블을 먼저 읽어야 조인의 횟수를 줄일 수 있는지에 대해 옵티마이저가 더 정확한 판단 가능

#### 코스트 모델(Cost Model)

MySQL 서버가 쿼리를 처리할 때 필요한 작업
	- 디스크로 부터 데이터 페이지 읽기
	- 메모리(InnoDB 버퍼 풀)로부터 데이터 페이지 읽기
	- 인덱스 키 비교
	- 레코드 평가 
	- 메모리 임시 테이블 작업
	- 디스크 임시 테이블 작업

MySQL 서버에서는 사용자 쿼리에 대해 이런 작업이 얼마나 필요한지 예측하고 비용을 계산해 최적의 실행 계획을 찾음

이 때 비용 계산에 필요한 단위 작업의 비용을 코스트 모델이라고 함
MySQL 5.7부터 단위작업 비용 상수를 DBMS 관리자가 조정할 수 있도록 개선
	- server_cost : 인덱스를 찾고 레코드를 비교하고 임시 테이블 처리에 대한 비용 관리
	- engine_cost : 레코드를 가진 데이터 페이지를 가져오는데 필요한 비용 관리

### 실행 계획 확인
EXPLAIN EXTENDED, EXPLAIN PARTITIONS 으로 구분된 명령은 MySQL 8.0 부터 통합
	- EXPLAIN : 테이블 포맷 표시
	- EXPLAIN FORMAT=TREE : 트리 포맷 표시
	- EXPLAIN FORMAT=JSON: JSON 포맷 표시
	- EXPLAIN ANALYZE : 쿼리의 실행 계획과 단계별 소요시간 정보 확인
		- actual time : 실제 소요된 시간
		- rows : 처리한 레코드 건수
		- loops : 반복 횟수

EXPLAIN ANALYZE 명령은 실행 계획만 추출하는 것이 아니라 실제 쿼리를 실행하고 사용된 실행 계획과 소요 시간을 보여줌


### 실행 계획 분석
#### 1. id 컬럼
하나의 SELECT 문장은 아래와 같이 1개 이상의 SUB SELECT 문장을 포함 가능
```
SELECT...
FROM (SELECT .. FROM tb1) tb1, tb2
WHERE tb1.id=tb2.id;
```

이런 문장을 SELECT 키워드 단위로 나눈 것을 단위 SELECT 쿼리라고 함 
실행 계획의 id 컬럼은 단위 SELECT 쿼리의 식별자 값
조인된 경우에는 테이블 개수만큼 실행 계획 레코드가 출력되지만 같은 id 값이 부여됨

#### 2. select_type 컬럼
각 단위 SELECT 쿼리가 어떤 타입의 쿼리인지 표시

1. SIMPLE
UNION이나 서브쿼리를 사용하지 않는 단순한 SELECT 쿼리에서 가장 바깥 쪽의 단위 쿼리
한 SQL 문에서 SIMPLE인 단위 쿼리는 하나만 존재

2. PRIMARY
UNION이나 서브쿼리를 사용하는 SELECT 쿼리에서 가장 바깥 쪽의 단위 쿼리
SIMPLE과 마찬가지로 한 SQL 문에 하나만 존재

3. UNION
UNION으로 결합하는 단위 SELECT 쿼리 중 두 번째 이후 단위 SELECT 쿼리
UNION의 첫 번째 단위 SELECT 쿼리는 DERIVED (임시 테이블)

4. DEPENDENT UNION
UNION이나 UNION ALL로 결합된 단위 쿼리가 외부 쿼리에 의해 영향을 받는 것을 의미 (내부 쿼리가 외부의 값을 참조해서 처리될 때)

5. UNION RESULT
UNION 결과를 담아두는 테이블
MySQL 8.0 부터는 UNION ALL의 경우에는 임시 테이블을 사용하지 않음
실제 쿼리에서 단위 쿼리가 아니기 때문에 id 값 부여되지 않음

6. SUBQUERY
FROM 절 이외에서 사용되는 서브쿼리

7. DEPENDENT SUBQUERY
서브쿼리가 바깥쪽 SELECT 쿼리에서 정의된 컬럼을 사용하는 경우
외부 쿼리가 먼저 수행되어야 하므로 일반 서브쿼리보다 처리 속도가 느린 경우가 많음

8. DERIVED
MySQL 5.5 버전까지는 FROM 절의 서브쿼리가 항상 DERIVED 였지만, 이후 버전은 옵티마이저 옵션에 따라 FROM 절의 서브쿼리가 외부 쿼리와 통합되는 최적화 수행되기도 함
단위 SELECT 쿼리의 실행 결과로 메모리나 디스크에 임시 테이블 생성을 의미

9. DEPENDENT DERIVED
MySQL 8.0 버전부터 사용 가능한 외부 컬럼을 사용한 FROM 절의 서브쿼리
(LATERAL JOIN)

10. UNCACHEABLE SUBQUERY
조건이 똑같은 서브쿼리가 실행될 때 다시 실행하지 않도록 서브쿼리 결과를 내부적인 캐시 공간에 담아둠

캐시를 사용하지 못하는 경우
	- 사용자 변수가 서브쿼리에 사용
	- NOT-DETERMINISTIC 속성의 스토어드 루틴이 서브쿼리 내 사용된 경우
	- UUID(), RAND() 등 결과값이 호출할 때마다 달라지는 함수가 서브쿼리에 사용

11. UNCACHEABLE UNION

12. MATERIALIZED
FROM 절이나 IN(subquery) 형태의 쿼리에 사용된 서브쿼리의 최적화를 위해 사용
MySQL 5.6 버전까지는 레코드마다 IN 쿼리의 서브쿼리가 실행
MySQL 5.7 버전부터는 서브쿼리의 내용을 임시 테이블로 구체화한 뒤, 임시 테이블과 조회 테이블을 조인하는 형태로 최적화


#### 3. table 컬럼
MySQL 서버의 실행계획은 테이블 기준으로 표시
테이블에 별칭이 부여된 경우 별칭이 표시
별도 테이블을 사용하지 않는 경우 NULL
임시 테이블을 사용하는 경우 <derived N> <union M,N>
여기서 N은 단위 SELECT 쿼리의 id 값을 지칭

#### 4. partitions 컬럼
MySQL 5.7 버전까지는 EXPLAIN PARTITION 명령으로 확인
MySQL 8.0 버전부터는 EXPLAIN 명령으로 파티션 관련 실행 계획까지 확인

#### 5. type 컬럼
MySQL 서버가 각 테이블의 레코드를 어떤 방식으로 읽었는지 나타냄

- system
레코드가 1건만 존재하거나 한 건도 존재하지 않는 테이블 참조
InnoDB 스토리지 엔진에서는 나타나지 않음(MyISAM, MEMORY)	

- const
프라이머리 키나 유니크 키 컬럼을 이용하는 WHERE 조건절
반드시 1건을 반환하는 쿼리의 처리 방식	
옵티마이저가 쿼리를 최적화하는 단계에서 쿼리를 먼저 실행해서 통째로 상수화

- eq_ref
여러 테이블이 조인되는 쿼리의 실행 계획
조인에서 처음 읽은 테이블의 컬럼값을 다음 읽을 테이블의 기본키나 유니크키 컬럼의 검색 조건에 사용
유니크 인덱스가 NOT NULL인 경우
다중 컬럼의 기본키 또는 유니크키면 모든 컬럼이 비교조건에 사용되는 경우

- ref
조인의 순서와 상관없이 인덱스의 종류와 관계없이 동등 조건으로 검색
1건의 레코드만 반환된다는 보장이 없어도 됨

-> const, eq_ref, ref 세 가지 접근 방식 모두 인덱스의 분포도가 나쁘지 않는다면 성능상의 문제를 일으키지 않기 때문에 쿼리를 튜닝하는 경우 크게 신경쓰지 않아도 괜찮다고 함

- fulltext
MySQL 서버의 전문 검색 인덱스를 사용해 레코드를 읽는 접근 방식
MATCH (…) AGAINST (…) 구문을 이용하는 경우에 해당하고 전문 검색용 인덱스가 있어야 함
일반 인덱스를 이용하는 방식보다 fulltext 접근 방식이 먼저 처리하지만, 느린 경우가 많으므로 조건별 성능 확인이 필요

- ref_or_null
ref 접근 방식과 같은데 NULL 비교가 추가된 형태

- unique_subquery
WHERE 조건절에서 사용되는 IN (subquery) 형태의 쿼리를 위한 접근 방식
서브쿼리에서 중복되지 않는 유니크한 값만 반환될 때 이 방식을 사용

- index_subquery
서브쿼리 결과에서 중복된 값이 있지만 인덱스를 이용해 서브쿼리 결과의 중복 값을 제거할 수 있을 때 사용

- range
인덱스를 하나의 값이 아니라 범위로 검색
필요한 레코드 양에 따라 차이는 있지만 상당히 빠른 접근 방식

- index_merge
2개 이상의 인덱스로 각각의 검색 결과를 만들고, 그 결과를 병합해서 처리하는 방식
여러 인덱스를 읽어야 하므로 일반적으로 range 접근 방식보다 효율성이 떨어짐
전문 검색 인덱스를 사용하는 쿼리에는 index_merge가 적용되지 않음
처리 결과가 항상 2개 이상의 집합이 되기 떄문에 추가적인 작업이 필요(교집합, 합집합, 중복제거 등)

- index
인덱스를 처음부터 끝까지 읽는 인덱스 풀 스캔을 의미
풀 테이블 스캔 방식과 비교하는 레코드 건수는 같지만 인덱스 크기가 데이터 파일 전체 크기보다 작으므로 더 빠르게 처리됨
range나 const, ref 같은 접근 방법으로 인덱스를 사용하지 못할 때, 인덱스에 포함된 컬럼만으로 처리할 수 있거나 인덱스로 정렬/그루핑 작업이 가능한 경우 사용

- ALL
풀 테이블 스캔을 의미
테이블을 처음부터 끝까지 읽어서 불필요한 레코드를 제거한 뒤 반환
가장 마지막에 선택되는 비효율적인 방식

Read Ahead
풀 테이블 스캔이나 인덱스 풀 스캔 같은 대량의 디스크 I/O를 유발하는 작업을 위해 한 번에 여러 페이지를 읽어서 처리하는 기능
데이터 웨어하우스나 배치 프로그램처럼 대용량의 레코드를 처리하는 쿼리에서 잘못 튜닝된 쿼리보다 더 나은 방식

-> index와 ALL 접근 방식은 작업 범위를 제한하지 않아 빠른 응답을 할 수 없음
테이블이 매우 작은 경우가 아니라면 데이터를 미리 저장해서 쿼리의 성능 확인이 필요

#### 6. possible_keys 컬럼
옵티마이저에서 실행계획을 만들기 위한 인덱스의 사용후보

#### 7. key 컬럼
실행계획에서 최종 선택된 인덱스
의도했던 인덱스가 표시되는지 확인이 필요 

#### 8. key_len 컬럼
중요한 정보 중 하나로, 쿼리를 처리하기 위해 인덱스의 각 레코드에서 몇 바이트까지 사용했는지 알려주는 값
CHAR(4) 인덱스의 경우 16이 표시 (4*4 byte)
CHAR(4) 인덱스와 INTEGER 타입의 컬럼을 조건에 동시에 사용(AND)한 경우 20이 표시
CHAR(4)(16 byte) + INTEGER(4byte) = 20

#### 9. ref 컬럼
접근 방식이 ref인 경우 참조 조건으로 어떤 값이 제공되었는지 표시(cost 또는 테이블명+컬럼명)
ref 컬럼 값이 func가 표시되는 경우 참조되는 값이 콜레이션 변환이나 값 자체 연산을 거쳐 참조된 것으로 확인이 필요 (조인시 문자집합 불일치 or 연산, or 타입 불일치)

#### 10. rows 컬럼
옵티마이저가 실행 계획의 효율성 판단을 위해 통계 정보를 참조해 산출해낸 예상 값
쿼리를 처리하기 위해 읽고 체크해야하는 레코드 수를 예측한 값

#### 11. filtered 컬럼
rows 컬럼의 값은 인덱스를 사용하는 조건에만 일치하는 레코드 건수를 예측
하지만 쿼리에서 모든 조건이 인덱스를 사용할 수 있는 것은 아니므로, 인덱스를 사용하지 못하는 조건에 일치하는 레코드 건수를 파악이 필요


#### 12. Extra 컬럼
주로 내부적인 처리 알고리즘에 대한 깊이 있는 내용으로 성능에 관련된 중요 내용도 포함되는 경우가 있음

##### using where
MySQL 엔진에서 스토리지 엔진으로부터 받은 레코드를 가공 또는 연산하는 작업을 수행한 경우

##### using filesort
쿼리에서 정렬을 수행할 때 사용할만한 인덱스를 찾지 못한 경우 내부 소트버퍼에서 정렬을 수행한 것
order by null을 통해 정렬을 수행하지 않도록 변경하거나
인덱스를 추가해 인덱스를 이용하여 정렬을 수행하도록 변경


## 쿼리 최적화
### SQL 모드(sql_mode)
#### MySQL 8.0 기본값
- ONLY_FULL_GROUP_BY (8.0 이전 버전에서 업데이트시 주의)
- STRICT_TRANS_TABLES
- NO_ZERO_IN_DATE
- NO_ZERO_DATE
- ERROR_FOR_DIVISION_BY_ZERO
- NO_ENGINE_SUBSTITION

#### sql_mode 종류
STRICT_ALL_TABLES & STRICT_TRANS_TABLES
MySQL 서버에서 INSERT나 UPDATE 문의 자동 타입 변경 수행시 적절히 변환되기 어려운 경우 쓰기작업을 계속 수행할지 에러 발생시킬지를 결정
설정시 Strict Mode 적용

ANSI_QUOTES
MySQL에서는 문자열 값(리터럴) 표현에서 홑따옴표와 쌍따옴표 동시 사용이 가능
보통 쌍따옴표는 컬럼명, 테이블명 등 식별자 구분 용도로 사용
홑 따옴표를 문자열 값 표기로만 사용할지 설정

ONLY_FULL_GROUP_BY
MySQL 8.0 부터 기본으로 활성화 (MySQL 5.7까지는 비활성화)
GROUP_BY에 포함되지 않은 칼럼이 집합 함수의 사용 없이 SELECT나 HAVING 절에 사용되지 못함

PIPE_AS_CONCAT 
|| 를 오라클과 같이 문자열 연결 연산자(CONCAT)로 사용

PAD_CHAR_TO_FULL_LENGTH
MySQL에서는 CHAR 타입이라도 VARCHAR 같이 유효 문자열 뒤 공백 문자 제거하여 반환
설정시 CHAR 타입은 공백문자 제거 X

NO_BACKSLASH_ESCAPES
역슬래시 문자를 이스케이프 문자 용도로 사용하지 않도록 함

IGNORE_SPACE
스토어드 프로시저나 함수명 뒤 공백이 있으면 에러가 출력
IGNORE_SPACE 추가시 프로시저명/함수명과 괄호사이의 공백 무시
MySQL 내장 함수에만 적용되며, 모두 예약어로 간주되어 테이블이나 컬럼명으로 사용 불가 (역따옴표 이용하면 사용 가능)

REAL_AS_FLOAT
부동 소수점 타입 FLOAT와 DOUBLE
기본적으로 REAL 타입은 DOUBLE 타입과 동의어로 사용
설정시 REAL 타입을 FLOAT 타입과  동의어로 사용

NO_ZERO_IN_DATE & NO_ZERO_DATE
설정시 2022-00-00 등의 잘못된 날짜 저장 불가

ANSI
앞선 여러 옵션을 조합해서 최대한 SQL 표준에 맞게 동작하도록 변경

TRADITIONAL
STRICT_TRANS_TABLES와 비슷하지만 조금 더 엄격하게 SQL의 작동을 제어 
설정시 경고로 처리되던 상황도 모두 에러로 바뀌며 SQL 문 실패


## 쿼리 작성 및 최적화

### 쿼리 실행 순서
where/join -> group by -> distinct -> having -> order by -> limit

group by 절이 없는 경우 예외적인 순서로 실행될 수 있음
where -> order by -> join -> limit

다른 순서로 실행시키기 위해서는
서브쿼리로 작성된 인라인 뷰(Inline View) 이용

### WHERE 절의 인덱스 사용
OR 조건에서 인덱스를 타지 않는 조건과 인덱스를 타는 조건이 있는 경우, 옵티마이저는 풀 테이블 스캔을 선택
-> 풀 테이블 스캔 + 인덱스 레인지 스캔 보다는 풀 테이블 스캔 한 번이 더 빠르기 때문

여러 조건이 다 인덱스를 사용할 수 있더라도 인덱스 하나를 레인지 스캔하는 것보다 느림

### GROUP BY 절의 인덱스 사용
GROUP BY 절에 명시된 컬럼이 하나라도 인덱스에 없으면 인덱스를 타지 못함

인덱스가 (COL_1, COL2, COL3) 이렇게 걸려 있는 경우

GROUP BY COL1 (사용가능)
GROUP BY COL1, COL2 (사용가능)
GROUP BY COL1, COL2, COL3 (사용가능)

GROUP BY COL2, COL1 (사용불가)
GROUP BY COL1, COL3 (사용불가)
GROUP BY COL1, COL2, COL3, COL4 (사용불가)

### ORDER BY 절의 인덱스 사용
GROUP BY 조건과 대체로 비슷함
내림차순과 오름차순이 인덱스와 같거나 아예 정반대의 경우만 사용 가능

### WHERE 절과 ORDER BY (GROUP BY) 절 인덱스 사용
WHERE 절과 ORDER BY 절이 같은 인덱스 사용 -> 이상적
WHERE 절만 인덱스 사용 -> 조건에 일치하는 레코드가 적을 때 유리
ORDER BY 절만 인덱스 이용 -> 정렬할 레코드가 아주 많을 때 유리

### GROUP BY 절과 ORDER BY 절의 인덱스 사용
하나의 인덱스를 사용하려면 컬럼의 순서와 내용이 모두 같아야 함
둘 중 하나라도 인덱스를 사용하지 못하는 경우 둘 다 인덱스를 사용하지 못함
MySQL 5.7 까지는 GROUP BY에서 GROUP BY에 대한 ORDER BY 까지 같이 수행
MySQL 8.0 부터는 명시하지 않는다면 GROUP BY 절이 정렬을 보장하지 않음

### JOIN 순서와 인덱스 사용
인덱스 레인지 스캔 = 인덱스 탐색(Index Seek) -> 인덱스 스캔(Index Scan)
인덱스 탐색 작업이 상대적으로 부하가 높음
드라이빙 테이블 : 인덱스 탐색 작업 1번 수행 이후 스캔
드리븐 테이블 : 인덱스 탐색 + 스캔을 드라이빙 테이블의 레코드 건수만큼 반복
1대1 조인이더라도 드리븐 테이블의 읽는 것이 부하가 더 큼

## 파티션
데이터와 인덱스를 조각화하여 물리적 메모리를 효율적으로 사용
MySQL에서는 테이블의 파티션 단위로 다른 인덱스 생성은 지원하지 않음
모든 인덱스는 파티션 단위로 생성
이력 데이터의 경우 파티션 테이블로 관리하면 불필요한 데이터의 삭제 작업의 부하를 줄일 수 있음

- 파티션 선택 가능 + 인덱스 사용 : 선택된 파티션의 인덱스만 레인지 스캔
- 파티션 선택 불가 + 인덱스 사용 : 각 파티션에 대해 인덱스 레인지 스캔하여 병합
- 파티션 선택 가능 + 인덱스 미사용 : 대상 파티션의 풀 테이블 스캔
- 파티션 선택 불가 + 인덱스 미사용 : 각 파티션에 대한 풀 테이블 스캔 수행

### MySQL 파티션 제약사항
모든 인덱스는 파티션 키 컬럼을 포함해야 함
유니크 인덱스는 그 값으로 어떤 파티션에 저장되었는지 알 수 있어야 함
```
CREATE TABLE table1 (
…
UNIQUE KEY (col1, col2)
…
) PARTITION BY HASH(col2)
PARTITIONS 4;
```


### MySQL 파티션 종류

#### 레인지 파티션
파티션 키의 연속된 범위로 파티션을 정의
날짜 기반, 범위 기반, 어떤 키로 자주 검색 실행
필요한 파티션만 접속할 때 많이 사용
`PARTITION BY RANGE` 키워드로 정의
가장 오래된 파티션만 삭제 가능

#### 리스트 파티션
레인지 파티션과 비슷하지만 값의 범위로 구성할 수 없는 경우 사용
파티션의 키 값이 코드나 카테고리 처럼 고정적
키 값이 연속되지 않고 정렬 순서와 무관
키 값을 기준으로 레코드 건수가 균일하고, 파티션 키가 검색 조건에 많이 사용
`PARTITION BY LIST` 키워드로 정의

#### 해시 파티션
해시 함수로 레코드가 저장될 파티션 결정
파티션 키로 정수 타입의 컬럼 또는 정수를 반환하는 표현식만 사용가능
파티션 추가/삭제에 테이블 전체 레코드의 재분배가 필요
레인지, 리스트 파티션을 통해 균등하게 나눌 수 없을 때
모든 레코드가 비슷한 사용빈도를 보이지만 테이블이 너무 커서 나눠야할 때
`PARTITION BY HASH` 키워드로 정의
파티션 키 또는 표현식이 반드시 정수 타입의 값을 반환
특정 파티션의 삭제가 불가
추가시 모든 테이블의 재배치 작업이 필요


#### 키 파티션
해시 파티션과 사용법과 특성이 거의 같음
해시 파티션은 해시 값의 계산을 사용자가 명시
키 파티션은 해시 값의 계산도 MySQL 서버에서 수행(MD5)
컬럼의 데이터 타입과 거의 상관없이 파티션 키 적용 가능
`PARTITION BY KEY` 키워드로 정의


#### 리니어 해시/키 파티션
Power-of-two 알고리즘으로 레코드가 분배되어 있어 새로운 파티션 추가시 특정 파티션 레코드만 재분배


## 데이터 타입
### 문자열 타입
#### CHAR
고정 길이이므로 저장 공간의 크기가 고정적으로 유효 크기 저장을 위한 추가 공간이 필요하지 않음

#### VARCHAR
가변길이로, 최대 길이 이하의 값이 저장될 때 저장공간이 줄어들지만, 유효 크기의 저장 때문에 1~2 바이트의 저장공간 별도로 필요
유효 크기는 2바이트 이내로 표현되어야 하므로 최대 길이는 65,536 바이트 이상으로 설정 불가
VARCHAR(10) 일 때 10은 10자리 문자열을 의미 (문자 타입에 따라 10 Byte ~ 40 Byte)

VARCHAR 타입의 경우 UPDATE할 때 길이가 더 큰 값으로 변경될 때는 레코드 자체를 다른 공간으로 옮겨서 저장
따라서 길이가 엇비슷한 데이터가 자주 변경되는 경우에는 CHAR 타입의 사용이 좋음 
ex) 부서번호, 게시글 상태 등

VARCHAR 타입의 컬럼 길이를 변경할 때 50 -> 63인 경우에는 잠금없이 빠르게 변경이 가능하지만 63 -> 64의 경우 읽기 잠금이 필요
원인 : VARCHAR(50)은 200(50*4) 바이트의 최대 길이를 갖기 때문에 문자열 길이를 저장하는 공간이 1바이트였지만, VARCHAR(64) 부터는 최대 길이가 256바이트 까지 가능해지기 때문에 문자열 길이 저장 공간이 2바이트로 늘어남

#### 콜레이션(Collation)
문자열 컬럼 값의 비교나 정렬 순서를 위한 규칙
한 문자 집합은 2개 이상의 콜레이션을 가짐

#### 문자열 비교
MySQL에서 문자열 컬럼의 비교는 CHAR와 VARCHAR가 거의 같음
기본적으로 다른 DBMS처럼 사용되지 않는 공간에 공백문자는 채워지지 않음
대부분의 콜레이션에서는 비교할 때 공백문자를 뒤로 붙여서 문자열의 길이를 동일하게 만든 뒤, 비교를 수행
SELECT ‘ABC’ = ‘ABC  ‘ # 1, TRUE
SELECT ‘   ABC’ = ‘ABC ‘ # 0, FALSE

하지만 COLLATIONS 테이블의 pad_attribute 컬럼의 값에 따라 문자열 비교에서 공백을 추가해서 시키지 않는 콜레이션도 존재함
-> 문자열 길이 차이가 큰 경우 더 빠른 비교 성능을 보임 

예외적으로 LIKE 조건의 비교에서는 공백이 유효 문자로 취급

### 숫자

값의 정확도에 따른 분류
- 참값 (Exact value) : INTEGER, …INT, DECIMAL
- 근삿값 :부동 소수점 (FLOAT, DOUBLE)

값이 저장되는 포맷에 따른 분류
- 십진 표기법 (DECIMAL) : 숫자 값의 한 자리당 4비트 또는 1바이트를 이용해 표기
- 이진 표기법 : 대부분의 숫자 타입 한바이트로 256까지의 숫자를 표현

근삿값은 저장시와 조회시 값이 정확히 일치하지 않음
또한 유효 자릿수를 넘어서는 소수점 이하 값은 계속 바뀔 수 있음
바이너리 로그 포맷이 STATEMENT인 경우 소스 서버와 레플리카 서버의 데이터가 달라질 수 있음

DECIMAL은 정확하지만 저장공간을 2배 이상 필요로 함
BIGINT와 비교했을 때 미세한 차이지만 BIGINT가 더 빠르므로 정수값은 INTEGER나 BIGINT 사용이 좋음


### 날짜
YEAR 1byte
DATE 4byte
TIME 3byte + milli
DATETIME 5byte + milli
TIMESTAMP 4byte+ milli

NOW(6)
밀리초의 자리수 
NOW() == NOW(0)

DATETIME, DATE는 타임존 정보가 변경되도 조회, 저장시 변경되지 않음
TIMESTAMP는 UTC 타입존으로 저장되므로 타입존이 달라지면 자동으로 값이 보정됨
-> MySQL 타임존 변경시 DATETIME, DATE 값은 타임존에 맞게 UPDATE 필요

어플리케이션에서 타임존을 변경하게되면 JDBC 드라이버가 날짜 및 시간 정보를 MySQL 타임존에서 JVM 타임존으로 변환해서 출력
-> ORM 사용시 사용하는 JDBC API에 따라 테스트 필요 

### ENUM / SET
#### ENUM
테이블 구조에 나열된 목록 중 하나의 값 (코드화된 값)
코드 값 사용 강제 및 저장공간을 절약
내부적으로는 정수 타입의 컬럼이기 때문에 데이터 정렬시 생성한 순서로 정렬
변경, 추가시 테이블 구조(메타데이터) 변경 필요 
순서 변경, 중간에 추가하는 경우 COPY 알고리즘 및 읽기 잠금이 필요

#### SET
ENUM과 저장방식은 비슷하지만 하나의 컬럼에 1개 이상 값을 저장

### TEXT, BLOB
거의 똑같은 설정 및 방식으로 작동
TEXT는 문자열 저장이므로 문자 집합과 콜레이션을 가짐
BLOB은 이진 데이터 타입이라서 별도의 문자 집합이나 콜레이션을 가지지 않음

### JSON
MySQL 5.7부터 지원
BLOB 타입에 바이너리 포맷인 BSON 타입으로 변환해서 저장 -> 공간 효율이 텍스트 타입 컬럼보다는 높은 편

MySQL 8.0 부터는 JSON 타입에 부분 업데이트 기능 제공

### 가상(파생) 컬럼(Generated Column)
가상 컬럼(Virtual Column)
값이 디스크에 저장 X
구조 변경에 테이블 리빌드 필요 X
값이 레코드 읽기 직전 계산

스토어드 컬럼(Stored Column)
값이 디스크에 저장
구조 변경에 필요시 테이블 리빌드 필요 (일반 테이블과 동일)
INSERT, UPDATE 시점에 값이 계산

### 복제
원본 데이터 : 소스(Source) 서버
복제 데이터 : 레플리카(Replica) 서버
소스 서버에서 데이터 및 스키마 변경이 최초로 발생
레플리카 서버는 변경 내역을 전달받아 데이터에 반영 -> 동기화

#### 복제 아키텍처 
MySQL 서버에서 발생하는 모든 변경사항 -> 바이너리 로그(Binary Log)
바이너리 로그의 각 변경 정보 -> 이벤트(Event)
레플리카 서버에서 바이너리 로그를 읽어 로컬 디스크에 저장한 파일 -> 릴레이 로그(Relay 로그)

- 바이너리 로그 덤프 스레드
: 바이너리 로그 내용을 레플리카 서버로 전송하는 스레드
일시적으로 바이너리 로그에 잠금을 수행

- 레플리케이션 I/O 스레드
레플리카 서버에서 복제 시작할때 생성
바이너리 로그를 릴레이 로그로 저장하는 역할

- 레플리케이션 SQL 스레드
릴레이 로그의 이벤트를 읽고 실행하는 역할

#### 복제 타입
바이너리 로그의 변경 내역을 식별하는 방식에 따른 구분
 
- 바이너리 로그 파일 위치 기반 복제
처음 도입됬을 때부터 제공
레플리카 서버에서 바이너리 로그 파일명과 파일 내 위치(Offset)로 식별
복제에 참여한 MySQL 서버가 모두 고유한 server_id 값을 갖고 있어야 함
(자신이 발생시킨 이벤트인지 구분하기 위함)

- 글로벌 트랜잭션 ID 기반 복제
MySQL 5.6에서 도입
바이너리 로그명과 파일 내 위치값의 조합이 꼭 같은 이벤트일거라는 보장이 없음
일부 서버에 장애가 발생했을 때 서버끼리 호환되지 않는 정보로 인해 복구가 불가한 경우가 생기기도 함 
각 이벤트가 복제에 참여한 MySQL 서버들에게서 동일한 고유 식별 값을 갖게 하는 것
GTID = [소스 서버 식별 값] : [ 트랜잭션 아이디] 로 구성
중간에 소스 서버에 장애가 발생하여 동기화되기 전 바이너리 로그 위치를 잃어버린 상황에서도 동기화가 가능 (동일한 식별 값의 GTID가 있으므로 동기화 되지 못한 이벤트를 알 수 있음)


## Performance 스키마
MySQL 서버에서 기본적으로 제공하는 시스템 데이터베이스 중 하나
MySQL 서버 내부 동작이나 쿼리 처리에 관련된 세부 정보들이 저장되는 테이블이 존재
MySQL 서버의 성능 분석 및 내부 처리 과정을 모니터링 하는데 사용

## Sys 스키마
Performance 스키마에서 어떤 정보가 어디에 저장되는지 알기 어려워 사용성을 개선하기 위해 도입
더 쉽게 이해할 수 있는 형태의 뷰와 스토어드 프로시저, 함수들이 제공

