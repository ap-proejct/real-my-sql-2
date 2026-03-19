# 📘 곽유섭 - Chapter 14 - 스토어드 프로그램 정리

> Real MySQL 8.0 2권 | Chapter 14 - 스토어드 프로그램

---

## 📝 정리 내용

# 14. 스토어드 프로그램

## 14-1. 스토어드 프로그램의 장단점

### 14-1-1. 스토어드 프로그램의 장점

- `데이터베이스의 보안 향상` :

  MySQL의 스토어드 프로그램은 자체적인 보안 설정 기능을 가지고 있으며, 스토어드 프로그램 단위로 실행 권한을 부여할 수 있다.

  이러한 보안 기능을 조합해서 특정 테이블의 읽기와 쓰기, 또는 특정 칼럼에 대해서만 권한을 설정하는 등 세밀한 권한 제어가 가능하다.

  주요 기능을 스토어드 프로그램으로 작성한다면 SQL 인젝션 같은 기본적인 보안 사고는 피할 수 있을 것이다.

  MySQL 서버의 스토어드 프로그램은 입력 값의 유효성을 체크한 후에야 동적인 SQL 문장을 생성하므로 SQL의 문법적인 취약점을 이용한 해킹은 어렵다.

- `기능의 추상화` :

  프로그래밍 언어의 라이브러리 기능을 활용해서 처리할 수도 있지만, MySQL 클라이언트를 이용하는 사용자는 활용할 수 없다. 또한 라이브러리를 사용한다고 하더라도 응용 프로그램이 여러 가지 언어를 사용하는 경우에는 관리가 어려워질 수도 있다.

  이러한 기능을 스토어듣 프로그램으로 작성해서 등록하면 개발 언어나 도구와 관계없이 쉽게 활용할 수 있다.

- `네트워크 소요 시간 절감` :

  하나의 프로그램에서 100, 200번씩 실행해야 하는 쿼리를 스토어드 프로그램으로 구현한다면 스토어드 프로그램을 호출할 때 한 번만 네트워크를 경우하면 되기 떄문에 네트워크 소요 시간을 줄이고 성능을 개선할 수 있다.

- `절차적 기능 구현` :

  DBMS 서버에서 사용하는 SQL 쿼리는 절차적인 기능을 제공하지 않는다. 그에 반해 스토어드 프로그램은 DBMS 서버에서 절차적인 기능을 실행할 수 있는 제어 기능을 제공한다.

  SQL 문장만으로 처리할 수 없는 문제를 해결하기 위해 애플리케이션과 MySQL 서버 간의 네트워크 통신 횟수를 늘리고, 필요한 데이터를 클라이언트와 서버 간에 주고받아야 하기 때문에 네트워크를 경유하는 데 시간이 소요된다.

  스토어드 프로그램을 이용해 절차적인 기능을 구현한다면 최소한 네트워크 경유에 걸리는 시간만큼은 줄일 수 있으며, 불필요한 애플리케이션 코드도 많이 줄일 수 있다.

- `개발 업무의 구분` :
  DBMS 코드를 개발하는 조직에서는 트랜잭션 단위로 데이터베이스 관련 처리를 하는 스토어드 프로그램을 만들어 API처럼 제공하고, 애플리케이션 개발자는 스토어드 프로그램을 호출해서 사용하는 형태로 역할을 구분해서 개발을 진행할 수 있다.

### 14-1-2. 스토어드 프로그램의 단점

- `낮은 처리 성능` : MySQL은 스토어드 프로그램과 같은 절차적 코드 처리를 주목적으로 하는 것이 아니어서 스토어드 프로그램의 처리 성능이 다른 프로그램 언어에 비해 떨어진다. 또한 다른 DBMS의 스토어드 프로그램과 달리 MySQL의 스토어드 프로그램은 실행 시마다 코드가 파싱돼야 한다.

- `애플리케이션 코드의 조각화` : 각 기능을 담당하는 프로그램 코드가 자바와 MySQL 스토어드 프로그램으로 분산된다면 애플리케이션의 설치나 배포가 더 복잡해지고 유지보수 또한 어려워질 수 있다.

## 14-2. 스토어드 프로그램의 문법

- `헤더(정의부)` :

  스토어드 프로그램의 이름과 입출력 값을 명시하는 부분

  보안이나 스토어드 프로그램의 작동 방식과 관련된 옵션을 명시할 수 있다.

- `바디(본문)` :

  스토어드 프로그램이 호출됐을 때 실행하는 내용을 작성하는 부분

### 14-2-1. 예제 테스트 시 주의사항

- 프로시저나 함수의 이름과 "(" 사이의 모든 공백을 제거하고 실행

- `thread_stack` 설정을 추가해서 각 스레드가 사용하는 스택의 크기를 늘리는 것이 좋다. (대략 512KB)

### 14-2-2. 스토어드 프로시저

