## 데이터베이스 조회
### 쿼리
```sql
show databases;
```
### 결과
```
+--------------------+
| Database           |
+--------------------+
| employees          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

## 테이블 스키마 조회
### 쿼리
```sql
select * from INFORMATION_SCHEMA.COLUMNS where table_name = '테이블명';
-> select * from INFORMATION_SCHEMA.COLUMNS where table_name = 'employees';
```
### 결과
```
+---------------+--------------+------------+-------------+------------------+----------------+-------------+-----------+--------------------------+------------------------+-------------------+---------------+--------------------+--------------------+--------------------+---------------+------------+-------+---------------------------------+----------------+-----------------------+--------+
| TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | COLUMN_NAME | ORDINAL_POSITION | COLUMN_DEFAULT | IS_NULLABLE | DATA_TYPE | CHARACTER_MAXIMUM_LENGTH | CHARACTER_OCTET_LENGTH | NUMERIC_PRECISION | NUMERIC_SCALE | DATETIME_PRECISION | CHARACTER_SET_NAME | COLLATION_NAME     | COLUMN_TYPE   | COLUMN_KEY | EXTRA | PRIVILEGES                      | COLUMN_COMMENT | GENERATION_EXPRESSION | SRS_ID |
+---------------+--------------+------------+-------------+------------------+----------------+-------------+-----------+--------------------------+------------------------+-------------------+---------------+--------------------+--------------------+--------------------+---------------+------------+-------+---------------------------------+----------------+-----------------------+--------+
| def           | employees    | employees  | emp_no      |                1 | NULL           | NO          | int       |                     NULL |                   NULL |                10 |             0 |               NULL | NULL               | NULL               | int           | PRI        |       | select,insert,update,references |                |                       |   NULL |
| def           | employees    | employees  | birth_date  |                2 | NULL           | NO          | date      |                     NULL |                   NULL |              NULL |          NULL |               NULL | NULL               | NULL               | date          |            |       | select,insert,update,references |                |                       |   NULL |
| def           | employees    | employees  | first_name  |                3 | NULL           | NO          | varchar   |                       14 |                     56 |              NULL |          NULL |               NULL | utf8mb4            | utf8mb4_general_ci | varchar(14)   | MUL        |       | select,insert,update,references |                |                       |   NULL |
| def           | employees    | employees  | last_name   |                4 | NULL           | NO          | varchar   |                       16 |                     64 |              NULL |          NULL |               NULL | utf8mb4            | utf8mb4_general_ci | varchar(16)   |            |       | select,insert,update,references |                |                       |   NULL |
| def           | employees    | employees  | gender      |                5 | NULL           | NO          | enum      |                        1 |                      4 |              NULL |          NULL |               NULL | utf8mb4            | utf8mb4_general_ci | enum('M','F') | MUL        |       | select,insert,update,references |                |                       |   NULL |
| def           | employees    | employees  | hire_date   |                6 | NULL           | NO          | date      |                     NULL |                   NULL |              NULL |          NULL |               NULL | NULL               | NULL               | date          | MUL        |       | select,insert,update,references |                |                       |   NULL |
+---------------+--------------+------------+-------------+------------------+----------------+-------------+-----------+--------------------------+------------------------+-------------------+---------------+--------------------+--------------------+--------------------+---------------+------------+-------+---------------------------------+----------------+-----------------------+--------+
6 rows in set (0.01 sec)
```


## 테이블 구조 조회
### 쿼리
```sql
desc [table명];
-> desc employees;
```
```
+------------+---------------+------+-----+---------+-------+
| Field      | Type          | Null | Key | Default | Extra |
+------------+---------------+------+-----+---------+-------+
| emp_no     | int           | NO   | PRI | NULL    |       |
| birth_date | date          | NO   |     | NULL    |       |
| first_name | varchar(14)   | NO   | MUL | NULL    |       |
| last_name  | varchar(16)   | NO   |     | NULL    |       |
| gender     | enum('M','F') | NO   | MUL | NULL    |       |
| hire_date  | date          | NO   | MUL | NULL    |       |
+------------+---------------+------+-----+---------+-------+
6 rows in set (0.01 sec)
```
### 결과

## 테이블 인덱스 조회
### 쿼리
```sql
show index from [테이블명];
-> show index from employees;
```
### 결과
```
+-----------+------------+---------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table     | Non_unique | Key_name            | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-----------+------------+---------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| employees |          0 | PRIMARY             |            1 | emp_no      | A         |      296392 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| employees |          1 | ix_hiredate         |            1 | hire_date   | A         |        9406 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| employees |          1 | ix_gender_birthdate |            1 | gender      | A         |           5 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| employees |          1 | ix_gender_birthdate |            2 | birth_date  | A         |        8142 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| employees |          1 | ix_firstname        |            1 | first_name  | A         |        1351 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
+-----------+------------+---------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
5 rows in set (0.00 sec)
```

## 세션 격리 수준 확인
### 쿼리
```sql
SELECT @@tx_isolation;
or show variables like 't%_isolation';
```

### 결과
```
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set (0.00 sec)
```

## 세션 격리 수준 변경
### 쿼리
```sql
set session transaction isolation level [isolation level];
-> set session transaction isolation level repeatable read;
```
