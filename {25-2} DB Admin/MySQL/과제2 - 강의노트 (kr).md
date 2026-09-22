#### >02. MySQL: 접근 제어, 권한 테이블, 계정, 역할

### 🩵결과의 수직 표시

쿼리를 `\G`로 끝내면 [`mysql`](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) 클라이언트가 결과를 수직으로 표시합니다. 많은 수의 컬럼을 표시할 때 특히 유용합니다.

**표준 쿼리:**

```mysql
USE db_chs;

SELECT table_name, 
       Engine,
       table_collation
FROM   information_schema.tables
WHERE  table_schema = DATABASE();
```

**`\G`를 사용한 버전:**

````mysql
SELECT table_name, 
       Engine,
       table_collation
FROM   information_schema.tables
WHERE  table_schema = DATABASE()\G
````

수직 형식으로 각 행의 컬럼이 별도의 줄에 표시됩니다.

---

## 접근 제어 및 계정 관리

MySQL은 클라이언트 사용자가 서버에 연결하고 데이터에 접근할 수 있도록 계정을 생성할 수 있습니다. MySQL 권한 시스템의 기본 기능은 특정 호스트에서 연결하는 사용자를 인증하고 해당 사용자를 데이터베이스의 권한(`SELECT`, `INSERT`, `UPDATE`, `DELETE` 등)과 연결하는 것입니다.

### 주요 특징

- **계정 식별**: 사용자 이름과 연결 호스트 모두로 식별
- **권한 관리**: SQL 문(`CREATE USER`, `GRANT`, `REVOKE`)을 통한 관리
- **호스트별 구분**: 같은 사용자 이름이라도 다른 호스트에서 연결하면 다른 권한 부여 가능

**예시:**
- `joe@office.example.com`과 `joe@home.example.com`은 서로 다른 계정

### 접근 제어의 두 단계

1. **연결 검증**: 서버가 제공된 ID와 비밀번호를 기반으로 연결 수락/거부
2. **요청 검증**: 연결 후 각 명령에 대해 충분한 권한이 있는지 확인

---

### 사용자 이름과 비밀번호

### MySQL과 운영체제의 차이점

- **사용자 이름**: MySQL 사용자 이름은 OS 로그인 이름과 무관
- **최대 길이**: MySQL 사용자 이름은 최대 32자
- **비밀번호**: MySQL 비밀번호는 OS 비밀번호와 별개
- **저장 방식**: `mysql.user` 테이블에 암호화되어 저장

---

## MySQL의 권한

권한은 적용 컨텍스트와 작업 수준에 따라 다릅니다:

- **관리 권한**: 글로벌, 서버 운영 관리
- **데이터베이스 권한**: 특정 데이터베이스 및 그 안의 모든 객체에 적용
- **객체 권한**: 테이블, 뷰, 저장 프로시저 등에 적용

### 🩵주요 권한 목록

| 권한 | 설명 |
|------|------|
| `ALL` | 모든 기본 권한 부여 (GRANT OPTION 제외) |
| `SELECT` | 테이블에서 행 조회 |
| `INSERT` | 테이블에 행 삽입 |
| `UPDATE` | 테이블의 행 수정 |
| `DELETE` | 테이블에서 행 삭제 |
| `CREATE` | 테이블 생성 |
| `DROP` | 데이터베이스, 테이블, 뷰 삭제 |
| `ALTER` | 테이블 구조 변경 |
| `INDEX` | 인덱스 생성 및 삭제 |
| `GRANT OPTION` | 자신이 가진 권한을 다른 사용자에게 부여/취소 |
| `CREATE USER` | 사용자 계정 관리 |
| `SUPER` | 고급 관리 작업 (KILL, SET GLOBAL 등) |
| `RELOAD` | FLUSH 명령 사용 |
| `SHUTDOWN` | 서버 종료 |

---

## 권한 테이블

`mysql` 시스템 데이터베이스의 주요 권한 테이블:

