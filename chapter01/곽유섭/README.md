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

  1. 두 옵션이 활성화되면 MySQL에서 DATE, DATETIME 타입의 칼럼에 "2020-00-00", "0000-00-00"과 같은 잘못된 날짜를 저장하는 것이 불가능해진다.

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

- `()` : 문자열 그룹을 표시. (xyz)라고 표현하면 "xyz"가 모두 있는지 확인하는 것이다.

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

1.  SYSDATE() 함수가 사용된 SQL은 레플리카 서버에서 안정적으로 복제되지 못한다.

2.  SYSDATE() 함수와 비교되는 칼럼은 인덱스를 효율적으로 사용하지 못한다.

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

| 지정문자 | 내용                          |
| -------- | ----------------------------- |
| %Y       | 4자리 연도                    |
| %m       | 2자리 숫자 표시의 월(01~12)   |
| %d       | 2자리 숫자 표시의 일자(01~31) |
| %H       | 2자리 숫자 표시의 시간(00~23) |
| %i       | 2자리 숫자 표시의 분(00~59)   |
| %s       | 2자리 숫자 표시의 초(00~59)   |

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

| 단위        | 의미        |
| ----------- | ----------- |
| YEAR        | 연          |
| MONTH       | 월          |
| DAY         | 일          |
| HOUR        | 시간        |
| MINUTE      | 분          |
| SECOND      | 초          |
| MICROSECOND | 마이크로 초 |
| QUARTER     | 분기        |
| WEEK        | 주          |

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

MySQL 서버에서 쿼리를 작성할 때 가능하면 복잡한 연산 또는 다른 테이블의 레코드를 읽어야 하는 서브쿼리 조건 등은 WHERE 절의 뒤쪽으로 배치하는 것이 성능상 도움이 된다.

### 11-4-4. DISTINCT

DISTINCT의 남용은 성능적인 문제도 있지만 쿼리의 결과가 의도한 바와 달라질 수 있으니 주의해서 사용하자.

### 11-4-5. LIMIT n

쿼리 결과에서 지정된 순서에 위치한 레코드만 가져오고자 할 때 사용.

모든 레코드의 정렬이 완료되지 않았다고 하더라도 필요한 레코드 건수만 준비되면 즉시 쿼리를 종료한다.

ORDER BY나 DISTINCT, GROUP BY처럼 전체 범위 작업이 선행되더라도 LIMIT 절이 있다면 크지는 않지만 나름의 성능 향상은 있다고 볼 수 있다.

```sql
mysql> SELECT * FROM employees LIMIT 10;
mysql> SELECT * FROM employees LIMIT 10, 10;
```

**주의**

1. LIMIT 인자로 표현식이나 별도의 서브쿼리를 사용할 수 없다.
2. 페이징을 할때 LIMIT n,m에 주어지는 수치가 매우 커질 경우, 오히려 실행에 상당히 오랜 시간이 걸릴 수 있다. 처음 몇 개의 페이지 조회로 끝나지 않을 가능성이 높다면 WHERE 조건 절로 읽어야하는 위치를 찾고 그 위치에서 몇 개만 읽는 형태의 쿼리를 사용하는 것이 좋다.

### 11-4-6. COUNT()

`COUNT()` : 결과 레코드의 건수를 반환하는 함수

`*` : 레코드 자체를 의미, 표현식의 인자로 사용할 수 있음.

MyISAM 스토리지 엔지을 사용하는 테이블은 전체 레코드 건수를 메타 정보에서 관리해서 WHERE 조건이 없는 COUNT(\*)는 바로 결과를 반환할 수 있지만 InnoDB 엔진을 사용하는 테이블에서는 그럴 수 없다.

**주의**

1. ORDER BY 구문이나 LEFT JOIN과 같은 레코드 건수를 가져오는데 무관한 작업은 제거하고 하자.
2. 인덱스를 제대로 사용하지 못하는 COUNT(\*) 쿼리는 일반 SELECT 쿼리보다 느리게 실행될 수도 있다.
3. COUNT()함수에 칼럼명이나 표현식이 인자로 사용될 경우, NULL이 아닌 레코드 건수만 반환하기에 NULL이 될 수 있는 칼럼은 의도대로 쿼리가 작동하는지 확인하는 것이 좋다.

### 11-4-7. JOIN

#### 11-4-7-1. JOIN의 순서와 인덱스

조인 작업에서 드라이빙 테이블을 읽을 때는 탐색 작업을 한 번만 수행하고, 이후로 스캔만 실행하면 된다. 하지만 드리븐 테이블에서는 드라이빙 테이블에서 읽는 레코드 건수만큼 반복해서 탐색과 스캔 작업을 실행해야 한다.

```sql
SELECT *
FROM employees e, dept_emp de
WHERE e.emp_no = de.emp_no;
```

- **두 칼럼 모두 각각 인덱스가 있는 경우** : 어느 쪽을 드라이빙으로 선택하든 인덱스를 이용해 빠르게 처리할 수 있기 때문에 옵티마이저가 선택하는 방법이 최적일 떄가 많다.

- **employees.emp_no만 인덱스가 있는 경우** : dept_emp는 풀 스캔해야하기 때문에 옵티마이저가 항상 dept_emp 테이블을 드라이빙 테이블로 선택하고, employees 테이블을 드리븐 테이블로 선택한다.

- **dept_emp.emp_no에만 인덱스가 있는 경우** : 위에 경우와 반대로 처리된다.

- **두 칼럼 모두 인덱스가 없는 경우** : 어느 테이블을 드라이빙 테이블로 선택하더라도 드리븐 테이블의 풀 스캔은 발생하기 때문에 옵티마이저가 적절한 드라이빙 테이블을 선택한다.(단 레코드 건수가 적은 테이블을 드라이빙 테이블로 선택하는 것이 훨씬 효율적이다.)

#### 11-4-7-2. JOIN 칼럼의 데이터 타입

조인 칼럼 간의 비교에서도 WHERE 절과 마찬가지로 각 컬럼의 데이터 타입이 일치하지 않으면 인덱스를 효율적으로 이용할 수 없다.

```sql
mysql> CREATE TABLE tb_test1 (user_id INT , user_type INT, PRIMARY KEY(user_id));
mysql> CREATE TABLE tb_test2 (user_type CHAR(1), type_desc VARCHAR(10), PRIMARY KEY(user_type));

mysql> SELECT *
        FROM tb_test1 tb1, tb_test2 tb2
        WHERE tb1.user_type = tb2.user_type;
```

이처럼 각 테이블의 칼럼 데이터 타입이 다르면 데이터 타입을 일치시키기 위해 변환 작업을 해야함으로 인덱스를 제대로 사용할 수 없게 된다.

**문제가 될 수 있는 비교 패턴**

- CHAR 타입과 INT 타입의 비교와 같이 데이터 타입의 종류가 완전히 다른 경우
- 같은 CHAR 타입이더라도 문자 집합이나 콜레이션이 다른 경우
- 같은 INT 타입이더라도 부호(Sign)의 존재 여부가 다른 경우

#### 11-4-7-3. OUTER JOIN의 성능과 주의사항

```sql
mysql> SELECT *
        FROM employees e
        LEFT JOIN dept_emp de ON de.emp_no = e.emp_no
        LEFT JOIN departments d ON d.dept_no = de.dept_no AND d.dept_name = 'Development';
```

위 쿼리의 실행 계획을 보면 제일 먼저 employees 테이블을 풀 스캔하면서 dept_emp테이블과 departments 테이블을 드리븐 테이블로 사용한 것을 알 수 있다.

아우터 조인은 드리븐 테이블의 데이터가 일관되지 않은 경우에만 필요한 경우이다.

옵티마이저는 절대 아우터로 조인되는 테이블을 드라이빙 테이블로 선택하지 못하기 때문에 시용에 주의가 필요하다.

또 주의해야할 사항은 아우터로 조인되는 테이블에 대한 조건을 WHERE 절에 함께 명시하는 것이다.

이 경우 옵티마이저가 LEFT JOIN을 INNER JOIN으로 변환해서 실행시켜버리기 때문이다.

예외적으로 OUTER JOIN으로 연결되는 테이블의 칼럼에 대한 조건을 WHERE 절에 사용해야 하는 경우가 있는데 이는 안티 조인(ANTI-JOIN) 효과를 기대하는 경우다.

```sql
mysql> SELECT *
        FROM employees e
        LEFT JOIN dept_manager dm ON dm.emp_no = e.emp_no
        WHERE dm.emp_no IS NULL
        LIMIT 10;
```

#### 11-4-7-4. JOIN과 외래키(FOREIGN KEY)