서로 데이터를 주고받아야 하는 여러 쿼리를 하나의 그룹으로 묶어서 독립적으로 실행하기 위해 사용하는 것.

스토어드 프로시저는 반드시 독립적으로 호출돼야 하며, SELECT나 UPDATE 같은 SQL 문장에서 스토어드 프룃저를 참조할 수 없다.

#### 14-2-2-1. 스토어드 프로시저 생성 및 삭제

```sql
mysql> CREATE PROCEDURE sp_sum(IN param1 INTEGER, IN param2 INTEGER, OUT param3 INTEGER)
        BEGIN
            SET param3 = param1 + param2;
        END ;;
```

**주의사항**

- 스토어드 프러시저는 기본 반환값이 없다. 즉, 스토어드 프로시저 내부에서는 값을 반환하는 RETURN 명령을 사용할 수 없다.

- 스토어드 프로시저의 각 파라미터는 3가지 특성 중 하나를 지닌다.
  - IN 타입으로 정의된 파라미터는 입력 전용 파라미터를 의미한다.

  - OUT 타입으로 정의된 파라미터는 출력 전용 파라미터다. OUT 파라미터는 스토어드 프로시저 외부에서 프로시저를 호출할 때 어떤 값을 전달하는 용도로는 사용할 수 없으며, 오직 외부 호출자로 결과 값을 전달하는 용도로만 사용된다.

  - INOUT 타입으로 정의된 파리미터는 입력 및 출력 용도로 모두 사용할 수 있다.

`DELIMITER` : 명령의 끝을 알려주는 종료 문자를 변경하는 명령어. 보통 ";;"또는 "//"를 종료 문자로 설정한다. (기본값 : ;)

`ALTER PROCEDURE` : 스토어드 프로시저를 변경

`DROP PROCEDURE` : 스토어드 프로시저를 삭제

```sql
mysql> ALTER PROCEDURE sp_sum SQL SECURITY DEFINER;
```

스토어드 프로시저의 파라미터나 프로시저의 처리 내용을 변경할 때는 `ALTER PROCEDURE` 를 사용할 수 없다. 이때는 스토어드 프로시저를 삭제한 후 다시 생성해야한다.

#### 14-2-2-2. 스토어드 프로시저 실행

```sql
mysql> SET @result:=0;
mysql> SELECT @result;  --// 0
mysql> CALL sp_sum(1,2,@result);
mysql> SELECT @result;  --// 3
```

INOUT이나 OUT 타입의 파라미터는 MySQL 세션 변수를 사용해야 한다.

#### 14-2-2-3. 스토어드 프로시저의 커서 반환

스토어드 프로그램은 명시적으로 커서를 파라미터로 전달받거나 반환할 수 없다. 하지만 스토어드 프로시저 내에서 커서를 오픈하지 않거나 SELECT 쿼리의 결과 셋을 페치하지 않으면 해당 쿼리의 결과 셋은 클라이언트로 바로 전송된다.

```sql
mysql> CREATE PROCEDURE sp_selectEmployees (IN in_empno INTEGER)
        BEGIN
            SELECT * FROM employees WHERE emp_no = in_empno;
        END;;
```

```sql
--// 디버깅 용도로 쉽게 테스트하는 방법
mysql> CREATE PROCEDURE sp_sum (IN param1 INTEGER, IN param2 INTEGER, OUT param3 INTEGER)
        BEGIN
            SELECT '> Stored procedure started. ' AS debug_message;
            SELECT CONCAT('> param1 : ', param1) AS debug_message;
            SELECT CONCAT('> param2 : ', param2) AS debug_message;

            SET param3 = param1 + param2;
            SELECT '> Stored procedure completed. ' AS debug_message;
        END;;
```

#### 14-2-2-4. 스토어드 프로시저 딕셔너리

`information_schema.ROUTINES` : 스토어드 프로시저의 정보를 조회할 수 있는 뷰

### 14-2-3. 스토어드 함수

스토어드 함수는 하나의 SQL문장으로 작성이 불가능한 기능을 하나의 SQL 문장으로 구현해야 할 때 사용한다.

SQL 문장과 관계없이 별도롤 실행되는 기능이라면 굳이 스토어드 함수를 개발할 필요가 없다.

독립적으로 실행돼도 된다면 스토어드 프로시저를 사용하는 것이 좋다. 왜냐하면 상대적으로 프로시저보다 제약 사항이 많기 때문이다.

#### 14-2-3-1. 스토어드 함수 생성 및 삭제

`CREATE FUNCTION` : 스토어드 함수 생성 명령. 모든 입력 파라미터는 읽기 전용이라 IN 이나 OUT, INOUT 같은 형식을 지정할 수 없다.

스토어드 함수는 반드시 정의부에 RETURNS 키워드를 이용해 반환되는 값의 타입을 명시해야 한다.

