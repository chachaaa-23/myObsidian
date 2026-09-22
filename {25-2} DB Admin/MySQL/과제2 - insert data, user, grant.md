##### User 생성
```mysql
CREATE USER 'andy'@'%' IDENTIFIED BY 'pazzw0Rd';
-- andy 라는 이름의 유저, 모든 호스트 접근가능, 비밀번호 = pazzw0Rd
```

##### 권한 부여
`SELECT` : view rows
```mysql
-- 모든 권한 부여
GRANT ALL
   ON *.*
   TO 'finley'@'localhost'
 WITH GRANT OPTION;

-- 🩵특정 권한 부여
GRANT SELECT, INSERT, UPDATE
   ON database_name.*
   TO 'username'@'hostname';

-- 🩵테이블 수준 권한
GRANT SELECT, INSERT
   ON database_name.table_name
   TO 'username'@'hostname';

-- 🩵컬럼 수준 권한
GRANT SELECT (col1), INSERT (col1, col2)
   ON database_name.table_name
   TO 'username'@'hostname';
```

--> 주요 권한 목록

| 권한             | 설명                            |
| -------------- | ----------------------------- |
| `ALL`          | 모든 기본 권한 부여 (GRANT OPTION 제외) |
| `SELECT`       | 테이블에서 행 조회                    |
| `INSERT`       | 테이블에 행 삽입                     |
| `UPDATE`       | 테이블의 행 수정                     |
| `DELETE`       | 테이블에서 행 삭제                    |
| `CREATE`       | 테이블 생성                        |
| `DROP`         | 데이터베이스, 테이블, 뷰 삭제             |
| `ALTER`        | 테이블 구조 변경                     |
| `INDEX`        | 인덱스 생성 및 삭제                   |
| `GRANT OPTION` | 자신이 가진 권한을 다른 사용자에게 부여/취소     |
| `CREATE USER`  | 사용자 계정 관리                     |
| `SUPER`        | 고급 관리 작업 (KILL, SET GLOBAL 등) |
| `RELOAD`       | FLUSH 명령 사용                   |
| `SHUTDOWN`     | 서버 종료                         |
##### 권한 확인

```mysql
-- 권한 확인
SHOW GRANTS FOR 'username'@'hostname';

-- 역할 포함하여 확인
SHOW GRANTS FOR 'username'@'hostname' 
          USING 'role_name';

-- 계정 속성 확인
-- 유저명, 비번, ... 알려준다
SHOW CREATE USER 'username'@'hostname'\G
```

#### 권한 취소

```mysql
-- 글로벌 권한 취소
REVOKE ALL
    ON *.*
  FROM 'username'@'hostname';

-- 데이터베이스 권한 취소
REVOKE CREATE, DROP
    ON database_name.*
  FROM 'username'@'hostname';

-- 테이블 권한 취소
REVOKE INSERT, UPDATE, DELETE
    ON database_name.table_name
  FROM 'username'@'hostname';
```

#### 계정 삭제

```mysql
DROP USER 'username'@'hostname';
```

#### View
- virtual tables that are used to view data from one or more tables
- 실제 데이터(row)를 저장하는 물리적인 테이블x
- 실제 테이블에 기반하여 정의된 테이블
- view 가 정의된 원본 테이블에서 데이터를 가져와 보여줌
- view 생성할 때 지정한 `SELECT` 쿼리문만 DB 에 저장됨. 

```mysql
CREATE VIEW <view_name> AS
SELECT <col1>, <col2>, ...
FROM <table-name>
WHERE [conditions];
```


#### 역할 생성

```mysql
CREATE ROLE 'app_developer', 
            'app_read',
            'app_write';
```

#### 역할에 권한 부여
-> DB와 table에 권한 부여하는 과정과 비슷.

```mysql
GRANT ALL
   ON app_db.* 
   TO 'app_developer';
   
GRANT SELECT 
   ON app_db.view_table
   TO 'app_read';
   
GRANT INSERT, UPDATE, DELETE 
   ON app_db.table_name 
   TO 'app_write';
```

#### 사용자에게 역할 할당

