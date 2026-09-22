##### >MySQL 접근 제어 및 계정 관리 핵심 요약

## 1. 기본 개념

**계정 식별 방식**

```
'username'@'hostname'
```

- 사용자 이름 + 호스트로 식별
- 같은 이름이라도 다른 호스트면 다른 계정

**접근 제어 2단계**

1. **연결 검증**: ID/비밀번호 확인
2. **요청 검증**: 각 명령의 권한 확인

## 2. 권한 수준 (우선순위)

```
글로벌 권한 (*.*)
  ↓
데이터베이스 권한 (db_name.*)
  ↓
테이블 권한 (db_name.table_name)
  ↓
컬럼 권한 (특정 컬럼)
```

## 3. 주요 SQL 명령어

### 계정 관리

mysql

```mysql
-- 생성
CREATE USER 'user'@'host' IDENTIFIED BY 'password';

-- 삭제
DROP USER 'user'@'host';

-- 비밀번호 변경
ALTER USER 'user'@'host' IDENTIFIED BY 'new_password';
```

### 권한 관리

mysql

```mysql
-- 권한 부여
GRANT SELECT, INSERT ON db.* TO 'user'@'host';

-- 권한 취소
REVOKE SELECT ON db.* FROM 'user'@'host';

-- 권한 확인
SHOW GRANTS FOR 'user'@'host';
```

### 역할(Role) 관리

mysql

````mysql
-- 역할 생성
CREATE ROLE 'role_name';

-- 역할에 권한 부여
GRANT SELECT ON db.* TO 'role_name';

-- 사용자에게 역할 할당
GRANT 'role_name' TO 'user'@'host';

-- 기본 역할 설정
SET DEFAULT ROLE ALL TO 'user'@'host';
```

## 4. 주요 권한 종류

| 권한 | 설명 |
|------|------|
| `SELECT` | 데이터 조회 |
| `INSERT` | 데이터 삽입 |
| `UPDATE` | 데이터 수정 |
| `DELETE` | 데이터 삭제 |
| `CREATE` | 테이블 생성 |
| `DROP` | 테이블/DB 삭제 |
| `ALL` | 모든 권한 |
| `GRANT OPTION` | 권한 부여 권한 |

## 5. 와일드카드

- `%` : 모든 호스트/데이터베이스
- `_` : 단일 문자
- 예: `'user'@'%.example.com'` → example.com 도메인 모든 호스트

## 6. 권한 테이블
```
mysql.user          → 글로벌 권한
mysql.db            → 데이터베이스 권한
mysql.tables_priv   → 테이블 권한
mysql.columns_priv  → 컬럼 권한
````

## 7. 권한 변경 적용

mysql

```mysql
-- 계정 관리 명령어 사용 시: 즉시 적용
-- 테이블 직접 수정 시: 재로드 필요
FLUSH PRIVILEGES;
```

## 8. 역할 활용 예시

mysql

```mysql
-- 1. 역할 생성 및 권한 부여
CREATE ROLE 'app_read', 'app_write';
GRANT SELECT ON app.* TO 'app_read';
GRANT INSERT, UPDATE, DELETE ON app.* TO 'app_write';

-- 2. 사용자 생성 및 역할 할당
CREATE USER 'user1'@'localhost' IDENTIFIED BY 'pass';
GRANT 'app_read', 'app_write' TO 'user1'@'localhost';

-- 3. 기본 역할 설정
SET DEFAULT ROLE ALL TO 'user1'@'localhost';
```

## 9. 보안 권장사항

✅ **해야 할 것**

- 계정 관리 명령어 사용 (`CREATE USER`, `GRANT`, `REVOKE`)
- 최소 권한 원칙 적용
- 역할을 활용한 권한 관리

❌ **피해야 할 것**

- 권한 테이블 직접 수정 (`INSERT`, `UPDATE`, `DELETE`)
- 글로벌 권한 남용
- 빈 비밀번호 사용

## 10. 유용한 확인 명령어

mysql

```mysql
-- 현재 사용자 확인
SELECT CURRENT_USER();

-- 현재 활성 역할 확인
SELECT CURRENT_ROLE();

-- 권한 확인
SHOW GRANTS;
SHOW GRANTS FOR 'user'@'host';

-- 계정 정보 확인
SHOW CREATE USER 'user'@'host';
```
