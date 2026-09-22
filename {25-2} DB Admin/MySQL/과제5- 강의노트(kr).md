##### >MySQL 8 파티셔닝 정리

### TABLE KEY 추가/삭제
`ALTER TABLE tableName ADD PRIBARY KEY (keyName)`
- 괄호 안에 여러개의 인자 가능
`ALTER TABLE tableName DROP PRIMARY KEY`


## 1. 파티셔닝 개념

파티셔닝은 데이터베이스에서 **데이터를 물리적으로 저장하는 방식**을 의미합니다. 
개별 테이블의 일부를 파일 시스템의 서로 다른 위치에 별도의 "**서브테이블**"로 저장할 수 있게 해줍니다.

**MySQL 8의 특징:**
- InnoDB 스토리지 엔진만 파티셔닝을 지원
- 별도의 활성화 설정 불필요
- 수평 파티셔닝만 지원 (행 단위 분할)

🩵**기본 예시:**

```sql
CREATE TABLE tp (
    tp_id INT, 
    amt DECIMAL(5,2),
    trx_date DATE
) -- CREATE 이후 세미컬럼 없음!
ENGINE = INNODB
PARTITION BY HASH (MONTH(trx_date)) -- CREATE 의 일부!
PARTITIONS 4;
```

## 2. 파티셔닝의 장점

1. **대용량 데이터 저장**: 단일 디스크나 파일 시스템 파티션보다 더 많은 데이터 저장 가능
2. **효율적인 데이터 삭제**: 불필요한 파티션만 제거하여 간편하게 데이터 삭제
3. **쿼리 성능 향상**: 여러 디스크에 데이터 검색을 분산하여 처리량 증가
4. **자동 쿼리 최적화**: WHERE 조건에 따라 관련 없는 파티션은 자동으로 검색에서 제외 (파티션 프루닝)

## 3. 파티셔닝 유형

### 🩵3.1 RANGE 파티셔닝

**값의 범위를 기준**으로 행을 파티션에 할당합니다.

**store_id 기준 예시:**

```mysql
CREATE TABLE employee (
    employee_id INT NOT NULL,
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    hired_date DATE NOT NULL DEFAULT '1990-01-01',
    termination_date DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (store_id) ( -- 🩵
    PARTITION p0 VALUES LESS THAN (6),    -- 매장 1-5
    PARTITION p1 VALUES LESS THAN (11),   -- 매장 6-10
    PARTITION p2 VALUES LESS THAN (16),   -- 매장 11-15
    PARTITION p3 VALUES LESS THAN (21),   -- 매장 16-20
    PARTITION p4 VALUES LESS THAN (26)    -- 매장 21-25
);
```

🩵**날짜 기준 예시:**

```mysql
...
PARTITION BY RANGE (YEAR(hired_date)) (
    PARTITION p0 VALUES LESS THAN (1996),
    PARTITION p1 VALUES LESS THAN (2001),
    PARTITION p2 VALUES LESS THAN (2006),
    PARTITION p3 VALUES LESS THAN MAXVALUE -- 🩵
);
```

TABLE 선언 없이 파티션만 추가하는 경우
```mysql
ALTER TABLE employees PARTITION BY RANGE (YEAR(hire_date)) (
    PARTITION p1980 VALUES LESS THAN (1980),
    PARTITION p2000 VALUES LESS THAN (2000),
    PARTITION p2020 VALUES LESS THAN (2020),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);
```

### 3.2 LIST 파티셔닝

명시적으로 정의된 값 목록을 기준으로 행을 할당합니다.

**지역별 파티셔닝 예시:**

```sql
CREATE TABLE employee (
    employee_id INT NOT NULL,
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    hired_date DATE NOT NULL DEFAULT '1990-01-01',
    termination_date DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY LIST (store_id) (
    PARTITION pNorth   VALUES IN (1, 2, 11, 12, 21, 22),
    PARTITION pSouth   VALUES IN (3, 4, 13, 14, 23, 24),
    PARTITION pEast    VALUES IN (5, 6, 15, 16, 25),
    PARTITION pWest    VALUES IN (7, 8, 17, 18),
    PARTITION pCentral VALUES IN (9, 10, 19, 20)
);
```

🩵**파티션 삭제:**

```sql
-- 데이터와 파티션 모두 삭제
ALTER TABLE employee DROP PARTITION pWest;

-- 데이터만 삭제 (파티션 구조 유지)
ALTER TABLE employee TRUNCATE PARTITION pWest;
```