외래키를 생성하는 주목적 : 데이터의 무결성을 보장하기 위해서

그러나 데이터 모델을 데이터베이스에 생성할 때 그 테이블 간의 관계는 외래키로 생성하지 않을 때가 더 많다.

#### 11-4-7-5. 지연된 조인(Delayed Join)

**지연된 조인** : 조인이 실행되기 이전에 GROUP BY나 ORDER BY를 처리하는 방식을 의미

```sql
--// 원본 쿼리
mysql> SELECT e.*
        FROM salaries s, employees e
        WHERE e.emp_no = s.emp_no
        AND s.emp_no BETWEEN 10001 AND 13000
        GROUP BY s.emp_no
        ORDER BY SUM(s.salary) DESC
        LIMIT 10;

--// 지연된 조인 쿼리
mysql> SELECT e.*
        FROM
        (
          SELECT s.emp_no
          FROM salaries s
          WHERE s.emp_no BETWEEN 10001 AND 13000
          GROUP BY s.emp_no
          ORDER BY SUM(s.salary) DESC
          LIMIT 10
        ) x,
        employees e
        WHERE e.emp_no = x.emp_no;
```

지연된 조인으로 개선된 쿼리는 파생 테이블을 사용하기 때문에 느리다고 예상할 수 있으나 실제로 임시 테이블에 저장할 레코드를 줄여 메모리를 이용해 빠르게 처리할 수 있다.

지연된 조인은 성능 향상을 가져올 수 있지만 모든 쿼리를 개선할 수 있는 것은 아니다.

**지연된 쿼리 변경 조건**

- LEFT (OUTER) JOIN인 경우 드라이빙 테이블과 드리븐 테이블은 1:1 또는 M:1 관계여야 한다.
- INNER JOIN인 경우 드라이빙 테이블과 드리븐 테이블은 1:1 또는 M:1 관계임과 동시에 드라이빙 테이블에 있는 레코드는 모두 드리븐 테이블에 존재해야 한다.

#### 11-4-7-6. 래터럴 조인(Lateral Join)

```sql
mysql> SELECT *
        FROM employees e
        LEFT JOIN LATERAL (SELECT *
                            FROM salaries s
                            WHERE s.emp_no = e.emp_no
                            ORDER BY s.from_date DESC LIMIT 2) s2 ON s2.emp_no = e.emp_no
        WHERE e.first_name = 'Matt';
```

래터럴 조인에서 가장 중요한 부분은 FROM 절에 사용된 서브쿼리에서 외부 쿼리의 FROM 절에 정의된 테이블의 칼럼을 참조할 수 있다는 것이다.

LATERAL 키워드를 가진 서브쿼리는 조인 순서상 후순위로 밀리고, 외부 쿼리의 결과 레코드 단위로 임시 테이블을 생성되기 때문에 꼭 필요한 경우에만 사용해야 한다.

#### 11-4-7-7. 실행 계획으로 인한 정렬 흐트러짐

8.0버전 이전에는 네스티드 루프 조인을 통해 드라이빙 테이블을 읽은 순서로 정렬된다고 생각하는 경우가 많았으며, 8.0버전부터 네스티드 루프 조인 대신 해시 조인이 사용되면서 레코드의 정렬 순서가 달라졌다.

이 정렬 결과는 옵티마이저에 의해 그때그때 상황에 따라 달라질 수 있으니 정렬된 결과가 필요하다면 테이블의 순서에 의존하지 말고 ORDER BY 절을 명시적으로 사용하는 것이 좋다.

### 11-4-8. GROUP BY

특정 칼럼의 값으로 레코드를 그루핑하고, 그룹별로 집계된 결과를 하나의 레코드로 조회할 떄 사용

#### 11-4-8-1. WITH GROUP

`ROLLUP` :

1. GROUP BY가 사용된 쿼리에서 그루핑된 그룹별로 소계를 가져오는 기능
2. 단순히 최종 합만 가져오는 것이 아니라 GROUP BY에 사용된 칼럼의 개수에 따라 소계의 레벨이 달리짐.

```sql
mysql> SELECT dept_no, COUNT(*)
        FROM dept_emp
        GROUP BY dept_no WITH ROLLUP;
```

WITH ROLLUP과 함께 사용된 GROUP BY 쿼리의 결과는 그룹별로 소계를 출력하는 레코드가 추가되어 표시된다.
이때 소계 레코드이 칼럼값은 항상 NULL로 표시된다.

이는 GROUP BY 절에 칼럼 개수에 따라 소계가 단계로 표시된다.

**GROUPING() 함수 지원**

```sql
mysql> SELECT
        IF(GROUPING(first_name), 'All first_name', first_name) AS first_name,
        IF(GROUPING(last_name), 'ALll last_name', last_name) AS last_name,
        COUNT(*)
        FROM employees
        GROUP BY first_name, last_name WITH ROLLUP;
```

GROUPING() 함수를 통해 결과에서 더이상 NULL 대신 명시한 문자열로 표시할 수 있다.

#### 11-4-8-2. 레코드를 칼럼으로 변환해서 조회

##### 11-4-8-2-1. 레코드를 칼럼으로 변환

SUM(CASE WHEN ...) 구문을 이용해서 레코드를 칼럼으로 변환할 수 있다.

```sql
SELECT
  SUM(CASE WHEN dept_no='d001' THEN emp_count ELSE 0 END) AS count_d001,
  SUM(CASE WHEN dept_no='d002' THEN emp_count ELSE 0 END) AS count_d002,
  SUM(CASE WHEN dept_no='d003' THEN emp_count ELSE 0 END) AS count_d003,
  SUM(CASE WHEN dept_no='d004' THEN emp_count ELSE 0 END) AS count_d004,
  SUM(CASE WHEN dept_no='d005' THEN emp_count ELSE 0 END) AS count_d005,
  SUM(CASE WHEN dept_no='d006' THEN emp_count ELSE 0 END) AS count_d006,
  SUM(CASE WHEN dept_no='d007' THEN emp_count ELSE 0 END) AS count_d007,
  SUM(CASE WHEN dept_no='d008' THEN emp_count ELSE 0 END) AS count_d008,
  SUM(CASE WHEN dept_no='d009' THEN emp_count ELSE 0 END) AS count_d009,
  SUM(emp_count) AS count_total
FROM (
  SELECT dept_no, COUNT(*) AS emp_count FROM dept_emp GROUP BY dept_no
) tb_derived;
```

이런 예지의 단점은 부서 번호가 쿼리의 일부로 사용되기 떄문에 부서 번호가 변경되거나 추가되면 쿼리까지도 변경돼야 한다는 것이다.

##### 11-4-8-2-2. 하나의 칼럼을 여러 칼럼으로 분리

SUM(CASE WHEN ...) 문장은 소그룹을 특정 조건으로 나눠서 사원의 수를 구하는 용도로도 사용할 수 있다.

```sql
SELECT de.dept_no,
      SUM(CASE WHEN e.hire_date BETWEEN '1980-01-01' AND '1989-12-31' THEN 1 ELSE 0 END) AS cnt_1980,
      SUM(CASE WHEN e.hire_date BETWEEN '1990-01-01' AND '1999-12-31' THEN 1 ELSE 0 END) AS cnt_1990,
      SUM(CASE WHEN e.hire_date BETWEEN '2000-01-01' AND '2009-12-31' THEN 1 ELSE 0 END) AS cnt_2000,
      COUNT(*) AS cnt_total
FROM dept_emp de, employees e
WHERE e.emp_no = de.emp_no
GROUP BY de.dept_no;
```

### 11-4-9. ORDER BY

**ORDER BY를 사용하지 않을 경우 순서 정렬**

- 인덱스를 사용한 SELECT의 겨웅에는 이덱스에 정렬된 순서대로 레코드를 가져온다.
- 풀 테이블 스캔을 실행하는 SELECT의 경우, MyISAM은 테이블에 저장된 순서대로 가져오고, InnoDB는 항상 프라이머리 키로 클러스터링돼 있기 때문에 프라이머리 키 순서대로 레코드를 가져온다.
- SELECT 쿼리가 임시 테이블을 거쳐 처리되면 조회되는 레코드의 순서를 예측하기 어렵다.

실행계획을 통해 정렬이 어떻게 수행됐는지 알 수 없지만 MySQL서버의 상태값을 통해 확인은 할 수 있다.

```sql
mysql> SHOW STATUS LIKE 'Sort_%';
```

`Sort_merge-passes` :

1. 메모리의 버퍼와 디스크에 저장된 레코드를 몇 번이나 병합했는지 보여줌.
2. 이 상태 값이 0보다 크다면 데이터가 정렬용 버퍼보다 커서 디스크를 이용했다는 것을 의미.