```sql
mysql> CREATE FUNCTION sf_sum(param INTEGER, param2 INTEGER)
        RETURNS INTEGER
        BEGIN
            DECLARE param3 INTEGER DEFAULT 0;
            SET param3 = param1 + param2;
            RETURN param3;
        END;;
```

**함수와 프로시저의 차이점**

- 함수 정의부에 RETURNS로 반환되는 값의 타입을 명시
- 함수 본문 마지막에 정의부에 지정된 타입과 동일한 타입의 값을 RETURN 명령으로 반환

**스토어드 함수 본문에서 사용하지 못하는 사항**

- PREPARE와 EXECUTE 명령을 이용한 프리페어 스테이트먼트를 사용할 수 없다.
- 명시적 또는 묵시적인 ROLLBACK/COMMIT을 유발하는 SQL문장을 사용할 수 없다.
- 재귀 호출(Recursive call)을 사용할 수 없다.
- 스토어드 함수 내에서 프로시저를 호출할 수 없다.
- 결과 셋을 반환하는 SQL 문장을 사용할 수 없다.

스토어드 프로시저와 마찬가지로 `ALTER FUNCTION` 명령을 사용할 수 있으나, 스토어드 함수의 입력 파라미터나 처리 내용을 변경할 수 없으며, 특성만 변경할 수 있다.

```sql
mysql> ALTER FUNCTION sf_sum SQL SECURITY DEFINER;
```

#### 14-2-3-2. 스토어드 함수 실행

스토어드 함수는 프로시저와 달리 CALL 명령으로 실행할 수 없으며, SELECT 문장을 이용해 실행한다.

```sql
mysql> SELECT sf_sum(1,2) AS sum;
```

### 14-2-4. 트리거

`트리거` :

    테이블의 레코드가 저장되거나 변경될 때 미리 정의해둔 작업을 자동으로 실행해주는 스토어드 프로그램.

    테이블 레코드가 INSERT, UPDATE, DELETE될 때 시작되도록 설정할 수 있다.

    트리거가 생성돼 있는 테이블에 칼럼을 추가하거나 삭제할 때 임시 테이블에 데이터를 복사하는 작업이 필요한데, 이때 레코드마다 트리거를 실행해야 하기 때문에 실행 시간이 더 오래 걸린다.

    하나의 테이블에 대해서 동일 이벤트에 2개 이상의 트리거를 생성할 수 있다.

    ROW 포맷의 바이너리 로그를 이용해서 복제를 하는 경우, 포맷과 관계없이 동일한 결과를 만들어내지만 트리거의 실행 위치가 다르다는 차이가 있다.

#### 14-2-4-1. 트리거 생성

`CREATE TRIGGER` : 트리거 생성 명령

    BEFORE나 AFTER 키워드와 INSERT, UPDATE, DELETE로 트리거가 실행될 이벤트와 시점을 명시할 수 있다.

    트리거 정의부 끝에 FOR EACH ROW 키워드를 붙여서 개별 레코드 단위로 트리거를 실행되게 한다.

```sql
mysql> CREATE TRIGGER on_delete BEFORE DELETE ON employees
        FOR EACH ROW
        BEGIN
            DELETE FROM salaries WHERE emp_no=OLD.emp_no;
        END;;
```

- 예제 트리거에서 사용된 `OLD` 키워드는 employees 테이블의 변경되기 전 레코드를 지칭. 변경될 레코드를 지칭하고자 할 때는 `NEW` 키워드를 사용하면 된다.

**트리거 이벤트**
| SQL 종류 | 발생 트리거 이벤트("==>"는 발생하는 이벤트의 순서를 의미) |
| -- | -- |
| INSERT | BEFORE INSERT ==> AFTER INSERT |
| LOAD DATA | BEFORE INSERT ==> AFTER INSERT |
| REPLACE | 종복 없을 때 : <br> BEFORE INSERT ==> AFTER INSERT <br> 중복 있을 때: <br> BEFORE DELETE ==> AFTER DELETE ==> BEFORE INSERT ==> AFTER INSERT |
| INSERT INTO ... ON DUPLICATE SET | 중복 없을 때 : <br> BEFORE INSERT ==> AFTER INSERT <br> 중복 있을 때 : <br> BEFORE UPDATE ==> AFTER UPDATE |
| UPDATE | BEFORE UPDATE ==> AFTER UPDATE |
| DELETE | BEFORE DELETE ==> AFTER DELETE |
| TRUNCATE | 이벤트 발생하지 않음 |
| DROP TABLE | 이벤트 발생하지 않음 |

- 칼럼의 값을 강제로 변환해서 테이블에 저장하는 것은 BEFORE 트리거에서만 가능하다.

**트리거 BEGIN ... END 코드 블록에서 사용하지 못하는 작업**