### 3.3 COLUMNS 파티셔닝

여러 컬럼을 사용하여 파티셔닝할 수 있습니다.

#### RANGE COLUMNS

```sql
CREATE TABLE trc (
    p INT,
    q INT,
    r CHAR(3),
    s INT
)
PARTITION BY RANGE COLUMNS (p, s, r) (
    PARTITION p0 VALUES LESS THAN (5, 10, 'ppp'),
    PARTITION p1 VALUES LESS THAN (10, 20, 'sss'),
    PARTITION p2 VALUES LESS THAN (15, 30, 'rrr'),
    PARTITION p3 VALUES LESS THAN (MAXVALUE, MAXVALUE, MAXVALUE)
);
```

**특징:**

- 여러 컬럼 사용 가능
- 표현식이 아닌 컬럼 이름만 사용
- 정수형 외에 문자열, DATE, DATETIME 타입 사용 가능

#### LIST COLUMNS

```sql
CREATE TABLE customer_z (
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    street_1 VARCHAR(35),
    street_2 VARCHAR(35),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY LIST COLUMNS (city) (
    PARTITION pZone_1 VALUES IN ('Ahmedabad', 'Surat', 'Mumbai'),
    PARTITION pZone_2 VALUES IN ('Delhi', 'Gurgaon', 'Punjab'),
    PARTITION pZone_3 VALUES IN ('Kolkata', 'Mizoram', 'Hyderabad'),
    PARTITION pZone_4 VALUES IN ('Bangalore', 'Chennai', 'Kochi')
);
```

### 3.4 HASH 파티셔닝

정의된 파티션 수에 데이터를 균등하게 분산합니다.

```sql
CREATE TABLE employee (
    employee_id INT NOT NULL,
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    hired_date DATE NOT NULL DEFAULT '1990-01-01',
    termination_date DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY HASH (store_id)
PARTITIONS 4;
```

**PARTITIONS 절 생략 시 기본값은 1**

### 3.5 LINEAR HASH 파티셔닝

선형 알고리즘(2^n)을 사용하는 해시 파티셔닝입니다.

```sql
CREATE TABLE employee (
    employee_id INT NOT NULL,
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    hired_date DATE NOT NULL DEFAULT '1990-01-01',
    termination_date DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY LINEAR HASH (YEAR(hired_date))
PARTITIONS 4;
```

**장점:** 파티션 추가, 삭제, 병합, 분할이 매우 빠름 (대용량 데이터에 유리)  
**단점:** 일반 해시에 비해 데이터 분산이 덜 균등함

### 🩵 3.6 KEY 파티셔닝

HASH와 유사하지만 MySQL 내장 해시 함수를 사용합니다.

```sql
-- PRIMARY KEY를 자동으로 파티셔닝 키로 사용
CREATE TABLE tk1 (
    tk1_id INT NOT NULL PRIMARY KEY,
    note VARCHAR(50)
)
PARTITION BY KEY ()
PARTITIONS 2;

-- 명시적으로 컬럼 지정
CREATE TABLE tk2 (
    cl1 INT NOT NULL,
    cl2 CHAR(10),
    cl3 DATE
)
PARTITION BY LINEAR KEY (cl1)
PARTITIONS 3;
```

**특징:**
- NULL이나 정수 외의 타입도 사용 가능
- LINEAR KEY도 지원

### 🩵3.7 서브파티셔닝

파티션을 추가로 하위 파티션으로 분할합니다

```sql
CREATE TABLE trs (
    trs_id INT,
    sold DATE
)
PARTITION BY RANGE (YEAR(sold))
SUBPARTITION BY HASH (TO_DAYS(sold))
SUBPARTITIONS 2 (
    PARTITION p0 VALUES LESS THAN (1991),
    PARTITION p1 VALUES LESS THAN (2001),
    PARTITION p2 VALUES LESS THAN MAXVALUE
);
```

