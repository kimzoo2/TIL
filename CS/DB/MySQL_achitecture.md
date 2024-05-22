# MYSQL 아키텍처

# MYSQL
## MySQL 아키텍처
- MySQL 엔진
- 스토리지 엔진
- 핸들러 API

### MySQL 엔진 (MySQL 서버의 두뇌)
클라이언트로부터 **접속 및 쿼리 요청 처리, 쿼리 최적화** 담당<br/>
커넥션 핸들러, SQL 파서, 전처리기, 옵티마이저 중심

### 스토리지 엔진 (MySQL 서버의 손발)
실제 **데이터를 디스크에 저장하거나 읽기** 담당<br/>
스토리지 엔진은 여러개를 사용할 수 있음<br/>
**InnoDB, MyISAM, Memory** 등 있으며 성능 향상을 위해 버퍼풀과 키캐시 기능 등을 갖고 있다.

### 핸들러 API
**MySQL 엔진**과 **스토리지 엔진**이 서로 데이터를 주고 받는 요청 API

**핸들러 API 통한 데이터 요청 조회 방법**
```sql
show global status like 'Handler%';
```
```
+----------------------------+---------+
| Variable_name              | Value   |
+----------------------------+---------+
| Handler_commit             | 3376    |
| Handler_delete             | 8       |
| Handler_discover           | 0       |
| Handler_external_lock      | 18779   |
| Handler_mrr_init           | 0       |
| Handler_prepare            | 614     |
| Handler_read_first         | 55      |
| Handler_read_key           | 6103    |
| Handler_read_last          | 0       |
| Handler_read_next          | 7436    |
| Handler_read_prev          | 0       |
| Handler_read_rnd           | 0       |
| Handler_read_rnd_next      | 9816    |
| Handler_rollback           | 0       |
| Handler_savepoint          | 0       |
| Handler_savepoint_rollback | 0       |
| Handler_update             | 582     |
| Handler_write              | 4819742 |
+----------------------------+---------+
18 rows in set (0.03 sec)
```
-> MySQL 엔진이 각 스토리지 엔진에게 보낸 명령의 횟수를 의미하는 변수들

### MySQL 스레딩
MySQL은 프로세스 기반이 아닌 **스레딩 기반** 작동<br/>
**포그라운드 스레드**와 **백그라운드 스레드**로 구분하여 실행

```sql
select thread_id, name, type, processlist_user, processlist_host
  from performance_schema.threads order by type, thread_id;
```
```
+-----------+---------------------------------------------+------------+------------------+------------------+
| thread_id | name                                        | type       | processlist_user | processlist_host |
+-----------+---------------------------------------------+------------+------------------+------------------+
|         1 | thread/sql/main                             | BACKGROUND | NULL             | NULL             |
|         2 | thread/mysys/thread_timer_notifier          | BACKGROUND | NULL             | NULL             |
|         4 | thread/innodb/io_ibuf_thread                | BACKGROUND | NULL             | NULL             |
|         5 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
|         6 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
|         7 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
|         8 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
|         9 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
|        10 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
|        11 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
|        12 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
|        13 | thread/innodb/page_flush_coordinator_thread | BACKGROUND | NULL             | NULL             |
|        14 | thread/innodb/log_checkpointer_thread       | BACKGROUND | NULL             | NULL             |
|        15 | thread/innodb/log_flush_notifier_thread     | BACKGROUND | NULL             | NULL             |
|        16 | thread/innodb/log_flusher_thread            | BACKGROUND | NULL             | NULL             |
|        17 | thread/innodb/log_write_notifier_thread     | BACKGROUND | NULL             | NULL             |
|        18 | thread/innodb/log_writer_thread             | BACKGROUND | NULL             | NULL             |
|        19 | thread/innodb/log_files_governor_thread     | BACKGROUND | NULL             | NULL             |
|        24 | thread/innodb/srv_lock_timeout_thread       | BACKGROUND | NULL             | NULL             |
|        25 | thread/innodb/srv_error_monitor_thread      | BACKGROUND | NULL             | NULL             |
|        26 | thread/innodb/srv_monitor_thread            | BACKGROUND | NULL             | NULL             |
|        27 | thread/innodb/buf_resize_thread             | BACKGROUND | NULL             | NULL             |
|        28 | thread/innodb/srv_master_thread             | BACKGROUND | NULL             | NULL             |
|        29 | thread/innodb/dict_stats_thread             | BACKGROUND | NULL             | NULL             |
|        30 | thread/innodb/fts_optimize_thread           | BACKGROUND | NULL             | NULL             |
|        31 | thread/mysqlx/worker                        | BACKGROUND | NULL             | NULL             |
|        32 | thread/mysqlx/worker                        | BACKGROUND | NULL             | NULL             |
|        33 | thread/mysqlx/acceptor_network              | BACKGROUND | NULL             | NULL             |
|        37 | thread/innodb/buf_dump_thread               | BACKGROUND | NULL             | NULL             |
|        38 | thread/innodb/clone_gtid_thread             | BACKGROUND | NULL             | NULL             |
|        39 | thread/innodb/srv_purge_thread              | BACKGROUND | NULL             | NULL             |
|        40 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |
|        41 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |
|        42 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |
|        44 | thread/sql/signal_handler                   | BACKGROUND | NULL             | NULL             |
|        45 | thread/mysqlx/acceptor_network              | BACKGROUND | NULL             | NULL             |
|        43 | thread/sql/event_scheduler                  | FOREGROUND | event_scheduler  | localhost        |
|        47 | thread/sql/compress_gtid_table              | FOREGROUND | NULL             | NULL             |
|        69 | thread/sql/one_connection                   | FOREGROUND | root             | localhost        |
+-----------+---------------------------------------------+------------+------------------+------------------+
39 rows in set (0.00 sec)

```
이중 **thread/sql/one_connection** 스레드만 실제 사용자 요청을 처리

