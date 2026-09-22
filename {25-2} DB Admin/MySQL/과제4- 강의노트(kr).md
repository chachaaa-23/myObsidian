##### >MySQL: 백업 및 데이터 복구

## 개요

MySQL은 다양한 백업 전략을 제공하며, 설치 환경의 요구사항에 맞는 방법을 선택할 수 있습니다. 데이터베이스 백업은 시스템 장애, 하드웨어 고장, 사용자의 실수로 인한 데이터 삭제 등의 문제 발생 시 데이터 복구 및 재시작을 위해 수행됩니다.

## `mysql` 도구의 배치 모드 사용

### 배치 모드란?

대화형 방식 대신, SQL 문을 파일에 저장하고 한 번에 실행하는 방식입니다.

```shell
$ mysql < batch-file
```

연결 매개변수 지정:

```shell
$ mysql -h host -u user -p < batch-file
```

### 배치 모드 사용 이유

- **반복 실행**: 매일 또는 매주 실행하는 쿼리를 매번 입력할 필요 없음
- **쿼리 재사용**: 기존 쿼리를 복사하고 편집하여 새 쿼리 생성
- **오류 수정 용이**: 스크립트를 수정하여 다시 실행
- **출력 관리**: 페이징 도구 사용 (`| less`) 또는 파일로 저장 (`> output.txt`)
- **자동화**: cron 작업 등에서 사용

### 출력 형식

- **대화형 모드**: 표 형식으로 출력
- **배치 모드**: 탭으로 구분된 간단한 형식

배치 모드에서 대화형 형식을 원할 경우: `mysql -t`

### 🩵스크립트 내부 실행

```mysql
SOURCE filename;
-- 또는
\. filename
```
-> restore data

## 백업 유형

### 1. 물리적(Physical) vs 논리적(Logical) 백업

#### 물리적 백업

- **특징**: 데이터베이스 디렉터리와 파일의 원본 복사
- **장점**: 빠른 백업 및 복구, 압축된 출력
- **단점**: 동일한 하드웨어에서만 복원 가능
- **도구**: `mysqlbackup`, `cp`, `scp`, `tar`, `rsync`
- **서버 상태**: 서버가 중지되어 있거나 적절한 잠금 필요

#### 논리적 백업

- **특징**: SQL 문(CREATE, INSERT) 형태로 저장
- **장점**: 플랫폼 독립적, 데이터 편집 가능, 다른 아키텍처로 이동 가능
- **단점**: 물리적 백업보다 느림, 출력 크기가 큼
- **도구**: `mysqldump`, `SELECT ... INTO OUTFILE`
- **서버 상태**: 서버 실행 중에 수행 가능

### 2. 온라인(Online) vs 오프라인(Offline) 백업

#### 온라인 백업

- 서버 실행 중 수행
- 클라이언트 작업에 최소한의 영향
- 적절한 잠금 필요

#### 오프라인 백업

- 서버 중지 후 수행
- 절차가 간단하지만 서비스 중단
- 복제 슬레이브 서버에서 수행 권장

### 3. 전체(Full) vs 증분(Incremental) 백업

#### 전체 백업

- 특정 시점의 모든 데이터 포함
- `mysqldump` 등의 도구 사용

#### 증분 백업

- 특정 기간 동안의 데이터 변경사항만 포함
- 바이너리 로그 활성화 필요
- 전체 백업 후 증분 백업을 적용하여 복구

## `mysqldump`를 이용한 백업

### 기본 사용법

#### 모든 데이터베이스 덤프

```shell
$ mysqldump --all-databases > dump.sql
```

#### 특정 데이터베이스 덤프

```shell
$ mysqldump --databases db1 db2 db3 > dump.sql
```

#### 단일 데이터베이스 덤프

```shell
$ mysqldump test > dump.sql
```

#### 🩵특정 테이블만 덤프

```shell
$ mysqldump test t1 t3 t7 > dump.sql
$ mysqldump [옵션들] db_name table_name [table_name2...]
```

**Practical example**
```shell
mysqldump --compact --skip-comments --skip-opt --databases school
```
- **--databases : DB 백업 시, dump 출력에 CREATE DATABASE & USE 문 포함시킴**
- --compact : 메괄호, 헤더, 도움말 제거
- --skip-comments : 주석 제거
- --skip-opt : 최적화 기본값 비활성화


### SQL 형식 백업 복원