`Sort_range` : 인덱스 레이진 스캔을 통해 읽은 레코드를 정렬한 횟수

`Sort_scan` : 풀 테이블 스캔을 통해서 읽은 레코드를 정렬한 횟수

`Sort_rows` : 정렬을 수행했던 전체 레코드 건수

#### 11-4-9-1. ORDER BY 사용법 및 주의사항

ORDER BY 절은 1개 또는 그 이상 여러 개의 칼럼으로 정렬을 수행할 수 있으며, 정렬 순서(오름차순, 내림차순)는 칼럼별로 다르게 명시할 수 있다.

일반적으로 정렬할 대상은 칼럼명이나 표현식으로 명시하지만 SELECT되는 칼럼의 순법을 명시할 수도 있다.

```sql
--// 같은 결과를 반환한다.
mysql> SELECT first_name, last_name FROM employees
        ORDER BY last_name;

mysql> SELECT first_name, last_name FROM employees
        ORDER BY 2;
```

단 숫자 값이 아닌 문자열 상수를 사용하는 경우네는 옵티마이저가 ORDER BY 절 자체를 무시한다.
(MySQL에서 쌍따옴표는 문자열 리터럴을 표현하는 데 사용되기 때문)

#### 11-4-9-2. 여러 방향으로 동시 정렬

8.0버전부터 오름차순과 내림차순을 혼용해서 인덱스를 생성할 수 있게 개선되서 정렬 순서를 혼용해서 인덱스를 이용할 수 있게 됐다.

```sql
mysql> ALTER TABLE salaries ADD INDEX ix_salary_fromdate (salary DESC, from _date ASC);
```

#### 11-4-9-3. 함수나 표현식을 이용한 정렬

함수 기반의 인덱스를 통해 결괏값을 정렬할 수 있다.

```sql
mysql> SELECT *
        FROM salaries
        ORDER BY COS(salary);
```

### 11-4-10. 서브쿼리

#### 11-4-10-1. SELECT 절에 사용된 서브쿼리

일반적으로 SELECT 절에 서브쿼리를 사용하면 그 서브쿼리는 항상 칼럼과 레코드가 하나인 결과를 반환해야 한다. 그 값이 NULL이든 아니든 관계없이 레코드가 1건이 존재해야한다.

```sql
mysql> SELECT emp_no, (SELECT dept_name FROM departments WHERE dept_name='Sales1')
        FROM dept_emp LIMIT 10;

mysql> SELECT emp_no, (SELECT dept_name FROM departments)
        FROM dept_emp LIMIT 10;

mysql> SELECT emp_no, (SELECT dept_no, dept_name FROM departments WHERE dept_name = 'Sales1')
        FROM dept_emp LIMIT 10;
```

**주의**

- 첫 번째 쿼리에서 사용된 서브쿼리는 항상 결과가 0건. 하지만 에러를 발생하지 않고, 서브쿼리의 결과는 NULL로 채워져서 반환된다.

- 두 번째 쿼리에서 서브쿼리가 2건 이상의 레코드를 반환하는 경우에는 에러가 나면서 쿼리가 종료된다.

- 세 번째 쿼리와 같이 SELECT 절에 사용된 서브쿼리가 2개 이상의 칼럼을 가져오려고 할 때도 에러가 발생한다.

`스칼라 서브쿼리` : 레코드의 칼럼이 각각 하나인 결과를 만들어내는 서브쿼리

`로우 서브쿼리` : 스칼라 서브쿼리보다 레코드 건수가 많거나 칼럼 수가 많은 결과를 만들어 내는 서브쿼리

```sql
--// 둘다 같은 결과를 도출하지만 조인으로 하는 편이 더 빠르다.
mysql> SELECT
        COUNT (CONCAT(e1.first_name,
                    (SELECT e2.first_name FROM employees e2 WHERE e2.emp_no = e1.emp_no))
                ) FROM employees e1;

mysql> SELECT COUNT(CONCAT(e1.first_name, e2.first_name))
        FROM employees e1, employees e2
        WHERE e1.emp_no = e2.emp_no;
```

```sql
--// 래터럴 조인을 통해 서브쿼리를 3번 남용하지 않아도 된다. 단 버그가 있어서 사용은 지양한다.
mysql> SELECT e.emp_no, e.first_name,
            (SELECT s.salary FROM salaries s
              WHERE s.emp_no = e.emp_no
              ORDER BY s.from_date DESC LIMIT 1) AS salary,
            (SELECT s.from_date FROM salaries s
              WHERE s.emp_no = e.emp_no
              ORDER BY s.from_date DESC LIMIT 1) AS salary_from_date,
            (SELECT s.to_date FROM salaries s
              WHERE s.emp_no = e.emp_no
              ORDER BY s.from_date DESC LIMIT 1) AS salary_to_date
        FROM employees e
        WHERE e.emp_no = 499999;

mysql> SELECT e.emp_no, e.first_name,
              s2.salary, s2.from_date, s2.to_date
        FROM employees e
        INNER JOIN LATERAL (
          SELECT * FROM salaries s
          WHERE s.emp_no = e.emp_no
          ORDER BY s.from_date DESC
          LIMIT 1) s2 ON s2.emp_no = e.emp_no
        WHERE e.emp_no = 499999;
```

#### 11-4-10-2. FROM 절에 사용된 서브쿼리

이전 버전에서는 FROM 절에 서브쿼리가 사용되면 서브쿼리의 결과를 임시 테이블로 저장하고 필요할 때 다시 임시 테이블을 읽는 방식으로 처리했다. 5.7버전부터는 옵티마이저가 FROM 절의 서브쿼리를 외부 쿼리로 병합하는 최적화를 수행하도록 개선됐다.

**외부 쿼리로 병합되지 못하는 FROM 절 서브쿼리**

- 집합 함수 사용(SUM(), MIN(), MAX(), COUNT() 등)
- DISTINCT
- GROUP BY 또는 HAVING
- LIMIT
- UNION(UNION DISTINCT) 또는 UNION ALL
- SELECT 절에 서브쿼리가 사용된 경우
- 사용자 변수 사용(사용자 변수에 값이 할당되는 경우)

외부 쿼리에서 GROUP BY나 DISTINCT와 같은 기능이 사용되고 있다면, 서브쿼리의 정렬 작업은 무의미하기 때문에 서브쿼리의 ORDER BY 절은 무시된다.

#### 11-4-10-3. WHERE 절에 사용된 서브쿼리

##### 11-4-10-3-1. 동등 또는 크다 작다 비교

5.5 이전 버전까지는 서브쿼리 외부의 조건으로 쿼리를 실행하고, 최종적으로 서브쿼리를 체크 조건으로 사용했다. 이 경우 풀 테이블 스캔이 필요한 경우가 많아서 성능 저하가 심각했다.

5.5 버전부터는 서브 쿼리를 먼저 실행한 후 상수로 변환한다. 그리고 상숫값으로 서브쿼리를 대체해서 나머지 쿼리 부분을 처리한다.

```sql
mysql> EXPLAIN FORMAT=TREE
        SELECT * FROM dept_emp de
        WHERE de.emp_no = (SELECT e.emp_no
                            FROM employees e
                            WHERE e.first_name = 'Georgi' AND e.last_name = 'Facello' LIMIT 1);
```

단일 값 비교가 아닌 튜플 비교 방식이 사용되면 서브쿼리가 먼저 처리되어 상수화되긴 하지만 외부 쿼리는 인덱스를 사용하지 못하고 풀 테이블 스캔을 실행하는 것을 확인할 수 있다. 그래서 튜플 형태의 비교는 주의해서 사용해야 한다.

```sql
mysql> EXPLAIN
        SELECT *
        FROM dept_emp de WHERE (emp_no, from_date) = (
          SELECT emp_no, from_date
          FROM salaries
          WHERE emp_no = 100001 limit 1);
```

##### 11-4-10-3-2. IN 비교( IN (subquery) )

```sql
mysql> SELECT *
        FROM employees e
        WHERE e.emp_no IN
          (SELECT de.emp_no FROM dept_emp de WHERE de.from_date = '1995-01-01');
```

5.5버전까지는 세미 조인의 최적화가 매우 부족해서 대부분 풀 테이블 스캔을 했다.

그러나 8.0 버전까지 세미 조인 최적화가 많이 개선되면서 이제는 쿼리 특성이나 조인 관계에 맞춰 5개의 최적화 전략을 선택적으로 사용한다.

**세미 조인 최적화 5가지**

- 테이블 폴-아웃(Table Pull-out)
- 퍼스트 매치(Firstmatch)
- 루스 스캔(Loosescan)
- 구체화(Materialization)
- 중복 제거(Duplicated Weed-out)

