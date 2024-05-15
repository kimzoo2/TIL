## GTID(global transaction id) 활성화하기
### gtid 확인
#### query
```sql
SHOW VARIABLES LIKE 'gtid_executed';
or
show master status \G
```
#### 결과
```
+---------------+----------------------------------------+
| Variable_name | Value                                  |
+---------------+----------------------------------------+
| gtid_executed |                                        |
+---------------+----------------------------------------+
1 row in set (0.01 sec)

or

*************************** 1. row ***************************
             File: binlog.028872
         Position: 453
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set, 1 warning (0.00 sec)
```
위와 같이 gtid가 없는 경우, **gtid_mode**를 확인한다
### gtid_mode 확인
#### query
```sql
SHOW VARIABLES LIKE 'gtid_mode';
```
#### 결과
```sql
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | OFF    |
+---------------+-------+
1 row in set (0.01 sec)
```
### 해결 방법
gtid_mode가 off인 경우, 아래의 순서를 따른다.
#### 1. mysql 서버 종료 
- brew로 시작한 경우
```
brew services stop mysql
```
#### 2. my.cnf 설정 추가 
- brew로 설치한 경우 opt/homebrew/etc 경로 확인
```
gtid-mode=ON
enforce-gtid-consistency=ON
```
#### 3. mysql 실행 후 gtid_mode 확인
```
brew services start mysql
```
```sql
SHOW VARIABLES LIKE 'gtid_mode';
AND
SHOW VARIABLES LIKE 'gtid_executed';
```
### 결과
```
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | ON    |
+---------------+-------+
1 row in set (0.01 sec)

or

+---------------+----------------------------------------+
| Variable_name | Value                                  |
+---------------+----------------------------------------+
| gtid_executed | d92b8184-0c43-11ef-b612-a7435c03fadd:1 |
+---------------+----------------------------------------+
1 row in set (0.00 sec)
```



