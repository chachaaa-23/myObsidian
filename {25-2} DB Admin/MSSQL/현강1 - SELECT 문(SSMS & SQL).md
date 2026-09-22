
##### 기초 SELECT
The `SELECT` command allows you to display data contained in one or more tables. 

```sql
SELECT --list of attributes 어떤 데이터를 출력할것인가?
FROM --list of tables 어디서 데이터를 가져올것인가?
WHERE --filtering conditions
GROUP BY --list of attributes
HAVING --conditions relating to groups

ORDER BY column -- 데이터 오름차순 정렬
ORDER BY column DESC -- 데이터 내림차순 정렬
-- 쉼표(,)통해 다중 열 정렬 가능. 왼쪽->오른쪽
-- 나만의 표현식을 통해 정렬가능 (ex. 급여의 10% 보너스 더한 총액 기준 정렬)
-- 별칭 사용 정렬 가능
```

##### Aliase (별칭)
```sql
SELECT [Alias] = original_column,
		original_column AS [Alias],
		original_column [Alias],
		original_column Alias,
		original_column 'Alias',
		original_column "Alias"
FROM Projects;
```

##### Row Function

| **`SUBSTRING(문자열, 시작위치, 반환개수)`** | **텍스트의 특정 부분 문자열(substring)을 반환** | `SUBSTRING('ABCDEFG', 2, 3)` $\rightarrow$ **'BCD'** 반환   |
| -------------------------------- | --------------------------------- | --------------------------------------------------------- |
| **`UPPER()`**                    | 대문자로 변환하여 반환                      | `UPPER('hello world')` $\rightarrow$ **'HELLO WORLD'** 반환 |
| **`LEN()`**                      | 텍스트 문자열의 **문자 개수(길이)** 반환         | `LEN('Data')` $\rightarrow$ **4** 반환                      |

##### Date and time functions
1. `GETDATE()` : returns the current date
2. `YEAR()` : returns the year of the given date
3. [`DATEDIFF()`](https://learn.microsoft.com/en-us/sql/t-sql/functions/datediff-transact-sql?view=sql-server-ver17) : calculates the difference between dates (eg, in days, minutes, etc)
4. `ISDATE()` : checks whether the given argument is a date