```mysql
-- 계정 생성
CREATE USER 'dev1'@'localhost' 
IDENTIFIED BY 'dev1pass';

-- 역할 할당
GRANT 'app_developer'
   TO 'dev1'@'localhost';
```

#### 역할 활성화

```mysql
-- 기본 역할 설정
SET DEFAULT ROLE ALL
              TO 'dev1'@'localhost';

-- 세션 내 역할 변경
SET ROLE NONE;                    -- 모든 역할 비활성화
SET ROLE ALL EXCEPT 'app_write';  -- 특정 역할 제외
SET ROLE DEFAULT;                 -- 기본 역할로 복원
SET ROLE 'read_view';             -- 특정 역할로 역할 변경

-- 🩵현재 활성 역할 확인
SELECT CURRENT_ROLE();
```


#### 역할 권한 취소

```mysql
-- 역할에서 권한 취소
REVOKE INSERT, UPDATE, DELETE
    ON app_db.*
  FROM 'app_write';

-- 사용자로부터 역할 취소
REVOKE 'app_write' FROM 'username'@'hostname';
```

#### 역할 삭제

```mysql
DROP ROLE 'app_read', 'app_write';
```


---
#### 🩵 **`mysql`** 
- database tables contain grant information
-  **Grant** table : may contain columns used for purposes other than scope or previlige assessment.
(*scope columns 범위컬럼 : 권한이 누구에게, 어디에서, 어느 db에 적용되는지 정의)
(*Privilege columns : 해당 범위 내에서 허용되는 구체적인 작업 정의. 값 주로 Y/N)

--> mysql.**db**, mysql.**tables_priv**, (.user, .global_grants, .columns_priv, ...)
##### 1) [**`mysql.db`** ](https://dev.mysql.com/doc/refman/8.0/en/grant-tables.html#grant-tables-tables-priv-columns-priv)
(+`mysql.user`)
	: db table scope columns determine which user can access which databases from which hosts. 

| Table Name                   | `user`                     | `db`                    |
| ---------------------------- | -------------------------- | ----------------------- |
| **Scope columns**            | `Host`                     | `Host`                  |
|                              | `User`                     | `Db`                    |
|                              |                            | `User`                  |
| **Privilege columns**        | `Select_priv`              | `Select_priv`           |
|                              | `Insert_priv`              | `Insert_priv`           |
|                              | `Update_priv`              | `Update_priv`           |
|                              | `Delete_priv`              | `Delete_priv`           |
|                              | `Index_priv`               | `Index_priv`            |
|                              |                            |                         |
|                              | `Alter_priv`               | `Alter_priv`            |
|                              | `Create_priv`              | `Create_priv`           |
|                              | `Drop_priv`                | `Drop_priv`             |
|                              | `Grant_priv`               | `Grant_priv`            |
|                              | `Create_view_priv`         | `Create_view_priv`      |
|                              | `Show_view_priv`           | `Show_view_priv`        |
|                              | `Create_routine_priv`      | `Create_routine_priv`   |
|                              | `Alter_routine_priv`       | `Alter_routine_priv`    |
|                              | `Execute_priv`             | `Execute_priv`          |
|                              | `Trigger_priv`             | `Trigger_priv`          |
|                              | `Event_priv`               | `Event_priv`            |
|                              | `Create_tmp_table_priv`    | `Create_tmp_table_priv` |
|                              | `Lock_tables_priv`         | `Lock_tables_priv`      |
|                              | `References_priv`          | `References_priv`       |
|                              | `Reload_priv`              |                         |
|                              | `Shutdown_priv`            |                         |
|                              | `Process_priv`             |                         |
|                              | `File_priv`                |                         |
|                              | `Show_db_priv`             |                         |
|                              | `Super_priv`               |                         |
|                              | `Repl_slave_priv`          |                         |
|                              | `Repl_client_priv`         |                         |
|                              | `Create_user_priv`         |                         |
|                              | `Create_tablespace_priv`   |                         |
|                              | `Create_role_priv`         |                         |
|                              | `Drop_role_priv`           |                         |
|                              |                            |                         |
| **Security columns**         | `ssl_type`                 | x                       |
|                              | `ssl_cipher`               |                         |
|                              | `x509_issuer`              |                         |
|                              | `x509_subject`             |                         |
|                              | `plugin`                   |                         |
|                              | `authentication_string`    |                         |
|                              | **`password_expired`**     |                         |
|                              | `password_last_changed`    |                         |
|                              | `password_lifetime`        |                         |
|                              | `account_locked`           |                         |
|                              | `Password_reuse_history`   |                         |
|                              | `Password_reuse_time`      |                         |
|                              | `Password_require_current` |                         |
|                              | `User_attributes`          |                         |
| **Resource control columns** | `max_questions`            | x                       |
|                              | `max_updates`              |                         |
|                              | `max_connections`          |                         |
|                              | `max_user_connections`     |                         |

