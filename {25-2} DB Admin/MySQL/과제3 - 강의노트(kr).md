##### >MySQL 계정 및 비밀번호 관리, 인증 플러그인, 프록시 및 권한 요약

## 1. 저장 객체의 접근 제어

### DEFINER 속성
- 모든 저장 객체(프로시저, 함수, 트리거, 이벤트, 뷰)는 `DEFINER` 속성을 가질 수 있음
- `DEFINER`를 명시하지 않으면 객체를 생성한 사용자가 기본값
- `SET_USER_ID` 또는 `SUPER` 권한이 있으면 임의의 계정을 `DEFINER`로 지정 가능

**DEFINER 란**
	: 누가 만든 것인지, 실행 시 어떤 계정 권한을 사용할지 지정하는 설정. 
	(호출하는 유저의 권한이 아니라 DEFINER 에 설정된 사용자의 권한을 사용해 실행됨)

**BEGIN / END 란**
	: 저장 프로시저. 하나의 블록으로 여러 SQL 문을 포함 가능
	- `BEGIN` \ `END` : 프로시저 블록 시작 / 종료

**Delimiter 란?**
	프로시저를 정의할 때 한 문장의 끝을 구분하는 기호로서 사용
	(why. MySQL 과 프로시저 내부에서 문장을 끝내는 경우를 구분짓기 위해)
	- 주로 BEGIN / END 와 함께 쓰이며, MySQL 에게 문장 종료 기호를 바꾼다고 알려줌.
### 🩵SQL SECURITY 속성

- `DEFINER` 또는 `INVOKER` 값 설정 가능
- **DEFINER**: 정의된 계정의 권한으로 실행
- **INVOKER**: 호출한 사용자의 권한으로 실행
- 기본값은 `DEFINER`
- 트리거와 이벤트는 항상 `DEFINER` 컨텍스트에서 실행

**예시:**

```mysql
CREATE DEFINER = 'admin'@'localhost' 
PROCEDURE p1()
SQL SECURITY DEFINER
BEGIN
    UPDATE t1 SET counter = counter + 1;
END;
```

```mysql
delimiter // -- 프로시저 정의 중 문장 종료 방지 위해 //로 변경
CREATE DEFINER='root'@'%' PROCEDURE show_workers() -- root 권한으로 실행되는 프로시저 생성
    SQL SECURITY DEFINER -- 실행될 때 DEFINER 의 권한 사용
BEGIN -- 저장 프로시저 시작
    SELECT User 
    FROM mysql.user
    WHERE User LIKE 'worker%';
END//
delimiter ; -- 종료문자 복원
```
## 2. 비밀번호 관리

### 🩵비밀번호 만료 정책

- **수동 만료**: `ALTER USER 'user'@'host' PASSWORD EXPIRE;`
- **자동 만료**: `default_password_lifetime` 시스템 변수로 설정 (일 단위)
- 개별 계정 설정:

```mysql
  ALTER USER 'user'@'host' PASSWORD EXPIRE INTERVAL 90 DAY;
  ALTER USER 'user'@'host' PASSWORD EXPIRE NEVER;
  ALTER USER 'user'@'host' PASSWORD EXPIRE DEFAULT;
```

### 비밀번호 재사용 제한

- `password_history`: 최근 N개의 비밀번호 재사용 방지
- `password_reuse_interval`: N일 이내의 비밀번호 재사용 방지

```mysql
ALTER USER 'user'@'host' 
PASSWORD HISTORY 5
PASSWORD REUSE INTERVAL 365 DAY;
```

### 현재 비밀번호 확인 요구

- `password_require_current` 시스템 변수로 전역 설정
- 개별 계정 설정:

```mysql
  ALTER USER 'user'@'host' PASSWORD REQUIRE CURRENT;
  ALTER USER 'user'@'host' PASSWORD REQUIRE CURRENT OPTIONAL;
```

### 이중 비밀번호

- 주 비밀번호와 보조 비밀번호 동시 사용 가능
- 애플리케이션 다운타임 없이 비밀번호 변경 가능

```mysql
-- 현재 비밀번호를 보조로 유지하며 새 비밀번호 설정
ALTER USER 'user'@'host'
IDENTIFIED BY 'new_password'
RETAIN CURRENT PASSWORD;

-- 보조 비밀번호 제거
ALTER USER 'user'@'host' DISCARD OLD PASSWORD;
```

### 랜덤 비밀번호 생성

```mysql
CREATE USER 'user'@'host' IDENTIFIED BY RANDOM PASSWORD;
ALTER USER 'user'@'host' IDENTIFIED BY RANDOM PASSWORD;
```

### 로그인 실패 추적 및 계정 임시 잠금

```mysql
CREATE USER 'user'@'host'
IDENTIFIED BY 'password'
FAILED_LOGIN_ATTEMPTS 3 
PASSWORD_LOCK_TIME 3;  -- 3일 동안 잠금
```

## 3. 인증 플러그인

### 🩵주요 인증 플러그인

1. **mysql_native_password**: 기본 네이티브 인증
2. **caching_sha2_password**: SHA-256 해싱 + 캐싱 (권장)
3. **sha256_password**: 기본 SHA-256 인증
4. **auth_socket**: Unix 소켓 파일을 통한 인증
5. 🩵**mysql_no_login**: 직접 로그인 방지 (프록시 전용)
	특정 사용자 계정의 권한은 유지하되, 클라이언트가 그 계정으로 직접 데이터베이스에 접속하는 것을 완전히 차단함
	


