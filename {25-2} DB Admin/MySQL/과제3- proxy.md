
**프록시 PROXY** : 클라이언트와 서버 사이의 통신을 중계.
	컴퓨터 네트워크에서 클라이언트(사용자)와 서버(서비스 제공자) 사이에 위치.
	클라이언트의 요청을 대신 받아 서버에 전달하고, 서버의 응답을 다시 클라이언트에게 전달해주는 중개 서버.

### 🩵프록시 설정 예시

```mysql
-- 프록시 계정 생성
CREATE USER 'employee_ext'@'localhost'
IDENTIFIED WITH my_auth_plugin
AS 'my_auth_string';

-- 프록시 대상 계정 (직접 로그인 방지)
CREATE USER 'employee'@'localhost'
IDENTIFIED WITH mysql_no_login BY 'pas$w0rd';

GRANT ALL ON employees.* TO 'employee'@'localhost';

-- 프록시 권한 부여
GRANT PROXY ON 'employee'@'localhost'
TO 'employee_ext'@'localhost';
```
`IDENTIFIED WITH [plugin_name] BY [password]`


---

시스템 변수 System variable: mysql 에서 서버의 동작 방식을 제어하는 변수
```mysql
SET GLOBAL variable_name = ON;
-- 변수 활성화 = ON / 1
```

---

`INFORMATION_SCHEMA.PLUGINS` : mysql 서버에 현재 설치&로드된 모든 플러그인에 대한 정보를 담고있는 가상 테이블(view)

`performance_schema.global_variables` : system variable information is available in these performance schema tables

->>global_variables 테이블의 컬럼 예시

| global_variable |
| --------------- |
| VARIABLE_NAME   |
| VARIABLE_VALUE  |
| THREAD_ID       |

---

##### `IN`,  `=` , `LIKE`

IN과 =
```mysql
WHERE VARIABLE_NAME IN ('var1', 'var2');
-- 위와 아래 코드는 같은내용임
WHERE VARIABLE_NABE = 'var1' OR VARIABLE_NABE = 'var2';
```

LIKE
- 문자열 전체가 아닌 일부(패턴) 가 일치하는지 확인할 때 사용
- %, $
- 변수의 전체이름을 정확히 아는 상황에선 비효율적. (성능저하 가능)

---

#####  [`INFORMATION_SCHEMA.ROUTINES`](https://dev.mysql.com/doc/refman/8.0/en/routines-table.html)

- display the metadata of the show_workers()

`@@proxy_user` : 원래 접속 시도한 유저
`CALL show_workers();` 
`CALL show_vault_pin();` 
-> display 시 CALL 사용 (not SELECT)