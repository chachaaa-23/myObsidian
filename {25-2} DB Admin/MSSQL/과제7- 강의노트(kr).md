#### SQL Server 데이터베이스 관리 - 보안 및 접근 제어

## mssql-cli 도구

### 개요

`mssql-cli`는 Python 기반의 SQL Server 관리 도구로, 기존 `sqlcmd`의 제한사항을 개선하여 다음과 같은 장점을 제공합니다:

- 여러 줄 쿼리 입력 및 붙여넣기 지원
- 향상된 쿼리 결과 표시
- 편리한 빠른 명령어

### 설치 방법


```bash
# 저장소 업데이트
sudo apt update

# Python 패키지 관리자 설치
sudo apt install -y python3-pip

# 수정된 버전 다운로드
curl -o /tmp/mssql_cli-1.0.1-cp310.cp311-none-manylinux1_x86_64.whl \
https://abd.wmi.amu.edu.pl/2025L/mssql_cli-1.0.1-cp310.cp311-none-manylinux1_x86_64.whl

# 패키지 설치 (sudo 없이)
pip install /tmp/mssql_cli-1.0.1-cp310.cp311-none-manylinux1_x86_64.whl

# PATH 변수에 추가
echo 'export PATH="$PATH:~/.local/bin"' >> ~/.bash_profile
source ~/.bash_profile
```

### 기본 사용법

**서버 접속:**

```bash
mssql-cli -S 127.0.0.1 -N -C -U sa
```

- `-N`: 암호화 지원 활성화
- `-C`: 서버 인증서 수락

**주요 기능:**

- `F2`: 자동 완성 켜기/끄기
- `F3`: 여러 줄 모드 켜기/끄기
- `Tab`: 구문 힌트 트리거
- `↑/↓`: 명령어 히스토리 탐색

### 유용한 명령어

|명령어|설명|
|---|---|
|`\?`|도움말 표시|
|`\d OBJECT`|데이터베이스 객체 설명|
|`\ld`|데이터베이스 목록|
|`\lt`|테이블 목록|
|`\lv`|뷰 목록|
|`\ll`|로그인 및 역할 목록|

## SQL Server 보안 모델

### 핵심 개념

SQL Server의 보안 모델은 세 가지 개념을 기반으로 합니다:

1. **권한(Permission)**: 특정 작업을 수행할 수 있는 허가 또는 금지
2. **보안 주체(Principal)**: 권한을 부여받을 수 있는 사용자, 그룹 또는 엔티티
3. **보안 개체(Securable)**: 권한을 할당할 수 있는 객체

**보안 범위 계층:**

- 서버 수준
- 데이터베이스 수준
- 스키마 수준

### 인증 방식

1. **Windows 인증**: Windows 또는 Active Directory 통합
2. **SQL Server 인증**: SQL Server 자체 인증 (Linux 기본값)

## 로그인 계정 관리

### 로그인 계정 조회

```sql
-- SQL 인증 로그인 목록
SELECT name, type_desc, is_disabled
FROM sys.sql_logins;

-- Windows 로그인 목록
SELECT name, type_desc
FROM sys.server_principals
WHERE type_desc LIKE 'WINDOWS%';
```

### 🩵로그인 계정 생성

```sql
CREATE LOGIN uzytkownik WITH PASSWORD = 'P@ssword';
```

### 🩵로그인 계정 수정

```sql
-- 비밀번호 변경 (사용자 본인)
ALTER LOGIN uzytkownik
WITH PASSWORD = 'P@ssword2'
OLD_PASSWORD = 'P@ssword'; -- 생략 가능
```

### 로그인 계정 삭제

```sql
DROP LOGIN LoginDoUsuniecia;
```

## 데이터베이스 사용자 계정

### 사용자 계정의 중요성

로그인 계정만으로는 데이터베이스에 접근할 수 없습니다. 각 데이터베이스에 사용자 계정이 필요합니다.

### 🩵사용자 계정 생성

```sql
USE db_uzytkownik;
CREATE USER uzytkownik FOR LOGIN uzytkownik;
```

### 사용자 계정 조회

```sql
SELECT name, type_desc, authentication_type_desc
FROM sys.database_principals
WHERE type_desc = 'SQL_USER';
```

**기본 계정:**

- `dbo`: 데이터베이스 소유자, 모든 권한 보유
- `guest`: 게스트 계정 (기본적으로 권한 없음)
- `INFORMATION_SCHEMA`, `sys`: 시스템 계정