- **`user`**: 사용자 계정, 글로벌 정적 권한
- **`global_grants`**: 글로벌 동적 권한
- **`db`**: 데이터베이스 수준 권한
- **`tables_priv`**: 테이블 수준 권한
- **`columns_priv`**: 컬럼 수준 권한
- **`procs_priv`**: 저장 프로시저/함수 권한

**중요**: 직접 수정(`INSERT`, `UPDATE`, `DELETE`)은 권장되지 않으며, 계정 관리 명령문 사용 권장

---

## 🩵계정 이름 지정

### 🩵구문

```
'username'@'hostname'
````

### 🩵규칙

- 사용자 이름만 지정하면 `'username'@'%'`와 동일
- 특수 문자가 있으면 따옴표 필요
- 호스트 부분에 와일드카드 사용 가능:
    - `%`: 모든 호스트
    - `_`: 단일 문자
    - 예: `'%.example.com'`, `'198.51.100.%'`

### 예시

|User|Host|허용되는 연결|
|---|---|---|
|`'fred'`|`'h1.example.net'`|h1.example.net에서 fred|
|`''`|`'h1.example.net'`|h1.example.net에서 모든 사용자|
|`'fred'`|`'%'`|모든 호스트에서 fred|
|`'fred'`|`'%.example.net'`|example.net 도메인에서 fred|

---

## 데이터베이스 연결

### 명령줄 클라이언트 사용

```shell
$ mysql --user=username --password database_name
# 또는 짧은 옵션
$ mysql -u username -p database_name
```

---

## 역할(Role) 이름 정의

역할은 권한 집합을 나타내는 이름입니다.

### 역할과 계정의 차이점

- 역할의 사용자 이름 부분은 비어있을 수 없음
- 호스트 부분 생략 시 `'%'`가 기본값 (와일드카드 속성 없음)
- 네트워크 마스크 표기법 무관

---

## 접근 제어 상세

### 1단계: 연결 검증

서버는 다음을 확인합니다:
- 사용자 신원 및 비밀번호
- 계정 잠금 상태

`mysql.user` 테이블의 `Host`, `User`, `authentication_string` 컬럼을 사용하여 검증합니다.

### 매칭 규칙

서버는 user 테이블을 정렬하여 메모리에 로드:
1. 가장 구체적인 호스트 값 우선
2. 같은 호스트 값이면 가장 구체적인 사용자 값 우선
3. 첫 번째 매칭 행 사용

**예시:**

정렬 전:
```
+-----------+----------+
| Host      | User     |
+-----------+----------+
| %         | root     |
| %         | jeffrey  |
| localhost | root     |
| localhost |          |
+-----------+----------+
```

정렬 후:
```
+-----------+----------+
| Host      | User     |
+-----------+----------+
| localhost | root     |
| localhost |          |
| %         | jeffrey  |
| %         | root     |
+-----------+----------+
```

### 2단계: 요청 검증

연결 후 각 요청마다 권한 확인:

1. **관리 작업**: `mysql.user` 및 `global_grants` 테이블만 확인
2. **데이터베이스 작업**: 
   - 글로벌 권한 확인 (`mysql.user`)
   - 데이터베이스 권한 확인 (`mysql.db`)
   - 테이블 권한 확인 (`mysql.tables_priv`)
   - 컬럼 권한 확인 (`mysql.columns_priv`)

**권한 계산 순서:**
```
글로벌 권한
OR (데이터베이스 권한 AND 호스트 권한)
OR 테이블 권한
OR 컬럼 권한
OR 프로시저 권한
````

---

## 🩵계정 생성 및 권한 부여

### 계정 생성

```mysql
CREATE USER 'finley'@'localhost'
IDENTIFIED BY 'Pa$sw0rd';
```

### 🩵권한 부여
`SELECT` : view rows

```mysql
-- 모든 권한 부여
GRANT ALL
   ON *.*
   TO 'finley'@'localhost'
 WITH GRANT OPTION;

-- 특정 권한 부여
GRANT SELECT, INSERT, UPDATE
   ON database_name.*
   TO 'username'@'hostname';

-- 테이블 수준 권한
GRANT SELECT, INSERT
   ON database_name.table_name
   TO 'username'@'hostname';

-- 컬럼 수준 권한
GRANT SELECT (col1), INSERT (col1, col2)
   ON database_name.table_name
   TO 'username'@'hostname';
```