```mysql
-- 방법1: 자동 subpartition 
	-- 각 파티션 내 subpartition 이름 자동생성 (ex. p0sp0, p0sp1)
CREATE TABLE exams (
    student INT NOT NULL,
    approach DATE NOT NULL,
    grade INT NOT NULL
)
PARTITION BY RANGE COLUMNS (approach) -- 이어서 subpartition 부분에서 정의.
SUBPARTITION BY HASH (TO_DAYS(approach))
SUBPARTITIONS 2 (
    PARTITION p0 VALUES LESS THAN ('2000-09-01'),
    PARTITION p1 VALUES LESS THAN ('2010-09-01'),
    PARTITION p2 VALUES LESS THAN (MAXVALUE)
);

-- 방법2: 각 partition 안에 subpartition 이름 직접 명시
CREATE TABLE exams (
    student INT NOT NULL,
    approach DATE NOT NULL,
    grade INT NOT NULL
)
PARTITION BY RANGE COLUMNS (approach) -- 이어서 subpartition 부분에서 정의.
SUBPARTITION BY HASH (TO_DAYS(approach))
SUBPARTITIONS 2
(
    PARTITION p0 VALUES LESS THAN ('2000-09-01') (
        SUBPARTITION p0sp0,
        SUBPARTITION p0sp1
    ),
    PARTITION p1 VALUES LESS THAN ('2010-09-01') (
        SUBPARTITION p1sp0,
        SUBPARTITION p1sp1
    ),
    PARTITION p2 VALUES LESS THAN (MAXVALUE) (
        SUBPARTITION p2sp0,
        SUBPARTITION p2sp1
    )
);
```

***HASH 서브 파티션** 사용 이유?*
	- 파티션 내부의 부하를 분산 (특정 기간에 데이터가 집중되는 문제 해소)
	- 범위 파티션 내부에서 데이터를 무작위로(랜덤하게) n개의 서브 파티션으로 다시 나눔

**규칙:**
- RANGE 또는 LIST와 KEY 또는 HASH 조합 가능 **(= 조합 파티셔닝)**
- 모든 파티션의 서브파티션 수가 동일해야 함
- 서브파티션 이름은 테이블 전체에서 고유해야 함

`TO_DAYS('date')` : 기원년(Year 0) 이후 며칠이 지났는지를 정수로 변환
- 결과 INT
- HASH 파티션을 날짜루 나누고 싶을 때 사용 (why. hash 파티션 키는 숫자나 문자열만 받음. DATE 자체를 바로 넣으면 MySQL 오류.)

## 4. NULL 값 처리

파티셔닝 타입별 NULL 처리 방식:
- **RANGE**: 가장 낮은 파티션에 삽입
- **LIST**: NULL이 명시적으로 값 목록에 포함된 경우에만 삽입 성공
- **HASH/KEY**: NULL을 0으로 처리

## 5. 파티션 관리

### 🩵5.1 RANGE/LIST 파티션 관리

**파티션 삭제:**

```sql
ALTER TABLE employee DROP PARTITION p2;
```

**파티션 추가:**

```sql
ALTER TABLE employee ADD PARTITION (
    PARTITION p4 VALUES LESS THAN (31)
);
```

### 5.2 HASH/KEY 파티션 관리

**파티션 수 줄이기:**

```sql
ALTER TABLE client COALESCE PARTITION 4;  -- 4개 파티션 감소
```

🩵 **파티션 삭제:**
- 파티션을 완전히 제거하고 NULL 상태로 만듬
```sql
ALTER TABLE tableName REMOVE PARTITIONING;
```

🩵 **파티션 추가:**

```sql
ALTER TABLE client ADD PARTITION PARTITIONS 4;
```

### 5.3 파티션 유지보수

**파티션 재구성 (조각 모음):**

```sql
ALTER TABLE trp REBUILD PARTITION p0, p1, p2;
```

🩵**파티션 분석 (analyze partion):**
`ANALYZE TABLE table1`
- 테이블 전체(모든 파티션 포함) 를 검사.
- 테이블의 키 분포(key distribution) 와 통계정보를 다시 계산 (이때 메타데이터 갱신, information_schema 조회 시 최신 table_row 값 확인가능!)

`ALTER TABLE tableName ANALYZE PARTITION partition1, partition2;`
- 선택한 테이블의 파티션을 스캔하여, 통계 데이터 값이 어떤지 다시 정확하게 추정하라고 지정. 
	`ANALYZE PARTITION p1` : 파티션된 테이블에서 개별 파티션의 통계를 분석함
	- 이때 `INFORMATION_SCHEMA.PARTITIONS` 테이블의 `TABLE_ROWS` 필드 값 갱신. 
	- `partition` 자리에 'ALL' 불가. 


**파티션 최적화:**

```sql
ALTER TABLE top OPTIMIZE PARTITION p0, p1, p2;
```

**파티션 복구:**

```sql
ALTER TABLE trp REPAIR PARTITION p3;
```

