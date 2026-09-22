### 데이터베이스 관리

## 08. _MSSQL: 데이터베이스 스키마 관리, 데이터 마스킹 기법_ [실습]

### 데이터베이스 스키마

SQL Server 데이터베이스에는 테이블과 같이 우리가 이미 알고 있는 객체 유형 외에도 스키마(schema)라는 특수 객체가 있습니다.

**스키마는** 데이터베이스 내에서 네임스페이스로 **논리적 분할**을 가능하게 하며, 객체를 **소유자로부터 분리**할 수 있게 합니다. 스키마는 **객체가** **생성되는 컨테이너**입니다.

SQL Server 보안 모델은 스키마를 사용하여 상속을 통해 권한 관리를 간소화할 수 있도록 합니다.

새로운 데이터베이스를 테이블과 샘플 데이터와 함께 생성해봅시다. 또한 이전 장의 `uzytkownik` 로그인 계정을 기반으로 `db_datareader`와 `db_ddladmin` 역할에 속하는 사용자 계정을 생성하겠습니다:

```sql
USE master;
go

CREATE DATABASE mssql08;
go

USE mssql08;
go

CREATE USER uzytkownik FOR LOGIN uzytkownik;
go
ALTER ROLE db_datareader ADD MEMBER uzytkownik;
ALTER ROLE db_ddladmin ADD MEMBER uzytkownik;
go

CREATE TABLE bank
(
  id              INT PRIMARY KEY,
  imie            VARCHAR(50), -- 이름
  nazwisko        VARCHAR(50), -- 성
  nazwisko_matki  VARCHAR(50), -- 어머니 성
  PESEL           CHAR(11),    -- 주민등록번호
  data_urodzenia  DATE,        -- 생년월일
  stan_konta      MONEY,       -- 계좌 잔액
  email           VARCHAR(50), -- 이메일 주소
  numer_karty     VARCHAR(16)  -- 신용카드 번호
);
go

INSERT INTO bank VALUES 
(1, 'Armando' ,'Gibbs'   ,'Riggs' , '60052349726', '1960-05-23', '27480',
    'iquet@primis.org'  , '9893436755178541'),
(2, 'Hakeem'  ,'Roman'   ,'Ramsey', '71030954241', '1971-03-09', '41798',
    'arcu@malesu.org'   , '9091680525982543'),
(3, 'Elliott' ,'Faulkner','Walls' , '78121066182', '1978-12-10', '40336',
    'sit@ultric.ca'     , '0628218134030380'),
(4, 'Charissa','Bernard' ,'Maddox', '64110432333', '1964-11-04', '39271',
    'risus@ssim.com'    , '6059056249431781'),
(5, 'Azalia'  ,'Stokes'  ,'Juarez', '87012837521', '1987-01-28', '79705',
    'aliquam@lectus.com', '2288549149712106');
go
```