---

## 🩵권한 확인 (user/role permission)

```mysql
-- 권한 확인
SHOW GRANTS FOR 'username'@'hostname';

-- 🩵역할 포함하여 확인
SHOW GRANTS FOR 'username'@'hostname' 
          USING 'role_name';

-- 계정 속성 확인
SHOW CREATE USER 'username'@'hostname'\G
```

---

## 🩵권한 취소

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

---

## 🩵계정 삭제

```mysql
DROP USER 'username'@'hostname';
```

---

## 예약된 계정

MySQL 설치 시 생성되는 특수 계정:

- **`'root'@'localhost'`**: 관리 목적, 모든 권한 보유
- **`'mysql.sys'@'localhost'`**: sys 데이터베이스 객체용 DEFINER
- **`'mysql.session'@'localhost'`**: 플러그인의 내부 사용
- **`'mysql.infoschema'@'localhost'`**: INFORMATION_SCHEMA 뷰용 DEFINER

---

## 역할(Role) 사용

역할은 명명된 권한 집합입니다.

### 🩵역할 생성

```mysql
CREATE ROLE 'app_developer', 
            'app_read',
            'app_write';
```

### 🩵역할에 권한 부여

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

### 🩵사용자에게 역할 할당

```mysql
-- 계정 생성
CREATE USER 'dev1'@'localhost' 
IDENTIFIED BY 'dev1pass';

-- 역할 할당
GRANT 'app_developer'
   TO 'dev1'@'localhost';
```

### 🩵역할 활성화

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

### 🩵역할 권한 취소

```mysql
-- 역할에서 권한 취소
REVOKE INSERT, UPDATE, DELETE
    ON app_db.*
  FROM 'app_write';

-- 사용자로부터 역할 취소
REVOKE 'app_write' FROM 'username'@'hostname';
```

### 🩵역할 삭제

```mysql
DROP ROLE 'app_read', 'app_write';
```

---

## 사용자와 역할의 상호 교환성

MySQL에서 사용자와 역할은 상호 교환 가능합니다:

```mysql
CREATE USER 'u1';
CREATE ROLE 'r1';
GRANT SELECT ON db1.* TO 'u1';
GRANT SELECT ON db2.* TO 'r1';

-- 사용자에게 사용자/역할 할당 가능
GRANT 'u1', 'r1' TO 'u2';

-- 역할에게 사용자/역할 할당 가능
GRANT 'u1', 'r1' TO 'r2';
```

### 주요 차이점

- **`CREATE ROLE`**: 기본적으로 잠긴 상태로 생성
- **`CREATE USER`**: 기본적으로 잠기지 않은 상태로 생성

---

## 권한 변경 적용 시점

### 자동 적용

계정 관리 명령문 사용 시 즉시 적용:

- `GRANT`, `REVOKE`
- `SET PASSWORD`
- `RENAME USER`

### 수동 재로드 필요

직접 테이블 수정 시 재로드 필요:

```mysql
FLUSH PRIVILEGES;
```

또는 명령줄:

```shell
$ mysqladmin flush-privileges
$ mysqladmin reload
```

### 세션별 적용 시점

- **테이블/컬럼 권한**: 다음 클라이언트 요청부터
- **데이터베이스 권한**: 다음 `USE dbname` 실행 시
- **글로벌 권한/비밀번호**: 새 연결부터

---

## 🩵비밀번호 할당

### 🩵새 계정 생성 시

```mysql
CREATE USER 'jeffrey'@'localhost'
IDENTIFIED BY 'password';
```

### 🩵기존 계정 변경

```mysql
ALTER USER 'jeffrey'@'localhost'
IDENTIFIED BY 'password';
```

### 자신의 비밀번호 변경

```mysql
ALTER USER USER()
IDENTIFIED BY 'new_password';
```
