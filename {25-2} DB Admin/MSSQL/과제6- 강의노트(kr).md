#### >데이터베이스 관리

#### 06. _MSSQL: 서버 설치; sqlcmd 도구 기본; 인코딩 및 콜레이션_

### 설치

본 과정에서는 Debian 계열의 Ubuntu Linux 시스템에서 작업하므로, [`APT`](https://en.wikipedia.org/wiki/APT_\(software\)) 저장소를 사용하여 Microsoft SQL Server를 설치합니다.

Microsoft SQL Server(줄여서 MSSQL) 설치를 준비할 때, 사용할 버전과 업데이트 배포 채널을 결정해야 합니다.

Microsoft는 다양한 버전과 업데이트 배포 채널을 포함하는 저장소를 제공합니다: _Cumulative Update_ (CU), _General Distribution Release_ (GDR). 때로는 _Preview_ 버전도 제공됩니다.

첫 번째 유형은 모든 중요(보안) 및 비중요 수정 사항을 포함합니다. 각 CU는 이전의 모든 CU 업데이트를 포함합니다. GDR 버전은 중요 업데이트만 포함하며 프로덕션 배포용입니다. Preview 버전은 아직 충분히 안정적이지 않은 새로운 소프트웨어 에디션이나 기능을 테스트하는 데 사용됩니다.

**참고**: 텍스트에서 명령줄과 명령 결과 줄을 구분하기 위해 실행되는 명령의 설명을 `$` 기호로 시작하는 것이 일반적입니다. 따라서 자료에 `$`로 시작하는 명령이 있으면, 터미널에 **`$` 기호 <u>다음</u>의 내용을 입력**합니다.

#### 초기 설정

설치 후, [`mssql-conf`](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-configure-mssql-conf?view=sql-server-linux-ver15) 도구를 실행하여 기본 서버 매개변수를 구성합니다:

shell

````shell
$ sudo /opt/mssql/bin/mssql-conf setup
```

Microsoft SQL Server는 상용 제품이므로 초기 구성의 첫 단계는 보유한 라이센스에 따라 서버 모드를 선택하는 것입니다. 사용 가능한 에디션은 다음과 같습니다:
```
  1) Evaluation (무료, 프로덕션 사용 권한 없음, 180일 제한)
  2) Developer (무료, 프로덕션 사용 권한 없음)
  3) Express (무료)
  4) Web (유료)
  5) Standard (유료)
  6) Enterprise (유료) - CPU 코어 사용이 20개 물리/40개 하이퍼스레드로 제한
  7) Enterprise Core (유료) - 운영 체제 최대치까지 CPU 코어 사용
  8) 소매 판매 채널을 통해 라이센스를 구입하여 입력할 제품 키가 있습니다.
````

본 과정에서는 유료 및 개발자 에디션보다 기능이 적지만 무료인 Express 버전으로 충분합니다. 평가판 에디션과 달리 상업적 목적으로 **사용할 수 있습니다**.

에디션을 선택한 후 라이센스 조항을 읽고 동의한 다음 서버 관리자 비밀번호를 입력합니다. 이것은 MSSQL에서 가장 권한이 높은 계정이므로 비밀번호는 복잡해야 합니다 - 구성 스크립트는 최소 8자를 요구합니다.

#### 도구 설치

MySQL과 달리 클라이언트 도구는 서버 설치 시 기본적으로 설치되지 않으며 수동으로 설치해야 합니다.

도구는 별도의 저장소에 있으며, APT에 추가해야 합니다:

```shell
$ wget -qO- https://packages.microsoft.com/config/ubuntu/22.04/prod.list | \
     sudo tee /etc/apt/sources.list.d/msprod.list
```

저장소 상태 업데이트:

```shell
$ sudo apt update
```

사용 가능한 클라이언트 도구 버전 검색:

```shell
$ apt search mssql-tools
```

최신 에디션의 클라이언트 도구 설치:

```shell
$ sudo apt install -y mssql-tools18
```

