#### Handling `NULL`
NULL 처리 시 등호 대신, 
`IS NULL` `IS NOT NULL` 사용하기.

옳은 예시
```
SELECT last_name,
       bonus
FROM   Employees
WHERE  bonus IS NULL;
```

Compare with (*NOT CORRECT!*):

```
SELECT last_name,
       bonus
FROM   Employees
WHERE  bonus = NULL;
```

=> `ISNULL(par1, par2)`
par1 이 NULL 이면, par2 값으로 대체.
(null 처리 훨씬 안정적)

#### 조건문
`CASE WHEN ... THEN ... END AS ...`

```sql
SELECT 
    CASE
        WHEN name = 'professor' OR name= 'lecturer' OR name= 'phd student' THEN 'research'
        ELSE 'administrative'
    END AS 'position type'
FROM Positions;
```

#### 문자열 검색
WHERE 절, **문자열** 찾기 (`LIKE`)

```sql
SELECT name AS 'shipName'
FROM Ships
WHERE name LIKE 'R%';
-- WHERE name = 'R%'; 하면 틀림. 
```

#### 중복 처리
`DISTINCT`