##### 11-4-10-3-3. NOT IN 비교( NOT IN (subquery) )

안티 세미 조인이라고도 불리며 2가지 방법으로 최적화를 수행한다.

- NOT EXISTS
- 구체화(Materialization)

두 방법 모두 성능 향상에 크게 도움이 되지는 않으니 최대한 다른 조건을 활용해서 데이터 검색 범위를 좁히는게 좋다.

### 11-4-11. CTE(Common Table Expression)

- 임시 테이블로서 SQL 문장 내에서 한 번 이상 사용될 수 있으며 SQL 문장이 종료되면 자동으로 CTE 임시 테이블은 삭제된다.

- 재귀적 반복 실행 여부를 기준으로 Non-recursive와 Recursive CTE로 구분됨.

**CTE 사용 위치**

- SELECT, UPDATE, DELETE 문장의 제일 앞쪽

- 서브쿼리의 제일 앞쪽

- SELECT 절의 바로 앞쪽

#### 11-4-11-1. 비 재귀적 CTE(Non-Recursive CTE)

MySQL 서버에서는 ANSI 표준을 그대로 이용해서 WITH 절을 이용해 CTE를 정의한다.

```sql
mysql> WITH cte1 AS (SELECT * FROM departments)
        SELECT * FROM cte1;

mysql> SELECT *
        FROM (SELECT * FROM departments) cte1;
```

**CTE의 장점**

- CTE 임시 테이블은 재사용 가능하므로 FROM 절의 서브쿼리보다 효율적이다.
- CTE로 선언된 임시 테이블을 다른 CTE 쿼리에서 참조할 수 있다.
- CTE는 임시 테이블의 생성 부분과 사용 부분의 코드를 분리할 수 있으므로 가독성이 높다.

#### 11-4-11-2. 재귀적 CTE(Recursive CTE)

```sql
mysql> WITH RECURSIVE cte (no) AS (
  SELECT 1
  UNION ALL
  SELECT (no + 1) FROM cte WHERE no < 5
)
SELECT * FROM cte;
```

재귀적 CTE 쿼리는 비 재귀적 쿼리 파트와 재귀적 파트로 구분되며, 이 둘은 UNION 또는 UNION ALL로 연결하는 형태로 반드시 쿼리를 작성해야 한다.

**작동 방법**

1. CTE 쿼리의 비 재귀적 파트의 쿼리를 실행
2. 1번 결과를 이용해 cte라는 임시 테이블 생성
3. 1번 결과를 cte라는 임시 테이블에 저장
4. 1번 결과를 입력으로 사용해 CTE 쿼리의 재귀적 파트의 쿼리를 실행
5. 4번 결과를 cte라는 임시 테이블에 저장(이때 UNION(UNION DISTINCT)라면 중복 제거 실행)
6. 전 단계으 결과를 입력으로 사용해 CTE 쿼리의 재귀적 파트 쿼리 실행
7. 6번 단계에서 쿼리 결과가 없으면 CTE 쿼리르 종료
8. 6번의 결과를 cte라는 임시 테이블에 저장
9. 6번으로 돌아가서 반복

**중요점**

1. CTE 쿼리의 비 재귀적 쿼리 파트의 결과가 CTE 임시 테이블의 구조를 결정한다.
2. 재귀적 퀄리 파트를 실행할 때는 지금까지의 모든 단게에서 만들어진 결과 셋이 아니라 직전 단계의 결과만 재귀 쿼리의 입력으로 사용된다.
3. 실제 재귀 쿼리가 반복을 멈추는 조건은 재귀 파트 쿼리의 결과가 0건일 때까지다.
4. 데이터의 오류나 쿼리 작성자의 실수로 종료 조건을 만족못해 무한 반복할 경우가 발생할 수 있는데, 이를 방지하기 위해 `cte_max_recursion_depth` 시스템 변수(기본값 : 1000)를 통해 최대 반복 실행 횟수를 제한할 수 있다.

### 11-4-12. 윈도우 함수(Window Function)

조회하는 현재 레코드를 기준으로 연관된 레코드 집합의 연산을 수행

**윈도우 함수와 집계 함수의 차이점** : 집계 함수는 주어진 그룹 별로 하나의 레코드로 묶어서 출력하지만 윈도우 함수는 조건에 일치하는 레코드 건수는 변하지 않고 그대로 유지한다.

#### 11-4-12-1. 쿼리 각 절의 실행 순서

윈도우 함수 이전 실행 : WHERE절, FROM절, GROUP BY절, HAVING절

-> 윈도우 함수 처리

윈도우 함수 이후 실행 : SELECT절, ORDER BY절, LIMIT절

```sql
--//윈도우 이전 실행 절들은 윈도우 함수에 적용할 수 없으니 FROM 절의 서브쿼리에 사용해야한다.
mysql> SELECT emp_no, from_date, salary,
              AVG(salary) OVER() AS avg_salary
        FROM (SELECT * FROM salaries WHERE emp_no=10001 LIMIT 5) s2;
```

#### 11-4-12-2. 윈도우 함수 기본 사용법

```sql
AGGREGATE_FUNC() OVER(<partition> <order>) AS window_func_column
```

OVER절을 이용해 연산 대상을 파티션하기 위한 옵션을 명시할 수 있다.

`프레임` :

- 윈도우 함수의 각 파티션 안에서도 연산 대상 레코드별로 연산을 수행할 소그룹
- 레코드의 순서대로 현재 레코드 기준 앞뒤 몇 건을 연산 범위로 제한하는 역할

```sql
AGGREGATE_FUNC() OVER(<partiion> <order> <frame>) AS window_func_column

frame:
  {ROWS | RANGE} {frame_start | frame_between}

frame_between:
  BETWEEN frame_start AND frame_end

frame_start, frame_end: {
    CURRENT ROWS
    | UNBOUNDED PRECEDING
    | UNBOUNDED FOLLOWING
    | expr PRECEDING
    | expr FOLLOWING
}
```

**프레임을 만드는 기준**

- `ROWS` : 레코드의 위치를 기준으로 프레임을 생성
- `RANGE` : ORDER BY 절에 명시된 칼럼을 기준으로 값의 범위로 프레임 생성

**프레임 시작과 끝 키워드 의미**

- `CURRENT ROW` : 현재 레코드
- `UNBOUNDED PRECEDING` : 파티션의 첫 번쨰 레코드
- `UNBOUNDED FOLLOWING` : 파티션의 마지막 레코드
- `expr PRECEDING` : 현재 레코드로부터 n번째 이전 레코드
- `expr FOLLOWING` : 현재 레코드로부터 n번째 이후 레코드

```sql
SELECT emp_no, from_date, salary,

    --// 현재 레코드의 from_date를 기준으로 1년 전부터 지금까지 급여 중 최소 급여
    MIN(salary) OVER(ORDER BY from_date RANGE INTERVAL 1 YEAR PRECEDING) AS min_1,

    --// 현재 레코드의 from_date를 기준으로 1년 전부터 2년 후까지의 급여 중 최대 급여
    MAX(salary) OVER(ORDER BY from_date RANGE BETWEEN INTERVAL 1 YEAR PRECEDING AND INTERVAL 2 YEAR FOLLOWING) AS max_1,

    --// from_date 칼럼으로 정렬 후, 첫 번쨰 레코드부터 현재 레코드까지의 평균
    AVG(salary) OVER(ORDER BY from_date ROWS UNBOUNDED PRECEDING) AS avg_1,

    --// from_date 칼럼으로 정렬 후, 현재 레코드를 기준으로 이전 건부터 이후 레코드까지의 급여 평균
    AVG(salary) OVER(ORDER BY from_date ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS avg_2

FROM salaries
WHERE emp_no = 100001;
```

> 주의 : 윈도우 함수에서 프레임이 별도로 명시되지 않으면 무조건 파티션의 모든 레코드가 연산의 대상이 되는 것은 아니다. ORDER BY 여부에 따라 프레임이 다음과 같이 묵시적으로 선택된다.
>
> - ORDER BY 사용 시 : RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
> - ORDER BY 미 사용 시 : RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING

**자동으로 프레임이 파티션의 전체 레코드로 설정되는 윈도우 함수**

- CUME_DIST()
- DENSE_RANK()
- LAG()
- LEAD()
- NTILE()
- PERCENT_RANK()
- RANK()
- ROW_NUMBER()

#### 11-4-12-3. 윈도우 함수

