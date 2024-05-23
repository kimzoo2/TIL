# MYSQL 아키텍처

# MySQL 아키텍처
- MySQL 엔진
- 스토리지 엔진
- 핸들러 API
- MySQL 스레딩
- 메모리 할당 및 구조
- 플러그인 스토리지 엔진 모델과 컴포넌트
- 쿼리 실행 구조
<br/>
<br/>


## MySQL 엔진 (MySQL 서버의 두뇌)
클라이언트로부터 **접속 및 쿼리 요청 처리, 쿼리 최적화** 담당<br/>
커넥션 핸들러, SQL 파서, 전처리기, 옵티마이저 중심

<br/>
<br/>

## 스토리지 엔진 (MySQL 서버의 손발)
실제 **데이터를 디스크에 저장하거나 읽기** 담당<br/>
**InnoDB, MyISAM, Memory** 등 있으며 성능 향상을 위해 버퍼풀(InnoDB)과 키캐시(MyISAM) 기능 등을 갖고 있음<br />
스토리지 엔진은 여러개를 사용할 수 있음<br/>

<br/>
<br/>

## 핸들러 API
**MySQL 엔진**과 **스토리지 엔진**이 서로 데이터를 주고 받는 요청 API
<br/>
<br/>

**핸들러 API 통한 데이터 요청 건수 조회 방법**
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

<br/>
<br/>

## MySQL 스레딩
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

<br/>
<br/>

### 스레드 기반이면 스레드 풀은?
커뮤니티 에디션은 스레드풀 X, 커넥션(사용자) 별로 스레드 생성<br/>
-> percona server(플러그인 형태) or 엔터프라이즈 버전 스레드 풀 기능 사용,

<br/>
<br/>

### 포그라운드 스레드(= 클라이언트 스레드)
커넥션 별로 스레드가 생성되기 때문에 클라이언트의 수만큼 스레드 존재<br/>
사용자가 요청하는 쿼리 문장을 처리하고 커넥션을 종료하기 전까지 유지 <br />
-> 이후, 커넥션을 종료하면 **스레드 캐시**로 반환됨. 스레드 캐시에 일정 개수 이상 스레드가 있으면 **스레드를 종료**함.


**역할**<br />
MySQL 데이터 버퍼이나 캐시로부터 데이터 가져오기<br />
버퍼나 캐시에 없으면 직접 디스크의 데이터나 인덱스 파일로부터 데이터 읽기<br />
MyISAM 스토리지 엔진인 경우, 포그라운드 스레드가 **디스크 쓰기 작업**까지 처리

<br/>
<br/>

### 백그라운드 스레드
- 인서트 버퍼 병합 스레드
- **로그를 디스크로 기록하는 스레드**
- **innoDB 버퍼풀 데이터를 디스크에 기록하는 스레드**
- 데이터를 버퍼로 읽어 오는 스레드
- 잠금이나 데드락을 모니터링하는 스레드

<br/>
<br/>

## 메모리 할당 및 구조
- 글로벌 메모리 영역
- 로컬 메모리 영역<br />
-> 차이 : **공유 여부**

### 글로벌 메모리 영역
MySQL 서버가 시작될 때 운영체제로부터 할당되는 하나의 메모리 공간, 모든 스레드에 의해 **공유**됨
- innoDB 버퍼풀
- MyISAM 키 캐시
- 바이너리 로그 버퍼
- 리두 로그 버퍼
- 테이블 캐시

### 로컬 메모리 영역(=세션 메모리 영역)
클라이언트 스레드(=포그라운드 스레드)가 쿼리를 처리할 때 사용하는 메모리 영역<br />
클라이언트 스레드 별로 독립적으로 할당되며 클라이언트 스레드가 사용하는 메모리 공간 **(공유X)**<br />
쿼리 용도에 따라 필요하지 않으면 비할당 가능 ex) 소트 버퍼, 조인 버퍼

<br/>
<br/>

## 플러그인 스토리지 엔진 모델과 컴포넌트
### 플러그인이란?
플러그인 : 추가 기능을 설치하기 위한 확장 프로그램이나 소프트웨어

MySQL의 독특한 구조 중 하나<br />
대표적으로 스토리지 엔진이 있고 파서도 플러그인 형태로 개발 가능<br />

**장점**
- 다른 라이브러리를 다운 받아 끼워 넣어 사용 가능
- 손쉽게 업그레이드 가능