향후 작업을 쉽게 하기 위해 클라이언트 도구의 설치 경로를 [`PATH`](http://www.linfo.org/path_env_var.html) 변수에 추가합니다:

```shell
$ echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> ~/.bash_profile
$ source ~/.bash_profile
```

모든 것이 올바르게 완료되면 [`sqlcmd`](https://docs.microsoft.com/en-us/sql/tools/sqlcmd-utility?view=sql-server-linux-ver15) 명령이 작동해야 합니다:

```shell
$ sqlcmd -?
```

#### 설치 확인

MSSQL 서버가 수신 대기하는 포트를 확인할 수 있습니다:

```shell
$ sudo netstat -tapnl | grep sqlservr
```

서비스 상태 확인:

```shell
$ sudo systemctl status mssql-server.service
```

#### 테스트

명령줄에서 실행되는 [`sqlcmd`](https://docs.microsoft.com/en-us/sql/tools/sqlcmd-utility?view=sql-server-linux-ver15)를 사용하여 Transact-SQL 스크립트를 실행할 수 있습니다. 예를 들어 [`@@VERSION`](https://docs.microsoft.com/en-us/sql/t-sql/functions/version-transact-sql-configuration-functions?view=sql-server-linux-ver15) 함수를 사용하여 서버 버전 정보를 표시할 수 있습니다.

대화형 세션을 시작하려면:

````shell
$ sqlcmd -S 127.0.0.1 -U sa -C
```
```
Password:
1> SELECT @@VERSION;
2> go
````

도구를 종료하려면 `:exit` 명령을 사용합니다.

#### 구성

주요 구성 파일은 `/var/opt/mssql/mssql.conf`이며, 수동 수정은 권장되지 않습니다. 대신 [`mssql-conf`](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-configure-mssql-conf?view=sql-server-linux-ver15) 도구를 사용합니다.

사용 가능한 구성 옵션 목록 보기:

```shell
$ sudo /opt/mssql/bin/mssql-conf list
```

서버 수신 포트를 기본값 `1433`에서 `14339`로 변경:

```shell
$ sudo /opt/mssql/bin/mssql-conf set network.tcpport 14339
```

구성 유효성 검사:

```shell
$ sudo /opt/mssql/bin/mssql-conf validate
```

서버 재시작 및 새 포트로 연결:

```shell
$ sudo systemctl restart mssql-server.service
$ sqlcmd -S 127.0.0.1,14339 -U sa -C
```

> MSSQL의 서버 이름 매개변수는 표준이 아닌 경우 포트 번호를 포함할 수 있습니다. `<서버>,<포트>` 형식을 사용합니다.

기본값 복원:

```shell
$ sudo /opt/mssql/bin/mssql-conf unset network.tcpport
$ sudo systemctl restart mssql-server.service
```

---

### `sqlcmd` 도구 기본

서버와 통신하기 위해 [`sqlcmd`](https://docs.microsoft.com/en-us/sql/tools/sqlcmd-utility?view=sql-server-linux-ver15)를 사용합니다. 데이터베이스와 테이블 생성, 데이터 선택 등의 기본 작업을 테스트합니다.

대화형 세션 시작:

```shell
$ sqlcmd -S 127.0.0.1 -U sa -C
```

#### 기본 명령

Transact-SQL 명령은 세미콜론 `;`으로 끝나고, 내부 명령은 콜론 `:`으로 시작합니다. `go` 명령은 버퍼의 모든 Transact-SQL 문을 실행합니다.

간단한 SELECT 문:

```sql
SELECT 1;
SELECT 2;
go
```

버퍼 초기화:

```sql
:reset
```

버퍼 내용 표시:

```sql
:list
```

도움말 보기:

```sql
:help
```

**이후 예제에서는 줄 번호를 생략합니다.**

서버 이름과 현재 날짜 확인:

```sql
SELECT @@SERVERNAME name, GETDATE() date;
go
```

더 나은 출력을 위해 `-Y 50` 매개변수와 함께 재시작:

```shell
$ sqlcmd -S localhost -U sa -C -Y 50
```

> 이후 과정의 결과는 `-Y 50` 매개변수 사용을 가정합니다.

#### 🩵데이터베이스 작업

🩵서버의 데이터베이스 이름 확인:
- displays the names of all db on the server
- ['sys.databases'](https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-databases-transact-sql?view=sql-server-ver17) 테이블 활용
```sql
SELECT name FROM sys.databases;
go
```

🩵현재 작업 중인 데이터베이스 확인:

```sql
SELECT DB_NAME();
go
```

🩵테스트 데이터베이스 생성:

```sql
CREATE DATABASE test;
go
```

🩵데이터베이스 선택/변경:

```sql
USE test;
go
```


🩵테이블 목록 표시:

```sql
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES;
go
```
- [INFORMATION_SCHEMA.TABLES](https://www.mssqltips.com/tutorial/information-schema-tables/) 테이블 활용
- 

🩵샘플 테이블 생성:

```sql
CREATE TABLE pet ( 
    name VARCHAR(20), 
    owner VARCHAR(20), 
    species VARCHAR(20), 
    sex CHAR(1), 
    birth DATE, 
    death DATE 
);
go
```
##### >문자열 type (`NVARCHAR`)
`CHAR` : 불변 길이 문자열 (Non-unicode)
`VARCHAR` : 가변 길이 문자열 (non-unicode)
`NVARCHAR` : 가변 길이 **유니코드** 문자열 (한글, 다국어 포함 모든 문자!)

🩵DB/테이블 삭제:
```sql
DROP DATABASE db;

DROP TABLE self_isolations;
-- 현재 db의 테이블 삭제
```


테이블 구조 확인:

```sql
SELECT COLUMN_NAME AS 'Name', DATA_TYPE AS 'Type', CHARACTER_MAXIMUM_LENGTH AS 'Max' 
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE TABLE_NAME = 'pet';
go
```

데이터 삽입:

```sql
INSERT INTO pet VALUES ('Puffball', 'Diane', 'hamster', 'f', '1999-03-30', NULL);
go
```

데이터 조회:

```sql
SELECT * FROM pet;
go
```

데이터 삭제:

```sql
DELETE FROM pet;
go
```

#### 파일에서 SQL 실행

`data.sql` 파일 생성:

```sql
USE test;
go
INSERT INTO pet VALUES 
('Fluffy'  , 'Harold' , 'cat'     , 'f'    , '1993-02-04', NULL),
('Claws'   , 'Gwen'   , 'cat'     , 'm'    , '1994-03-17', NULL),
('Buffy'   , 'Harold' , 'dog'     , 'f'    , '1989-05-13', NULL),
('Fang'    , 'Benny'  , 'dog'     , 'm'    , '1990-08-27', NULL),
('Bowser'  , 'Diane'  , 'dog'     , 'm'    , '1979-08-31', '1995-07-29'),
('Chirpy'  , 'Gwen'   , 'bird'    , 'f'    , '1998-09-11', NULL),
('Whistler', 'Gwen'   , 'bird'    , NULL   , '1997-12-09', NULL),
('Slim'    , 'Benny'  , 'snake'   , 'm'    , '1996-04-29', NULL),
('Puffball', 'Diane'  , 'hamster' , 'f'    , '1999-03-30', NULL);
go
```

세션에서 파일 실행:

```sql
:r data.sql
```

명령줄에서 직접 쿼리 실행:

```shell
$ sqlcmd -S 127.0.0.1 -U sa -C -Q "USE TEST; SELECT * FROM pet WHERE name LIKE '%fy';"
```

파일에서 스크립트 실행:

```shell
$ sqlcmd -S 127.0.0.1 -U sa -C -i end.sql
```

---

### 데이터 인코딩 및 콜레이션

MSSQL에서 **콜레이션**은 문자 집합과 정렬 방법을 모두 정의합니다.

MSSQL 데이터베이스에서는:

- 여러 콜레이션 방법을 사용하여 문자열 저장 가능
- 동일한 서버, 데이터베이스, 열 또는 쿼리에서 다른 콜레이션 방법으로 문자열 저장 및 조작 가능
- 언급된 모든 수준에서 콜레이션 방법 사양 제어 가능

#### 지원되는 콜레이션 확인

````sql
SELECT * FROM sys.fn_helpcollations();
go
```

결과 예시:
```
name             description
---------------- ---------------------
Polish_BIN       Polish, binary sort
Polish_BIN2      Polish, binary code point comparison sort
Polish_CI_AI     Polish, case-insensitive, accent-insensitive
Polish_CI_AI_WS  Polish, case-insensitive, accent-insensitive, width-sensitive
````

#### 콜레이션 이름 구조

`Polish_CI_AI`의 경우:

- 식별자: `Polish`
- 옵션:
    - `_CI` - 대소문자 구분 안 함 (case-insensitive)
    - `_AI` - 악센트 구분 안 함 (accent-insensitive, 예: `ą`를 `a`와 동일하게 처리)
    - `_KS` - 가나 구분 (kana-sensitive)
    - `_WS` - 너비 구분 (width-sensitive)

#### 🩵서버 수준 콜레이션

🩵기본 콜레이션 확인:

```sql
SELECT SERVERPROPERTY('Collation');
go
```

결과: `SQL_Latin1_General_CP1_CI_AS`

#### 데이터베이스 수준 콜레이션

🩵특정 콜레이션으로 데이터베이스 생성:

```sql
USE master;
CREATE DATABASE db_pl COLLATE Polish_CS_AS;
go
```

🩵데이터베이스 콜레이션 확인:
- displays the current configured collation at the db lever
```sql
USE db_pl;
SELECT DATABASEPROPERTYEX('db_pl','collation');
go
```

#### 열 수준 콜레이션

🩵특정 열에 대한 콜레이션 설정:
- 컬럼에 콜레이션 설정해서 테이블 생성하기

```sql
CREATE TABLE t1 ( 
    x VARCHAR(5) COLLATE Czech_CI_AS,
    y VARCHAR(5) COLLATE Slovak_CI_AS 
);
go
```

🩵열 콜레이션 정보 확인:

```sql
SELECT COLUMN_NAME, COLLATION_NAME 
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_CATALOG = 'db_name' AND TABLE_NAME = 'table_name';
go
```


#### 🩵유니코드 문자열 처리

**중요**: 폴란드어와 같은 특수 문자를 저장하려면 `VARCHAR` 대신 `NVARCHAR`를 사용해야 합니다.

잘못된 예 (VARCHAR):

```sql
CREATE DATABASE db_default;
USE db_default;
CREATE TABLE tabela ( imie VARCHAR(20) );
INSERT INTO tabela VALUES (N'Łucja'), (N'Stanisław'), (N'Świetlana');
SELECT * FROM tabela;
go
```

결과: 악센트 누락 (`Lucja`, `Stanislaw`, `Swietlana`)

🩵올바른 예 (NVARCHAR):

```sql
ALTER TABLE tabela ALTER COLUMN imie NVARCHAR(20);
INSERT INTO tabela VALUES (N'Łucja'), (N'Stanisław'), (N'Świetlana'), (N'Leon'), (N'Lucyna');
SELECT * FROM tabela;
go
```

결과: 올바른 문자 표시 (`Łucja`, `Stanisław`, `Świetlana`)
***❗️유의사항* : 여러 컬럼을 동시에 수정 불가 (=/ MySQL 은 가능)**


#### 정렬 순서

기본 콜레이션으로 정렬:

```sql
SELECT * FROM tabela ORDER BY imie;
go
```

특정 콜레이션으로 정렬:

```sql
SELECT * FROM tabela ORDER BY imie COLLATE Polish_CS_AS;
go
```

열 콜레이션 변경:

```sql
ALTER TABLE tabela ALTER COLUMN imie NVARCHAR(20) COLLATE Polish_CS_AS;
go
```

이후 `ORDER BY`는 올바른 폴란드어 정렬 순서를 따릅니다.

---

### 핵심 요약

1. **설치**: Ubuntu에서 APT 저장소를 통해 MSSQL 설치
2. **sqlcmd**: 대화형 및 스크립트 모드로 SQL 실행
3. **콜레이션**:
    - 문자 집합과 정렬 방법을 함께 정의
    - 서버, 데이터베이스, 열 수준에서 설정 가능
    - 유니코드 문자에는 `NVARCHAR` 사용 필수
    - 폴란드어 데이터에는 `Polish_CS_AS` 콜레이션 권장

### 추가 참고 자료

1. [SQL Server on Linux](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-overview?view=sql-server-linux-ver15)
2. [sqlcmd Utility](https://docs.microsoft.com/en-us/sql/tools/sqlcmd-utility?view=sql-server-linux-ver15)
3. [Collation and Unicode support](https://docs.microsoft.com/en-us/sql/relational-databases/collations/collation-and-unicode-support?view=sql-server-linux-ver15)

`qwaesrdtyuiop-[]=≈ vbm,./?>`