**집계 함수**
| 함수 | 설명 |
| -- | -- |
| AVG() | 평균 값 반환 |
| BIT*AND() | AND 비트 연산 결과 반환 |
| BIT_OR() | OR 비트 연산 결과반환 |
| COUNT() | 건수 반환 |
| JSON_ARRAYAGG() | 결과를 JSON 배열로 반환 |
| JSON_OBJECTAGG() | 결과를 JSON OBJECT 배열로 반환 |
| MAX() | 최댓값 반환 |
| MIN() | 최솟값 반환 |
| STDDEV_POP(), STDDEV(), STD() | 표준 편차 값 반환 |
| STDDEV* SAMP() | 표본 표준 편차 값 반환 |
| SUM() | 합계 값 반환 |
| VAR_POP(), VARIANCE() | 표준 분산 값 반환 |
| vAR_SAMP() | 표준 분산 값 반환 |

**비 집계 함수**
| 함수 | 설명 |
| -- | -- |
| CUME_DIST() | 누적 분포 값 반환 |
| DENSE_RANK() | 랭킹 값 반환(Gap 없음) |
| FIRST_VALUE() | 파티션의 첫 번쨰 레코드 값 반환 |
| LAG() | 파티션 내에서 파라미터(N)를 이용해 N번째 이전 레코드 값 반환 |
| LAST_VALUE() | 파티션의 마지막 레코드 값 반환 |
| LEAD() | 파티션 내에서 파라미터(N)를 이용해 N번째 이후 레코드 값 반환 |
| NTH_VALUE() | 파티션의 n번째 값 반환 |
| NTILE() | 파티션별 전체 건수를 파라미터(N)로 N-등분한 값 반환 |
| PERCENT_RANK() | 퍼센트 랭킹 값 반환 |
| RANK() | 랭킹 값 반환(Gap있음) |
| ROW_NUMBER() | 파티션의 레코드 순번 반환 |

#### 11-4-12-4. 윈도우 함수와 성능

윈도우 함수를 사용한 쿼리는 인덱스를 충분히 활용하기 어렵기 때문에 성능 상 어쩔 수 없이 느리다.

때문에 가능한 윈도우 함수에 너무 의존하지 않는 편이 좋다.

### 11-4-13. 잠금을 사용하는 SELECT

**잠금 옵션**

- `FOR SHARE` : SELECT 쿼리로 읽은 레코드에 대해서 읽기 잠금
- `FOR UPDATE` : SELECT 쿼리가 읽은 레코드에 대해서 쓰기 잠금

> 참고 : 8.0 이전 버전에는 읽기 잠을 위해 LOCK IN SHARE MODE절을 사용했다.

**전제조건** : 두 가지 잠금 옵션 모두 자동 커밋이 비활성화이거나 BEGING 명이나 START TRANSACTION 명령으로 트랜잭션이 시작된 상태에서만 잠금이 유지된다.

- FOR SHARE 절은 SELECT된 레코드에 대해서 읽기 잠금(공유 잠금, Shared lock)을 설정하고 다른 세션에서 해당 레코드를 변경하지 못하게 한다. 다른 세션에서 잠금이 걸린 레코드를 읽는 것은 가능하다.

- FOR UDPATE 절은 쓰기 잠금(배타 잠금, Exclusive lock)을 설정하고, 다른 트랜잭션에서는 그 레코드를 변경하는 것뿐만 아니라 읽기(FOR SHARE 절을 사용하는 SELECT 쿼리)도 수행할 수 없다.

#### 11-4-13-1. 잠금 테이블 선택

```sql
--// 전체 잠금
mysql> SELECT *
        FROM employees e
        INNER JOIN dept_emp de ON de.emp_no = e.emp_no
        INNER JOIN departments d ON d.dept_no = de.dept_no
        FOR UPDATE;

--// OF를 통한 선택 잠금
mysql> SELECT *
        FROM employees e
        INNER JOIN dept_emp de ON de.emp_no = e.emp_no
        INNER JOIN departments d ON d.dept_no = de.dept_no
        WHERE e.emp_no = 100001
        FOR UPDATE OF e;
```

#### 11-4-13-2. NOWAIT \* SKIP LOCKED

`NOWAIT` : SELECT 쿼리에서 해당 레코드가 다른 트랜잭션에 의해서 잠겨진 상태라면 즉시 에러를 반환하고 쿼리를 종료하는 옵션

`SKIP LOCKED` : SELECT하려는 레코드가 다른 트랜잭션에 의해 이미 잠겨지 상태라면 에러를 반환하지 않고 잠긴 레코드를 무시하고 잠금이 걸리지 않은 레코드만 가져오는 옵션

둘 다 FOR UPDATE에서만 쓰인다.

`FOR UPDATE SKIP LOCKED` 절을 사용하면 트랜잭션이 수행되는 데 걸리는 시간과 관계없이 다른 트랜잭션에 의해서 이미 사용 중인 레코드를 스킵하는 시간만 지나면 각자의 트랜잭션을 실행할 수 있다.
그래서 실제 레코드를 스캔하는데 걸리는 시간이 매우 짧아 다량의 트랜잭션을 동시에 처리하게 되는 효과를 얻을 수도 있다.

> 주의 : NOWAIT나 SKIP LOCKED는 SELECT ... FOR UPDATE 구문에서만 사용할 수 있음.

## 11-5. INSERT

### 11-5-1. 고급 옵션

#### 11-5-1-1. INSERT IGNORE

저장하는 레코드의 프라이머리 키나 유니크 인덱스 칼럼의 값이 이미 테이블에 존재하는 레코드와 중복되는 경우, 그리고 저장하는 레코드의 칼럼이 테이블의 칼럼과 호환되지 않는 경우 모두 무시하고 다음 레코드를 처리할 수 있게 하는 옵션

```sql
INSERT IGNORE INTO salaries
  SELECT emp_no, (salary+100), '2020-01-01', '2022-01-01'
  FROM salaries WHERE to_date >= '2020-01-01';
```

INSERT IGNORE 옵션은 단순히 유니크 인덱스의 중복뿐만 아니라 데이터 타입이 일치하지 않아서 INSERT 할 수 없는 경우에도 칼럼의 기본 값으로 INSERT를 하도록 만들기도 한다.

```sql
--// IGNORE 키워드가 있으면, NOT NULL 칼럼인 emp_no와 from_date에 각 타입별 기본 값을 저장
mysql> INSERT IGNORE INTO salaries VALUES (NULL, NULL, NULL, NULL);
```

#### 11-5-1-2. INSERT ... ON DUPLICATE KEY UPDATE

프라이머리 키나 유니크 인덱스의 중복이 발생하면 UPDATE 문장의 역할을 수행하게 해준다.

```sql
--// VALUES() 함수를 활용
mysql> INSERT INTO daily_statistic
        SELECT DATE(visited_at), 'VISIT', COUNT(*)
        FROM access_log
        GROUP BY DATE(visited_at)
        ON DUPLICATE KEY UPDATE stat_value=stat_value + VALUES(stat_value);

--// 8.0.20이후 VALUES() 대체 문법
mysql> INSERT INTO daily_statistic
        SELECT target_date, stat_name, stat_value
        FROM (
          SELECT DATE(visited_at) target_date, 'VISIT' stat_name, COUNT(*) stat_value
          FROM access_log
          GROUP BY DATE(visited_at)
        ) stat
        ON DUPLICATE KEY UPDATE
          daily_statistic.stat_value = daily_statistic.stat_value + stat.stat_value;
```

INSERT ... SELECT ... 헝태의 문법이 아닌 경우에는 INSERT되는 레코드에 대해 별칭을 부여해서 참조하는 방식으로 VALUES() 사용을 피할 수 있다.

```sql
mysql> INSERT INTO daily_statistic (target_date, stat_name, stat_value)
        VALUES ('2020-09-01', 'VISIT', 1),
                ('2020-09-02', 'VISIT', 1)
                AS new /* new라는 이름으로 별칭을 부여 */
        ON DUPLICATE KEY
          UPDATE daily_statistic.stat_value = daily_statistic.stat_value + new.stat_value;

mysql> INSERT INTO daily_statistic
        /* new라는 이름으로 별칭을 부여 */
        SET target_date='2020-09-01', stat_name = 'VISIT', stat_value = 1 AS new
        ON DUPLICATE KEY
          UPDATE daily_statistic.stat_value = daily_statistic.stat_value + new.stat_value;

mysql> INSERT INTO daily_statistic
        /* new라는 이름으로 별칭을 부여하면서 칼럶의 별칭까지 부여 */
        SET target_date  ='2020-09-01', stat_name = 'VISIT', stat_value = 1 AS new(fd1, fd2, fd3)
        ON DUPLICATE KEY
          UPDATE daily_statistic.stat_value = daily_statistic.stat_value + new.fd3;
```

### 11-5-2. LOAD DATA 명령 주의 사항

**LOAD DATA 장점**

