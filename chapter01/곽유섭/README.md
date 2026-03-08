# 📘 곽유섭 - Chapter 01 정리

> Real MySQL 8.0 2권 | Chapter 11 - 쿼리 작성 및 최적화

---

## 📝 정리 내용

> # 11. 쿼리 작성 및 최적화

`DDL` : 데이터베이스나 테이블의 구조를 변경하기 위한 문장

`DML` : 테이블의 데이터를 조작(읽고, 쓰기)하기 위한 문장

## 11-1. 쿼리 작성과 연관된 시스템 변수

### 11-1-1. SQL 모드

sql_mode를 설정할 때는 구분자(,)를 이용해 키워드들을 동시에 설정할 수 있다.

```
주의 : MySQL 서버에 사용자 테이블을 생성하고 데이터를 저장하기 시작했다면 가급적 sql_mode는 변경하지 않는 편이 좋다.

MySQL 8.0 서버 sql_mode 기본값

 - ONLY_FULL_GROUP_BY

 - STRICT_TRANS_TABLES

 - NO_ZERO_IN_DATE

 - NO_ZERO_DATE

 - ERROR_FOR_DIVISION_BY_ZERO

 - NO_ENGINE_SUBSTITUTION
```

 - `STRICT_ALL_TABLES & STRICT_TRANS_TABLES` : 
    1. INSERT나 UPDATE 문장으로 데이터를 변경하는 경우 칼럼의 타입과 저장되는 값의 타입이 다를 때 자동으로 타입 변경을 수행한다.
    
    2. 타입이 적절히 변환되기 어렵거나 칼럼에 저장될 값이 없거나 길이가 최대 길이보다 큰 경우 MySQL이 해당 쿼리를 계속 실행할지, 아니면 에러를 발생시킬지 결정한다.

    3. STRICT_ALL_TABLES는 모든 테이블에 적용되지만 STRICT_TRANS_TABLES는 InnoDB 스토리지 엔진에만 적용된다.

    4. 두 옵션 모두 서비스에 적용하기 전에 반드시 활성화할 것을 권장하며, 서비스 도중 변경해야 한다면 INSERT와 DELETE 문장을 검토해 의도하지 않은 결과가 발생하지 않도록 주의해야 한다.

 - `ANSI_QUOTES` : 
    1. MySQL에서는 문자열 값(리터럴)을 표현하기 위해 홑따옴표('), 쌍따옴표(")를 동시에 사용할 수 있다. 반면 오라클에서 홑따옴표(')는 문자열 값을 표기하는 데 사용하고, 쌍따옴표(")는 칼럼명, 테이블명과 같은 식별자를 구분하는 용도로만 사용된다.

    2. 이러한 혼란을 방지하기 위해서 ANSI_QUOTES 옵션을 설정하면 홑따옴표(')만 문자열 값 표기로 사용할 수 있고, 쌍따옴표(")는 칼럼명이나 테이블명과 같은 식별자 표기로만 사용할 수 있다.

 - `ONLY_FULL_GROUP_BY` : 
    1. MySQL에서 GROUP BY 절에 명시되지 않은 칼럼이라도 집합 함수없이 SELECT, HAVING 절에 사용할 수 있다.

    2. 이러한 부분은 SQL 표준이나 다른 DBMS와 다른 동작이기에 ONLY_FULL_GROUP_BY 옵션을 통해 엄격한 규칙을 적용할 수 있다.

    3. 5.7버전까진 기본값이 비활성화였지만 8.0버전부터 활성화가 기본값이 되었다.

 - `PIPE_AS_CONCAT` : 
    1. MySQL에서 `||`는 OR 연산자와 같은 의미지만 설정 시 오라클과 같이 문자열 연결 연산자(CONCAT)으로 사용할 수 있다.

 - `PAD_CHAR_TO_FULL_LENGTH` : 
    1. MySQL에서는 CHAR 타입이라고 하더라도 VARCHAR와 같이 유효 문자열 뒤의 공백 문자는 제거되어 반환된다.

    2. PAD_CHAR_TO_FULL_LENGTH 옵션을 설정하면 CHAR 타입의 칼럼에 저장된 값의 뒤따르는 공백 문자를 제거하지 않고 그대로 반환한다.

 - `NO_BACKSLASH_ESCAPES` : 
    1. MySQL에서는 역슬래시 문자(\\)를 이스케이프 문자로 사용할 수 있다.

    2. NO_BACKSLASH_ESCAPES 옵션을 설정하면 역슬래시 문자를 이스케이프 문자로 사용할 수 없게 된다.

 - `IGNORE_SPACE` :
    1. MySQL에서 스토어드 프로시저나 함수의 이름 뒤에 공백이 있으면 "스토어드 프로시저나 함수가 없습니다."라는 에러를 반환할 수도 있다.(공백도 이름으로 인식하기 때문)

    2. IGNORE_SPACE 옵션을 설정하면 스토어드 프로시저나 함수의 이름 뒤에 공백이 있어도 무시한다.

    3. IGNORE_SPACE 옵션은 내장 함수에만 적용되며, 활성화되면 내장 함수는 모두 에약어로 간주되어 테이블이나 칼럼 명으로 사용할 수 없다.

 - `REAL_AS_FLOAT` :
    1. MySQL에서 부동 소수점 타입은 FLOAT, DOUBLE 타입이 지원되는데 REAL 타입은 DOUBLE 타입과 동일한 의미로 사용된다.

    2. REAL_AS_FLOAT 옵션을 설정하면 REAL 타입이 FLOAT 타입과 동일한 의미로 사용된다.

 - `NO_ZERO_IN_DATE & NO_ZERO_DATE` :
    1. 두 옵션이 활성화되면 MySQL에서 DATE, DATETIME 타입의 칼럼에 "2020-00-00", "0000-00-00"과 같은  잘못된 날짜를 저장하는 것이 불가능해진다.

 - `ANSI` :
    1. 앞에서 설명한 "REAL_AS_FLOAT", "PIPE_AS_CONCAT", "ANSI_QUOTES", "IGNORE_SPACE", "ONLY_FULL_GROUP_BY" 모드의 조합으로 구성된 모드다.

 - `TRADITIONAL` :
    1. "STRICT_ALL_TABLES", "STRICT_TRANS_TABLES","NO_ZERO_IN_DATE", "NO_ZERO_DATE", "ERROR_FOR_DIVISION_BY_ZERO", "NO_ENGINE_SUBSTITUTION" 모드의 조합으로 구성된 모드다.

    2. TRADITIONAL 모드를 설정하면 해당 모드가 아닐 때 경고로 처리되던 상황이 모두 에러로 바뀌고 SQL 문장은 실패한다.


### 11-1-2. 영문 대소문자 구분

MySQL 서버는 설치된 운영체제에 따라 테이블명의 대소문자를 구분한다.

 - 유닉스 계열 : 대소문자를 구분한다.

 - Window : 대소문자를 구분하지 않는다.

`lower_case_table_names` : 대소문자 구분 여부를 결정하는 MySQL 시스템 변수

### 11-1-3. MySQL 예약어

생성하는 데이터베이스나 테이블, 칼럼의 이름을 예약어와 같은 키워드로 생성하면 해당 칼럼이나 테이블을 SQL에서 사용하기 위해 항상 역따옴표(`)나 쌍따옴표(")로 감싸야 한다.

## 11-2. 매뉴얼의 SQL 문법 표기를 읽는 방법
```
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY]       
   [IGNORE] 
   [INTO] tbl_name
   [PARTITION (partition_name [, partition_name] ...)]
   [(col_name [, col_name] ...)]
   {VALUES | VALUE} (value_list) [, (value_list)] ...
   [ON DUPLICATE KEY UPDATE assingment_list]

value: {expr | DEFAULT}

value_list: value[, value]...

assingment: col_name = value

assingment_list: assingment[, assingment]...
```

`대괄호([])` : 해당 키워드난 표현식 자체가 선택 사항

`파이프(|)` : 앞과 뒤의 키워드나 표현식 중 하나를 선택해서 사용할 수 있음.

`중괄호({})` : 괄호 내의 아이템 중에서 반드시 하나를 사용해야 함.

`...` : 앞에 명시된 키워드나 표현식의 조합이 반복될 수 있음.

## 11-3. MySQL 연산자와 내장 함수

가능하면 SQL의 가독성을 높이기 위해 ANSI 표준 형태의 연산자를 사용하는 것을 권장

### 11-3-1. 리터럴 표기법 문자열

#### 11-3-1-1. 문자열

SQL 표준에서 문자열은 항상 홑따옴표(')를 사용해서 표시한다. 단 MySQL에서는 쌍따옴표를 사용해서 문자열을 표기할 수 있다.

```sql
SELECT * FROM departments WHERE dept_no = 'd001';
SELECT * FROM departments WHERE dept_no = "d001";
```

```sql
SELECT * FROM departments WHERE dept_no = 'd''001'; -- // SQL 표준
SELECT * FROM departments WHERE dept_no = 'd"001'; -- // SQL 표준
SELECT * FROM departments WHERE dept_no = "d'001"; -- // MySQL
SELECT * FROM departments WHERE dept_no = "d""001"; -- // MySQL
```

SQL에선 사용되는 식별자(테이블명이나 칼럼명 등)가 키워드와 충돌할 때 오라클이나 PostgreSQL에서는 쌍따옴표나 대괄호로 감싸서 충돌을 피한다. MySQL에서는 역따옴표(`)를 사용한다.


```sql
CREATE TABLE tab_test (`table` VARCHAR(20) NOT NULL, ...);
SELECT `column` FROM tab_test;
```
단, ANSI_QUOTES 옵션을 설정하면 쌍따옴표(")를 사용해야 한다.

```sql
CREATE TABLE tab_test ("table" VARCHAR(20) NOT NULL, ...);
SELECT "column" FROM tab_test;
```

#### 11-3-1-2. 숫자

숫자를 사용할땐 보통 따옴표 없이 숫자 값을 입력하면 되나, 문자열 형태이더라도 비교 대상이 숫자 값이거나 숫자 타입의 칼럼이면 MySQL에서 자동으로 숫자 값으로 변환한다.

```sql
SELECT * FROM tab_test WHERE number_column = '10001';
SELECT * FROM tab_test WHERE string_column = 10001;
```

단 두 번째 쿼리는 문자열 칼럼을 숫자로 변환해서 비교 수행해야 하므로 인덱스가 있더라도 이를 활용하지 못하며, 만약 string_column에 알파벳과 같은 문자가 포홤된 경우에는 숫자 값으로 변환할 수 없으므로 쿼리 자체가 실패할 수도 있다.

#### 11-3-1-3. 날짜

다른 DBMS는 날짜 타입을 비교하거나 INSERT하려면 문자열을 DATE타입으로 변환하는 코드가 필요하다.

하지만 MySQL 서버는 자동으로 DATE나 DATETIIME 값으로 변환하기 때문에 `STR_TO_DATE()` 같은 함수를 사용하지 않아도 된다.

```sql
SELECT * FROM dept_emp WHERE from_date = '2011-04-29';

SELECT * FROM dept_emp WHERE from_date = STR_TO_DATE('2011-04-29', '%Y-%m-%d');
```

#### 11-3-1-4. 불리언

`BOOL, BOOLEAN` : `TINYINT` 타입에 대한 동의어

### 11-3-2. MySQL 연산자

#### 11-3-2-1. 동등(Equal) 비교(=, <=>)

`=` : 일반적인 동등 비교 연산자

`<=>` : NULL값에 대한 비교할 수 있는 동등 비교 연산자(NULL-Safe 비교 연산자) -> MySQL에서만 지원

#### 11-3-2-2. 부정(Not-Equal) 비교(<>, !=)

"값지 않다" 비교를 위한 연산자로 `<>`와 `!=` 두 가지가 있다.

#### 11-3-2-3. NOT 연산자(!)

`NOT` , `!` : TRUE 또는 FALSE 연산의 결과를 반대로 만드는 연산자

#### 11-3-2-4. AND(&&)와 OR(||) 연산자

`AND`, `OR` : d일반적인 DBMS에서 불리언 표현식의 결과를 결합하기 위한 연산자

`&&`, `||` : MySQL에서만 사용가능한 `AND`, `OR` 연산자의 표기 방식

`||`는 오라클에서 문자열을 결합하는 연산자로 사용되기에 사용에 주의가 필요하다.

```
주의 : OR과 AND를 혼용해서 사용할 경우 AND가 우선적으로 처리된다음 OR로 처리되기 때문에 괄호를 사용해서 처리하는게 좋다.
```

#### 11-3-2-5. 나누기(/, DIV)와 나머지(%, MOD) 연산자

```sql
SELECT 29 / 9;
SELECT 29 DIV 9;
SELECT MOD(29, 9);
SELECT 29 MOD 9;
SELECT 29 % 9;
```

#### 11-3-2-6. REGEXP 연산자

`REGEXP` : 문자열 값이 어떤 패턴을 만족하는지 확인하는 연산자

`RLIKE` : `REGEXP`와 동일한 기능을 수행하는 연산자(정규 표현식을 비교하는 연산자)

```sql
SELECT 'abc' REGEXP '^[x-z]';
```

**대표적인 심벌 - 자세한 건 POSIX 정규 표현식 매뉴얼 참조**

- `^` : 문자열의 시작을 표시. "^"심벌을 표현식의 앞쪽에 넣어주면 일치하는 부분이 반드시 문자열의 시작 부분에 있어야함을 의미한다.

- `$` : 문자열의 끝을 표시. "$"심벌을 표현식의 뒤쪽에 넣어주면 일치하는 부분이 반드시 문자열의 끝 부분에 있어야함을 의미한다.

- `[]` : 문자 그룹을 표시. [xyz] 또는 [x-z]라고 표현하면 'x','y','z' 중 하나인지 확인하는 것이다. 대괄호는 문자열이 아니라 문자 하나와 일치하는지를 확인하는 것이다.

- `()` : 문자열 그룹을 표시. (xyz)라고 표현하면 "xyz"가  모두 있는지 확인하는 것이다.

- `|` : (abc|xyz)라고 표현하면 "abc" 또는 "xyz" 중 하나인지 확인하는 것이다.

- `.` : 어떠한 문자든지 1개의 문자를 표시. "..."라고 표현했다면 3개의 문자로 구성된 문자열을 찾는 것이다.

- `*` : 이 기호 앞에 표시된 정규 표현식이 0번 또는 1번 이상 반복될 수 있다는 표시.

- `+` : 이 기호 앞에 표시된 정규 표현식이 1번 이상 반복될 수 있다는 표시.

- `?` : 이 기호 앞에 표시된 정규 표현식이 0번 또는 1번만 올 수 있다는 표시.


REGEXP 연산자를 문자열 칼럼 비교에 사용할 때 인덱스 레인지 스캔을 사용할 수 없다.

#### 11-3-2-7. LIKE 연산자

REGEXP 연산자보다 훨씬 단순한 문자열 패턴 비교 연산자로 인덱스를 이용해 처리할 수도 있다.

```sql
mysql> SELECT 'abcdef' LIKE 'abc%';

mysql> SELECT 'abcdef' LIKE '%abc';

mysql> SELECT 'abcdef' LIKE '%ef';
```

 - `%` : 0개 또는 1개 이상의 모든 문자에 일치(문자의 내용과 관계없이)
 - `_` : 정확히 1개의 임의의 문자와 일치(문자의 내용과 관계없이)

 ESCAPE 절을 LIKE 조건 뒤에 추가해 이스케이프 문자를 설정할 수 있다.

 ```sql
 mysql> SELECT 'abc' LIKE 'a/%' ESCAPE '/';
 mysql> SELECT 'a%' LIKE 'a/%' ESCAPE '/';
 ```

 와일드카드 문자가 검색어 뒤쪽에 있다면 인덱스 레인지 스캔으로 사용할 수 잇지만 검색어 앞쪽에 있다면 인덱스를 전혀 활용할 수 없다.

 #### 11-3-2-8. BETWEEN 연산자

 "크거나 같다"와 "작거나 같다"라는 2개의 연산자를 하나로 합친 연산자.

 ```sql
 SELECT * FROM dept_emp WHERE dept_no = 'd003' AND emp_no = 10001;

 SELECT * FROM dept_emp WHERE dept_no BETWEEN 'd003' AND 'd005' AND emp_no = 10001;
 ```
dept_emp 테이블에서 (dept_no, emp_no) 칼럼으로 구성된 프라이머리 키가 존재한다면, 첫 번째 쿼리는 두 조건 모두 인덱스를 이용해 범위를 줄여줄 수 있다.

단 두 번째 쿼리에서 사용한 BETWEEN 연산자는 dept_no를 모든 인덱스 범위에서 검색해야해서 emp_no가 비교 범위를 줄이는 역할을 할 수 없다.

```sql
SELECT * FROM dept_emp WHERE dept_no IN ('d003', 'd004','d005') AND emp_no = 10001;
```
BETWEEN이 선형으로 인덱스를 검색해야 하는 것과 달리 IN은 동등 비교를 여러번 수행하는 것과 같은 효과가 있기 때문에 인덱스를 최적으로 사용할 수 있다.

#### 11-3-2-9. IN 연산자

`IN` : 여러 개의 값에 대해 동등 비교 연산을 수행하는 연산자. 여러 개의 값이 비교되지만 범위로 검색하는 것이 아니라 여러 번의 동등 비교로 실행하기 때문에 일반적으로 빠르게 처리된다.

 - 상수가 사용된 경우 - IN (?, ?, ?)

 - 서브쿼리가 사용된 경우 - IN (SELECT ... FROM ...)

 8.0 버전부터는 IN 절에 튜플을 그대로 나열해도 인덱스를 최적으로 사용할 수 있게 개선됐다.

 ```sql
 mysql> SELECT *
         FROM dept_emp
         WHERE (dept_no, emp_no) IN (('d001', 10017), ('d002', 10144), ('d003', 100545));
 ```  

 NOT IN의 실행 계획은 인덱스 풀 스캔으로 표시되는데, 동등이 아닌 부정형 비교여서 인덱스를 이용해 처리 범위를 줄이는 조건으로는 사용할 수 없기 때문이다.

 ### 11-3-3. MySQL 내장 함수

 MySQL 함수는 2가지로 구분

  - 내장 함수

  - 사용자 정의 함수(UDF, User Defined Function)

#### 11-3-3-1. NULL 값 비교 및 대체(IFNULL,ISNULL)

`IFNULL()` : 칼럼이나 표현식의 값이 NULL인지 비교하고, NULL이면 다른 값으로 대체하는 용도

`ISNULL()` : 칼럼이나 표현식의 값이 NULL인지 아닌지 비교하는 용도

```sql
mysql> SELECT IFNULL(NULL, 1) --// 1

mysql> SELECT IFNULL(0,1)  --// 0

mysql> SELECT ISNULL(0) --// 0

mysql> SELECT ISNULL(1/0) --// 1
```
#### 11-3-3-2. 현재 시각 조회(NOW, SYSDATE)

두 함수 모두 현재의 시간을 반환하는 함수로서 같은 기능을 수행한다.

NOW() : 하나의 SQL에서 모두 같은 값을 반환

SYSDATE() : 호출될 때마다 다른 값을 반환

SYSDATE() 함수의 잠재적인 문제

 1. SYSDATE() 함수가 사용된 SQL은 레플리카 서버에서 안정적으로 복제되지 못한다.

 2. SYSDATE() 함수와 비교되는 칼럼은 인덱스를 효율적으로 사용하지 못한다.

```sql
mysql> EXPLAIN
         SELECT emp_no, salary, from_date, to_date
         FROM salaries
         WHERE emp_no = 10001 AND from_date > NOW();

mysql> EXPLAIN
         SELECT emp_no, salary, from_date, to_date
         FROM salaries
         WHERE emp_no = 10001 AND from_date > SYSDATE();
```

SYSDATE() 함수는 호출될 때마다 다른 값을 반환하므로 상수가 아니다. 그래서 인덱스를 스캔할 때도 매번 비교되는 레코드마다 함수를 실행해야 하기에 인덱스를 효율적으로 사용할 수 없는 것이다.

#### 11-3-3-3. 날짜와 시간의 포맷(DATE_FORMAT, STR_TO_DATE)

**DATE_FORMAT()의 대표적인 지정자**

| 지정문자 | 내용 |
| -- | -- |
| %Y | 4자리 연도 |
| %m | 2자리 숫자 표시의 월(01~12) |
| %d | 2자리 숫자 표시의 일자(01~31) |
| %H | 2자리 숫자 표시의 시간(00~23) |
| %i | 2자리 숫자 표시의 분(00~59) |
| %s | 2자리 숫자 표시의 초(00~59) |

```sql
mysql> SELECT DATE_FORMAT(NOW(), '%Y-%m-%d') AS current_dt;

mysql> SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s') AS current_dttm;
```

SQL에서 표준 형태(년-월-일 시:분:초) 문자열이나 이 밖에도 DATETIME으로 자동 변환 가능한 형태가 있다.

그렇지 않은 경우 STR_TO_DATE() 함수를 이용해 문자열을 DATETIME타입으로 변환할 수 있다.

```sql
mysql> SELECT STR_TO_DATE('2020-08-23', '%Y-%m-%d') AS current_dt;

mysql> SELECT STR_TO_DATE('2020-08-23 15:06:45', '%Y-%m-%d %H:%i:%s') AS current_dttm;
```

#### 11-3-3-4. 날짜와 시간의 연산(DATE_ADD, DATE_SUB)

```sql
mysql> SELECT DATE_ADD(NOW(), INTERVAL 1 DAY) AS tomorrow;

mysql> SELECT DATE_ADD(NOW(), INTERVAL -1 DAY) AS yesterday;
```

| 단위 | 의미 |
| -- | -- |
| YEAR | 연 |
| MONTH | 월 |
| DAY | 일 |
| HOUR | 시간 |
| MINUTE | 분 |
| SECOND | 초 |
| MICROSECOND | 마이크로 초 |
| QUARTER | 분기 |
| WEEK | 주 |

#### 11-3-3-5. 타임스탬프 연산(UNIX_TIMESTAMP, FROM_UNIXTIME)

`UNIX_TIMESTAMP()` : '1970-01-01 00:00:00'으로부터 경과된 초의 수를 반환하는 함수. 인자가 없으면 현재 날짜와 시간의 타임스탬프 값을, 인자로 특정 날짜를 전달하면 그 날짜와 시간으 타임스탬프를 반환한다.

`FROM_UNIXTIME()` : 인자로 전달한 타임스탬프 값을 DATETIME타입으로 변환하는 함수.

```sql
mysql> SELECT UNIX_TIMESTAMP();

mysql> SELECT UNIX_TIMESTAMP('2020-08-23 15:06:45');

mysql> SELECT FROM_UNIXTIME(UNIX_TIMESTAMP('2020-08-23 15:06:45'));
```

MYSQL TIMESTAMP 타입은 '1970-01-01 00:00:01' ~ '2038-01-09 03:14:07' 사이 값만 가능하다.

#### 11-3-3-6. 문자열 처리(RPAD, LPAD / RTRIM, LTRIM, TRIM)

`RPAD()`,`LPAD()` : 문자열의 좌측 또는 우측에 문자를 덧붙여서 지정된 길이의 문자열로 만드는 함수.

`RTRIM()`,`LTRIM()`,`TRIM()` : 문자열의 우측, 좌측, 양쪽에서 공백을 제거하는 함수.

```sql
mysql> SELECT RPAD('Cloee', 10, '_');
mysql> SELECT LPAD('123', 10, '0');
mysql> SELECT RTRIM('Cloee   ');
mysql> SELECT LTRIM('   Cloee');
mysql> SELECT TRIM('   Cloee   ');
```

#### 11-3-3-7. 문자열 연결(CONCAT)

`CONCAT()` : 여러 개의 문자열을 연결하는 함수. 숫자 값을 인자로 전달하면 자동으로 문자열로 변환되지만, 의도된 결과가 아닌 경우 명시적으로 CAST() 함수를 사용해 변환할 수 있다.

```sql
mysql> SELECT CONCAT('Georgi', 'Christian') AS name;

mysql> SELECT CONCAT('Georgi', 'Christian',2) AS name;

mysql> SELECT CONCAT('Georgi', 'Christian',CAST(2 AS CHAR)) AS name;
```

`CONCAT_WS()` : 각 문자열을 연결할 때 구분자를 넣어준다는 점을 제외하면 CONCAT()와 동일하다.

```sql
mysql> SELECT CONCAT_WS(',', 'Georgi', 'Christian') AS name;
```

#### 11-3-3-8. GROUP BY 문자열 결합(GROUP_CONCAT)

`GROUP_CONCAT()` : GROUP BY 절과 함께 사용하여 GROUP BY 절로 묶인 레코드의 칼럼 값을 하나의 문자열로 결합하는 함수.

GROUP BY가 없는 SQL에서 사용하면 하나의 결괏값만 만들어낸다.

```sql
mysql> SELECT GROUP_CONCAT(dept_no) FROM departments;

mysql> SELECT GROUP_CONCAT(dept_no SEPARATOR '|') FROM departments;

mysql> SELECT GROUP_CONCAT(dept_no ORDER BY emp_no DESC) 
         FROM departments
         WHERE emp_no BETWEEN 100001 AND 100003;

mysql> SELECT GROUP_CONCAT(DISTINCT dept_no ORDER BY emp_no DESC) 
         FROM departments
         WHERE emp_no BETWEEN 100001 AND 100003;

```

- 첫 번째 예제는 가장 기본적인 형태로 쿼리를 실행하면 모든 레코드에서 dept_no 칼럼의 값을 기본 구분자(,)로 연결한 값을 반환

- 두 번째 예제는 dept_no 칼럼의 값을 구분자(|)로 연결한 값을 반환

- 세 번째 예제는 emp_no 칼럼의 역순으로 정렬해서, dept_no 칼럼의 값을 연결해서 가져오는 쿼리.

- 네 번째 예제는 중복된 dept_no 값을 제거하고 유니크한 dept_no 값만을 연결해서 값을 가져오는 쿼리.

GROUP_CONCAT() 함수는 지정된 칼럼의 값들을 연결하기 위해 제한적인 메모리 버퍼 공간을 사용한다. 이 때 결과가 지정된 크기를 초과하면 SQL GUI 도구에선 경고로 그치지만, JDBC로 실행될 때는 경고가 아닌 에러로 취급되어 쿼리가 실패하기 때문에 초과되지 않게 주의해야한다.

`group_concat_max_len` : GROUP_CONCAT() 함수가 사용할 수 있는 버퍼 공간의 크기를 지정하는 시스템 변수.

8.0버전부터 용도에 맞게 래터럴 조인이나 윈도우 함수를 이용할 수 있게 됐다.

```sql
--// 윈도우 함수를 이용해 최대 5개 부서만 GROUP_CONCAT 실행
mysql> SELECT GROUP_CONCAT(dept_no ORDER BY emp_no DESC) 
         FROM (
            SELECT *, RANK() OVER (ORDER BY dept_no) AS rnk
            FROM departments
         ) as x
         WHERE rnk <= 5;

--// 래터럴 조인을 이용해 부서별로 10명씩만 GROUP_CONCAT 실행
mysql> SELECT d.dept_no, GROUP_CONCAT(de2.emp_no)
         FROM departments d
         LEFT JOIN LATERAL (
            SELECT de.dept_no, de.emp_no
            FROM dept_emp de
            WHERE de.dept_no = d.dept_no
            ORDER BY de.emp_no ASC
            LIMIT 10
         ) as de2 ON de2.dept_no = d.dept_no
         GROUP BY d.dept_no;
```

#### 11-3-3-9. 값의 비교와 대체(CASE WHEN ... THEN ... END)

프로그래밍 언어에서 SWITCH 구문과 같은 역할하는 SQL 구문.

```sql
mysql> SELECT emp_no, first_name,
         CASE gender
            WHEN 'M' THEN 'Man'
            WHEN 'F' THEN 'Woman'
            ELSE 'Unknown' END AS gender
         FROM employees
         LIMIT 10;

mysql> SELECT emp_no, first_name,
         CASE
            WHEN hire_date < '1995-01-01' THEN 'Old'
            ELSE 'New' END AS employee_type
         FROM employees
         LIMIT 10;
```

`CASE WHEN`구문에서 중요한 것은 CASE WHEN 절이 일치하는 경우에만 THEN 이하의 표현식이 실행된다는 점이다. 이 점을 잘 활용하면 쿼리 튜닝이 가능할 수도 있다.

#### 11-3-3-10. 타입의 변환(CAST, CONVERT)

`CAST()` : 지정된 표현식을 지정된 타입으로 변환하는 함수.

변환할 수 있는 데이터 타입 : `DATE`, `TIME`, `DATETIME`, `BINARY`, `CHAR`, `DECIMAL`, `SIGNED INTEGER`, `UNSIGNED INTEGER`

`CONVERT()` : 지정된 표현식을 지정된 타입으로 변환하는 용도와 문자열의 문자 집합을 변환하는 용도 2가지로 사용할 수 있다.

```sql
mysql> SELECT CAST('1234' AS SIGNED INTEGER) AS converted_integer;

mysql> SELECT CAST('2000-01-01' AS DATE) AS converted_date;

mysql> SELECT CAST(1-2 AS UNSIGNED);

mysql> SELECT 1-2;

mysql> SELECT CONVERT(1-2, UNSIGNED);

mysql> SELECT CONVERT('ABC' USING 'utf8mb4');
```

#### 11-3-3-11. 이진값과 16진수 문자열(Hex String) 변환(HEX, UNHEX)

`HEX()` : 이진값을 16진수 문자열(Hex String)로 변환하는 함수.

`UNHEX()` : 16진수 문자열(Hex String)을 이진값(BINARY)으로 변환하는 함수.

#### 11-3-3-12. 암호화 및 해시 함수(MD5, SHA, SHA2)

`MD5`, `SHA` : 비대칭형 암호화 알고리즘, 인자로 전달한 문자열을 각각 지정된 비트 수의 해시 값을 만들어내는 함수

`SHA()` : SHA-1 알고리즘을 사용해 160비트 해시 값을 반환하는 함수. 저장 공간 : 40바이트

`SHA2()` : SHA-2 알고리즘을 사용해 224비트 ~ 512비트 해시 값을 반환하는 함수. 저장 공간 : 사용된 인자의 2배 크기

`MD5()` : 메시지 다이제스트 알고리즘을 사용해 128비트(16바이트) 해시 값을 반환하는 함수. 저장 공간 : 32바이트

```sql
mysql> SELECT MD5('abc');

mysql> SELECT SHA('abc');

mysql> SELECT SHA2('abc', 256);
```

저장 공간을 줄이려면 CHAR나 VARCHAR가 아닌 BINARY 또는 VARBINARY 타입을 저장하면 된다. 이때 함수의 결과를 UNHEX(), HEX() 함수를 이용해 변환해야 한다.

이 함수들의 결괏값은 중복 가능성이 매우 낮기 때문에 길이가 긴 데이터를 크기를 줄여서 인덱싱(해시)하는 용도로도 사용된다.

```sql
mysql> SELECT * FROM tb_accesslog WHERE MD5(access_url) = MD5('http://matt.com');

mysql> SELECT * FROM tb_accesslog WHERE UNHEX(MD5(access_url)) = UNHEX(MD5('http://matt.com'));
```

#### 11-3-3-13. 처리 대기(SLEEP)

`SLEEP()` : 프로그래밍 언어나 셸 스크립트 언어에서 제공하는 "sleep" 기능을 수행.

```sql
mysql> SELECT SLEEP(1.5)
         FROM employees
         WHERE emp_no BETWEEN 10001 AND 10010;
```

#### 11-3-3-14. 벤치마크(BENCHMARK)

`BENCHMARK()` : 지정된 횟수만큼 지정된 표현식을 실행하는 함수.

이때 표현식은 스칼라값(하나의 칼럼을 가진 하나의 레코드)을 반환하는 표현식이여야 하며, SELECT 쿼리를 사용하는 것도 가능하나 스칼라값을 반환해야만 한다.

```sql
mysql> SELECT BENCHMARK(1000000, MD5('abcdefghijk'));

mysql> SELECT BENCHMARK(1000000, (SELECT COUNT(*) FROM salaries));
```
`BENCHMARK()`로 얻은 쿼리나 함수의 성능은 그 자체로는 큰 의미가 없으며, 2개의 동일 기능을 상대적으로 비교 분석하는 용도로 사용할 것을 권장한다.

#### 11-3-3-15. IP 주소 변환(INET_ATON, INET_NTOA)

MySQL에서는 `INET_ATON()`과 `INET_NTOA()`를 이용해 IPv4 주소를 문자열이 아닌 부호 없는 정수 타입에 저장할 수 있게 제공한다.

`INET_ATON()` : IPv4 주소 문자열을 부호 없는 정수로 변환하는 함수

`INET_NTOA()` : 부호 없는 정수를 IPv4 주소 문자열로 변환하는 함수

`INET6_ATON()` : IPv4,IPv6 주소 문자열을 BINARY타입으로 변환하는 함수

`INET6_NTOA()` : BINARY타입을 IPv4, IPv6 주소 문자열로 변환하는 함수

이때 IPv4는 BINARY(4), IPv6는 BINARY(16) 타입으로 사용해야한다.

```sql
mysql> SELECT HEX(INET6_ATON('fdfe:5a55:caff:fefa:9089')); --// FDFE0000000000005A55CAFFFEFA9089

mysql> SELECT HEX(INET6_ATON('10.0.5.9')); --// 0A000509

mysql> SELECT INET6_NTOA(UNHEX('FDFE0000000000005A55CAFFFEFA9089')); --// fdfe:5a55:caff:fefa:9089

mysql> SELECT INET6_NTOA(UNHEX('0A000509')); --// 10.0.5.9
```

#### 11-3-3-16. JSON 포맷(JSON_PRETTY)

`JSON_PRETTY()` : JSON 칼럼의 값을 읽기 쉬운 포맷으로 변환해주는 함수

#### 11-3-3-17. JSON 필드 크기(JSON_STORAGE_SIZE)

JSON 데이터를 실제 디스크에 저장할 때 BSON(Binary JSON) 포맷을 사용하나, 이는 저장 공간의 크기가 얼마나 될지 예측하기 어렵다.

`JSON_STORAGE_SIZE()` : JSON 칼럼의 값을 저장하는데 사용된 바이트 수를 반환하는 함수

#### 11-3-3-18. JSON 필드 추출(JSON_EXTRACT)

`JSON_EXTRACT()` : JSON 도큐먼트에서 특정 필드의 값을 가져오는 함수

```sql
mysql> SELECT emp_no, JSON_EXTRACT(doc, "$.first_name") FROM employee_docs;

mysql> SELECT emp_no, JSON_UNQUOTE(JSON_EXTRACT(doc, "$.first_name")) FROM employee_docs;
```

`JSON_UNQUOTE()` : JSON 문자열에서 따옴표를 제거하는 함수 

`->` : JSON_EXTRACT()의 대체연산자

`->>` : JSON_UNQUOTE(JSON_EXTRACT())의 대체연산자

#### 11-3-3-19. JSON 오브젝트 포함 여부 확인(JSON_CONTRAINS)

`JSON_CONTRAINS()` : JSON 도큐먼트 또는 지정된 JSON 경로에 JSON 필드를 가지고 있는지 확인하는 함수

```sql
mysql> SELECT emp_no FROM employee_docs
         WHERE JSON_CONTRAINS(doc, '{"first_name":"Christian"}');

mysql> SELECT emp_no FROM employee_docs
         WHERE JSON_CONTAINS(doc, '"Christian"', '$.first_name');
```

#### 11-3-3-20. JSON 오브젝트 생성(JSON_OBJECT)

`JSON_OBJECT()` : RDBMS 칼럼의 값을 이용해 JSON 오브젝트를 생성하는 함수

```sql
mysql> SELECT JSON_OBJECT("empNo", emp_no, 
                           "salary", salary,
                           "fromDate", from_date,
                           "toDate", to_date) AS as_json
         FROM salaries LIMIT 3;
```

#### 11-3-3-21. JSON 칼럼으로 집계(JSON_OBJECTAGG & JSON_ARRAYAGG)

`JSON_OBJECTAGG()`, `JSON_ARRAYAGG()` : GROUP BY 절과 함께 사용되는 집계 함수로서, RDBMS 칼럼의 값들을 모아 JSON 배열 또는 도큐먼트를 생성하는 함수.

```sql
mysql> SELECT dept_no, JSON_OBJECTAGG(emp_no, from_date) AS agg_manager
         FROM dept_manager
         WHERE dept_no IN ('d001', 'd002', 'd003')
         GROUP BY dept_no;

mysql> SELECT dept_no, JSON_ARRAYAGG(emp_no) as agg_manager
         FROM dept_manager
         WHERE dept_no IN ('d001', 'd002', 'd003')
         GROUP BY dept_no;
```

#### 11-3-3-22. JSON 데이터를 테이블로 변환(JSON_TABLE)

`JSON_TABLE()` : JSON 데이터의 값들을 모아서 RDBMS 테이블을 만들어 반환하는 함수. 이때 함수가 만들어서 반환하는 테이블의 레코드 건수는 원본 테이블과 동일한 레코드 건수를 가진다.

```sql
mysql> SELECT e2.emp_no, e2.first_name, e2.gender
         FROM employee_docs e1,
              JSON_TABLE(doc, "$" COLUMNS(
                emp_no INT PATH "$.emp_no",
                gender CHAR(1) PATH "$.gender",
                first_name VARCHAR(20) PATH "$.first_name"
              )) AS e2
         WHERE e1.emp_no IN (10001, 10002);
```

## 11-4. SELECT

### 11-4-1. SELECT 절의 처리 순서
```
// 일반적인 경우
드라이빙 테이블, 드리븐 테이블 WHERE 적용 및 조인 실행
-> GROUP BY
-> DISTINCT
-> HAVING 조건 적용
-> ORDER BY
-> LIMIT
```

```
// ORDER BY가 조인보다 먼저 실행되는 경우(GROUP BY 없이 ORDER BY만 사용된 쿼리)

드라이빙 테이블 WHERE 적용
-> ORDER BY 
-> 드리븐 테이블 조인 실행
-> LIMIT
```

### 11-4-2. WHERE 절과 GROUP BY 절, ORDER BY 절의 인덱스 사용

#### 11-4-2-1. 인덱스를 사용하기 위한 기본 규칙

WHERE 조건이나 GROUP BY 또는 ORDER BY에서 원본값을 검색하거나 정렬할 때만 B-Tree에 정렬된 인덱스를 이용한다.

```sql
mysql> SELECT * FROM salaries WHERE salary*10>150000; // 인덱스 사용 불가

mysql> SELECT * FROM salaries WHERE salary>150000/10; // 인덱스 사용 가능
```
복잡한 연산을 수행한다거나 해시 값을 만들어서 비교해야 하는 경우 미리 계산된 값을 저장하도록 MySQL의 가상 칼람을 추가하고 그 칼럼에 인덱스를 생성하거나 함수 기반의 인덱스를 사용해야 한다.

WHERE 절에 사용되는 비교 조건에서 연산자 양쪽의 두 비교 대상 값은 데이터 타입이 일치해야만 인덱스를 효율적으로 사용할 수 있다.

#### 11-4-2-2. WHERE 절의 인덱스 사용

WHERE 조건이 인덱스를 사용하는 방법은 크게 작업 범위 결정 조건과 체크 조건 두 가지 방식으로 구분된다.

작업 범위 결정 조건은 WHERE 절에서 동등 비교 조건이나 IN으로 구성된 조건에 사용된 칼럼들이 인덱스의 칼럼 구성과 좌측에서부터 비교했을 때 얼마나 일치하는가에 따라 달라진다.

WHERE 절의 나열 순서는 인덱스의 사용 여부와 관계없으며, 오로지 인덱스 순서상 동등 비교로 된 칼럼까지만 작업 범위 결정 조건으로 사용될 수 있다.

```sql
ALTER TABLE ... ADD INDEX ix_col1234 (col_1 ASC, col_2 DESC, col_3 ASC, col_4 ASC);
```
8.0버전부터 인덱스를 구성하는 칼럼별로 정렬을 혼합해서 생성할 수 있게 개선되었지만, 권장하진 않는다.

WHERE 조건에 OR 연산자가 있을 경우 AND 연산자와 다르게 비교해야 할 레코드가 더 늘어나 풀 테이블 스캔을 할 가능성이 높아진다.

#### 11-4-2-3. GROUP BY 절의 인덱스 사용

GROUP BY 절에 명시된 칼럼의 순서가 인덱스를 구성하는 칼럼의 순서와 같으면 인덱스를 사용할 수 있다.

 - GROUP BY 절에 명시된 칼럼이 인덱스 칼럼의 순서와 위치가 같아야 한다.

 - 인덱스를 구성하는 칼럼 중에서 뒤쪽에 있는 칼럼은 GROUP BY 절에 명시되지 않아도 인덱스를 사용할 수 있지만 인덱스의 앞쪽에 있는 칼럼이 GROUP BY 절에 명시되지 않으면 사용할 수 없다.

 - WHERE 조건절과는 달리 GROUP BY 절에 명시된 칼럼이 하나라도 인덱스에 없다면 GROUP BY 절은 인덱스를 이용하지 못한다.

**GROUP BY에서 인덱스를 이용하지 못하는 경우**

```
// 인덱스(COL_1, COL_2, COL_3, COL_4)

... GROUP BY COL_2, COL_1
... GROUP BY COL_1, COL_3, COL_2
... GROUP BY COL_1, COL_3
... GROUP BY COL_1, COL_2, COL_3, COL_4, COL_5
```
 - 1, 2번째 예제는 GROUP BY 칼럼이 인덱스를 구성하는 칼럼의 순서와 일치하지 않음.

 - 3번째 예제는 COL_3가 명시됐지만 COL_2가 그 앞에 명시되지 않음.

 - 4번째 예제는 인덱스에 없는 COL_5가 GROUP BY 절에 추가됨.


```
--// 원본 쿼리
... WHERE COL_1='상수' ... GROUP BY COL_2, COL_3

--// WHERE 조건절의 COL_1 칼럼을 GROUP BY 절의 앞쪽으로 포함시켜 본 쿼리
... WHERE COL_1='상수' ... GROUP BY COL_1, COL_2, COL_3
```
 - WHERE 조건절에서 앞에 인덱스 컬럼이 사용됐다면 GROUP BY 절에서 해당 칼럼이 명시되지 않아도 인덱스를 이용할 수 있다.

 - 앞에 인덱스 컬럼이 WHERE 절에서 상숫값과 비교되었다면 2개의 쿼리는 같은 결과를 반환한다.

#### 11-4-2-4. ORDER BY 절의 인덱스 사용

ORDER BY 절은 GROUP BY와 인덱스 사용 조건이 유사하지만 한 가지 추가 조건이 있다.

 - 정렬되는 각 컬럼의 오름차순(ASC) 및 내림차순(DESC) 옵션이 인덱스와 같거나 정반대인 경우에만 사용할 수 있다.

#### 11-4-2-5. WHERE 조건과 ORDER BY(또는 GROUP BY) 절의 인덱스 사용

일반적으로 WHERE 조건절과 ORDER BY(또는 GROUP BY) 절이 서로 다른 인덱스를 사용할 수 없다.

**WHERE 절과 ORDER BY 절이 같이 사용된 쿼리에서의 인덱스 사용 방법**

 - WHERE 절과 ORDER BY 절이 동시에 같은 인덱스 이용 : 가장 빠른 성능

 - WHERE 절만 인덱스를 이용 : 인덱스를 통해 검색된 결과 레코드를 별도의 정렬 처리 과정을 거쳐 정렬하는 방식

 - ORDER BY 절만 인덱스를 이용 : ORDER BY 절의 순서대로 인덱스를 읽으면서 레코드 한 건씩 WHERE 절의 조건에 일치하는 비교하고, 일치하지 않을 때는 버리는 형태.

WHERE 절과 ORDER BY 절에 명시된 칼럼은 순서대로 인덱스 칼럼 왼쪽부터 일치해야하며, 중간에 빠진 칼럼이 있으면 주로 WHERE 절만 인덱스를 이용할 수 있다.

#### 11-4-2-6. GROUP BY 절과 ORDER BY 절의 인덱스 사용

GROUP BY 절에 명시된 칼럼과 ORDER BY에 명시된 칼럼의 순서와 내용이 모두 같아야 하나의 인덱스를 사용할 수 있으며, 하나라도 인덱스를 이용할 수 없을 때는 둘 다 인덱스를 사용하지 못한다.

8.0 버전부터 GROUP BY 절이 칼럼의 정렬까지는 보장하지 않는 형태로 바뀌었기 때문에 칼럼의 그루핑과 정렬을 모두 수행하려면 GROUP BY 절과 ORDER BY 절 모두 명시해야한다.

### 11-4-3. WHERE 절의 비교 조건 사용 시 주의사항

#### 11-4-3-1. NULL 비교

MySQL에서는 NULL 값이 포함된 레코드도 인덱스로 관리하며, 이땐 "IS NULL"(또는 "<=>") 연산자를 사용해야 한다.

```sql
mysql> SELECT NULL = NULL;

mysql> SELECT NULL <=> NULL;

mysql> SELECT CASE WHEN NULL = NULL THEN 1 ELSE 0 END;

mysql> SELECT CASE WHEN NULL IS NULL THEN 1 ELSE 0 END;
```

**ISNULL() 함수 사용시 주의점**

```sql
mysql> SELECT * FROM titles WHERE to_date IS NULL; --// 레인지 스캔

mysql> SELECT * fROM titles WHERE ISNULL(to_date); --// 레인지 스캔

mysql> SELECT * FROM titles WHERE ISNULL(to_date) = 1; --// 인덱스 or 테이블 풀 스캔

mysql> SELECT * FROM titles WHERE ISNULL(to_date) = true; --// 인덱스 or 테이블 풀 스캔
```

#### 11-4-3-2. 문자열이나 숫자 비교

문자열 칼럼이나 숫자 칼럼을 비교할 때는 반드시 그 타입에 맞는 상숫값을 사용할 것을 권장.

```sql
mysql> SELECT * FROM employees WHERE emp_no = 10001;

mysql> SELECT * FROM employees WHERE first_name = 'Smith';

mysql> SELECT * FROM employees WHERE emp_no = '10001';

mysql> SELECT * FROM employees WHERE first_name = 10001;
```

1,2,3번째 쿼리는 성능 저하는 발생하지 않으나, 4번째 쿼리는 우선순위를 가지는 숫자 타입을 비교를 수행하려고 실행계획 수립.
그래서 first_name 칼럼의 타입 변환이 필요하기 때문에 인덱스를 사용하지 못한다.

#### 11-4-3-3. 날짜 비교

##### 11-4-3-3-1. DATE 또는 DATETIME과 문자열 비교

DATE나 DATETIME 칼럼을 비교할때는 칼럼에 함수를 사용해 변경하지 말고, 상수를 변경하는 형태로 조건을 사용해야 인덱스를 이용할 수 있다.

```sql
mysql> SELECT COUNT(*)
         FROM employees
         WHERE hire_date>STR_TO_DATE('2011-07-23', '%Y-%m-%d'); --// 인덱스 사용

mysql> SELECT COUNT(*)
         FROM employees
         WHERE hire_date>'2011-07-23' --// 인덱스 사용

mysql> SELECT COUNT(*)
         FROM employees
         WHERE DATE_FORMAT(hire_date, '%Y-%m-%d')>'2011-07-23'; --// 인덱스 사용X
```

##### 11-4-3-3-2. DATE 또는 DATETIME과 비교

UNIX_TIMESTAMP() 함수의 결괏값을 DATE, DATETIME과 비교하려면 반드시 FROM_UNIXTIME() 함수를 이용해 변환하고 비교해야 한다.

#### 11-4-3-4. Short-Circuit Evaluation

`Short0circuit Evaluation` : 여러 개의 표현식이 AND 또는 OR 논리 연산자로 연결된 경우 선행 표현식의 결과에 따라 후행 표현식을 평가할지 말지 결정하는 최적화

```sql
mysql> SELECT * FROM salaries
         WHERE CONVERT_TZ(from_date, '+00:00','+09:00') > '1991-01-01'
            AND to_date < '1985-01-01'
=> (0.73 sec)

mysql> SELECT * FROM salaries
         WHERE to_date < '1985-01-01'
            AND CONVERT_TZ(from_date, '+00:00','+09:00') > '1991-01-01'
=> (0.52 sec)
```

MySQL 서버는 쿼리의 WHERE 절에 나열된 조건을 "Short-circuit Evaluation" 방식으로 평가해서 해당 레코드를 반환해야 할지 말지 결정한다. 이 때, WHERE 절의 조건 중에서 인덱스를 사용할 수 있는 조건이 있다면 그 조건을 가장 최우선적으로 사용한다.

EXIST (subquery) 
---

## 🖼️ 이미지

> `images/` 폴더에 캡처 이미지를 추가하고 아래에 삽입하세요.

```md
![설명](./images/파일명.png)
```

---

## 💬 느낀 점 / 질문

> 스터디에서 나눌 질문이나 인사이트를 적어주세요.