### 사용자 계정 수정

```sql
ALTER USER uzytkownik WITH NAME = uzytkownik2;
```

### 🩵사용자 계정 삭제
##### 1) 서버 수준의 login account 를 제거할 때
```sql
DROP LOGIN dante;
```

##### 2) DB 수준의 USER 를 제거할 때

```sql
DROP USER LoginToBeDeleted;
```

## 역할(Role) 관리

### 서버 수준 기본 역할

- `sysadmin`: 모든 권한
- `serveradmin`: 서버 구성 변경 및 종료
- `securityadmin`: 로그인 및 권한 관리
- `dbcreator`: 데이터베이스 관리
- `public`: 모든 로그인이 속한 특수 역할
**❗️🩵 `ALTER SERVER ROLE **
서버 관리 권한 부여 시: `ALTER SERVER ROLE`

### 데이터베이스 수준 기본 역할

- `db_owner`: 데이터베이스 모든 권한
- `db_datareader`: 모든 사용자 테이블 읽기 허용
- `db_datawriter`: 데이터 추가, 삭제, 변경 허용
- `db_denydatareader`: 읽기 **금지**
- `db_denydatawriter`: 쓰기 **금지**
- `db_ddladmin`: 모든 DDL 실행 허용
- `db_backupoperator`: 백업 생성 허용
**❗️🩵 `ALTER ROLE`**
- DB 관리 권한 부여 시: `ALTER ROLE`
	- 현재 db내의 로컬 역할을 찾으려 함

### 역할 멤버십 관리


🩵**사용자를 역할에 추가:**
❗️한 line에 한 가지 역할만 한 명에서 부여 가능

```sql
ALTER ROLE db_datareader ADD MEMBER uzytkownik;
```

🩵**사용자를 역할에서 제거:**

```sql
ALTER ROLE db_owner DROP MEMBER uzytkownik;
```

🩵**역할 멤버십 조회:**
[sys.server_role_members](https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-server-role-members-transact-sql?view=sql-server-linux-2017)

| Column name             | Data type | Description                        |
| ----------------------- | --------- | ---------------------------------- |
| **role_principal_id**   | **int**   | Server-Principal ID of the role.   |
| **member_principal_id** | **int**   | Server-Principal ID of the member. |
[sys.server_principals](https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-server-principals-transact-sql?view=sql-server-linux-2017)

| Column name      | Data type   | Description                                            |
| ---------------- | ----------- | ------------------------------------------------------ |
| **name**         | **sysname** | Name of the principal. Is unique within a server.      |
| **principal_id** | **int**     | ID number of the Principal. Is unique within a server. |

```sql
SELECT p1.name AS RoleName,
       p2.name AS UserName
FROM sys.database_role_members AS drm
JOIN sys.database_principals AS p1
  ON drm.role_principal_id = p1.principal_id
JOIN sys.database_principals AS p2
  ON drm.member_principal_id = p2.principal_id;
```

## Contained Database (포함된 데이터베이스)

### 개념

내부 인증을 사용하는 데이터베이스로, 데이터베이스 수준에서 권한을 관리하여 이식성이 향상됩니다.

### 특징

- 서버 수준 리소스 접근 불가
- 연결 해제 없이 다른 데이터베이스 변경 불가

### 설정 및 생성

```sql
-- Contained Database 기능 활성화
USE master;
sp_configure 'contained database authentication', 1;
RECONFIGURE WITH OVERRIDE;

-- Contained Database 생성
CREATE DATABASE db_contained CONTAINMENT = PARTIAL;
```

### 사용자 생성 및 접속

```sql
-- 데이터베이스에 사용자 생성 (로그인 계정 불필요)
USE db_contained;
CREATE USER zwarty WITH PASSWORD = 'iGotowy4';
```

```bash
# 데이터베이스 지정하여 접속
mssql-cli -S 127.0.0.1 -N -C -U zwarty -d db_contained
```

## 권한(Permission) 관리

### 🩵권한 부여
- 여러개의 권한, 한 줄로 동시에 부여 가능!
```sql
-- 테이블 SELECT 권한 부여
GRANT SELECT ON OBJECT::tab2 TO zwarty;
```

--* `OBJECT::` 를 붙이는 이유
	`개체종류::이름` 형식의 범위 한정자임. 
	스키마 범위의 객체(table, view) 라고 명시적으로 선언 (안 써도 ㄱㅊ)
### 🩵권한 거부

```sql
DENY SELECT ON table_name(col_name) TO user_account
  