#### 전체 데이터베이스 복원

```shell
$ mysql < dump.sql
```

또는 mysql 클라이언트 내에서:

```mysql
SOURCE dump.sql;
```

#### 특정 데이터베이스로 복원

```shell
$ mysqladmin create db1
$ mysql db1 < dump.sql
```

### CSV 형식 백업 (`--tab` 옵션)

```shell
$ mysqldump --tab=/tmp db1
```

- 각 테이블마다 두 개의 파일 생성:
    - `table_name.sql`: CREATE TABLE 문
    - `table_name.txt`: 데이터 (탭으로 구분)

#### CSV 형식 옵션

```shell
$ mysqldump --tab=/tmp \
  --fields-terminated-by=, \
  --fields-enclosed-by='"' \
  --lines-terminated-by=0x0d0a \
  db1
```

#### CSV 백업 복원

```shell
$ mysql db1 < t1.sql
$ mysqlimport db1 t1.txt
```

🩵또는 LOAD DATA 사용:

[`LOAD DATA`](https://dev.mysql.com/doc/refman/8.0/en/load-data.html "15.2.9 LOAD DATA Statement") can be used to read files obtained from external sources. 
For example, many programs can export data in comma-separated values (CSV) format, such that lines have fields **separated by commas** and **enclosed within double quotation marks**, with an initial **line of column names**. 
If the lines in such a file are terminated by carriage return/newline pairs, the statement shown here illustrates the field- and line-handling options you would use to load the file:


```mysql
LOAD DATA INFILE 'data.txt' 
INTO TABLE tbl_name   

FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'   
LINES TERMINATED BY '\r\n'   
IGNORE 1 LINES;
```

### 유용한 옵션

#### 저장 프로시저, 함수, 이벤트, 트리거 포함

- `--events`: 이벤트 스케줄러
- `--routines`: 저장 프로시저 및 함수
- `--triggers`: 트리거 (기본값: 활성화)

#### 🩵정의와 데이터 분리

```shell
# 🩵테이블 정의만 (--no-data)
$ mysqldump --no-data --routines --events test > dump-defs.sql

# 🩵데이터만
$ mysqldump --no-create-info test > dump-data.sql
```

### 실용 예제

#### 데이터베이스 복사

```shell
$ mysqldump db1 > dump.sql
$ mysqladmin create db2
$ mysql db2 < dump.sql
```

#### 다른 서버로 복사

서버 1:
```shell
$ mysqldump --databases db1 > dump.sql
```

서버 2:
```shell
$ mysql < dump.sql
```

## `mysqlpump`를 이용한 백업

### 주요 특징

- **병렬 처리**: 여러 데이터베이스와 객체를 동시에 처리
- **세밀한 제어**: 특정 데이터베이스와 객체 선택 가능
- **사용자 계정**: CREATE USER와 GRANT 문으로 저장
- **압축 출력**: 가능
- **진행 표시기**: 제공

### 객체 선택

```mysql
mysqlpump --default-parallelism=1 --skip-tz-utc --skip-set-charset --skip-watch-progress --include-tables='ye%' --no-create-info --no-create-db --skip-routines --skip-triggers academy
```
`--default-parallelism=1   --skip-tz-utc   --skip-set-charset  --skip-watch-progress`
	- 용량줄이기용 코드(주석제거, ...)
`  --include-tables='ye%'`
	- 이름이 ye로 시작하는 테이블만 포함

**`--no-create-info` : CREATE TABLE 문을 출력 x**
**`--no-create-db` : CREATE DATABASE 문을 출력 x**
+) **`--exclude-tables=%` : 모든 테이블을 덤프 대상에서 제외**