**단점**
- 플러그인 간 통신 불가, 플러그인은 오직 MySQL 서버와 인터페이스 가능
- 플러그인은 MySQL 변수나 함수를 직접 호출 (캡슐화 X)
- 상호 의존 관계를 설정할 수 없기 때문에 초기화 어려움

MySQL 8.0부터는 플러그인의 단점을 해소하고자 **컴포넌트 아키텍처** 지원

<br/>
<br/>

## 쿼리 실행 구조
![mysql_architecture.png](..%2F..%2Fimg%2Fdb%2Fmysql_architecture.png)

client SQL 요청 -> MySQL 엔진 -> 스토리지 엔진 -> MySQL 엔진이 SQL 결과 전달<br />

쿼리 작업은 여러 하위 작업으로 분리됨 각 하위 작업이 **MySQL 엔진 영역**에서 처리되는지, **스토리지 엔진 영역**에서 처리되는지 구분할 줄 알아야 함. <br />
**왜일까?** 각 엔진 영역 간의 차이를 설명하기 위해

<br/>
<br/>

### 쿼리 파서
SQL 쿼리를 토큰으로 분리해 트리 형태의 구조(=파서 트리) 생성<br />
기본 문법 오류 검증

### 전처리기
파서 트리를 기반으로 쿼리의 구조적 문제점(해당 객체의 존재 여부, 접근 권환) 확인<br />
존재하지 않거나 권한상 사용할 수 없는 개체의 토큰 검증

### 옵티마이저
쿼리를 최적화하고 실행 계획 수립

### 실행 엔진
실행 계획을 기반으로 각 핸들러(=스토리지 엔진) 에게 요청해서 받은 결과를 다른 핸들러 요청의 입력으로 연결
**GROUP BY**나 **ORDER BY** 등의 복잡한 쿼리는 실행 엔진에서 처리

ex) group by를 처리하기 위해 임시 테이블을 사용하는 경우
1. 실행 엔진 -> 핸들러 : create 임시 테이블
2. 실행 엔진 -> 핸들러 : read WHERE절
3. 실행 엔진 -> 핸들러 : save 레코드를 임시 테이블에
4. 실행 엔진 -> 핸들러 : read 임시 테이블에서 필요한 방식(group by)
5. 실행 엔진 -> 유저, 다른 모듈 : return 결과

### 핸들러(=스토리지 엔진)
실행 엔진의 요청에 따라 데이터를 **디스크에 저장**하거나 **디스크로부터 읽는 역할**을 담당

### 쿼리 캐시 (8.0부터 삭제)
실행 쿼리와 결과값을 **캐시**하는 공간<br />
변경이 생기면 변경된 테이블과 관련된 것들을 모두 삭제해야 함 -> 8.0부터는 **성능상의 이슈로 제거**