**파티션 검사:**

```sql
ALTER TABLE tcp CHECK PARTITION p0;
```

**모든 파티션에 적용:**

```sql
ALTER TABLE tap ANALYZE PARTITION ALL;
```

🩵**Reorganize partition:**
```mysql
ALTER TABLE table_cars -- 해당 테이블의
    REORGANIZE PARTITION p2_fast INTO( -- p2 파티션 조건을 변경한다
    PARTITION p2_fast VALUES LESS THAN (325), 
    PARTITION p3_turbo VALUES LESS THAN MAXVALUE
);
```

## 🩵6. 파티션 정보 조회

##### *`SHOW` 와 `SELECT` 차이*
**`SHOW`**: 테이블 자체의 구조(schema) 및 관리정보(metadata) 확인하는 관리명령어.
	(db에 저장된 실제 데이터 x)
**`SELECT`** : db와 테이블에 저장된 실제 데이터를 검색하고 추출.

```sql
-- 테이블 생성 정보 확인
SHOW CREATE TABLE employee\G

-- 🩵테이블 상태 확인
SHOW TABLE STATUS LIKE 'employee';
SHOW TABLE STATUS FROM mydb;

-- 쿼리 실행 계획 확인
EXPLAIN SELECT * FROM employee WHERE id < 10;

-- INFORMATION_SCHEMA에서 조회
SELECT * FROM INFORMATION_SCHEMA.PARTITIONS 
WHERE TABLE_NAME = 'employee';
```

***`EXPLAIN` 명령어***
	MySQL 서버가 해당 쿼리의 실행 계획을 분석하여 출력함
	(어떻게 처리할지 explain)
	- 주로 `partitions` 필드에서의 파티셔닝 효과 확인


##### [`INFORMATION_SCHEMA.PARTITIONS`](https://dev.mysql.com/doc/refman/9.0/en/information-schema-partitions-table.html)
	PARTITIONS 에 관한 정보가 담겨있는 테이블.

| Table Name  | `PARTITIONS`     |                                                   |
| ----------- | ---------------- | ------------------------------------------------- |
| **columns** | `TABLE_SCHEMA`   | name of the schema(DB) to which the table belongs |
|             | `TABLE_NAME`     | name of the table containg the partition          |
|             | `PARTITION_NAME` |                                                   |
|             | ...              |                                                   |

## 7. 파티션 프루닝 (Partition Pruning)

쿼리 조건에 따라 불필요한 파티션을 자동으로 검색에서 제외하는 최적화 기법입니다.

**예시:**

```sql
CREATE TABLE tp1 (
    first_name VARCHAR(30) NOT NULL,
    last_name VARCHAR(30) NOT NULL,
    zone_code TINYINT UNSIGNED NOT NULL,
    doj DATE NOT NULL
)
PARTITION BY RANGE (zone_code) (
    PARTITION p0 VALUES LESS THAN (65),
    PARTITION p1 VALUES LESS THAN (129),
    PARTITION p2 VALUES LESS THAN (193),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);

-- 이 쿼리는 p1, p2만 검색
SELECT first_name, last_name, doj
FROM tp1
WHERE zone_code > 126 AND zone_code < 131;
```

**프루닝이 적용되는 경우:**

- `partition_column = constant`
- `partition_column IN (constant1, constant2, ...)`
- `partition_column BETWEEN value1 AND value2`

**지원 명령문:** SELECT, UPDATE, DELETE

## 🩵8. 파티션 선택 (Partition Selection)

특정 파티션을 명시적으로 지정하여 쿼리할 수 있습니다.

```sql
-- p1 파티션에서만 조회 🩵
SELECT * FROM employee PARTITION (p1);

-- 여러 파티션 지정
SELECT * FROM employee PARTITION (p1, p2);
```

## 9. 외부 테이블과 파티션

`DATA DIRECTORY` 옵션으로 테이블이나 파티션을 외부 위치에 저장할 수 있습니다.

```sql
-- 테이블 전체를 외부에 저장
CREATE TABLE foo1 (
    x INT PRIMARY KEY
)
DATA DIRECTORY = '/tmp/externaldir-1';

-- 파티션별로 다른 위치에 저장
CREATE TABLE foo2 (
    x INT PRIMARY KEY
)
PARTITION BY RANGE (x) (
    PARTITION p0_slow VALUES LESS THAN (100)
        DATA DIRECTORY = '/tmp/externaldir-2',
    PARTITION p1_medium VALUES LESS THAN (MAXVALUE)
        DATA DIRECTORY = '/tmp/externaldir-3'
);
```