**프록시 PROXY** : **클라이언트와 서버 사이의 통신을 중계하는 제3자**
	컴퓨터 네트워크에서 클라이언트(사용자)와 서버(서비스 제공자) 사이에 위치하여 **클라이언트의 요청을 대신 받아 서버에 전달하고, 서버의 응답을 다시 클라이언트에게 전달해주는 중개 서버**.


### SHA-2 캐싱 인증

```mysql
CREATE USER 'sha2user'@'localhost'
IDENTIFIED WITH caching_sha2_password 
BY 'password';
```

### Unix 소켓 인증

- OS 사용자명과 MySQL 사용자명 매칭

```mysql
CREATE USER 'valerie'@'localhost'
IDENTIFIED WITH auth_socket;

-- 추가 OS 사용자 허용
ALTER USER 'valerie'@'localhost' 
IDENTIFIED WITH auth_socket 
AS 'stephanie';
```

### 🩵No-Login 플러그인

- 직접 로그인 차단, 프록시나 stored program에서만 사용

```mysql
INSTALL PLUGIN mysql_no_login SONAME 'mysql_no_login.so';

CREATE USER 'nologin'@'localhost'
IDENTIFIED WITH mysql_no_login;
```

```mysql
-- verify mysql_no_login plugin
SELECT PLUGIN_NAME, PLUGIN_STATUS
FROM INFORMATION_SCHEMA.PLUGINS
WHERE PLUGIN_NAME LIKE '%login%';
```
## 4. 프록시 계정

### 프록시 개념

- **프록시 사용자**: 외부에서 연결하는 사용자
- **프록시 대상 사용자**: 권한을 위임받는 실제 사용자

### 🩵프록시 설정 예시

```mysql
-- 프록시 계정 생성
CREATE USER 'employee_ext'@'localhost'
IDENTIFIED WITH my_auth_plugin
AS 'my_auth_string';

-- 프록시 대상 계정 (직접 로그인 방지)
CREATE USER 'employee'@'localhost'
IDENTIFIED WITH mysql_no_login;

GRANT ALL ON employees.* TO 'employee'@'localhost';

-- 🩵프록시 권한 부여
GRANT PROXY ON 'super_worker'@'localhost'
TO 'worker1'@'localhost';
```

### 기본 프록시 사용자

```mysql
-- 모든 사용자에 대한 기본 프록시
CREATE USER ''@''
IDENTIFIED WITH ldap_auth
AS 'O=Oracle, OU=MySQL';

GRANT PROXY ON 'developer'@'localhost' TO ''@'';
GRANT PROXY ON 'manager'@'localhost' TO ''@'';
```

### 🩵프록시 확인

```mysql
SELECT USER(), CURRENT_USER();
-- USER(): 실제 연결 사용자
-- CURRENT_USER(): 권한을 가진 프록시 대상 사용자
```

## 🩵5. 계정 잠금

```mysql
-- 계정 생성 시 잠금
CREATE USER 'user'@'host' 
IDENTIFIED BY 'password' 
ACCOUNT LOCK;

-- 기존 계정 잠금/해제
ALTER USER 'user'@'host' ACCOUNT LOCK;
ALTER USER 'user'@'host' ACCOUNT UNLOCK;
```

## 🩵6. 리소스 제한

```mysql
CREATE USER 'francis'@'localhost'
IDENTIFIED BY 'password'
WITH MAX_QUERIES_PER_HOUR 20
     MAX_UPDATES_PER_HOUR 10
     MAX_CONNECTIONS_PER_HOUR 5
     MAX_USER_CONNECTIONS 2;

-- 제한 수정
ALTER USER 'francis'@'localhost' 
WITH MAX_QUERIES_PER_HOUR 100;

-- 제한 제거 (0으로 설정)
ALTER USER 'francis'@'localhost'
WITH MAX_CONNECTIONS_PER_HOUR 0;
```
-> 추가 컬럼들, [공식 문서](https://dev.mysql.com/doc/refman/9.3/en/alter-user.html?utm_source=chatgpt.com)에서 확인 가능 
- `PASSWORD HISTORY n`
- `FAILED_LOGIN_ATTEMPTS n`
- `PASSWORD_LOCK_TIME n`  (<< n days)

## 7. SQL 기반 계정 활동 감사

### 사용자 식별 함수

- `USER()`: 실제 연결 사용자 (와일드카드 없음)
- `CURRENT_USER()`: 권한 검사에 사용되는 계정 (와일드카드 포함 가능)

```mysql
-- 사용자명/호스트명 추출
SELECT SUBSTRING_INDEX(CURRENT_USER(),'@',1) AS user_name;
SELECT SUBSTRING_INDEX(CURRENT_USER(),'@',-1) AS host_name;
```

## 핵심 포인트

1. **보안 강화**: SHA-256 기반 인증 사용, 비밀번호 정책 설정
2. **권한 분리**: `DEFINER`와 `SQL SECURITY`로 권한 제어
3. **프록시 활용**: 중앙집중식 인증 관리, 직접 로그인 방지
4. **리소스 관리**: 사용자별 리소스 제한으로 서버 보호
5. **감사 추적**: `USER()`와 `CURRENT_USER()` 구분으로 정확한 감사