- 트리거는 외래키 관계에 의해 자동으로 변경되는 경우 호출되지 않는다.
- 복제에 의해 레플리카 서버에 업데이트되는 데이터는 레코드 기반의 복제에서는 레플리카 서버의 트리거를 기동시키지 않지만 문장 기반의 복제에서는 레플리카 서버에서도 트리거를 기동시킨다.
- 명시적 또는 묵시적 ROLLBACK/COMMIT을 유발하는 SQL 문장은 사용할 수 없다.
- RETURN 문장을 사용할 수 없으며, 트리거를 종료할 때는 LEAVE 명령을 사용한다.
- mysql과 information_schema, performance_schema 데이터베이스에 존재하는 테이블에 대해서는 트리거를 생성할 수 없다.

#### 14-2-4-2. 트리거 실행

트리거는 스토어드 프로시저나 함수와 같이 작동을 확인하기 위해 명시적으로 실행해 볼 수 있는 방법이 없다.

#### 14-2-4-3. 트리거 딕셔너리

`information_schema.TRIGGERS` : 트리거 딕셔너리 정보를 볼 수 있는 뷰

### 14-2-5. 이벤트

`이벤트` : 주어진 특정 시간에 스토어드 프로그램을 실행할 수 있는 스케줄러 기능

스케줄링을 전담하는 스레드를 활성화한 경우(`event_scheduler` 변수를 ON이나 1로 설정)에만 이벤트를 실행할 수 있다.

```sql
mysql> SHOW GLOBAL VARIABLES LIKE 'event_scheduler';

mysql> SHOW PROCESSLIST;
```

`information_schema.EVENTS` : 가장 최근에 실행된 이벤트 정보를 뷰로 확인할 수 있다.

#### 14-2-5-1. 이벤트 생성

```sql
--// 일회성 이벤트
mysql> CREATE EVENT onetime_job
        ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 HOUR
        DO
        INSERT INTO daily_rank_log VALUES (NOW(), 'Done');
```

단 한 번 실행되는 일회성 이벤트를 등록하려면 `ON SCHEDULE AT` 절을 명시하면 된다.

```sql
--// 반복성 이벤트
mysql> CREATE EVENT daily_ranking
        ON SCHEDULE EVERY 1 DAY STARTS '2020-09-07 01:00:00' ENDS '2021-01-01 00:00:00'
        DO
        INSERT INTO daily_rank_log VALUES (NOW(), 'Done');
```

이벤트 생성 명령의 `EVERY` 절에는 `DAY`뿐만 아니라 `YEAR`,`QUARTER`,`MONTH`,`HOUR`,`MINUTE`,`WEEK`,`SECOND`,... 등의 반복 주기를 사용할 수 있기 때문에 원하는 형태의 반복 스케줄링을 쉽게 만들어 낼 수 있다.

여러 개의 SQL 문장과 연산 작업이 필요하다면 `BEGIN ... END` 블록을 통해 작성하면 된다.

이벤트의 반복성 여부와 관계없이 `ON COMPLETION` 절을 이용해 완전히 종료된 이벤트를 삭제할지, 그대로 유지할지 선택할 수 있다.(기본 : 완전히 종료된 이벤트는 자동 삭제)

`ON COMPLETION PRESERVE` : 이벤트 실행이 완료돼도 MySQL 서버는 그 이벤트를 삭제하지 않는다.

이벤트를 생성할땐 `ENABLE`, `DISABLE`, `DISABLE ON SLAVE` 3가지 상태로 생성할 수 있다.

기본적으로 레플리카 서버는 `SLAVESIDE_DISABLED`상태이지만 레플리카 서버가 소스 서버로 승격될 경우, 수동으로 이벤트 상태를 `ENABLE`상태로 변경해야 한다.

```sql
--// 레플리카 서버에서만 DISABLE된 이벤트 목록 확인
mysql> SELECT event_schema, event_name
        FROM information_schema.EVENTS
        WHERE STATUS = 'SLAVESIDE_DISABLED';

--// 수동으로 ENABLE 변경
mysql> ALTER EVENT myevent ENABLE;
```

#### 14-2-5-3. 이벤트 딕셔너리

모든 이벤트 정보는 `information_schema.EVENTS` 뷰를 통해 이벤트 목록과 상세 내용을 확인할 수 있다.

### 14-2-6. 스토어드 프로그램 본문(Body) 작성

#### 14-2-6-1. BEGIN ... END 블록과 트랜잭션

본문(Body)는 `BEGIN`으로 시작해서 `END`로 끝나며, 하나의 `BEGIN ... END` 블록은 또 다른 여러 개의 `BEGIN .. END`블록을 중첩해서 포함할 수 있다.

**트린잭션을 시작하는 명령어**

- BEGIN
- START TRANSACTION

`BEGIN ... END` 블록 내에서 사용된 `BEGIN` 명령은 모두 트랜잭션의 시작이 아니라 `BEGIN ... END` 블록으 시작 키워드로 해석한다.

