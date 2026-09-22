# MSSQL 파일 관리 및 백업 정리

## 데이터 저장 모델 - 파일

### 파일 그룹(Filegroup) 개념

- 데이터는 하나 이상의 **데이터 파일**에 저장
- 데이터 파일은 **파일 그룹**이라는 컨테이너로 그룹화
- **트랜잭션 로그 파일**은 별도로 관리 (파일 그룹에 속하지 않음)

### 기본 구성

- **Primary 파일**: 첫 번째 데이터 파일 (`.mdf` 확장자)
    - 메타데이터와 다른 그룹/파일 정보 포함
- **추가 파일**: `.ndf` 확장자 사용
- **Primary 파일 그룹**: Primary 파일이 속한 기본 그룹

### 데이터 저장 구조

- **페이지(Page)**: 8KB 크기
    - 96바이트 헤더 + 8096바이트 데이터 영역
- **익스텐트(Extent)**: 8개의 연속된 페이지
    - SQL Server가 작업하는 최소 단위

### FILESTREAM 파일 그룹

- 대용량 바이너리 데이터(보통 1MB 이상) 저장용
- 파일 시스템에 직접 저장하여 성능 향상
- 별도 설정 및 분석 필요

## 파일 관리 실습

### 🩵데이터베이스 생성 및 확인
- in SQL server, `type = 0` represents rows that are data files, while `type = 1` represents log files. 

```sql
CREATE DATABASE mssql09;
USE mssql09;

-- 파일 구성 확인
SELECT type_desc, name, physical_name, size, max_size, growth
FROM sys.database_files;
```


**기본 설정값**:

- 초기 크기: 8MB (1024 페이지 × 8KB)
- 자동 증가: 64MB씩
- 최대 크기: 트랜잭션 로그 2TB, 데이터 파일 무제한(-1)

### 🩵데이터 파일 추가

```sql
USE master;
ALTER DATABASE mssql09
ADD FILE
(
  NAME = 'mssql09_2',
  SIZE = 8MB,
  FILEGROWTH = 64MB,
  FILENAME = '/var/opt/mssql/data/mssql09_2.ndf'
)
TO FILEGROUP [PRIMARY];
```

### 🩵파일 속성 수정

```sql
-- 최대 크기를 1GB로 제한
ALTER DATABASE mssql09
MODIFY FILE
(
  NAME = 'mssql09',
  MAXSIZE = 1GB
);
```

### 🩵새 파일 그룹 생성

```sql
-- 🩵파일 그룹 생성
ALTER DATABASE mssql09
ADD FILEGROUP FG1;

-- 파일 그룹에 파일 추가
ALTER DATABASE mssql09
ADD FILE
(
  NAME = 'mssql09_fg1',
  SIZE = 8MB,
  FILEGROWTH = 64MB,
  FILENAME = '/var/opt/mssql/data/mssql09_fg1.ndf'
)
TO FILEGROUP FG1;

-- 🩵특정 파일 그룹에 테이블 생성
CREATE TABLE data_fg1
(
    col1 char(4000),
    col2 char(4000)
)
ON FG1;
```

## 트랜잭션 로그

### 로그의 역할

- 데이터 수정 작업을 먼저 로그에 기록
- 성공 시 데이터 파일에 적용
- 실패 시 롤백
- 복구, 가용성 그룹, 복제 등 지원

### 로그 구조

- **가상 로그 파일(VLF)** 로 구성
- 순차적으로 사용, 끝까지 사용 후 처음부터 재사용
- 여유 공간 부족 시 자동 확장

### 🩵로그 파일 관리

```sql
-- 로그 파일 추가
ALTER DATABASE mssql09
ADD LOG FILE
(
  NAME = 'mssql09_log2',
  SIZE = 8MB,
  MAXSIZE = 2TB,
  FILEGROWTH = 64MB,
  FILENAME = '/var/opt/mssql/data/mssql09_log2.ldf'
);

-- 로그 파일 삭제
ALTER DATABASE mssql09
REMOVE FILE mssql09_log2;
```

## 복구 모델(Recovery Model)

### 세 가지 복구 모델

1. **SIMPLE**
    - 최소 로깅
    - 트랜잭션 완료 후 자동 로그 잘림
    - 로그 백업 불가
    - 데이터 수정이 드문 경우 적합
2. **FULL** (기본값)
    - 전체 로깅
    - 로그 백업 가능
    - 특정 시점 복구 가능
    - 디스크 요구사항 높음
3. **BULK_LOGGED**
    - 대량 데이터 가져오기 시 임시 사용
    - 최소 로깅으로 성능 향상
    - 작업 후 FULL 모델로 전환 권장

### 🩵복구 모델 변경