- 내부적으로 MySQL 엔진과 스토리지 엔진의 호출 횟수를 최소화
- 스토리지 엔진이 직접 데이터를 적재하기 때문에 일반적인 INSERT 명령보다 매우 빠르다.

**LOAD DATA 단점**

- 단일 스레드로 실행
- 단일 트랜잭션으로 실행

**사용법** : 가능한 LOAD DATA 문장으로 적재할 데이터 파일을 하나보다는 여러 개로 준비해서 동시에 여러 트랜잭션으로 나눠서 실행되게 하는 것이 좋다.

### 11-5-3. 성능을 위한 테이블 구조

#### 11-5-3-1. 대량 INSERT 성능

**대량 INSERT할 때 신경써야할 것**

- `버퍼 풀의 크기` : 스토리지 버퍼 풀이 커서 테이블의 인덱스가 모두 메모리에 적재돼 있다면 차이가 줄어든다.
- `프라이머리 키 정렬 유무` : 프라이머리 키로 정렬된 데이터를 적재할 때는 항상 메모리에 프라이머리 키의 마지막 페이지만 적재돼 있으면 따로 저장할 위치를 찾지 않아도 된다.
- `세컨더리 인덱스` : 세컨더리 인덱스가 너무 남용되어있다면 백그라운드 작업의 부하를 유발하므로 성능을 떨어드릴 수 있다.

#### 11-5-3-2. 프라이머리 키 선정

온라인 트랜잭션 처리를 위한 테이블들은 읽기 쿼리의 비율이 압도적으로 높기 때문에 이런 류에 테이블은 SELECT 쿼리를 빠르게 만드는 방향으로 프라이머리 키를 선정해야하며, 반대로 INSERT가 많은 테이블에서는 인덱스의 개수를 최소화하는 것이 좋다.

#### 11-5-3-3. Auto_Increment 칼럼

INSERT에 최적화된 테이블을 생성하기 위해서는 2가지 요소를 갖춰 테이블을 준비해야한다.

- 단조 증가 또는 단조 감소되는 값으로 프라이머리 키 선정
- 세컨더리 인덱스 최소화

자동 증가 값을 프라이머리 키로 해서 테이블을 생성하는 것은 MySQL에서 가장 빠른 INSERT를 보장하는 방법.

**AUTO-INC 잠금 변경 방법(innodb_autoinc_lock_mode)**

- `innodb_autoinc_lock_mode = 0` : 항상 AUTO_INC 잠금을 걸고 한 번에 1씩만 증가된 값을 가져온다.
- `innodb_autoinc_lock_mode = 1 (Consecutive mode)` : 단순히 레코드 한 건식 INSERT하는 쿼리에선 뮤텍스를 이용해 더 빠르게 처리한다. 하지만 여러 레코드를 INSERT하거나 LOAD DATA 하는 쿼리는 AUTO_INC잠금을 걸고 필요한 만큼 값을 가져와 사용한다.
- `innodb_autoinc_lock_mode = 2 (Interleaved mode)` : 자동 증가 값을 적당히 미리 할당받아서 처리하는 가장 빠른 방식이다. 채번한 번호는 유니크한 번호까지만 보장하며, 연속성은 보장하지 않는다.
  그래서 쿼리 기반의 복제(SBR, State Based Replication)를 사용하는 MySQL에서는 소스 서버와 레플리카 서버의 자동 증가 값이 동기화되지 못할 수도 있으니 주의해야 한다.

## 11-6. UPDATE와 DELETE

### 11-6-1. UPDATE ... ORDER BY ... LIMIT n

MySQL에서는 UPDATE나 DELETE 문장에 ORDER BY 절과 LIMIT 절을 동시에 사용해 특정 칼럼으로 정렬해서 상위 몇 건만 변경 및 삭제하는 것도 가능하다.

한 번에 너무 많은 레코드를 변경 및 삭제하는 작업은 서버에 과부하를 유발하거나 다른 커넥션의 쿼리 처리르 방해할 수 있기에 LIMIT를 이용해 조금씩 잘라서 변경하거나 삭제하는 방식으로 처리할 수 있다.

복제 소스 서버에서 ORDER BY ... LIMIT이 포함된 UPDATE나 DELETE 문장은 바이너리 로그의 포맷이 로우(ROW)일 때는 문제가 되지 않지만 문장(STATEMENT) 기반의 복제에서는 주의가 필요하다.

```sql
mysql> SET binlog_format=STATEMENT;

mysql> DELETE FROM employees ORDER BY last_name LIMIT 10;
```

경고 메시지가 나오는 이유는 ORDER BY에 정렬되더라도 중복된 값의 순서는 복제 소스 서버와 레플리카 서버에서 달라질 수도 있기 때문에 자동으로 기록된다.

### 11-6-2. JOIN UPDATE

두 개 이상의 테이블을 조인해 조인된 결과 레코드를 변경 및 삭제하는 쿼리

- 조인된 테이블 주에서 특정 테이블의 칼럼값을 다른 테이블의 칼럼에 업데이트해야 할 때
- 다른 테이블의 칼럼값을 참조하지 않더라도 조인되는 양쪽 테이블에 공통으로 존재하느 레코드만 찾아서 업데이트할 때

웹 서비스 같은 OLPT 환경에선 데드락을 유발할 가능성이 높아 지양하는게 좋고, 배치 프로그램이나 통계용 UPDATE 문장에서는 유용하게 사용할 수 있다.

```sql
mysql> UPDATE tb_test1 t1, employees e
        SET t1.first_name = e.first_name
        WHERE e.emp_no = t1.emp_no;
```

JOIN UPDATE 문자에서는 GROUP BY나 ORDER BY 절을 사용할 수 없기 떄문에 서브쿼리를 통해 우회해야 한다.

```sql
mysql> UPDATE departments d
        (SELECT de.dept_no, COUNT(*) AS emp_count
          FROM dept_emp de
          GROUP BY de.dept_no) dc
        SET d.emp_count = dc.emp_count
        WHERE dc.dept_no = d.dept_no;
```

원하는 조인의 방향을 옵티마이저에게 알려주고 싶다면 STRAIGHT_JOIN이나 JOIN_ORDER를 사용하면 된다.

```sql
mysql> UPDATE (SELECT de.dept_no, COUNT(*) AS emp_count
                FROM dept_emp de
                GROUP BY de.dept_no) dc
        STRAIGHT_JOIN departments d ON dc.dept_no = d.dept_no
        SET d.emp_count = dc.emp_count;

mysql> UPDATE /*+ JOIN_ORDER (dc, d) */
        (SELECT de.dept_no, COUNT(*) AS emp_count
          FROM dept_emp de
          GROUP BY de.dept_no) dc
        INNER JOIN departments d ON dc.dept_no = d.dept_no
        SET d.emp_count = dc.emp_count;
```

### 11-6-3. 여러 레코드 UPDATE

8.0버전부터 레코드 생성(Row Constructor) 문법을 이용해 레코드별로 서로 다른 값을 업데이트할 수 있다.

```sql
--// VALUES ROW(...), ROOW(...), ... 문법을 사용하면 SQL 문장 내에서 임시 테이블을 생성하는 효과
mysql> UPDATE user_level ul
        INNER JOIN (VALUES ROW(1, 1),
                            ROW(2, 4)) new_user_level (user_id, user_lv)
                                        ON new_user_level.user_id = ul.user_id
        SET ul.user_lv = ul.user_lv + new_user_level.user_lv;
```

### 11-6-4. JOIN DELETE

JOIN DELETE 또한 하나의 테이블만 삭제하는 게 아닌 여러 테이블을 삭제할 수 있다.

```sql
mysql> DELETE e
        FROM employees e, dept_emp de, departments d
        WHERE e.emp_no = de.emp_no AND de.dept_no = d.dept_no AND d.dept_no = 'd001';

mysql> DELETE e, de
        FROM employees e, dept_emp de, departments d
        WHERE e.emp_no = de.emp_no AND de.dept_no = d.dept_no AND d.dept_no = 'd001';

mysql> DELETE e, de, d
        FROM employees e, dept_emp de, departments d
        WHERE e.emp_no = de.emp_no AND de.dept_no = d.dept_no AND d.dept_no = 'd001';
```

STRAIGHT_JOIN이나 JOIN_ORDER 힌트로 조인의 순서를 옵티마이저에게 지시할 수 있다.

```sql
mysql> DELETE e, de, d
        FROM departments d
          STRAIGHT_JOIN dept_emp de ON de.dept_no = d.dept_no
          STRAIGHT_JOIN employees e ON e.emp_no = de.emp_no
        WHERE d.dept_no = 'd001';

mysql> DELETE /*+ JOIN_ORDER(d, de, e) */ e, de, d
        FROM departments d
          INNER JOIN dept_emp de ON de.dept_no = d.dept_no
          INNER JOIN employees e ON e.emp_no = de.emp_no
        WHERE d.dept_no = 'd001';
```