본문에서 트랜잭션을 시작할 때는 `START TRANSACTION` 명령을 사용해야하며, 종료할 때는 `COMMIT` 또는 `ROLLBACK` 명령을 사용하면 된다.

트랜잭션은 스토어드 프로시저나 이벤트의 본뭄에서만 사용할 수 있다.

```sql
mysqk> CREATE PROCEDURE sp_hello ( IN name VARCHAR(50) )
        BEGIN
            START TRANSACTION;
            INSERT INTO tb_hello VALUES (name, CONCAT('Hello ', name));
            COMMIT;
        END ;;
```

프로시저 내부에서 `COMMIT`이나 `ROLLBACK` 명령으로 트랜잭션을 완료하면 외부에서 `COMMIT`이나 `ROLLBACK`을 실행해도 아무런 의미가 없다.

스토어드 프로시저 내부에서 트랜잭션을 완료할지 또는 프로시저를 호출하는 클라이언트에서 확인 과정을 거친 후에 커밋, 롤백할지는 중요한 문제이니 주의해야 한다.

스토어드 함수와 트리거는 본문 내에서 트랜잭션을 커밋하거나 롤백할 수 없으므로 "프로시저 외부에서 트랜잭션 완료"와 똑같은 형태로만 처리된다.

#### 14-2-6-2. 변수

`변수` : 스토어드 프로그램의 `BEGIN ... END` 블록 내에서만 사용하는 변수(= 스토어드 프로그램 로컬 변수, 로컬 변수)

> 주의 : 로컬 변수가 사용자 변수보다 빠르게 처리되므로 스토어드 프로그램 내부에서는 가능한 한 사용자 변수 대신 로컬 변수를 사용하는 편이 좋다.

`DECLARE` 명령으로 정의되고 반드시 타입이 함께 명시돼야 한다.

`BEGIN ... END` 블록 내에서만 변수가 유효하며, 사용자 변수보다 빠르고 다른 쿼리나 스토어드 프로그램과의 간섭을 발생시키지 않는다.

로컬 변수는 반드시 타입과 함께 정의되기 때문에 컴파일러 수준으로 타입 오류를 체크할 수 있다.

```sql
--// 로컬 변수 정의
DECLARE v_name VARCHAR(50) DEFAULT 'Matt';
DECLARE v_email VARCHAR(50) DEFAULT 'matt@email.com';

--// 로컬 변수에 값을 할당
SET v_name = 'Kim', v_email = 'kim@email.com';

--// SELECT ... INTO 구문을 이용한 값의 할당
SELECT emp_no, first_name, last_name INTO v_empno, v_firstname, v_lastname
FROM employees
WHERE emp_no = 10001
LIMIT 1;
```

`SET` 명령은 `DECLARE`로 정의한 변수에 값을 저장하는 할당 명령. 하나의 `SET` 명령으로 여러 개의 로컬 변수에 값을 할당할 수도 있다.

`SELECT ... INTO` 명령은 SELECT한 레코드의 칼럼값을 로컬 변수에 할당하는 명령. 이때 SELECT 명령은 반드시 1개 레코드를 반환하는 SQL이어야 한다.

`BEGIN ... END`블록에서 입력 파라미터와 로컬 변수, 그리고 테이블의 칼럼명은 모두 같은 이름을 가질 수 있다.

**우선순위**

1. DECLARE로 정의한 로컬 변수
2. 스토어드 프로그램의 입력 파라미터
3. 테이블의 칼럼

중첩된 `BEGIN ... END` 블록은 각각 똑같은 이름의 로컬 변수를 정의할 수 있으며, 이는 일반적인 프로그래밍 언어의 스코프와 같은 범위를 가진다.

#### 14-2-6-3. 제어문

제어문은 스토어드 프로그램의 `BEGIN ... END` 블록 내부에서만 사용할 수 있다.

##### 14-2-6-3-1. IF ... ELSEIF ... ELSE ... END IF

```sql
mysql> CREATE FUNCTION sf_greatest(p_value1 INT, p_value2 INT)
        RETURNS INT
        BEGIN
            IF p_value1 IS NULL THEN
                RETURN p_value2;
            ELSEIF p_value2 IS NULL THEN
                RETURN p_value1;
            ELSEIF p_value1 > = p_value2 THEN
                RETURN p_value1;
            ELSE
                RETURN p_value2;
            END IF;
        END;;
```

##### 14-2-6-3-2. CASE WHEN ... THEN ... ELSE ... END CASE

```
--// 동등 비교에서만 사용가능
CASE 변수
    WHEN 비교대상값1 THEN 처리내용1
    WHEN 비교대상값2 THEN 처리내용2
    ELSE 처리내용3
END CASE;

CASE
    WHEN 비교조건식1 THEN 처리내용1
    WHEN 비교조건식2 THEN 처리내용2
    ELSE 처리내용3
END CASE;
```