데이터베이스에서 특정 사용자 계정의 기본 스키마를 확인하려면 [`SCHEMA_NAME()`](https://docs.microsoft.com/en-us/sql/t-sql/functions/schema-name-transact-sql?view=sql-server-linux-ver15) 함수를 사용합니다.

```sql
SELECT SCHEMA_NAME() AS CurrentSchema;
```

| CurrentSchema |
| ------------- |
| dbo           |

`dbo` 스키마는 `dbo` 사용자 계정의 기본 스키마이며, 이는 데이터베이스에 연결하는 데 사용된 `sa` 로그인 계정을 매핑한 후 작업하는 계정입니다.

지금까지 사용한 대부분의 데이터베이스 수준 명령은 이 스키마에서 작동했습니다. 특정 사용자 계정이 생성한 객체는 초기에 해당 계정의 기본 스키마로 설정된 스키마에 위치합니다. 이 메커니즘은 강의 후반부에서 살펴보겠습니다.

SQL Server에서 객체는 다음 형식의 정규화된 이름을 사용하여 지정됩니다: `instance_name.database_name.schema_name.object_name`.

`instance_name`은 서버 이름이며, [`@@SERVERNAME`](https://docs.microsoft.com/en-us/sql/t-sql/functions/servername-transact-sql?view=sql-server-linux-ver15) 함수를 사용하여 확인할 수 있습니다:

```sql
SELECT @@SERVERNAME as Name;
```

| Name |
| ---- |
| abd  |

`mssql08` 데이터베이스의 `bank` 테이블은 다음과 같은 방법으로 지정할 수 있습니다:

```sql
SELECT * FROM bank;
```

이 경우 서버는 우리 사용자 계정(`sa` 로그인 계정에 매핑된 `dbo`)의 기본 스키마를 사용하여 정규화된 이름을 자동으로 완성하지만, 직접 지정할 수도 있습니다:

```sql
SELECT * FROM dbo.bank;
```

전체 정규화된 테이블 이름을 지정할 수도 있으며, 이는 예를 들어 `mssql08` 데이터베이스 외부에서 쿼리를 호출할 수 있게 합니다:

```sql
USE master;
SELECT * FROM abd.mssql08.dbo.bank;
```

매우 고급 SQL Server 토폴로지에서는 서버 간에 Transact-SQL 표현식을 실행할 수 있지만, 이러한 솔루션의 구성은 이 강의의 범위를 벗어납니다.

일반적으로 `schema_name.object_name` 형식이 대부분의 경우 데이터베이스에서 객체를 지정하는 최적의 방법으로 받아들여집니다.

#### 🩵데이터베이스 스키마 생성

데이터베이스에서 스키마를 생성하려면 `CREATE SCHEMA` 권한이 있어야 합니다. 데이터베이스의 다른 사용자 계정에 대해 스키마를 생성하려면 생성하는 계정에 대상 계정에 대한 `IMPERSONATE` 권한도 있어야 합니다. 스키마가 데이터베이스 수준 역할(사용자 계정 대신)의 소유가 되려면 해당 역할의 멤버십 또는 해당 역할에 대한 `ALTER` 권한이 필요합니다.

`sa` 로그인 계정에서 작업할 때는 당연히 이러한 권한이 있습니다.

🩵[`CREATE SCHEMA`](https://docs.microsoft.com/en-us/sql/t-sql/statements/create-schema-transact-sql?view=sql-server-linux-ver15)를 사용하여 `MojSchemat` 스키마를 생성해봅시다:
*❗️여러 개의 스키마 생성 시, batch 분리를 위해 GO 사용 필요*

```sql
USE mssql08;
CREATE SCHEMA MojSchemat;
```

#### 데이터베이스 스키마 수정

시스템 뷰 [`sys.schemas`](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/schemas-catalog-views-sys-schemas?view=sql-server-linux-ver15)를 쿼리하여 데이터베이스의 스키마 목록을 표시할 수 있습니다:

```sql
SELECT * FROM sys.schemas;
```

기본적으로 데이터베이스에는 삭제할 수 없는 중요한 스키마인 `dbo`, `guest`, `sys`, `INFORMATION_SCHEMA`와 데이터베이스 수준의 역할에 해당하는 스키마가 포함되어 있습니다. 이들은 하위 호환성을 위해 생성되며 사용되지 않는 경우 삭제할 수 있습니다.

`MojSchemat` 스키마가 `dbo`와 동일한 `principal_id`를 가지고 있다는 점에 주목하세요. 이는 **새로 생성된 스키마의 소유자**가 기본적으로 **이를 생성한 사용자**(우리의 경우 `sa` 로그인 계정에 매핑된 `dbo` 사용자 계정)임을 의미합니다. 그러나 `AUTHORIZATION` 표현식을 사용하여 특정 사용자에 대한 스키마를 생성할 수 있습니다.

🩵[`ALTER AUTHORIZATION`](https://docs.microsoft.com/en-us/sql/t-sql/statements/alter-authorization-transact-sql?view=sql-server-linux-ver15)을 사용하여 `MojSchemat` 스키마의 소유자를 `uzytkownik`으로 변경해봅시다:
❗️`schema::` 범위 한정자 필수!
```sql
ALTER AUTHORIZATION ON schema::MojSchemat TO uzytkownik;
```

스키마의 새 소유자를 확인해봅시다:

```sql
SELECT  s.name as SchemaName,
        p.name as Principal
FROM    sys.schemas as s
        JOIN sys.database_principals AS p
          ON s.principal_id = p.principal_id
WHERE s.name = 'MojSchemat';
```

| SchemaName   | Principal   |
|--------------+-------------|
| MojSchemat   | uzytkownik  |


#### 🩵Default Schema기본 사용자 계정 스키마
`IMPERSONATE` 권한이 있는 사용자 계정은 로그인 계정과 사용자 계정 모두의 ID를 가정할 수 있습니다. [`EXECUTE AS`](https://docs.microsoft.com/en-us/sql/t-sql/statements/execute-as-transact-sql?view=sql-server-linux-ver15)를 사용하여 사용자 가장을 통해 기본 스키마를 확인해봅시다. 원래 보안 컨텍스트로 돌아가려면 `REVERT` 표현식을 사용합니다:

```sql
EXECUTE AS USER = 'uzytkownik';
SELECT SCHEMA_NAME() AS CurrentSchema;
REVERT;
```



| CurrentSchema   |
|-----------------|
| dbo             |

기본 사용자 스키마는 `dbo`이므로 사용자가 생성한 모든 객체는 이 스키마에 위치합니다. `uzytkownik` 사용자 계정으로 테이블을 생성해봅시다:

```sql
EXECUTE AS USER = 'uzytkownik';
CREATE TABLE tabela1 ( imie VARCHAR(20) );
REVERT;
```

객체(우리의 경우 테이블)에 대한 스키마 할당을 확인하려면 시스템 뷰 [`sys.objects`](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?view=sql-server-linux-ver15)와 [`sys.schemas`](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/schemas-catalog-views-sys-schemas?view=sql-server-linux-ver15)를 사용합니다:


```sql
SELECT o.name AS ObjectName, s.name AS SchemaName
FROM sys.objects AS o
  JOIN sys.schemas AS s
    ON o.schema_id = s.schema_id
WHERE o.name = 'tabela1';
```

| ObjectName   | SchemaName   |
|--------------+--------------|
| tabela1      | dbo          |

🩵[`ALTER USER`](https://docs.microsoft.com/en-us/sql/t-sql/statements/alter-user-transact-sql?view=sql-server-linux-ver15)를 사용하여 `uzytkownik` 계정의 기본 스키마를 변경해봅시다:
- ❗️원래는 `schema.table_name` 처럼 스키마명을 함께써야함. 하지만 기본 스키마를 설정하면 앞의 스키마명을 대신 입력한 상태로 간주함. 

```sql
ALTER USER uzytkownik WITH DEFAULT_SCHEMA = MojSchemat;
```

기본 스키마가 변경되었는지 확인해봅시다:

```sql
EXECUTE AS USER = 'uzytkownik';
SELECT SCHEMA_NAME() AS CurrentSchema;
REVERT;
```

| CurrentSchema   |
|-----------------|
| MojSchemat      |


`uzytkownik` 사용자 계정으로 다른 테이블을 생성해봅시다:

```sql
EXECUTE AS USER = 'uzytkownik';
CREATE TABLE tabela2 ( imie VARCHAR(20) );
INSERT INTO tabela2 VALUES (N'Janina'), (N'Wiesław'), (N'Marcin'), (N'Piotr'), (N'Anna');
REVERT;
```

두 테이블의 레이아웃을 비교해봅시다:

```sql
SELECT o.name AS ObjectName, s.name AS SchemaName
FROM sys.objects AS o
  JOIN sys.schemas AS s
    ON o.schema_id = s.schema_id
WHERE o.name LIKE 'tabela%';
```


| ObjectName   | SchemaName   |
|--------------+--------------|
| tabela1      | dbo          |
| tabela2      | MojSchemat   |

스키마(또는 전체 정규화된 이름)를 지정하지 않으면 `dbo` 계정에서 `tabela2` 테이블을 쿼리할 수 없다는 점에 주목하세요:

```sql
SELECT * FROM tabela2;
```

```null
Msg 208, Level 16, State 1, Line 1
Invalid object name 'tabela2'.
```

```sql
SELECT * FROM MojSchemat.tabela2;
```

| imie    |
|---------|
| Janina  |
| Wieslaw |
| Marcin  |
| Piotr   |
| Anna    |


물론 `uzytkownik` 계정에서 테이블 이름만으로 쿼리를 실행하면 정상적으로 작동합니다. 이는 이 테이블의 스키마가 기본 계정 스키마와 동일하기 때문입니다:

```sql
EXECUTE AS USER = 'uzytkownik';
SELECT * FROM tabela2;
REVERT;
```

| imie    |
|---------|
| Janina  |
| Wieslaw |
| Marcin  |
| Piotr   |
| Anna    |

#### 🩵스키마에 객체 할당

시스템 뷰 [`INFORMATION_SCHEMA.TABLES`](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/tables-transact-sql?view=sql-server-linux-ver15)를 사용하여 스키마에 대한 `bank` 테이블의 할당을 나열할 수 있습니다:

```sql
SELECT * 
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_NAME = 'bank';
```

| TABLE_CATALOG   | TABLE_SCHEMA   | TABLE_NAME   | TABLE_TYPE   |
|-----------------+----------------+--------------+--------------|
| mssql08         | dbo            | bank         | BASE TABLE   |



🩵[`ALTER SCHEMA`](https://docs.microsoft.com/en-us/sql/t-sql/statements/alter-schema-transact-sql?view=sql-server-linux-ver15)를 사용하여 `bank` 테이블의 할당을 이전에 생성한 `MojSchemat` 스키마로 변경해봅시다:
`ALTER SCHEMA toChange_schema_name TRANSFER original_schema_name.table_name`

```sql
ALTER SCHEMA MojSchemat TRANSFER dbo.bank;
```

🩵스키마 할당을 다시 확인해봅시다:

```sql
SELECT * 
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_NAME = 'bank';
```


| TABLE_CATALOG   | TABLE_SCHEMA   | TABLE_NAME   | TABLE_TYPE   |
|-----------------+----------------+--------------+--------------|
| mssql08         | MojSchemat     | bank         | BASE TABLE   |


#### 🩵스키마 삭제

[`DROP SCHEMA`](https://docs.microsoft.com/en-us/sql/t-sql/statements/drop-schema-transact-sql?view=sql-server-linux-ver15)를 사용하여 스키마를 삭제합니다. 이 표현식을 사용하여 `MojSchemat` 스키마를 삭제해봅시다:

```sql
DROP SCHEMA MojSchemat;
```

빈 스키마만 삭제할 수 있기 때문에 작업이 오류로 끝납니다:

```null
Msg 3729, Level 16, State 1, Line 1
Cannot drop schema 'MojSchemat' because it is being referenced by object 'bank'.
```

🎈따라서 `bank`와 `tabela2` 테이블을 `dbo` 스키마로 이동하고 `MojSchemat` 스키마를 삭제합시다:

```sql
ALTER SCHEMA dbo TRANSFER MojSchemat.bank;
ALTER SCHEMA dbo TRANSFER MojSchemat.tabela2;
DROP SCHEMA MojSchemat;
```

### 동적 데이터 마스킹

이전 장에서 [`DENY`](https://docs.microsoft.com/en-us/sql/t-sql/statements/deny-object-permissions-transact-sql?view=sql-server-linux-ver15)를 사용하여 컬럼 수준에서 액세스를 제한하는 방법을 배웠습니다.

이 방법은 액세스가 거부된 객체에 접근하려고 하면 오류가 발생할 수 있으므로 모든 애플리케이션에 적합하지 않습니다. 일부 애플리케이션에서는 Transact-SQL 쿼리 결과에서 민감한 데이터를 마스킹해야 합니다.

이를 위해 SQL Server 2016부터 사용 가능한 [동적 데이터 마스킹(Dynamic Data Masking)](https://docs.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking?view=sql-server-linux-ver15) 기술이 개발되었습니다.

DDM을 사용하여 테이블을 생성하는 데 필요한 권한은 표준 `CREATE TABLE`과 테이블이 위치한 스키마에 대한 `ALTER`입니다.

컬럼에 대한 마스킹 추가, 수정 및 제거는 `ALTER ANY MASK`와 테이블 수준의 `ALTER` 권한이 필요합니다.

`SELECT` 권한이 있는 사용자는 마스킹된 결과만 볼 수 있습니다. `UNMASK` 권한을 부여하면 마스킹되지 않은 데이터를 얻을 수 있습니다.

데이터베이스 수준의 `CONTROL` 권한에는 `ALTER ANY MASK`와 `UNMASK`가 모두 포함됩니다.

#### 원시 상태

`bank` 테이블에서 마스킹되지 않은 데이터의 가시성을 확인해봅시다:

```sql
SELECT * FROM bank;
```


|id | imie     | nazwisko | nazwisko | PESEL       | data_urodz | stan_konta |...
|   |          |          | _matki   |             | enia       |            |...
|---+----------+----------+----------+-------------+------------+------------+...
| 1 | Armando  | Gibbs    | Riggs    | 60052349726 | 1960-05-23 | 27480.00   |...
| 2 | Hakeem   | Roman    | Ramsey   | 71030954241 | 1971-03-09 | 41798.00   |...
| 3 | Elliott  | Faulkner | Walls    | 78121066182 | 1978-12-10 | 40336.00   |...
| 4 | Charissa | Bernard  | Maddox   | 64110432333 | 1964-11-04 | 39271.00   |...
| 5 | Azalia   | Stokes   | Juarez   | 87012837521 | 1987-01-28 | 79705.00   |...
+---+----------+----------+----------+-------------+------------+------------+...

...+--------------------+-----------------+
...| email              | numer_karty     |
...|                    |                 |
...+--------------------+-----------------|
...| iquet@primis.org   | 9893436755178541|
...| arcu@malesu.org    | 9091680525982543|
...| sit@ultric.ca      | 0628218134030380|
...| risus@ssim.com     | 6059056249431781|
...| aliquam@lectus.com | 2288549149712106|
...+--------------------+-----------------+


#### 🩵마스킹 방법

DDM 구현을 용이하게 하기 위해 Microsoft는 여러 **데이터 마스킹** 방법을 구현했습니다. 기본 방법을 적용한 결과는 컬럼 유형에 따라 다릅니다:

- 텍스트 (`char`, `nchar`, `varchar`, `nvarchar`, `text`, `ntext`) - 마스킹 결과는 `xxxx` 문자열 또는 전체 필드를 채우는 여러 `x` 문자가 됩니다.
- 숫자 (`bigint`, `bit`, `decimal`, `int`, `money`, `numeric`, `smallint`, `smallmoney`, `tinyint`, `float`, `real`)는 `0`으로 마스킹됩니다.
- 날짜 및 시간 (`date`, `datetime2`, `datetime`, `datetimeoffset`, `smalldatetime`, `time`)은 1900년 1월 1일 자정으로 마스킹됩니다.
- 이진 (`binary`, `varbinary`, `image`)은 ASCII 값이 `0`인 단일 바이트로 표현됩니다.

또한 **내장 마스킹 함수**를 사용할 수 있습니다:

- 이메일 주소용
- 주어진 범위에서 임의의 숫자 반환
- 텍스트 유형의 시작과 끝에서 지정된 수의 문자 반환

#### 🩵마스킹 추가

마스킹은 테이블을 생성할 때 또는 [`ALTER TABLE`](https://docs.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql?view=sql-server-linux-ver15)을 사용하여 정의할 수 있습니다. 여러 컬럼에 기본 마스킹을 추가해봅시다:

```sql
ALTER TABLE bank  
  ALTER COLUMN nazwisko_matki VARCHAR(50) MASKED WITH (FUNCTION = 'default()'); -- nazwisko_matki: 어머니 성
  
ALTER TABLE bank
  ALTER COLUMN stan_konta MONEY MASKED WITH (FUNCTION = 'default()'); -- stan_konta: 계좌 잔액
ALTER TABLE bank  
  ALTER COLUMN PESEL CHAR(11) MASKED WITH (FUNCTION = 'default()'); -- PESEL: 주민등록번호
ALTER TABLE bank  
  ALTER COLUMN data_urodzenia DATE MASKED WITH (FUNCTION = 'default()'); -- data_urodzenia: 생년월일
```

데이터의 가시성을 확인해봅시다:

```sql
SELECT * FROM bank;
```


|id | imie     | nazwisko | nazwisko | PESEL       | data_urodz | stan_konta |...
|   |          |          | _matki   |             | enia       |            |...
|---+----------+----------+----------+-------------+------------+------------+...
| 1 | Armando  | Gibbs    | Riggs    | 60052349726 | 1960-05-23 | 27480.0000 |...
| 2 | Hakeem   | Roman    | Ramsey   | 71030954241 | 1971-03-09 | 41798.0000 |...
| 3 | Elliott  | Faulkner | Walls    | 78121066182 | 1978-12-10 | 40336.0000 |...
| 4 | Charissa | Bernard  | Maddox   | 64110432333 | 1964-11-04 | 39271.0000 |...
| 5 | Azalia   | Stokes   | Juarez   | 87012837521 | 1987-01-28 | 79705.0000 |...

...| email              | numer_karty     |
...+--------------------+-----------------|
...| iquet@primis.org   | 9893436755178541|
...| arcu@malesu.org    | 9091680525982543|
...| sit@ultric.ca      | 0628218134030380|
...| risus@ssim.com     | 6059056249431781|
...| aliquam@lectus.com | 2288549149712106|


🩵우리의 사용자 계정 `dbo`가 `UNMASK` 권한을 가지고 있기 때문에 결과가 마스킹되지 않습니다. [`EXECUTE AS`](https://docs.microsoft.com/en-us/sql/t-sql/statements/execute-as-transact-sql?view=sql-server-linux-ver15)를 사용하여 `uzytkownik` 사용자 가장을 통해 데이터베이스를 쿼리해봅시다. 원래 보안 컨텍스트로 돌아가려면 `REVERT` 표현식을 사용합니다:

- 실행 권한을 바꾸는 코드, `uzytkownik` 권한으로 테이블 조회
```sql
EXECUTE AS USER = 'uzytkownik';
SELECT * FROM bank;
REVERT;
```

|id | imie     | nazwisko | nazwisko | PESEL       | data_urodz | stan_konta |...
|   |          |          | _matki   |             | enia       |            |...
|---+----------+----------+----------+-------------+------------+------------+...
| 1 | Armando  | Gibbs    | xxxx     | 60052349726 | 1900-01-01 | 0.00       |...
| 2 | Hakeem   | Roman    | xxxx     | 71030954241 | 1900-01-01 | 0.00       |...
| 3 | Elliott  | Faulkner | xxxx     | 78121066182 | 1900-01-01 | 0.00       |...
| 4 | Charissa | Bernard  | xxxx     | 64110432333 | 1900-01-01 | 0.00       |...
| 5 | Azalia   | Stokes   | xxxx     | 87012837521 | 1900-01-01 | 0.00       |...


...| email              | numer_karty     |
...|                    |                 |
...+--------------------+-----------------|
...| iquet@primis.org   | 9893436755178541|
...| arcu@malesu.org    | 9091680525982543|
...| sit@ultric.ca      | 0628218134030380|
...| risus@ssim.com     | 6059056249431781|
...| aliquam@lectus.com | 2288549149712106|


수정한 컬럼의 데이터가 이제 올바르게 마스킹됩니다. 다음 컬럼에 **마스킹을 추가**해봅시다.

🩵이메일 주소를 마스킹하는 `email()` 함수는 매개변수를 받지 않습니다:
```sql
ALTER TABLE bank  
  ALTER COLUMN email VARCHAR(50) MASKED WITH (FUNCTION = 'email()');
```

마스킹된 컬럼을 확인해봅시다:
```sql
EXECUTE AS USER = 'uzytkownik';
SELECT id, email FROM bank;
REVERT;
```


| id   | email         |
|------+---------------|
| 1    | iXXX@XXXX.com |
| 2    | aXXX@XXXX.com |
| 3    | sXXX@XXXX.com |
| 4    | rXXX@XXXX.com |
| 5    | aXXX@XXXX.com |


🩵`random()` 함수를 사용하여 100에서 9001 사이의 임의 값을 반환하도록 `stan_konta` 컬럼을 수정해봅시다.
```sql
ALTER TABLE bank
  ALTER COLUMN stan_konta MONEY MASKED WITH (FUNCTION = 'random(100, 9001)');
```

마스킹된 컬럼을 확인해봅시다:
```sql
EXECUTE AS USER = 'uzytkownik';
SELECT id, stan_konta FROM bank;
REVERT;
```

> 이 쿼리를 여러 번 반복하세요:

| id   | stan_konta   |
|------+--------------|
| 1    | 3361.9497    |
| 2    | 2955.0875    |
| 3    | 985.0494     |
| 4    | 5371.5021    |
| 5    | 6854.8999    |

| id   | stan_konta   |
|------+--------------|
| 1    | 8722.8612    |
| 2    | 5846.4494    |
| 3    | 7756.4977    |
| 4    | 3009.2331    |
| 5    | 8546.6492    |


#### 🩵[partial()](https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking?view=sql-server-ver17) 함수
**`partial(prefix, [padding], surffix)`**
- prefix(int): 가리지 않고 보여줄 앞부분 문자 개수
- padding(string): 앞과 뒤 사이에 삽일될 가림 문자열
- suffix(int): -가리지 않고 보여줄 뒷부분 문자개수

`partial()` 함수를 사용하여 마지막 네 자리만 표시하여 신용카드 번호를 부분적으로 마스킹해봅시다:
```sql
ALTER TABLE bank  
  ALTER COLUMN numer_karty VARCHAR(16) MASKED WITH (FUNCTION = 'partial(0,"---UKRYTE---",4)');
```

마스킹된 컬럼을 확인해봅시다:
```sql
EXECUTE AS USER = 'uzytkownik';
SELECT id, numer_karty FROM bank;
REVERT;
```

| id   | numer_karty      |
|------+------------------|
| 1    | ---UKRYTE---8541 |
| 2    | ---UKRYTE---2543 |
| 3    | ---UKRYTE---0380 |
| 4    | ---UKRYTE---1781 |
| 5    | ---UKRYTE---2106 |


#### 마스킹 제거

시스템 뷰 [`sys.masked_columns`](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-masked-columns-transact-sql?view=sql-server-linux-ver15)를 쿼리하여 테이블의 현재 마스킹 상태를 확인할 수 있습니다:

```sql
SELECT c.name, c.masking_function  
FROM sys.masked_columns AS c  
  JOIN sys.tables AS t
    ON c.[object_id] = t.[object_id]
WHERE t.name = 'bank'
```

| name           | masking_function              |
|----------------+-------------------------------|
| nazwisko_matki | default()                     |
| PESEL          | default()                     |
| data_urodzenia | default()                     |
| stan_konta     | random(100, 9001)             |
| email          | email()                       |
| numer_karty    | partial(0, "---UKRYTE---", 4) |


🩵`ALTER TABLE`을 사용하여 **마스킹을 제거**할 수 있습니다:
```sql
ALTER TABLE bank
  ALTER COLUMN numer_karty DROP MASKED;
```

`bank` 테이블에서 마스킹된 컬럼을 나열하는 쿼리를 다시 실행해봅시다:
```sql
SELECT c.name, c.masking_function  
FROM sys.masked_columns AS c  
  JOIN sys.tables AS t
    ON c.[object_id] = t.[object_id]
WHERE t.name = 'bank'
```

| name           | masking_function   |
|----------------+--------------------|
| nazwisko_matki | default()          |
| PESEL          | default()          |
| data_urodzenia | default()          |
| stan_konta     | random(100, 9001)  |
| email          | email()            |