## 11-7. 스키마 조작(DDL)

DBMS 서버의 모든 오브젝트를 생성한거나 변경하는 쿼리.

스토어드 프로시저나 함수, DB나 테이블 등을 생성하거나 변경하는 대부분의 명령이 해당.

### 11-7-1. 온라인 DDL

8.0 버전으로 업그레이드되면서 대부분의 스키마 변경 작업은 MySQL 서버에 내장된 온라인 DDL 기능으로 처리가 가능해졌다.

#### 11-7-1-1. 온라인 DDL 알고리즘

온라인 DDL은 스키마의 변경하는 작업 도중에도 다른 커넥션에서 해당 테이블의 데이터를 변경하거나 조회하는 작업을 가능하게 해준다.

온라인 DDL은 ALGORITHM과 LOCK옵션을 이용해 어떤 모드로 스키마 변경을 실행할지를 결정할 수 있다.

`old_alter_table` : ALTER TABLE 명령이 온라인 DDL로 작동할지, 아니면 예전 방식(읽고 쓰기를 막고 스키마 변경하는 방식)으로 처리할지를 결정할 수 있다.

**ALTER TABLE 명령 실행 시 순서**

1. ALGORITHM=INSTANT로 스키마 변경시 가능한지 확인 후, 가능하다면 선택
2. ALGORITHM=INPLACE로 스키마 변경시 가능한지 확인 후, 가능하다면 선택
3. ALGORITHM=COPY 알고리즘 선택

`INSTANT` :

- 테이블의 데이터는 전혀 변경하지 않고, 메타데이터만 변경하고 작업 완료.
- 테이블이 가진 레코드 건수와 무관하게 작업 시간은 매우 짧다.
- 스키마 변경 도중 테이블의 읽고 쓰기는 대기하게 되지만 스키마 변경 시간이 매우 짧기 때문에 다른 커넥션의 쿼리 처리에는 크게 영향을 미치지 않는다.

`INPLACE` :

- 임시 테이블로 데이터를 복사하지 않고 스키마 변경을 실행한다.
- 내부적으로 테이블의 리빌드를 실행할 수도 있으며, 레코드의 복사 작업은 없지만 모든 레코드를 리빌드해야하기 때문에 테이블의 크기에 따라 많은 시간이 소요될 수도 있음.
- 스키마 변경 중에도 테이블의 읽기와 쓰기 모두 가능하다.

`COPY` :

- 변경된 스키마를 적용한 임시 테이블을 생성하고, 테이블의 레코드를 모두 임시 테이블로 복사한 후 최종적으로 임시 테이블을 RENAME해서 스키마 변경 완료.
- 테이블 읽기만 가능하고 DML은 실행 불가.

**INPLACE나 COPY 알고리즘의 경우 명시할 수 있는 LOCK**

- `NONE` : 아무런 잠금을 걸지 않음. (대부분 INPLACE의 설정값)
- `SHARED` : 읽기 잠금을 걸고 스키마 변경을 실행하기 떄문에 스키마 변경 중 읽는 가능하지만 쓰기는 불가함.
- `EXCLUSIVE` : 쓰기 잠금을 걸고 스키마 변경을 실행하기 때문에 테이블의 읽고 쓰기 불가.

**INPLACE 알고리즘을 사용하는 경우의 구분**

- 데이터 재구성이 필요한 경우 : 잠금을 필요로 하지 않기 때문에 읽고 쓰기는 가능하지만 여전히 테이블의 레코드 건수에 따라 상당히 많은 시간이 소요될 수도 있다.
- 데이터 재구성이 필요치 않은 경우 : INPLACE 알고리즘을 사용하지만 INSTANT 알고리즘과 비슷하게 매우 빨리 작업이 완료될 수 있다.

### 11-7-2. 데이터베이스 변경

데이터베이스에 설정할 수 있는 옵션은 기본 문자 집합이나 콜레이션을 설정하는 정도이다.

#### 11-7-2-1. 데이터베이스 생성

```sql
mysql> CREATE DATABASE [IF NOT EXISTS] employees;
mysql> CREATE DATABASE [IF NOT EXISTS] employees CHARACTER SET utf8mb4;
mysql> CREATE DATABASE [IF NOT EXISTS] employees
          CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

#### 11-7-2-2. 데이터베이스 목록

```sql
mysql> SHOW DATABASES;
mysql> SHOW DATABASES LIKE '&emp&';
```

#### 11-7-2-3. 데이터베이스 선택

```sql
mysql USE employees;
```

#### 11-7-2-4. 데이터베이스 속성 변경

```sql
mysql> ALTER DATABASE employees CHARACTER SET=euckr;
mysql> ALTER DATABASE employees CHARACTER SET=euckr COLLATE=euckr_korean_ci;
```

#### 11-7-2-5. 데이터베이스 삭제

```sql
mysql> DROP DATABASE [IF EXISTS] employees;
```

### 11-7-3. 테이블 스페이스 변경

**제너럴 테이블스페이스 제약사항**

- 파티션 테이블은 제너럴 테이블스페이스를 사용하지 못함
- 복제 소스와 레플리카 서버가 동일 호스트에서 실행되는 경우 ADD DATAFILE 문장은 사용 불가
- 테이블 암호화(TDE)는 테이블스페이스 단위로 설정
- 테이블 압축 가능 여부는 테이블스페이스의 블록 사이즈와 InnoDB 페이지 사이즈에 의해 결정됨
- 특정 테이블을 삭제(DROP TABLE)해도 디스크 공간이 운영체제로 반납되지 않음

**제너럴 테이블스페이스 장점**

- 제너럴 테이블스페이스를 사용하면 파일 핸들러를 최소화
- 테이블스페이스 관리에 필요한 메모리 공간을 최소화

### 11-7-4. 테이블 변경

#### 11-7-4-1. 테이블 생성

```sql
mysql> CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tb_test (
        member_id BIGINT [UNSIGNED] [AUTO_INCREMENT],
        nickname CHAR(20) [CHARACTER SET 'utf8'] [COLLATE 'utf8_general_ci'] [NOT NULL],
        home_url VARCHAR(200) [COLLATE 'latin1_general_cs'],
        birth_year SMALLINT [(4)] [UNSIGNED] [ZEROFILL],
        member_point INT [NOT NULL] [DEFAULT 0],
        registered_dttm DATETIME [NOT NULL],
        modified_ts TIMESTAMP [NOT NULL] [DEFAULT CURRENT_TIMESTAMP],
        gender ENUM('Female', 'Male') [NOT NULL],
        hobby SET('Reading', 'Game', 'Sports'),
        profil TEXT [NOT NULL],
        session_data BLOB,
        PRIMARY KEY (member_id),
        UNIQUE INDEX ux_nickname (nickname),
        INDEX ix_registerddttm (registered_dttm)
        ) ENGINE=InnoDB;
```

 - 모든 칼럼은 NULL 또는 NOT NULL 제약을 명시할 수 있다.
 - 문자열 타입은 타입 뒤에 반드시 최대한 저장할 수 있는 문자 수를 명시해야한다. CHARACTER SET절은 칼럼에 저장되는 문자열 값이 어떤 문자 집합을 사용할지를 결정, COLLATE로 문자열 비교나 정렬 규칙을 나타내기 위한 콜레이션을 설정할 수 있다.
 - 숫자 타입은 선택적으로 길이를 가질 수 있지만, 저장될 값의 길이가 아닌 단순히 보여줄 길이를 지정하는 것이다. 
 - SIGNED는 음수와 양수 모두 저장할 수 있다.
 - ZEROFILL은 왼쪽에 '0'을 패딩할지 결정하는 옵션이다.
 - DATE, DATETIME, TIMESTAMP 모두 값이 자동으로 현재 시간으로 업데이트되도록 기본 값을 명시할 수 있다.
 - ENUM 또는 SET 타입은 타입의 이름 뒤에 해당 칼럼이 가질 수 있는 값을 괄호로 정의해야 한다.

 #### 11-7-4-2. 테이블 구조 조회

 `SHOW CREATE TABLE` : 테이블의 구조를 확인하는 쿼리. 테이블의 CREATE TABLE 문장을 표시한다.

 `DESC` : `DESCRIBE`의 약어 형태, 테이블의 칼럼 정보를 보기 편한 표 형태로 표시해준다.

 #### 11-7-4-3. 테이블 구조 변경

`ALTER TABLE` : 테이블의 구조를 변경하는 쿼리, 테이블 자체의 속성 뿐만 아니라 인덱스의 추가/삭제, 칼럼의 추가/삭제 등 다양한 용도로 사용된다.

```sql
 mysql> ALTER TABLE employees
        CONVERT TO CHARACTER SET UTF8MB4 UTF8MB4_GENERAL_CI,
        ALGORITHM = INPLACE,
        LOCK = NONE;