`--skip-routines` : 프로시저/함수(=루틴) 생성을 생략. [dev.mysql.com](https://dev.mysql.com/doc/refman/8.4/en/mysqlpump.html?utm_source=chatgpt.com)
`--skip-triggers` : 트리거 생성을 생략. [dev.mysql.com](https://dev.mysql.com/doc/refman/8.4/en/mysqlpump.html?utm_source=chatgpt.com)
`--skip-definer` : `DEFINER=` 같은 소유자 관련 구문을 제거(루틴/트리거 관련 문제 예방). [Percona](https://www.percona.com/blog/migrating-ownership-of-your-stored-routines-views-and-triggers-in-mysql/?utm_source=chatgpt.com)


```shell
# 특정 데이터베이스 제외
$ mysqlpump --exclude-databases=test,world

# 특정 테이블만 포함
$ mysqlpump --include-tables=customer,invoice

# 조합 사용
$ mysqlpump --include-databases=db1,db2 --exclude-tables=db1.t1,db2.t2
```

### 병렬 처리 설정

```shell
# 기본 2개 스레드
$ mysqlpump --default-parallelism=4

# 데이터베이스별 스레드 지정
$ mysqlpump --parallel-schemas=5:db1,db2 --parallel-schemas=3:db3
```

### 제한사항

- 기본적으로 `performance_schema`, `ndbinfo`, `sys` 데이터베이스는 제외
- `INFORMATION_SCHEMA`는 덤프하지 않음
- InnoDB `CREATE TABLESPACE` 문은 저장하지 않음

## `SELECT ... INTO OUTFILE`를 이용한 데이터 내보내기

### 기본 사용법

```mysql
SELECT a, b, a+b
INTO OUTFILE '/tmp/result.txt'
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM test_table;
```

### TABLE 문 사용

```mysql
TABLE employees
ORDER BY lname 
LIMIT 1000
INTO OUTFILE '/tmp/employee_data.txt'
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

### 주요 특징

- 서버 호스트에 파일 생성
- `FILE` 권한 필요
- 기존 파일이 있으면 오류 발생
- `INTO DUMPFILE`: 한 행만 저장 (BLOB 데이터용)

## `LOAD DATA`를 이용한 데이터 로딩

### 기본 사용법

```mysql
LOAD DATA LOCAL INFILE '/path/pet.txt' 
INTO TABLE pet;
```

### 줄바꿈 문자 지정
(Win 기준)
```mysql
LOAD DATA LOCAL INFILE '/path/pet.txt'
INTO TABLE pet
LINES TERMINATED BY '\r\n';
```

### LOCAL 기능 설정

#### 서버 측

```shell
# local_infile 변수 활성화
$ mysqld --local-infile=1
```

#### 클라이언트 측

```shell
# mysql 클라이언트
$ mysql --local-infile=1

# mysqlimport
$ mysqlimport --local=1
```

### 파이프에서 데이터 로딩 (Unix)

```shell
$ mkfifo /mysql/data/db1/ls.dat
$ chmod 666 /mysql/data/db1/ls.dat
$ find / -ls > /mysql/data/db1/ls.dat &
$ mysql -e "LOAD DATA INFILE 'ls.dat' INTO TABLE t1" db1
```

## 🩵`mysqlimport`를 이용한 데이터 가져오기

### 기본 사용법

```shell
$ mysqlimport [options] db_name textfile1 [textfile2 ...]
```
파일명에서 확장자를 제거한 이름이 테이블명이 됩니다.
>옵션 정리
- `mysqlimport` : LOAD DATA sql문의 명령줄 인터페이스 역할. 데이터 파일을 읽어 서버 전송
- `--local` : 서버의 보안 제한 우회. 가져올 파일이 클라이언트 컴퓨터에 있음을 서버에 알림. 
- `--ignore-lines=1` : 첫번째 행 무시. 테이블의 열 이름 포함 시 사용

### 🩵실용 예제

```shell
# 테이블 생성
$ mysql -e 'CREATE TABLE imptest(id INT, n VARCHAR(30))' test

# 데이터 파일 생성
$ cat > imptest.txt << EOF
100,Max Sydow
101,Count Dracula
EOF

# 🩵데이터 가져오기
$ mysqlimport --local --fields-terminated-by=, --ignore-lines=1 db_name file_name.txt


# 결과 확인
$ mysql -e 'SELECT * FROM imptest' test
```

## 권장 백업 전략

1. **바이너리 로그 활성화**
```shell
   $ mysqld --log-bin
```

2. **정기적인 전체 백업**
```shell
   $ mysqldump --all-databases > backup.sql
```

3. **정기적인 증분 백업**
```shell
   $ mysqladmin flush-logs
```

4. **백업 저장 위치**: 데이터 디렉터리와 다른 안전한 매체에 보관
5. **복구 절차**:
    - 전체 백업 복원
    - 바이너리 로그를 이용한 증분 복구

## 참고사항

- InnoDB는 자동 복구 메커니즘 제공
- 백업 파일의 소유자와 권한 확인 필요
- 압축 및 암호화는 외부 도구 사용
- 정기적인 백업 일정 수립 및 자동화 권장