##### 14-2-6-3-3. 반복 루프

`LOOP`, `REPEAT`, `WHILE` 구문을 사용해 반복 루프 처리를 할 수 있다.

`LOOP`문은 별도의 반복 조건을 명시하지 못하는 반면 `REPEAT`나 `WHILE`은 반복 조건을 명시할 수 있다.

`LOOP` 구문에서 반복 루프를 벗어나려면 `LEAVE` 명령을 사용하면 된다.

`REPEAT`문은 먼저 본문을 처리하고 그다음 반복 조건을 체크하지만 `WHILE1`은 그 반대로 실행된다.

```sql
mysql> CREATE FUNCTION sf_factorial1 (p_max INT)
            RETURNS INT
        BEGIN
            DECLARE v_factorial INT DEFAULT 1;

            factorial_loop : LOOP
                SET v_factorial = v_factorial * p_max;
                SET p_max = p_max - 1;
                IF p_max<=1 THEN
                    LEAVE factorial_loop;
                END IF;
            END LOOP;

            RETURN v_factorial;
        END;;
```

```sql
mysql> CREATE FUNCTION sf_factorial2 (p_max INT)
            RETURNS INT
        BEGIN
            DECLARE v_factorial INT DEFAULT 1;
            REPEAT
                SET v_factorial = v_factorial * p_max;
                SET p_max = p_max - 1;
            UNTIL p_max <= 1 END REPEAT;

            RETURN v_factorial;
        END;;
```

```sql
mysql CREATE FUNCTION sf_factorial3 (p_max INT)
            RETURNS INT
        BEGIN
            DECLARE v_factorial INT DEFAULT 1;

            WHILE p_max>1 DO
                SET v_factorial = v_factorial * p_max;
                SET p_max = p_max - 1;
            END WHILE;

            RETURN v_factorial;
        END;;
```

#### 14-2-6-4. 핸들러와 컨디션을 이용한 에러 핸들링

`핸들러` : 이미 정의한 컨디션 또는 사용자가 정의한 컨디션을 어떻게 처리(핸들링)할지 정의하는 기능

`컨디션` : SQL문장의 처리 상태에 대해 별명을 붙이는 것과 같은 역할 수행

##### 14-2-6-4-1. SQLSTATE와 에러 번호(Error No)

`ERROR-NO` :

    4자리 숫자 값으로 구성된 에러코드, MySQL에서만 유효한 에러 식별 번호다.

`SQL-STATE` :

    다섯 글자의 알파벳과 숫자로 구성되며, 에러뿐만 아니라 여라 가지 상태를 의미하는 코드. 이 값은 ANSI SQL 표준을 준수하는 DBMS에서는 모두 똑같은 값과 의미를 가진다.

    대부분의 ErrorNo는 특정 SqlState 값과 매핑돼 있으며, 매핑되지 않는 ErrorNo는 SqlState값 "HY000"으로 설정된다.

    - "00" 정상 처리됨
    - "01" 경고 메시지
    - "02" Not found
    - 그 이외의 값은 DBMS별로 할당된 각자의 에러 케이스를 의미한다.

`ERROR-MESSAGE` :

    포매팅된 텍스트 문장. 사람이 읽을 수 있는 형태의 에러 메시지.

##### 14-2-6-4-2. 핸들러

```sql
--// 핸들러 정의
DECLARE handler_type HANDLER
    FOR condition_value [, condition_value] ... handler_statements
```

**핸들러 타입**

- `CONTINUE` : handler_statements를 실행하고 스토어드 프로그램의 마지막 실행 지점으로 다시 돌아가서 나머지 코드를 처리한다.
- `EXIT` :

  정의된 handler_statements를 실행한 뒤에 이 핸들러가 정의된 `BEGIN ... END` 블록을 벗어난다.

  최상위 블록에 정의됐다면 스토어드 프로그램을 벗어나서 종료된다.

  핸들러의 handler_statements 부분에 함수의 반환 타입에 맞는 적절한 값을 반환하는 코드가 반드시 포함돼 있어야 한다.

**핸들러 정의 문장의 컨디션 값**

- `SQLSTATE` : 스토어드 프로그램이 실행되는 도중 어떤 이벤트가 발생했을 때 해당 이벤트의 SQLSTATE 값이 일치할 때 실행되는 핸들러를 정의

- `SQLWARNING` : 스토어드 프로그램에서 코드를 실행하던 중 경고가 발생했을 때 실행되는 핸들러를 정의

- `NOT FOUND` : SELECT 쿼리 문의 결과 건수가 1건도 없거나 CURSOR의 레코드를 마지막까지 읽은 뒤에 실행하는 핸들러를 정의

- `SQLEXCEPTION` : 경고와 NOT FOUND, "00"으로 시작하는 SQLSTATE 이외의 모든 케이스를 의미

- MySQL의 에러 코드 값을 직접 명시할 때도 있다.