##### 2) 🩵 **`mysql.tables_priv`**
(+`mysql.columns_priv`)
	: db 테이블과 비슷하지만 세분화됨
	they apply at the table and column levels rather tham at the db level.

| Table Name            | `tables_priv` | `columns_priv` |
| --------------------- | ------------- | -------------- |
| **Scope columns**     | `Host`        | `Host`         |
|                       | `Db`          | `Db`           |
|                       | `User`        | `User`         |
|                       | `Table_name`  | `Table_name`   |
|                       |               | `Column_name`  |
| **Privilege columns** | `Table_priv`  | `Column_priv`  |
|                       | `Column_priv` |                |
| **Other columns**     | `Timestamp`   | `Timestamp`    |
|                       | `Grantor`     |                |

##### 3) 🩵 `mysql.proxies_priv`
	: records information about proxy accounts.

| Table Name  | `proxies_priv`                 |
| ----------- | ------------------------------ |
| **columns** | `Host`, `User`                 |
|             | `Proxied_host`, `Proxied_user` |
|             | `Grantor`, `Timestamp`         |
|             | `With_grant`                   |

(참고 - MySQL 공식문서 'set-type privilege column values' https://dev.mysql.com/doc/refman/8.0/en/grant-tables.html#grant-tables-tables-priv-columns-priv)


**step-up example**
~ 유저가 특정 table의 DROP 권한을 가지고 있는지 확인하고, 결과를 true/false 로 띄워라 (use privilege columns)
```mysql
SELECT IF(LOCATE('Drop', Table_priv) > 0, 'true', 'false') AS 'DROP' -- Table_priv 에 권한 존재하는지 체크
FROM mysql.tables_priv -- 사용자의 테이블 단위 권한정보 저장되있는 시스템 테이블
WHERE User = 'andy' AND
	  Host = '%' AND
	  Db = 'smart_data' AND
	  Table_name = 'animals';
	  
```

`IF(..., '참', '거짓') AS 'COL' ...`
	: 첫번째 인자가 참이면 '참', 거짓이면 '거짓' 을 최종결과로 출력함. 이때 컬럼이름은 'COL'

`LOCATE('Drop', Table_priv) > 0`
	: 첫번째 문자열이 두 번째 문자열 안에 존재하는지 확인
	있으면 1이상의 수를, 없으면 0을 반환

`Table_priv` 
	: 부여된 권한들이 저장되는 `SET` 타입의 컬럼.

|Table Name|Column Name|Possible Set Elements|
|---|---|---|
|`tables_priv`|`Table_priv`|`'Select', 'Insert', 'Update', 'Delete', 'Create', 'Drop', 'Grant', 'References', 'Index', 'Alter', 'Create View', 'Show view', 'Trigger'`|

---
#### The role_edges Grant Table

**`mysql.role_edges`**: MySQL이 **역할-사용자 간의 연결(assignment)** 정보를 저장하는 내부 시스템 테이블입니다. (어떤 역할이 누구에게 부여됐는가?)
- The `role_edges` table lists edges for role subgraphs.

- `FROM_HOST`, `FROM_USER`: The account that is granted a role. (FROM: 출발점, 역할 자체)
- `TO_HOST`, `TO_USER`: The role that is granted to the account. (TO: 도착점, 역할을 받는 계정)
- `WITH_ADMIN_OPTION`: Whether the account can grant the role to and revoke it from other accounts by using `WITH ADMIN OPTION`.

**`CONCAT` 함수**
- 둘 이상의 문자열을 결합해 하나의 문자열로 반환
`CONCAT (string1, string2, ...)`;