#### 스레드 기반이면 스레드 풀은?
커뮤니티 에디션은 스레드풀 X, 커넥션(사용자) 별로 스레드 생성<br/>
-> 엔터프라이즈 버전, percona server의 스레드 풀 기능 사용

#### 포그라운드 스레드(= 클라이언트 스레드)
커넥션 별로 스레드가 생성되기 때문에 클라이언트의 수만큼 존재<br/>
사용자가 요청하는 쿼리 문장을 처리하고 커넥션을 종료하기 전까지 유지 <br />
-> 이후, 커넥션을 종료하면 **스레드 캐시**로 반환됨. 스레드 캐시에 일정 개수 이상 스레드가 있으면 **스레드를 종료**함.

**역할**<br />
MySQL 데이터 버퍼풀이나 캐시로부터 데이터 가져오기<br />
버퍼나 캐시에 없으면 직접 디스크의 데이터나 인덱스 파일로부터 데이터 읽기<br />
MyISAM 스토리지 엔진인 경우, 포그라운드 스레드가 **디스크 쓰기 작업**까지 처리

#### 백그라운드 스레드
- 인서트 버퍼 병합 스레드
- **로그를 디스크로 기록하는 스레드**
- **innoDB 버퍼풀 데이터를 디스크에 기록하는 스레드**
- 데이터를 버퍼로 읽어 오는 스레드
- 잠금이나 데드락을 모니터링하는 스레드


### 메모리 할당 및 구조
- 글로벌 메모리 영역
- 로컬 메모리 영역<br />
-> 차이 :공유 여부

#### 글로벌 메모리 영역
MySQL 서버가 시작될 때 운영체제로부터 할당되는 하나의 메모리 공간, 모든 스레드에 의해 **공유**됨
- innoDB 버퍼풀
- MyISAM 키 캐시
- 바이너리 로그 버퍼
- 리두 로그 버퍼
- 테이블 캐시

#### 로컬 메모리 영역(=세션 메모리 영역)
클라이언트 스레드(=포그라운드 스레드)가 쿼리를 처리할 때 사용하는 메모리 영역<br />
클라이언트 스레드 별로 독립적으로 할당되며 클라이언트 스레드가 사용하는 메모리 공간 **(공유X)**<br />
쿼리 용도에 따라 필요하지 않으면 비할당 가능 ex) 소트 버퍼, 조인 버퍼


### 플러그인 스토리지 엔진 모델과 컴포넌트
#### 플러그인이란?
플러그인 : 추가 기능을 설치하기 위한 확장 프로그램이나 소프트웨어

MySQL의 독특한 구조 중 하나<br />
대표적으로 스토리지 엔진이 있고 파서도 플러그인 형태로 개발 가능<br />

**장점**
- 다른 라이브러리를 다운 받아 끼워 넣어 사용 가능
- 손쉽게 업그레이드 가능

**단점**
- 플러그인 간 통신 불가, 플러그인은 오직 MySQL 서버와 인터페이스 가능
- 플러그인은 MySQL 변수나 함수를 직접 호출 (캡슐화 X)
- 초기화가 어렵다.

MySQL 8.0부터는 플러그인의 단점을 해소하고자 **컴포넌트 아키텍처** 지원.



### 쿼리 실행 구조
![mysql_architecture.png](..%2F..%2Fimg%2Fdb%2Fmysql_architecture.png)