```sql
-- 현재 복구 모델 확인
SELECT name, recovery_model_desc  
FROM sys.databases
WHERE name = 'mssql09';

-- 복구 모델 변경
ALTER DATABASE mssql09
SET RECOVERY SIMPLE;
```

## 백업(Backup)

### 백업 유형

#### 🩵1. 전체 백업(Full Backup)

- 모든 복구 모델에서 사용 가능
- CHECKPOINT 후 모든 페이지 복사
- 트랜잭션 일관성을 위한 로그 일부 포함

```sql
BACKUP DATABASE mssql09
    TO DISK = '/var/opt/mssql/data/mssql09.bak'
    WITH
        NOFORMAT,
        NAME = 'mssql09 Full Backup';
```

#### 🩵2. 차등 백업(Differential Backup)

- 마지막 전체 백업 이후 변경된 페이지만 복사
- 빠르고 디스크 공간 절약
- 복구 시 전체 백업 + 최신 차등 백업만 필요

```sql
BACKUP DATABASE mssql09
    TO DISK = '/var/opt/mssql/data/mssql09.bak'
    WITH
        DIFFERENTIAL,
        NOFORMAT,
        NAME = 'mssql09 Differential Backup';
```

#### 3. 트랜잭션 로그 백업(Transaction Log Backup)
*DB Log BackUp*

- SIMPLE 모델에서는 불가
- 마지막 백업 이후의 모든 로그 복사
- 로그 잘림(truncate) 수행
- 특정 시점 복구 가능
- 리소스 부담 최소

```sql
BACKUP LOG mssql09
    TO DISK = '/var/opt/mssql/data/mssql09.bak'
    WITH
        NOFORMAT,
        NAME = 'mssql09 Log Backup';
```

### 백업 미디어 개념

- **백업 장치(Backup Device)**: 디스크 파일, 테이프, SMB 리소스 등
- **미디어 세트(Media Set)**: 최대 64개 장치 포함 가능
- **백업 세트(Backup Set)**: 미디어 세트 내 각 백업

## 복원(Restore)
##### 🎈*복원 작업 진행 시, 반드시 `master` db 로 접속!*
🎈 `FILE` option corresponds to the `Position` column in the backup set header information.

| `name`              | `backup set` |
| ------------------- | ------------ |
| Full Backup         | 1            |
| Differential Backup | 2            |
| Log Backup          | 3            |

### 백업 정보 확인

```sql
-- 백업 세트 헤더 정보 확인
RESTORE HEADERONLY FROM DISK = '/var/opt/mssql/data/mssql09.bak';

-- 백업 유효성 검증
RESTORE VERIFYONLY FROM DISK = '/var/opt/mssql/data/mssql09.bak';
```

### `ALTER` 사용 복원 
- DB가 서버에 존재하는 경우, 현재 DB를 오프라인 상태로 바꾸기

```sql
USE master;
ALTER DATABASE mssql09
SET OFFLINE WITH ROLLBACK IMMEDIATE;

RESTORE DATABASE mssql09
    FROM DISK = '/var/opt/mssql/data/mssql09.bak'
    WITH
        FILE = 1,
        REPLACE;
```

### Backup 복원
- DB가 서버에 존재하지 않는 경우

```sql
-- 1. 전체 백업 복원 (NORECOVERY)
RESTORE DATABASE mssql09
    FROM DISK = '/var/opt/mssql/data/mssql09.bak'
    WITH
        FILE = 1,
        NORECOVERY, -- 복원 중 상태 (복원 우에도 db 잠김 상태)
        REPLACE; -- 기존 데이터 유무에 상관없이 덮어쓰기

-- 2. 차등 백업 복원 (NORECOVERY)
RESTORE DATABASE mssql09
    FROM DISK = '/var/opt/mssql/data/mssql09.bak'
    WITH
        FILE = 2,
        NORECOVERY;

-- 3. 로그 백업 복원 (마지막에만 RECOVERY)
RESTORE DATABASE mssql09
    FROM DISK = '/var/opt/mssql/data/mssql09.bak'
    WITH
        FILE = 3;
```

### 복원 시 주의사항

- **NORECOVERY**: 여러 백업을 연속으로 복원할 때 마지막 전까지 사용
- **REPLACE**: 복원 체인의 첫 번째 명령에만 사용
- **FILE**: 백업 세트의 Position 값 지정

## 핵심 포인트

1. **파일 관리**: 여러 파일과 파일 그룹으로 성능과 관리 효율성 향상
2. **복구 모델**: 용도에 맞는 모델 선택 (SIMPLE/FULL/BULK_LOGGED)
3. **백업 전략**: 전체 + 차등 + 로그 백업 조합으로 최적화
4. **복원 절차**: 올바른 순서와 옵션 사용이 중요