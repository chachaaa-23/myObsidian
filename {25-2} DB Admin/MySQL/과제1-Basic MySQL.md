##### DB 생성 (MYSQL)
```mysql
CREATE DATABASE mydb;  -- 'mydb' 데이터베이스 생성
-- character set 설정가능(아스키코드, 유니코드 ...)
-- collation method 설정가능(문자열 순서 정렬)
CREATE TABLE mydb_table1 (,,,); -- 테이블 생성

-- ex. 각각의 컬럼을 갖는 테이블 tab1 생성.
	CREATE TABLE tab1 (x INT, y CHAR(3), fin DATE);
-- 테이블 생성 시 저장엔진 별도 설정 가능. 타입 뒤에 `ENGINE = "..."` 붙이기

USE mydb; -- mydb 데이터베이스 선택/변경

SHOW DATABASES; -- MySQL 서버 속 모든 DB들을 보여줌
SHOW TABLES; -- 현재 데이터베이스의 테이블 목록 확인
```

```mysql
SELECT DATABASE(); -- returns the default(current) DB name as a string. 
SELECT * FROM pet; -- 'pet' 테이블의 모든 데이터 조회
-- WHERE 절 통해 어느 테이블을 가져올지 세부설정 가능
-- FROM 없이 단독으로 지정 가능

-- ex. 현재 DB에서 테이블의 이름이 숫자로 끝나는 table을 띄우기
	SELECT TABEL_NAME -- 테이블 이름을 띄우는데,
	FROM INFORMATION_SCHEMA.TABLES -- 전체 서버의 모든 테이블 중
	WHERE table_schema = DATABASE() AND -- 현재 선택된 DB의 테이블 중 찾아
	      REGEXP_LIKE(TABLE_NAME, '[0-9]$'); -- 숫자로 끝나야 해

```

```mysql
DESCRIBE mydb_table1; -- 테이블 discription(정보) 확인 (컬럼 이름, 테이터타입)

SHOW FULL COLLUMNS FROM tab1; -- 테이블의 '모든'정보 표시 (collation, Privileges comment 표시가능)

INSERT INTO mydb_table1 (...) VALUES (,,,) -- 데이터 추가
DELETE FROM mydb_table1; -- 데이터 삭제
UPDATE _table_name_  -- 데이터 수정
SET _column1_ = _value1_, _column2_ = _value2_, ...  
WHERE _condition_;

ALTER TABLE tab_name <작업명>;
-- DB 수정시에는 ALTER DATABASE db_name <작업명>;
-- 데이터가 저장되거나 구성되는 구조와 정의 자체 변경o (데이터 자체 바꾸기x)
```

* 서버 아래에(ex. 차차상사) 여러개의 DB가 존재하고(ex. 인사 DB, 재고 DB), 하나의 DB 아래에 여러개의 테이블이 존재한다 (ex. 육류재고 테이블, 와인재고 테이블)
- mysql 명령어들은 뒤에 꼭 세미콜론을 붙여야 함 (=/ ssms)

(참고: https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_database)