**주의사항:**

- 프로덕션 환경에서는 `innodb_directories` 변수 설정 필요
- 서버 재시작 필요

## 10. 파티셔닝 제약사항

### 10.1 PRIMARY KEY와 UNIQUE KEY

**중요 규칙:** 파티셔닝 키(함수)는 모든 PRIMARY KEY와 UNIQUE KEY에 포함된 컬럼을 포함해야 합니다.

**잘못된 예:**

```sql
-- 오류: cl3이 UNIQUE KEY에 없음
CREATE TABLE tk1 (
    cl1 INT NOT NULL,
    cl2 DATE NOT NULL,
    cl3 INT NOT NULL,
    cl4 INT NOT NULL,
    UNIQUE KEY (cl1, cl2)
)
PARTITION BY HASH(cl3)
PARTITIONS 4;
```

**올바른 예:**

```sql
CREATE TABLE tk1 (
    cl1 INT NOT NULL,
    cl2 DATE NOT NULL,
    cl3 INT NOT NULL,
    cl4 INT NOT NULL,
    UNIQUE KEY (cl1, cl2, cl3)  -- cl3 포함
)
PARTITION BY HASH(cl3)
PARTITIONS 4;
```

**키가 없는 테이블은 제약 없음**

### 10.2 스토리지 엔진

- MySQL 8에서는 **InnoDB만** 파티셔닝 지원
- 다른 엔진(MyISAM, MEMORY 등)은 파티셔닝 불가

### 10.3 파티셔닝 표현식에 사용 가능한 함수

**허용되는 함수:**

- 수학: `ABS()`, `CEILING()`, `FLOOR()`, `MOD()`
- 날짜/시간: `YEAR()`, `MONTH()`, `DAY()`, `HOUR()`, `MINUTE()`, `SECOND()`
- 날짜 변환: `TO_DAYS()`, `TO_SECONDS()`, `UNIX_TIMESTAMP()`, `TIME_TO_SEC()`
- 요일: `DAYOFWEEK()`, `DAYOFYEAR()`, `WEEKDAY()`
- 기타: `QUARTER()`, `YEARWEEK()`, `EXTRACT()`, `DATEDIFF()`, `MICROSECOND()`

**MySQL 8에서 트렁케이션 지원:**

- `TO_DAYS()`, `TO_SECONDS()`, `YEAR()`, `UNIX_TIMESTAMP()`

## 11. 실무 활용 팁

### 날짜 기반 파티셔닝 (가장 일반적)

```sql
-- 월별 파티셔닝
PARTITION BY RANGE (YEAR(date_column) * 100 + MONTH(date_column))

-- 연도별 파티셔닝
PARTITION BY RANGE (YEAR(date_column))
```

### 파티션 수 선택

- HASH/KEY: 2의 제곱수 추천 (4, 8, 16, 32...)
- RANGE/LIST: 비즈니스 요구사항에 맞게

### 성능 고려사항

- 파티션 프루닝을 활용할 수 있도록 WHERE 절에 파티셔닝 키 포함
- 너무 많은 파티션은 오히려 성능 저하 (수백 개 이상 비추천)
- 파티션별 데이터 크기가 균등하도록 설계

### Location of partition & subpartitions

- [`INFORMATION_SCHEMA.INNODB_TABLESPACES_BRIEF`](https://dev.mysql.com/doc/refman/8.4/en/information-schema-innodb-tablespaces-brief-table.html) table 사용.
	- `NAME`, `PATH` column 사용
```mysql
SELECT NAME, PATH
FROM INFORMATION_SCHEMA.INNODB_TABLESPACES_BRIEF
WHERE NAME LIKE 'bronx/alonzo%';
-- 와일드카드 사용시, LIKE 연산자 사용필수.
```

##### ***`=` `LIKE` 연산자 차이***
1) **`=`** 
- '정확히' 일치하는 값 비교
- 와일드카드 해석x, '문자열' 그대로 인식

2) **`LIKE`**
- 패턴 매칭용 연산자
- %, _ 등의 와일드카드 지원 

***+) NULL handling 시에는...?***
-> `IS NULL` / `IS NOT NULL` 사용.
why?
	NULL 은 값이 존재하지 않는 상태를 의미한다. 
	따라서 다른값과 같음`=`, 다름`< >` 비교가 정의되지 않는다. 