`"00000"인 SQLSTATE와 에러 번호 0`은 모두 정상적으로 처리됐음을 의미하는 값이라서 `condition_value`에 사용해서는 안된다.

```sql
--//SQLEXCEPTION이 발생했을때 error_flag를 1로 설정, 코드로 다시 돌아가 계속 실행
DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET error_flag=1;

--//SQLEXCEPTION 발생 시 블록으로 감싼 문장 실행 후, 에러가 발생한 코드가 포함된 블록을 벗어난다.
DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT 'Error occurred - terminating';
    END;;
```

##### 14-2-6-4-3. 컨디션

MySQL에서 어떤 조건(이벤트)이 발생했을 때 실행할지를 명시하는 방법

```sql
--// 컨티션 정의 방법
DECLARE condition_name CONDITION FOR condition_value;
```

- `condition_value`에 에러 번호를 사용할 때는 MySQL의 에러번호를 입력하면 된다. 이때 여러 개를 동시에 명시할 수 없다.

- `condition_value`에 SQLSTATE를 명시하는 경우 SQLSTATE 키워드를 입력하고 값을 입력하면 된다.

##### 14-2-6-4-4. 컨디션을 사용하는 핸들러 정의

```sql
mysql> CREATE FUNCTION sf_testfunc()
            RETURNS BIGINT
        BEGIN
            DECLARE dup_key CONDITION FOR 1062;
            DECLARE EXIT HANDLER FOR dup_key
                BEGIN
                    RETURN -1;
                END;

            INSERT INTO tb_test VALUES (1);
            RETURN 1;
        END ;;
```

#### 14-2-6-5. 시그널을 이용한 예외 발생

`시그널(SIGNAL)` : 스토어드 프로그램에서 사용자가 직접 예외나 에러를 발생시키는 명령

##### 14-2-6-5-1. 스토어드 프로그램의 BEGIN ... END 블록에서 SIGNAL 사용

```sql
mysql> CREATE FUNCTION sf_divide (p_dividend INT, p_divisor INT)
            RETURNS INT
        BEGIN
            DECLARE null_divisor CONDITION FOR SQLSTATE '45000';

            IF p_divisor IS NULL THEN
                SIGNAL null_divisor
                    SET MESSAGE_TEXT='Divisor can not be null', MYSQL_ERRNO=9999;
            ELSEIF p_divisor=0 THEN
                SIGNAL SQLSTATE '45000'
                    SET MESSAGE_TEXT='Divisor can not be 0', MySQL_ERRNO=9998;
            ELSEIF p_dividend IS NULL THEN
                SIGNAL SQLSTATE '01000'
                    SET MESSAGE_TEXT='Dividend is null, so regarding dividend as 0', MysQL ERRNO=9997;
                RETURN 0;
            END IF;

            RETURN FLOOR(p_dividend / p_divisor);
        END;;
```

`SIGNAL` 명령은 에러와 경고를 모두 발생시킬 수 있다.

| SQLSTATE 클래스(종류) | 의미                       |
| --------------------- | -------------------------- |
| "00"                  | 정상 처리됨(Success)       |
| "01"                  | 처리 중 경고 발생(Warning) |
| 그 밖의 값            | 처리 중 오류 발생(Error)   |

"00"은 `SIGNAL` 명령문과 연결해서는 안된다,

"01"과 연결된 `SIGNAL`은 스토어드 프로그램을 종료시키진 않지만, 종료된 이후 경고 메시지를 출력한다.

그 밖의 모든 값은 에러로 간주해서 즉시 처리를 종료하고 호출자에게 메시지와 코드를 반환한다.

##### 14-2-6-5-2. 핸들러 코드에서 SIGNAL 사용

```sql
mysql> CREATE PROCEDURE sp_remove_user (IN p_userid INT)
        BEGIN
            DECLARE v_affectedrowcount INT DEFAULT 0;
            DECLARE EXIT HANDLER FOR SQLEXCEPTION
                BEGIN
                    SIGNAL SQLSTATE '45000'
                        SET MESSAGE_TEXT='Can not remove user information', MYSQL_ERRNO=9999;
                END;

            --// 사용자 정보 삭제
            DELETE FROM tb_user WHERE user_id=p_userid;
            --// 위에서 실행된 쿼리로 삭제된 레코드 건수 확인
            SELECT ROW_COUNT() INTO v_affectedrowcount;
            --// 삭제된 레코드 건수가 1건이 아닌 경우 에러 발생
            IF v_affectedrowcount<>1 THEN
                SIGNAL SQLSTATE '45000';
            END IF;
        END;;
```

#### 14-2-6-6. 커서

`커서` : JDBC 프로그램에서 자주 사용하는 결과 셋

- 스토어드 프로그램의 커서는 전 방향(전진) 읽기만 가능하다.

- 스토어드 프로그램에서는 커서의 칼럼을 바로 업데이트 하는 것이 불가능하다.