-- 특정 컬럼 SELECT 거부
DENY SELECT ON OBJECT::tab2(tajne) TO zwarty;
```

**중요:** `DENY` 권한은 항상 `GRANT` 권한보다 우선합니다.

### 권한 취소

```sql
REVOKE SELECT ON OBJECT::tab2 TO zwarty;
```

### 권한 조회

🩵**부여된 권한 확인 (사용자):**
##### 🩵[sys.fn_my_permissions](https://learn.microsoft.com/en-us/sql/relational-databases/system-functions/sys-fn-my-permissions-transact-sql?view=sql-server-linux-ver15)
`fn_my_permissions ( securable, 'securable_class' )`
- securable: 
	권한 확인할 대상 객체 이름 (entity_name)
	multipart name 가능
- securable_class:
	대상의 클래스 지정 (table 등은 객체 - OBJECT)

```sql
SELECT * FROM fn_my_permissions('tab2', 'object');
```

**거부된 권한 확인:**

```sql
SELECT l.name AS grantee_name,
       p.state_desc,
       p.permission_name,
       o.name
FROM sys.database_permissions AS p
JOIN sys.database_principals AS l
  ON p.grantee_principal_id = l.principal_id
JOIN sys.sysobjects O
  ON p.major_id = O.id
WHERE p.state_desc = 'DENY';
```

### Create compact DB (~contained db)
- 모든 인증 정보와 메타데이터를(ex. server 수준에서 만든 login 정보) DB 내부에 직접 가지고 있는 독립적인 형태.
- server 에 의존 x, DB 스스로 사용자를 인증.
- portability
- 보안 이슈, 중복로그인 등의 구조적 위험성 유
```sql
-- 보안 비활성화 후,
EXEC sp_configure 'contained database authentication', 1;
RECONFIGURE WITH OVERRIDE;
-- compact DB 생성
CREATE DATABASE db_contained CONTAINMENT = PARTIAL;
USE db_contained;
```

We can check the compactness of the database using the system view [`sys.databases`](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-databases-transact-sql?view=sql-server-linux-ver15):
```sql
SELECT name,
       containment_desc
FROM sys.databases
WHERE name = 'db_contained';
```


## 실습 예제

### 기본 보안 설정 시나리오

```sql
-- 1. 데이터베이스 생성
CREATE DATABASE db_uzytkownik;
USE db_uzytkownik;

-- 2. 테이블 및 데이터 생성
CREATE TABLE tab1 (x VARCHAR(10), y VARCHAR(20), z VARCHAR(10));
INSERT INTO tab1 VALUES ('wiersz1', 'dane', 'testowe');

-- 3. 로그인 생성
CREATE LOGIN uzytkownik WITH PASSWORD = 'P@ssword';

-- 4. 사용자 생성
CREATE USER uzytkownik FOR LOGIN uzytkownik;

-- 5. 읽기 권한 부여
ALTER ROLE db_datareader ADD MEMBER uzytkownik;
```

### 컬럼 수준 보안 시나리오

```sql
-- 1. Contained Database 생성
CREATE DATABASE db_contained CONTAINMENT = PARTIAL;
USE db_contained;

-- 2. 기밀 및 공개 데이터 테이블 생성
CREATE TABLE tab2 (tajne NVARCHAR(100), publiczne NVARCHAR(100));
INSERT INTO tab2 VALUES (N'tajny wiersz 1', N'publiczny wiersz 1');

-- 3. 사용자 생성
CREATE USER zwarty WITH PASSWORD = 'iGotowy4';

-- 4. 테이블 SELECT 권한 부여
GRANT SELECT ON OBJECT::tab2 TO zwarty;

-- 5. 기밀 컬럼 접근 거부
DENY SELECT ON OBJECT::tab2(tajne) TO zwarty;

-- 결과: 사용자는 publiczne 컬럼만 읽을 수 있음
```

## 핵심 요약

1. **mssql-cli**는 향상된 대화형 SQL Server 관리 도구입니다
2. **로그인 계정**은 서버 접속에 필요하고, **사용자 계정**은 데이터베이스 접근에 필요합니다
3. **역할**을 사용하면 권한 관리가 간편해집니다
4. **DENY는 항상 GRANT보다 우선**합니다
5. **Contained Database**는 이식성이 높은 독립적인 인증을 제공합니다
6. 권한은 서버, 데이터베이스, 스키마, 객체 수준에서 계층적으로 관리됩니다