### 스레드 풀
#### 스레드풀 X
스레드 당 하나의 커넥션 => 1:1 <br/>
[스레드 기반이면 스레드 풀은?](####스레드 기반이면 스레드 풀은?)

<br/>

#### 스레드풀 O<br />
**하나의 스레드**가 **여러개의 커넥션**을 전담 => 1:N <br />

![threadpool.png](..%2F..%2Fimg%2Fdb%2Fthreadpool.png)
출처 - [mysql dev blog](https://dev.mysql.com/blog-archive/the-new-mysql-thread-pool/)

**receiver thread** - 들어오는 연결 요청을 처리하는 스레드, 라운드 로빈 방식으로 스레드 그룹에 커넥션을 할당 <br />
**query worker thread** - 할당된 사용자 연결로부터 쿼리 실행을 담당하는 하나 이상의 스레드 <br/>


스레드 풀은 스레드 그룹으로 구성 (기본값 : 16)<br />
사용자 연결은 라운드 로빈 방식으로 스레드 그룹에 할당<br />
각 스레드 그룹 내에는 할당된 사용자 연결로부터 쿼리 실행을 담당하는 하나 이상의 스레드가 존재<br/>

**만약 스레드 그룹의 모든 스레드가 일을 처리하고 있다면?**<br />
해당 스레드 그룹에 worker thread를 추가할지, 처리 완료할 때까지 기다릴지 판단 필요
-> **타이머 스레드**가 스레드 그룹 상태를 체크해서 thread_pool_stall_limit 에 지정된 밀리초만큼 작업을 처리하지 못하면 새로운 스레드를 생성
<br/>
<br/>

### 트랜잭션 지원 메타데이터
메타데이터 or 데이터 딕셔너리 : 테이블의 구조 정보와 스토어드 프로그램 등의 정보
~ 5.7ver : 파일 기반 관리
8.0ver : innodb 테이블에 시스템 테이블(MySQL 서버가 작동할 때 기본적으로 필요한 테이블들)과 함께 저장

데이터 딕셔너리 데이터 조회 방법
```sql
select * from information_schema.tables; -- mysql.tables는 임의 수정 불가하도록 접근 불가함
```

**왜 innodb 스토리지 엔진 기반으로 변경했을까?** <br />
**파일 기반**<br />
즉, 트랜잭션을 지원하지 않아서 테이블 생성 및 변경 중 서버가 종료되면 데이터가 일관되지 않는 문제가 발생 <br />

예외 - InnoDB 외의 MyISAM이나 CSV 같은 스토리지 엔진의 메타 정보는 여전히 파일(.SDI) 기반으로 저장, innodb 테이블들의 구조도 원하면 SDI파일로 변환 가능

<br/>
<br/>

# 스토리지 엔진
- 디스크에 접근하여 **데이터를 읽고 데이터를 쓰는 엔진**이다. 대표적으료 innoDB, MyISAM, Memory 등이 있다.

# InnoDB
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

<br/>
<br/>

## 프라이머리 키에 의한 클러스터링
프라이머리 키 기준으로 클러스터링되어 저장 즉, **프라이머리 키 값이 정렬되어 디스크에 저장** <br />
이는 **프라이머리 키를 이용한 인덱스 레인지 스캔이 가능**, 원하는 페이지의 위치를 빠르게 스캔 가능 <br />
  - 참고 : MyISAM은 **논클러스터링**(insert 되는 순서대로 데이터에 저장)

<br/>
<br/>

## MVCC (Multi Version Concurrency Control)
locking의 **속도 저하 문제**, X-Lock이 걸리면 읽기 행위를 하지 못함 <br />
목적 - 잠금을 사용하지 않는 일관된 읽기를 제공

-> **하나의 레코드에 대한 여러개의 버전을 관리**하여 잠금을 사용하지 않는 일관된 읽기(MVCC 목적)를 제공한다.

## MVCC 예시
#### data insert
```sql
insert into member (id, name, area) values (20, '김주희', '서울');
commit;
```
#### database status
![mvcc_insert.png](..%2F..%2Fimg%2Fdb%2Fmvcc_insert.png)
- 메모리의 버퍼풀과 디스크에만 데이터가 저장된다.

#### data update
```sql
update member set area='제주' where id=20;
```
#### database status
![mvcc_update.png](..%2F..%2Fimg%2Fdb%2Fmvcc_update.png)
- **변경 전 값**만 **undo log**에 복사
- 위의 데이터가 아직 commit 되기 전이라면? 다른 사용자가 id=20을 조회했을 때 어떤 데이터가 조회될까?
- 이는, **isolation level**(격리 수준)에 따라 다르다.

![mvcc_isolationlevel.png](..%2F..%2Fimg%2Fdb%2Fmvcc_isolationlevel.png)

#### commit 후엔?
- 언두로그는 유지될까? <br />
-> 언두 영역을 필요로 하는 트랜잭션이 더는 없으면 제거

<br/>
<br/>

## 잠금 없는 일관된 읽기
innoDB는 MVCC를 이용하여 다른 트랜잭션이 레코드를 변경 중이어도 lock을 기다리지 않고 읽기 작업 수행 가능 <br />
isolation level이 **serializable**(읽기 작업에도 S-Lock이 걸림)이 아니라면 insert과 연결되지 않은 읽기(SELECT) 작업은 잠금 대기하지 않고 수행

<br/>
<br/>

## 자동 데드락 감지
innoDB는 내부적으로 lock이 deadlock 상태인지 확인하기 위해 **잠금 대기 목록**을 그래프 형태로 관리<br />
-> 데드락 감지 스레드가 잠금 대기 그래프를 검사해 deadlock에 빠진 트랜잭션 중 하나를 강제 종료 (기준 : 언두 로그양) <br />

**동시 처리 스레드가 매우 많아지거나 트랜잭션이 가진 잠금의 개수가 많아지면?** <br />
데드락 감지 스레드는 잠금 목록을 검사하기 위해 잠금 목록이 저장된 리스트 잠금 <br />
-> 이때 **트랜잭션들이 잠금 처리할 수 없기 때문에** 전체 성능이 느려질 수 있음 <br />

<br/>
<br/>

## 자동화된 장애 복구
MySQL 서버가 시작될 때 완료되지 못한 트랜잭션이나 일부만 기록된 데티어 페이지에 대한 복구 작업이 자동으로 진행 <br />
하지만, 자동 복구 불가능한 경우도 발생 가능 <br />
**innodb_force_recovery** 시스템 변수를 설정해 MySQL 서버 시작하여 검사 가능 <br />

1 : 테이블스페이스나 인덱스 손상 시<br />
2 : 메인 스레드가 언ㄷ 데이터 삭제하는 과정에서 장애 발생 시<br />
3 : 커밋되지 않은 트랜잭션 존재 시<br />
4 : 인서트 버퍼 손상<br />
5 : 언두로그 사용 불가 시<br />
6 : 리두로그 손상 시

<br/>
<br/>

## 버퍼풀
### 버퍼풀이란?
데이터 캐시 : 인덱스 및 테이블 데이터를 읽어 메모리에 캐시해두는 공간<br />
쓰기 버퍼링 : 쓰기 작업을 지연하여 일괄적으로 처리하게 해주는 버퍼 역할 (랜덤 i/o를 줄임)<br />
**단위** : 페이지 or 블록

### 버퍼풀 크기 설정
버퍼풀은 내부적으로 128MB 청크 단위로 쪼개서 관리<br />
버퍼 풀을 여러 개로 쪼개어 관리 -> 버퍼 풀 전체를 관리하는 잠금 때문에 내부 잠금 경합 가능성 낮추기 위해

### 버퍼풀 구조
버퍼풀을 페이지 크기로 쪼개고 해당 데이터 페이지를 읽어서 각 조각에 저장
아래와 같은 3개의 자료 구조가 관리됨
- LRU 리스트
- 플러시 리스트
- 프리 리스트 (실제 사용자 데이터로 채워지지 않은 빈 페이지들의 목록)

#### LRU 리스트
![lru_list](https://dev.mysql.com/doc/refman/8.0/en/images/innodb-buffer-pool-list.png)
목적 : 한 번 읽어온 페이지를 최대한 오래 메모리에 유지해 디스크 i/o를 감소 시키는 것

#### 과정
1. 버퍼 풀에 필요한 레코드가 저장된 데이터 페이지 유무 확인
   - innoDB 어댑티브 해시 인덱스를 통해 페이지 검색
   - 해당 테이블의 인덱스를 이용해 버퍼 풀에서 페이지 검색
   - 버퍼 풀에 있으면 포인터를 MRU 방향으로 올림
2. 디스크에서 데이터 페이지를 버퍼풀에 적재, 적재된 페이지에 대한 포인터를 LRU 헤더에 추가
3. 버퍼 풀의 LRU 헤더에 적재된 데이터가 읽히면 MRU 방향으로 이동 (대량 읽기의 경우 MRU로 이동하진 않음)
4. 버퍼 풀의 데이터 페이지에 age 부여, 오래동안 사용되지 않으면(aging) 버퍼풀에서 제거(=evictiton)
5. 필요한 데이터가 자주 접근되면 인덱스 키가 어댑티브 해시 인덱스에 추가됨

#### 데이터 변경 과정과 플러시 리스트
플러시(flush list)란?<br />
디스크로 동기화되지 않은 데이터를 가진 데이터 페이지(=더티 페이지)의 변경 시점 기준 페이지 목록을 관리<br />

데이터 변경 -> 리두 로그에 기록 -> 버퍼 풀의 데이터 페이지에 변경 내용 반영 -> 체크포인팅되어야 디스크에 동기화<br />

![bufferpool.png](..%2F..%2Fimg%2Fdb%2Fbufferpool.png)

리두 로그의 각 엔트리는 특정 데이터 페이지와 연결됨

<br/>
<br/>

### 버퍼 풀과 리두 로그
버퍼 풀은 클린 페이지(읽었으나 변경되지 않은)와 더티 페이지(변경된)를 가짐<br />
더티 페이지는 디스크에 기록되어 있지 않기 때문에 언젠가 디스크에 기록되어야 함<br />
하지만, 더티 페이지는 버퍼 풀에 무한정 머무를 수 없음. 계속 다른 데이터들이 변경되면서 리두 로그 파일에 기록됐던 **로그 엔트리가 덮어 씌워지기** 때문임.
-> 그래서 innodb는 리두 로그 파일의 당장 재사용 가능 공간과 불가능한 공간(=활성 리두 로그)을 구분해서 관리함. 그림의 화살표를 가진 엔트리들이 활성 리두 로그

#### LSN
리두 로그 파일이 순환되어 재사용될 때마다 로그 포지션(LSN)을 계속 증가시킴<br />
이 로그 포지션을 기준으로 리두 로그와 버퍼 풀의 더티 페이지를 디스크로 동기화함

- 데이터 변경되어 올라간 LSN(Log Sequence Number)
![lsn.png](..%2F..%2Fimg%2Fdb%2Flsn.png)
- 체크 포인트 이벤트 후 LSN
![checkpoint.png](..%2F..%2Fimg%2Fdb%2Fcheckpoint.png)
- 체크 포인트 후 데이터 변경
![checkpointlsn.png](..%2F..%2Fimg%2Fdb%2Fcheckpointlsn.png)
-> 체크 포인트 lsn과 리두 로그 엔트리의 lsn 차이를 **체크포인트 에이지**라고 함

이러한 이유로 버퍼 풀의 공간과 리두 로그 공간은 서로 영향을 주고 받음

<br/>
<br/>

### 버퍼 풀 플러시
스토리지 엔진은 성능 영향 없이 더티 페이지를 동기화하기 위해 2개의 플러시 기능을 백그라운드로 실행함 (클리너 스레드)
- 플러시 리스트 플러시
- LRU 리스트 플러시

#### 플러시 리스트 플러시
리두 로그 공간을 재활용 하기 위해 오래된 리두 로그 엔트리가 사용하는 공간을 비워야 함 <br />
이때 리두 로그 공간이 지워지려면 버퍼풀의 더티 페이지가 먼저 디스크로 동기화 필요 <br />
-> 이를 위해 사용하는 것이 **플러시 리스트 플러시** 기능

주의점 : 더티 페이지가 많으면 디스크 쓰기 폭발(disk i/o burst) 발생 가능성 ↑ <br />
        -> 디스크 쓰기가 폭증할 수 있기 때문에 innodb는 더티 페이지가 발생하면 더티 페이지를 조금씩 기록하도록 함

#### LRU 리스트 플러시
LRU 리스트에서 사용 빈도가 낮은 데이터 페이지를 제거하는 플러시 기능 <br />
더티 페이지는 디스크에 동기화하고 클린 페이지는 프리 리스트로 옮김

<br/>
<br/>

## Double Write Buffer
더티 페이지를 디스크에 플러시할 때 일부만 기록되는 파셜 페이지(partial-page) or 톤페이지(torn-page)가 발생할 수 있음 <br/>
 -> 하드웨어 오작동이나 비정상 종료로 이어짐<br/>
위와 같은 문제를 막기 위한 게 **Double-Write** 기법

**무결성**이 중요한 서비스에서 활성화하는 것을 추천

### 작동 방식
1. 더티 페이지를 플러시 하기 전에 DoubleWrite 버퍼에 기록
2. 각 스토리지 엔진은 더티 페이지를 파일에 랜덤 I/O로 쓰기

### 문제 상황 발생 시
1. innoDB는 스토리지 엔진을 시작할 때 DoubleWrite 버퍼의 내용과 데이터 파일의 페이지를 모두 비교
2. 다른 경우 DoubleWrite 버퍼 내용을 데이터 파일 페이지로 복사

<br/>
<br/>

## 언두 로그
DML(INSERT, UPDATE, DELETE) 변경되기 이전 데이터를 별도로 백업한 데이터 <br />

**언두 로그의 용도**
- 트랜잭션 보장 (롤백 대비용)
- 트랜잭션 격리 수준 보장 (동시성 제공)

### 언두 테이블스페이스 관리
언두 테이블 스페이스란? 언두 로그가 저장되는 공간

~ 5.6 ver : 시스템 테이블스페이스(idbata.ibd)에 저장
5.6 ver : 언두 로그 파일 or 테이블스페이스에 저장 선택
~ 8.0 ver : 언두 로그 파일에 기록

<br/>
<br/>

## 체인지 버퍼
레코드 변경 시, 테이블에 포함된 인덱스를 업데이트 하는 작업이 필요함<br />
인덱스 페이지가 버퍼 풀에 없을 때 즉시 실행하지 않고 저장하는 공간

**유니크 인덱스**는 결과 전달 전 반드시 중복 여부를 체크해야 하기 때문에 체인지 버퍼 사용 불가

### 체인지 버퍼 머지 스레드
체인지 버퍼에 저장된 인덱스 레코드 조각을 병합하는 스레드

<br/>
<br/>

## 리두 로그 및 로그 버퍼
데이터 파일에 기록되지 못한 데이터를 잃지 않게 해주는 안전장치<br />
-> 그래서 트랜잭션의 ACID 속성 중에 영속성과 관련 있음 <br />
서버의 비정상 종료가 발생하면 리두 로그로 데이터 파일을 서버 종료 전으로 복구시킴

**8.0ver**<br/>
데이터를 복구하거나 대용량 데이터를 한 번에 적재하는 경우, 리두 로그를 비활성화해서 데이터의 적재 시간을 단축 시킬 수 있음

### 로그 버퍼
innoDB 스토리지 엔진이 변경된 내용을 버퍼 풀에 모았다가 디스크에 기록하는데(체크 포인트 이벤트) 변경 작업이 많은 경우, 리두 로그의 기록 작업이 문제될 수 있음 <br />
이때 리두 로그 버퍼링에 사용되는 공간이 **로그 버퍼**

## 어댑티브 해시 인덱스
innoDB가 스토리지 엔진에서 버퍼풀에 있는 사용자가 자주 요청하는 데이터에 대해 자동으로 생성하는 인덱스 <br />
인덱스 스캔 시 루트 노트 - 브랜치 노드 - 리프 노드까지 매번 찾아야하니, B-Tree의 검색 시간을 줄이기 위해 도입된 기능 <br />
{인덱스 key , 데이터 페이지 주소}(버퍼 풀에 로딩된 페이지 주소)으로 관리

### 주의사항
버퍼풀 내의 접근을 빠르게 하는 목적이기 때문에 디스크에서 읽어오는 경우, 도움이 되지 않음 <br />
메모리 공간을 사용하기 때문에 주의해야 함 <br />
테이블 삭제 및 변경 시 어댑티브 해시 인덱스를 모두 삭제해야하기 때문에 성능에 영향

<br/>
<br/>

# MyISAM
- 키 캐시
- 운영체제의 캐시 및 버퍼
- 데이터 파일과 프라이머리 키 구조

## 키 캐시
innoDB 버퍼 풀과 비슷한 역할을 함. 하지만 **인덱스만을 대상으로 작동** <br/>
인덱스의 디스크 쓰기 작업에 대해서만 부분적으로 버퍼링

## 운영체제의 캐시 및 버퍼
MyISAM은 디스크의 i/o를 해결해주는 캐시나 버퍼링이 없기 때문에 운영체제가 그 역할을 수행함<br/>
운영체제가 디스크로부터의 i/o를 해결해주는 매커니즘을 탑재하고 있기 때문임

### 성능 저하 요소
운영체제가 사용할 수 있는 캐시 공간 필요. 남는 메모리가 없다면 데이터를 캐시하지 못해 쿼리 성능 ↓

## 데이터 파일과 프라이머리 키 구조
MyISAM은 테이블에 insert 되는 순서대로 데이터 파일이 저장 <br />
테이블에 저장된 레코드는 ROWID라는 물리적 주솟값을 가지고 PK와 세컨더리 인덱스는 모두 ROWID를 포인터로 가짐

<br/>
<br/>

# MySQL 로그 파일
- 에러 로그 파일
- 제너럴 쿼리 로그 파일
- 슬로우 쿼리 로그

## 에러 로그 파일
MySQL이 실행되는 도중 발생하는 에러나 경고 메시지가 출력되는 로그 파일 <br />
설정 파일에서 log_error라는 이름의 파라미터로 정의된 경로에 생성

## 제너럴 쿼리 로그 파일
모든 실행 쿼리가 저장되는 파일 <br />
쿼리가 실행되기 전에 MySQL이 쿼리 요청을 받으면 바로 기록하기 때문에 에러가 발생해도 기록

## 슬로우 쿼리 로그
정상적으로 실행되었으나 걸린 시간이 long_query_time에 정의된 시간보다 많이 걸린 쿼리 <br />
서버 실행 중 문제되는 쿼리를 파악하는데 도움이 됨

Time : 쿼리가 종료된 시점<br />
User@Host : 실행한 사용자의 계정<br />
Query_time : 쿼리가 실행되는데 걸린 전체 시간<br />
Lock_time : MySQL 엔진 레벨에서 실행된 테이블 잠금에 대한 대기 시간<br />
  -> 잠금 체크와 같은 실행 부분도 계산되기 때문에 작은 값이면 무시 가능<br />
Rows-examined : 쿼리 처리를 위해 접근한 레코드 건수



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