client SQL 요청 -> MySQL 엔진 -> 스토리지 엔진 -> MySQL 엔진이 SQL 결과 전달<br />

**!중요**
쿼리 작업은 여러 하위 작업으로 분리됨 각 하위 작업이 **MySQL 엔진 영역**에서 처리되는지, **스토리지 엔진 영역**에서 처리되는지 구분할 줄 알아야 함.

#### 쿼리 파서
SQL 쿼리를 토큰으로 분리해 트리 형태의 구조(=파서 트리) 생성<br />
기본 문법 오류 검증

#### 전처리기
파서 트리를 기반으로 쿼리의 구조적 문제점(해당 객체의 존재 여부, 접근 권환) 확인<br />
존재하지 않거나 권한상 사용할 수 없는 개체의 토큰 오류

#### 옵티마이저
쿼리를 최적화하고 실행 계획 수립

#### 실행 엔진
실행 계획을 기반으로 각 핸들러(=스토리지 엔진) 에게 요청해서 받은 결과를 다른 핸들러 요청의 입력으로 연결
**GROUP BY**나 **ORDER BY** 등의 복잡한 쿼리는 실행 엔진에서 처리

ex) group by를 처리하기 위해 임시 테이블을 사용하는 경우
1. 실행 엔진 -> 핸들러 : create 임시 테이블
2. 실행 엔진 -> 핸들러 : read WHERE절
3. 실행 엔진 -> 핸들러 : save 레코드를 임시 테이블에
4. 실행 엔진 -> 핸들러 : read 임시 테이블에서 필요한 방식(group by)
5. 실행 엔진 -> 유저, 다른 모듈 : return 결과

#### 핸들러(=스토리지 엔진)
실행 엔진의 요청에 따라 데이터를 **디스크에 저장**하거나 **디스크로부터 읽는 역할**을 담당

#### 쿼리 캐시 (8.0부터 삭제)
실행 쿼리와 결과값을 **캐시**하는 공간<br />
하지만 변경이 생기면 변경된 테이블과 관련된 것들을 모두 삭제해야하기 때문에 8.0부터는 **성능상의 이슈로 제거**

#### 스레드 풀
스레드풀 X<br />
스레드 당 하나의 커넥션 <br/>
[스레드 기반이면 스레드 풀은?](####스레드 기반이면 스레드 풀은?)

스레드풀 O
하나의 스레드가 여러개의 커넥션을 전담


[mysql dev blog](https://dev.mysql.com/blog-archive/the-new-mysql-thread-pool/)



#### 트랜잭션 지원 메타데이터
메타데이터 or 데이터 딕셔너리 : 테이블의 구조 정보와 스토어드 프로그램 등의 정보
~ 5.7ver : 파일 기반 관리
8.0ver : innodb 테이블에 시스템 테이블(MySQL 서버가 작동할 때 기본적으로 필요한 테이블들)과 함께 저장

데이터 딕셔너리 데이터 조회 방법
```sql
select * from information_schema.tables; -- mysql.tables는 임의 수정 불가하도록 접근 불가함
```

**왜 innodb 스토리지 엔진 기반으로 변경했을까?**

**파일 기반**이었기 때문이다. 즉, 트랜잭션을 지원하지 않아서 테이블 생성 및 변경 중 서버가 종료되면 데이터가 일관되지 않는 문제가 발생했다.
InnoDB 외의 MyISAM이나 CSV 같은 스토리지 엔진의 메타 정보는 여전히 파일(.SDI) 기반으로 저장된다. innodb 테이블들의 구조도 원하면 SDI파일로 변환할 수 있다.



## 스토리지 엔진
- 디스크에 접근하여 **데이터를 읽고 데이터를 쓰는 엔진**이다. 대표적으료 innoDB, MyISAM, Memory 등이 있다.

## InnoDB
![innodb-architecture-8-0.png](..%2F..%2Fimg%2Fdb%2Finnodb-architecture-8-0.png)
### InnoDB 특징
- **프라이머리 키에 의한 클러스터링**
- 외래키 지원
- **MVCC (Multi Version Concurrency Control)**
- **잠금 없는 일관된 읽기**
- 자동 데드락 감지
- 자동화된 장애 복구
- **버퍼 풀**
- **double write buffer**
- **언두 테이블스페이스 관리**
- **체인지 버퍼**
- **리두 로그 및 로그 버퍼**
- **어댑티브 해시 인덱스**

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


## MyISAM
- 키 캐시
- 운영체제의 캐시 및 버퍼
- 데이터 파일과 프라이머리 키 구조

## MySQL 로그 파일
- 에러 로그 파일
- 제너럴 쿼리 로그 파일
- 슬로우 쿼리 로그

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
