# MYSQL 아키텍처

# MYSQL
- 프로그래밍 언어로 MYSQL에 접근할 수 있는 거?
- MYSQL 엔진(두뇌)
  - 데이터베이스에 접근하고 실행하는 엔진
  - cache (8.0부터 성능 이슈로 제거 됨)
  - parser
  - optimizer
  - preprocessor
- 핸들러 API
- 스토리지 엔진(=핸들러, 손발) ex) innoDB, MyISAM, memory
- 디스크를 읽고 쓰는 주체
  - 인덱스 클러스터링
  - MVCC
    - 잠금 없는 읽기
    - isolation level
    - 언두로그
  - 버퍼풀, 어댑티드 해시 인덱스?
    - 버퍼풀과 리두로그
  - double write buffer
- 운영체제 등

## MYSQL 엔진
- 쿼리를 실행하고 최적화하는 엔진

### 쿼리 캐시
- 실행 쿼리와 결과값을 캐싱하는 공간이다. 하지만 변경이 생기면 관련 데이터들을 모두 삭제해야하기 때문에 8.0부터는 성능상의 이슈로 제거되었다.


## 스토리지 엔진
- 디스크에 접근하여 **데이터를 읽고 데이터를 쓰는 엔진**이다. 대표적으료 innoDB, MyISAM, Memory 등이 있다.

## InnoDB
![innodb-architecture-8-0.png](..%2F..%2Fimg%2Fdb%2Finnodb-architecture-8-0.png)
### InnoDB 특징
- 프라이머리 키에 의한 클러스터링
- 외래키 지원
- 레코드 락
- MVCC (Multi Version Concurrency Control)
- 잠금 없는 일관된 읽기
- 자동 데드락 감지
- 자동화된 장애 복구
- 버퍼 풀
- double write buffer
- 언두 로그
- 체인지 버퍼
- 리두 로그

### 프라이머리 키에 의한 클러스터링
- 프라이머리 키 기준으로 클러스터링되어 저장한다. 즉, **프라이머리 키 값이 정렬되어 디스크에 저장**된다.
- 이는 **프라이머리 키를 이용한 인덱스 레인지 스캔을 가능**하게 한다. 원하는 페이지의 위치를 빠르게 찾을 수 있게 된다.
  - 참고 :MyISAM은 논클러스터링이다.

### 외래키 지원
### MVCC (Multi Version Concurrency Control)
- 기존 locking이 **속도가 저하되는 문제**가 있어서 **lock을 사용하기 않기 위해** 제공하는 기능이다.
- **하나의 레코드에 대한 여러개의 버전을 관리**하여 잠금을 사용하지 않는 일관된 읽기(MVCC 목적)를 제공한다.

#### MVCC 예시
- data insert
```sql
insert into member (id, name, area) values (20, '김주희', '서울');
commit;
```
- database status
![mvcc_insert.png](..%2F..%2Fimg%2Fdb%2Fmvcc_insert.png)
- 메모리의 버퍼풀과 디스크에만 데이터가 저장된다.

- data update
```sql
update member set area='제주' where id=20;
```
- database status
![mvcc_update.png](..%2F..%2Fimg%2Fdb%2Fmvcc_update.png)
- 위의 데이터가 아직 commit 되기 전이라면? 다른 사용자가 id=20을 조회했을 때 어떤 데이터가 조회될까?
- 이는, **isolation level**(격리 수준)에 따라 다르다.
![mvcc_isolationlevel.png](..%2F..%2Fimg%2Fdb%2Fmvcc_isolationlevel.png)

#### commit 후엔?
- 언두로그는 유지될까?

### 잠금 없는 일관된 읽기
- innoDB는 MVCC를 이용하여 다른 트랜잭션이 변경 시에도 lock을 기다리지 않고 읽기 작업을 수행할 수 있게 되었다.
- 격리 수준이 serializabel이 아니라면 insert과 연결되지 않은 읽기(SELECT) 작업은 잠금을 대기하지 않고 수행한다.


## 단어 모음집
- MySQL서버
  - MySQL 엔진, 스토리지 엔진
  - 핸들러 API
  - MySQL 스레딩 구조
    - 포그라운드 스레드 (=클라이언트 스레드)
    - 백그라운드 스레드
      - 로그 스레드
      - 쓰기 스레드
    - 스레드 풀
  - 메모리 할당 및 사용 구조
    - 글로벌 메모리 영역
      - 버퍼 풀
      - 어댑티브 해시 인덱스
      - 리두 로그 버퍼
      - 테이블 캐시
    - 로컬 메모리 영역
      - 정렬 버퍼
      - 조인 버퍼
      - 바니어리 로그 캐시
      - 네트워크 버퍼
  - 플러그인 스토리지 엔진 모델
  - 컴포넌트
- MySQL 엔진
  - 쿼리 캐시
  - SQL 파서
  - 전처리기
  - 옵티마이저
  - 실행 엔진
- 스토리지 엔진
  - 클러스터링 인덱스
  - 레코드락
  - 외래키 지원
  - MVCC(Multi value Concurrency Control)
    - 격리 수준 (Isolation level)
  - 자동 데드락 감지
  - 자동화된 장애 복구
  - 버퍼 풀
    - 레코드 버퍼
    - LRU 리스트, 플러시 리스트, 프리 리스트
    - 더티 페이지, 클린 페이지, LSN(Long Sequence Number), 채크포인트 에이지
    - 버퍼 풀 플러시
      - 플러시 리스트 플러시
      - LRU 리스트 플러시
  - double write buffer
  - 언두 로그
    - 언두 테이블스페이스
    - 롤백 세그먼트
    - 언두 슬롯
  - 체인지 버퍼
    - 체인지 버퍼 머지 스레드
  - 리두 로그
    - 로그 버퍼


참고
- Real MySQL 8.0
- [mysql 8.0 docs](https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html)
