## Handling null values

 [`ISNULL( check_expression, replacement_value );`](https://learn.microsoft.com/en-us/sql/t-sql/functions/isnull-transact-sql?view=sql-server-ver17)

ex. handling null value (bonus)

```jsx
SELECT id, (salary+ ISNULL(bonus, 0))*12 AS [annual remuneration]
FROM Employees;
```

##### *property of `NULL`

- null 을 더하거나 곱하면 똑같이 null이다

→ **ssms 말고 모든 sql 에서 쓰이는 표현은?**

--> [`CALESCE();`](https://learn.microsoft.com/en-us/sql/t-sql/language-elements/coalesce-transact-sql?view=sql-server-ver17)

- can be extended with a large number of arguments
- 괄호 안에 검사되는 변수 중, 제일먼저 null 이 아닌 녀석을 반환

## Data types and conversion

--> [data types](https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-types-transact-sql?view=sql-server-ver17)

- `CHAR`(static string bytes) , `VARCHAR`(dynamic string byte)
- `DECIMAL` , `NUMERIC` (fixed-point real numbers)
- `MONEY` (money and currency)
- `DATETIME` (date and time)

```jsx
CREATE TABLE Employees(
	  id           INT NOT NULL PRIMARY KEY,
    last_name    VARCHAR(20) NOT NULL,
    manager      INT REFERENCES Employees(id),
    salary       MONEY,
    bonus        MONEY,
    position     VARCHAR(15) REFERENCES Positions(name),
    hired_date   DATETIME
); 
```

--> [conversion](https://learn.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql?view=sql-server-ver17)

- `CAST(col1 AS VARCHAR)`
- `CONVERT(VARCHAR, col1)`
    - you can convert the data type in a predefined format `103 = dd/mm/year`

```jsx
SELECT last_name, 
			 CAST( salary AS VARCHAR) + 'EUR',
			 CONVERT( VARCHAR, salary) + 'EUR',
			 CONVERT( VARCHAR, hired_date, 103)  
FROM Employees;
```

- `DECIMAL(p[,s])`, `numeric(p, [,s])`
    - p - total digit of number
    - s- right point digit of number
    - you can use `NUMERIC()` as type

```sql
SELECT CASE(2.0/4.0 AS NUMERIC(3,2)) AS number;

--result : 0.50
```

### expressions

- `DISTINCT` : remove duplicate