mysql> ALTER TABLE employees
        ENGINE = InnoDB,
        ALGORITHM = INPLACE,
        LOCK = NONE;
```

#### 11-7-4-4. 테이블 명 변경

`RENAME TABLE` : 테이블 명을 변경하거나 다른 데이터베이스로 테이블을 이동할 때 사용하는 쿼리.

```sql
mysql> RENAME TABLE table1 TO table2;
mysql> RENAME TABLE db1.table1 TO db2.table2;
```

두번째 쿼리 데이터베이스를 옮기는 작업을 하면 교체하는 동안 일시적으로 기존 테이블이 없어지는 시점이 발생한다.

이를 방지하기 RENAME 쿼리를 하나의 문장으로 묶어서 실행할 수 있으며, 이때 명시된 모든 테이블은 잠금을 걸고 변경 작업을 진행한다.

```sql
mysql> RENAME TABLE batch TO batch_old,
        batch_new TO batch;
```

#### 11-7-4-5. 테이블 상태 조회

```sql
mysql> SHOW TABLE STATUS LIKE 'employees' \G

mysql> SELECT * FROM information_schema.TABLES
        WHERE TABLE_SCHEMA = 'employees'
        AND TABLE_NAME = 'employees' \G;
```

> 참고 : `\G`는 레코드의 칼럼을 라인당 하나씩만 표현하게 하는 옵션이다.

> 참고 : information_schema 데이터베이스의 테이블들은 MyySQL 서버가 가진 테이블들에 대한 다양한 정보를 제공한다.
> - 데이터베이스 객체에 대한 메타 정보
> - 테이블과 칼럼에 대한 간략한 통계 정보
> - 전문 검색 디버깅을 위한 뷰
> - 압축 실행과 실패 횟수에 대한 집계


#### 11-7-4-6. 테이블 구조 복사

```sql
--// 테이블 구조 복사
mysql> CREATE TABLE temp_employees LIKE employees;

--// 테이블 데이터 복사
mysql> INSERT INTO temp_employees SELECT * FROM employees;
```

#### 11-7-4-7. 테이블 삭제

```sql
mysql> DROP TABLE [IF EXISTS] table1;
```
**주의** : 어댑티브 해시 인덱스가 활성화돼 있는 경우, 삭제시 어댑티브 해시 인덱스 정보도 모두 삭제되기 때문에 서버의 부하가 높아지고 간접적으로 다른 쿼리 처리에 영향을 미칠 수도 있다.

### 11-7-5. 칼럼 변경

#### 11-7-5-1. 칼럼 추가

```sql
--// 테이블의 제일 마지막에 새로운 칼럼을 추가
mysql> ALTER TABLE employees ADD COLUMN emp_telno VARCHAR(20), ALGORITHM=INSTANT;

--// 테이블의 중간에 새로운 칼럼을 추가
mysql> ALTER TABLE employees ADD COLUMN emp_telno VARCHAR(20) AFTER emp_no, ALGORITHM=INSTANT, LOCK=NONE;
```

#### 11-7-5-2. 칼럼 삭제

삭제는 항상 INPLACE 알고리즘으로만 가능하다.

```sql
mysql> ALTER TABLE employees DROP COLUMN emp_telno,
        ALGORITHM=INPLACE, LOCK=NONE;
```

#### 11-7-5-3. 칼럼 이름 및 칼럼 타입 변경

```sql
--// 칼럼의 이름 변경
mysql> ALTER TABLE salaries CHANGE to_date end_date DATE NOT NULL, ALGORITHM=INPLACE, LOCK=NONE;

--// INT 칼럼을 VARCHAR 타입으로 변경
mysql> ALTER TABLE salaries MODIFY salary VARCHAR(20), ALGORITHM=COPY, LOCK=SHARED;

--// VARCHAR 타입의 길이 확장
mysql> ALTER TABLE employees MODIFY last_name VARCHAR(30) NOT NULL, ALGORITHM=INPLACE, LOCK=NONE;

--// VARCHAR 타입의 길이 축소
mysql> ALTER TABLE employees MODIFY last_name VARCHAR(10) NOT NULL, ALGORITHM=COPY, LOCK=SHARED;
```

### 11-7-6. 인덱스 변경

#### 11-7-6-1. 인덱스 추가

```sql
mysql> ALTER TABLE employees ADD PRIMARY KEY (emp_no)
        ALGORITHM=INPLACE, LOCK=NONE;

mysql> ALTER TABLE employees ADD UNIQUE INDEX ux_empno (emp_no)
        ALGORITHM=INPLACE, LOCK=NONE;

mysql> ALTER TABLE employees ADD INDEX ix_lastname (last_name)
        ALGORITHM=INPLACE, LOCK=NONE;

mysql> ALTER TABLE employees ADD FULLTEXT INDEX fx_firstname_lastname (first_name, last_name)
        ALGORITHM=INPLACE, LOCK=SHARED;

mysql> ALTER TABLE employees ADD SPATIAL INDEX fx_loc (last_location)
        ALGORITHM=INPLACE, LOCK=SHARED;
```
#### 11-7-6-2. 인덱스 조회

`SHOW INDEXES` : 테이블의 인덱스만 표시. 인덱스 칼럼별로 한 줄씩 표시해준다.

`SHOW CREATE TABLE` : 테이블의 생성 구문을 그대로 보여준다.

#### 11-7-6-3. 인덱스 이름 변경

```sql
mysql> ALTER TABLE salaries RENAME INDEX ix_salary TO ix_salary2, ALGORITHM=INPLACE, LOCK=NONE;
```

#### 11-7-6-4. 인덱스 가시성 변경

인덱스를 삭제했다가 새로 생성하는 것은 많은 시간이 걸린다. 그래서 쿼리 실행할때 해당 인덱스를 사용할 수 있게 할지 말지를 결정하는 기능을 도입했다.

```sql
mysql> ALTER TABLE employees ALTER INDEX ix_firstname INVISIBLE;
```

#### 11-7-6-4. 인덱스 삭제

`ALTER TABLE ,,, DROP INDEX ...` : 인덱스를 삭제하는 명령.

```sql
mysql> ALTER TABLE employees DROP PRIMARY KEY, ALGORITHM=COPY, LOCK=SHARED;

mysql> ALTER TABLE employees DROP INDEX ux_empno, ALGORITHM=INPLACE, LOCK=NONE;

mysql> ALTER TABLE employees DROP INDEX fx_loc, ALGORITHM=INPLACE, LOCK=NONE;
```

### 11-7-7. 테이블 변경 묶음 실행

```sql
mysql> ALTER TABLE employees
        ADD INDEX ix_lastname (last_name, first_name),
        ADD INDEX ix_birthdate (birth_date),
        ALGROITHM=INPLACE, LOCK=NONE;
```

### 11-7-8. 프로세스 조회 및 강제 종료

`SHOW PROCESSLIST` : 서버에 접속된 사용자의 목록이나 각 클라이언트 사용자가 현재 어떤 쿼리를 실행하고 있는지 확인하는 명령.

특정 스레드의 실행 중인 쿼리나 커넥션 자체를 강제 종료하려면 KILL 명령으로 ID를 지정해주면 된다.

#### 11-7-9. 활성 트랜잭션 조회

```sql
--// 5초 이상 실행된 트랜잭션 조회
mysql> SELECT trx_id,
        (SELECT CONCAT(user,'@',host)
        FROM information_schema.proceesslist
        WHERE id=trx_mysql_thread_id) AS source_info,
        trx_state,
        trx_started,
        now(),
        (unix_timestamp(now()) - unix_timestamp(trx_started)) AS lasting_sec,
        trx_requested_lock_id,
        trx_wait_started,
        trx_mysql_thread_id,
        trx_tables_in_use,
        trx_tables_locked
        FROM information_schema.innodb_trx
        WHERE (unix_timestamp(now()) - unix_timestamp(trx_started)) > 5; \G
```

## 11-8. 쿼리 성능 테스트

### 11-8-1. 쿼리의 성능에 영향을 미치는 요소

 - 운영체제의 캐시

 - MySQL 서버의 버퍼 풀(InnoDB 버퍼 풀과 MyISAM의 키 캐시)

 - 독립된 MySQL 서버

 - 쿼리 테스트 횟수