**커서 구분**

- `센서티브 커서` : 일치하는 레코드에 대한 정보를 실제 레코드의 포인터만으로 유지하는 형태
  - 커서를 이용해 칼럼의 데이터를 변경하거나 삭제하는 것이 가능하다.

  - 칼럼의 값이 변경되 커서를 생성한 SELECT 쿼리의 조건에 일치하지 않거나 레코드가 삭제되면 커서에도 즉시 반영된다.

  - 임시테이블로 레코드를 복사하지 않기 떄문에 커서의 오픈이 빠르다.

- `인센서티브 커서` : 일치하는 레코드를 별도의 임시 테이블로 복사해서 가지고 있는 형태
  - SELECT 쿼리에 부합되는 결과를 우선적으로 임시 테이블로 복사하기 떄문에 느리다.

  - 임시 테이블로 복사된 데이터를 조회하는 것이라 칼럼 값의 변경이나 삭제가 불가능하다.

  - 다른 트랜잭샨과의 충돌이 발생하지 않는다.

`어센서티브` : 센서티브 커서와 인센서티브 커서를 혼용해서 사용하는 방식. ( = MySQL 스토어드 프로그램에서 정의되는 커서 )

**DECLARE 명령으로 정의하는 순서에 대한 주의사항**

1. 로컬 변수와 CONDITION
2. CURSOR
3. HANDLER

## 14-3. 스토어드 프로그램의 보안 옵션

### 14-3-1. DEFINER와 SQL SECURITY 옵션

- `DEFINER` : 해당 스토어드 프로그램의 소유권. 소토어드 프로그램이 실행될 때의 권한으로 사용됨.

- `SQL SECURITY` : 스토어드 프로그램을 실행할 때 누구의 권한으로 실행할지 결정하는 옵션.
  - `DEFINER` : 스토어드 프로그램을 생성한 사용자
    - 모든 스토어드 프로그램이 기본적으로 가지는 옵션

  - `INVOKER` : 스토어드 프로그램을 호출(실행)한 사용자
    - 스토어드 프로시저와 함수, 뷰만 가질 수 있다.

### 14-3-2. DETERMINISTIC과 NOT DETERMINISTIC 옵션

- `DETERMINISTIC` : 스토어드 프로그램의 입력이 가탇면 시점이나 상황에 관계없이 결과가 항상 같다.

- `NOT DETERMINISTIC` : 입력이 같아도 시점에 따라 결과가 달라질 수도 있음을 의미한다.

해당 옵션은 SQL 문장에서 반복적으로 호출될 수 있는 스토어드 함수한테 영향이 많으며, 쿼리의 성능을 급격하게 떨어뜨리기도 한다.

스토어드 함수에선 풀 테이블 스캔을 유도하는 NOT DETERMINISTIC 옵션이 기본값으로 설정되어 있기 때문에 스토어드 함수 사용 시 DETERMINISTIC 옵션을 설정할 것을 권장한다.

## 14-4. 스토어드 프로그램의 참고 및 주의사항

### 14-4-1. 한글 처리

스토어드 프리소저나 함수에서 값을 넘겨받을 때 문자 집합을 별도로 지정해 한글이 깨지는 문자를 피할 것을 권장한다.

```sql
--// 문자 집합 확인
mysql> SHOW VARIABLES LIKE 'character%';

--// 문자 집합 지정
mysql> CREATE FUNCTION sf_getstring()
            RETURNS VARCHAR(20) CHARACTER SET utf8mb4
        BEGIN
            RETURN '한글 테스트';
        END;;
```

### 14-4-2. 스토어드 프로그램과 세션 변수

스토어드 프로그램에선 로컬 변수와 사용자 변수 모두 사용할 수 있다.

로컬 변수는 정의할 때 정확한 타입과 길이를 명시해야 하지만 사용자 변수는 이런 제약이 없기 때문에 사용에 주의해야 한다.

### 14-4-3. 스토어드 프로시저와 재귀 호출

스토어드 프로그램에서도 재귀 호출이 가능한데 이는 스토어드 프로시저에서만 사용 가능하다.

`max_sp_recursion_depth` : 최대 몇 번까지 재귀 호출을 허용할지를 설정하는 변수

### 14-4-4. 중첩된 커서 사용

일반적으로 하나의 커서를 열고 사용이 끝나면 닫고 다시 새로운 커서를 열어서 사용하는 형태지만, 중첩된 루프 안에서 두 개의 커서를 동시에 열어서 사용할 때도 있다.

이렇게 두 개의 커서를 동시에 열어서 사용할 때는 예외 핸들링에 주의해야한다.

이처럼 반복 루프가 여러 번 중첩되어 커서가 사용될때 서로 다른 BEGIN ... END 블록으로 구분해서 작성하면 쉽고 깔끔하게 해결할 수 있다.

---
