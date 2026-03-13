## 0. 개요
- 절차적인 쿼리를 위해 스토어드 프로그램(스토어드 루틴)을 이용할 수 있다
- 스토어드 프로시저는 프토어드 함수, 트리거와 이벤트 등을 모두 아우르는 명칭이다

## 1. 스토어드 프로그램의 장단점
### 1. 스토어드 프로그램의 장점
- 데이터베이스 보안 향상
  - 스토어드 프로그램은 자체적인 보안 설정 기능이 있어 스토어드 프로그램 단위로 실행 권한을 부여할 수 있다
  - 세밀한 권한제어가 가능하고, SQL 인젝션 같은 보안사고도 피할 수 있다
- 기능의 추상화
  - 개발 언어나 도구와 상관없이 쉽게 활용할 수 있다
- 네트워크 소요 시간 절감
  - 스토어드 프로그램을 호출할 때 한 번만 네트워크를 경유하기 떄문에 네트워크 소요 시간을 줄이고 성능을 개선할 수 있다
- 절차적 기능 구현
  - SQL 쿼리는 절차적 기능(IF, WHILE)을 제공하지 않지만 스토어드에서는 가능하기 때문에 불필요한 애플리케이션 코드를 줄일 수 있다
- 개발 업무의 구분
  - 애플리케이션 개발 조직과 SQL 개발 조직이 구분돼 있는 회사도 있다.
  - DBMS 코드를 개발하는 조직에서는 트랜잭션 단위로 데이터베이스 관련 처리를 하는 스토어드 프로그램을 만들어 API처럼 제공하고, 해당 프로그램을 호출해서 사용하는 형태로 구분해서 개발을 진행할 수 있다

### 2. 스토어드 프로그램의 단점
- 낮은 처리 성능
  - 스토어드 프로그램은 MySQL 엔진에서 해석되고 실행되기 때문에 처리 성능이 다른 프로그램 언어에 비해 상대적으로 떨어진다
- 애플리케이션 코드의 조각화
  - 프로그램 코드가 자바와 스토어드 프로그램으로 분산된다면 설치나 배포가 더 복잡해지고 유지보수 또한 어려워질 수 있다

## 2. 스토어드 프로그램의 문법
- 헤더 부문과 본문 부분으로 나눌 수 있다
- 헤더 부분(정의부): 스토어드 프로그램 이름과 입출력 값을 명시하는 부분이다.
  - 보안이나 스토어드 프로그램의 작동 방식과 관련된 옵션도 명시할 수 있다
- 본문 부분(바디): 스토어드 프로그램이 호출됐을 때 실행하는 내용을 작성하는 부분이다

### 2. 스토어드 프로시저
- 서로 데이터를 주고받아야 하는 여러 쿼리를 하나의 그룹으로 묶어서 독립적으로 실행하기 위해 사용하는 것이다
- 각 쿼리가 연관되어 데이터를 주고받으면서 반복적으로 실행돼야 할 때 이용하면 네트워크 전송을 최소화하고 수행 시간을 줄일 수 있다
- 반드시 독립적으로 호출돼야 하며, SELECT나 UPDATE 같은 SQL 문장에서 스토어드 프로시저를 참조할 수 없다

#### 1. 스토어드 프로시저 생성 및 삭제
- `CREATE PROCEDURE`명령으로 생성할 수 있다
- 두 가지 주의 사항이 있다
  - 기본 반환값이 없다. 즉 프로시저 내부에서 값을 반환하는 RETURN 명령을 사용할 수 없다
  - 프로시저의 각 파라미터는 다음 3가지 특성 중 하나를 가진다
    - IN 타입으로 정의된 파라미터는 입력 전용 파라미터를 의미한다. 읽기 전용이라고 생각하면 된다
    - OUT 타입으로 정의된 파라미터는 출력 전용 파라미터이다
    - INOUT 타입으로 정의된 파라미터는 입력 및 출력 용도로 모두 사용할 수 있다

- `;`는 쿼리의 끝을 의미하므로 스토어드 프로그램 내부에 무수히 많은 `;` 문자가 있어 스토어드 프로그램에서는 SQL 문장의 구분자를 구분해야 한다
- 명령의 끝을 알려주는 종료 문자를 변경하는 명령어는 `DELIMITER`이다
  - 보통 스토어드 프로그램을 생성할 때는 `;;` 또는 `//`과 같이 연속된 2개의 문자열을 종료 문자로 설정한다
  - 이렇게 변경하면 SELECT, INSERT 같은 명령도 `;;`을 사용해야 하기 때문에 되돌리는 것이 좋다
- 변경할 때는 `ALTER PROCEDURE`을 사용하고 삭제할 때는 `DROP PROCEDURE` 명령을 사용하면 된다
- 하지만 프로시저의 파라미터나 처리 내용을 변경할 때는 삭제한 후 다시 생성하는 것이 유일한 방법이다

```sql
-- 프로시저 이름은 sp_sum이며, param1, param2, param3 파라미터를 필요로 한다
-- BEGIN부터 END까지가 본문에 속하며 파라미터로 받은 param1과 param2를 더해 param3에 저장한다
CREATE PROCEDURE sp_sum (IN param1 INTEGER, IN param2 INTEGER, OUT param3 INTEGER)
    BEGIN
        SET param3 = param1 + aparam2;
    END ;;

-- 종료 문자를 ;;로 변경
DELIMITER ;;

CREATE PROCEDURE sp_sum (IN param1 INTEGER, IN param2 INTEGER, OUT param3 INTEGER)
    BEGIN
        SET param3 = param1 + aparam2;
    END ;;

-- 스토어드 프로그램의 생성이 완료되면 다시 기본 종료 문자인 ;로 복구
DELIMITER ;

-- 보안 옵션을 DEFINER로 변경
ALTER PROCEDURE sp_sum SQL SECURITY DEFINER
```

#### 2. 스토어드 프로시저 실행
- 반드시 CALL 명령어로 실행해야 한다
- IN 타입의 파라미터는 상숫값을 그대로 전달해도 무방하지만 OUT이나 INOUT 타입의 파라미터는 세션 변수를 이용해 값을 주고받아야 한다
- 자바같은 프로그래밍 언어에서는 세션 변수를 사용하지 않고 바로 OUT이나 INOUT 타입의 변숫값을 읽어올 수 있다

```sql
SET @result:=0;
-- IN 타입은 값을 다시 전달받을 필요 없어 리터럴 형태로 사용해도 된다
-- 하지만 OUT 타입은 값을 넘겨받아야 하므로 세션 변수를 사용해야 한다
CALL sp_sum(1,2,@result);

SELECT @result;
```

#### 3. 스토어드 프로시저의 커서 반환
- 명시적으로 커서를 파라미터로 전달받거나 반환할 수 없다
- 하지만 프로시저 내에서 커서를 오픈하지 않거나 SELECT 쿼리의 결과 셋을 페치하지 않으면 해당 쿼리의 결과 셋은 클라이언트로 바로 전송된다
- 하나의 프로시저에서 2개 이상의 결과 셋을 반환할 수 있다
- 메시지를 화면에 출력하는 기능을 제공하지 않으며, 별도의 로그 파일에 기록하는 기능도 없다
  - 프로시저가 잘못돼서 디버깅하려고 해도 원인을 찾기가 쉽지 않다

```sql
-- SELECT 쿼리를 실행하지만 그 결과를 전혀 사용하지 않는다
CREATE PROCEDURE sp_selectEmployees (IN in_empno INTEGER)
    BEGIN
        SELECT * FROM employees WHERE empno = in_empno;
    END ;;

-- OUT 변수에 화면에 출력하는 처리를 하지도 않았는데 쿼리의 결과가 전송된다
CALL sp_selectEmployees(10001);
```

- 아래 방법으로 쿼리 결과를 화면에 출력되는 걸 확인해서 디버깅해볼 수 있다
- `SIGNAL`과 `RESIGNAL`명령을 이용하거나 `DIAGNOSTICS`도 좋은 디버깅 방법이지만 복잡하다
```sql
CREATE PROCEDURE sp_sum (IN param1 INTEGER, IN param2 INTEGER, OUT param3 INTEGER)
  BEGIN
    SELECT '> Stored procedure started' AS debug_message;
    SELECT CONCAT(' > param1 : ', param1) AS debug_message;
    SELECT CONCAT(' > param2 : ', param2) AS debug_message;
    SET param3 = param1 + param2;
    SELECT '> Stored procedure finished' AS debug_message;
  END ;;

CALL sp_sum(1,2,@result);
```

#### 4. 스토어드 프로시저 딕셔너리
- 8.0부터 사용자에게 보이지 않는 시스템 테이블로 저장된다
- `information_schema` 데이터베이스의 `ROUTINES` 뷰를 통해 프로시저의 정보를 조회할 수 있다

```sql
SELECT routine_schema, routine_name, routine_type
FROM information_schema.ROUTINES
WHERE routine_schema = 'test';
```

### 3. 스토어드 함수
- 하나의 SQL 문장으로 작성이 불가능한 기능을 하나의 SQL 문장으로 구현해야 할 때 사용한다
- 별도로 실행되는 기능이라면 스토어드 함수는 사용하지 말고 프로시저를 사용하는 것이 좋다
  - 스토어드 함수의 유일한 장점이 SQL을 사용한다는 것이고, 프로시저보다 제약 사항이 많기 때문이다

```sql
-- 부서별로 최근 배속된 사원 2명씩 가져온다
-- dept_emp 테이블을 부서별로 그루핑까지는 가능하지만 2명을 가져올 수는 없다. 이럴 때 스토어드 함수를 사용한다
SELECT dept_no, sf_getRecentEmp(dept_no)
FROM dept_emp
GROUP BY dept_no;
```

#### 1. 스토어드 함수 생성 및 삭제
- `CREATE FUNCTION` 명령으로 생성할 수 있고, 모든 파라미터는 읽기 전용이라 `IN`, `OUT`, `INOUT` 키워드를 사용하지 않는다
- 반드시 정의부에 RETURNS 키워드를 이용해 반환되는 값의 타입을 명시해야 한다

```sql
-- 정의부에 RETURNS로 반환되는 값의 타입을 명시해야 한다
-- 본문 마지막에 정의부에 지정된 타입과 동일한 타입의 값을 RETURN 명령으로 반환해야 한다
CREATE FUNCSTION sf_sum(param1 INTEGER, param2 INTEGER)
  RETURNS INTEGER
  BEGIN
    DECLARE param3 INTEGER DEFAULT 0;
    SET param3 = param1 + param2;
    RETURN param3;
  END ;;
```

- 프로시저와 달리 본문에서는 다음과 같은 사항을 사용하지 못한다
  - PREPARE와 EXECUTE 명려을 이용한 프리페어 스테이트먼트를 사용할 수 없다
  - 명시적 또는 묵시적인 ROLLBACK/COMMIT을 유발하는 SQL 문장을 사용할 수 없다
  - 재귀 호출을 사용할 수 없다
  - 스토어드 함수 내에서 프로시저를 호출할 수 없다
  - 결과 셋을 반환하는 SQL 문장을 사용할 수 없다
- `ALTER FUNCTION`을 사용할 수 있지만 파라미터나 처리 내용을 변경할 수는 없고, 함수의 특성만 변경할 수 있다
  - `ALTER FUNCTION sf_sum SQL SECURITY DEFINER;`

#### 2. 스토어드 함수 실행
- SELECT 문장을 이용해 실행한다
  - `SELECT sf_sum(1, 2) AS sum;`

### 4. 트리거
- 테이블에 레코드가 저장되거나 변경될 때 미리 정의해둔 작업을 자동으로 실행해주는 스토어드 프로그램이다
- 이름 그대로 데이터 변화가 생길 때 다른 작업을 수행한다
- INSERT, UPDATE, DELETE될 때 시작되도록 설정할 수 있다
- 애플리케이션에서 개발하는 것이 더 효율적이라 필요성이 떨어지고, 트리거가 있으면 칼럼을 추가하거나 삭제할 때 실행 시간이 훨씬 오래걸린다
  - 해당 작업을 할 때 데이터를 복사하는 작업이 필요한데, 이때 레코드마다 트리거를 한 번씩 실행해야 하기 때문이다

#### 1. 트리거 생성
- `CREATE TRIGGER` 명령으로 생성한다
- 정의부 끝에 `FOR EACH ROW` 키워드를 붙여서 개별 레코드 단위로 트리거가 실행되게 한다
  - 트리거는 레코드 단위로만 작동하지만 해당 키워드는 그대로 사용하도록 문법이 구현돼 있다

```sql
-- 트리거 이름 뒤에는 언제 실행될지를 명시한다
-- OLD 키워드는 employees 테이블의 변경되기 전 레코드를 지칭한다
-- 변경될 레코드를 지칭하고자 할 때는 NEW 키워드를 사용하면 된다
CREATE TRIGGER on_delete BEFORE DELETE ON employees
  FOR EACH ROW
BEGIN
    INSERT INTO salareis VALUES emp_no=OLD.emp_no;
END;;
```

| SQL 종류 | 발생 트리거 이벤트 |
|---|---|
| INSERT | BEFORE INSERT => AFTER INSERT |
| LOAD DATA | BEFORE INSERT => AFTER INSERT |
| REPLACE | 중복 레이터가 없을 때 <br>BEFORE INSERT => AFTER INSERT <br>중복 레이터가 있을 때 <br>BEFORE DELETE => AFTER DELETE => BEFORE INSERT => AFTER INSERT |
| INSERT ... ON DUPLICATE KEY UPDATE | 중복 레이터가 없을 때 <br>BEFORE INSERT => AFTER INSERT <br>중복 레이터가 있을 때 <br>BEFORE DELETE => AFTER DELETE => BEFORE INSERT => AFTER INSERT |
| UPDATE | BEFORE UPDATE => AFTER UPDATE |
| DELETE | BEFORE DELETE => AFTER DELETE |
| TRUNCATE | 이벤트 발생하지 않음 |
| DROP TABLE | 이벤트 발생하지 않음 |

- 칼럼의 값을 체크해서 강제로 변환하는 것도 가능하지만 `BEFORE`에서만 가능하다
- BEGIN ... END 블록에서 사용하지 못하는 몇 가지 유형이 있다
  - 트리거는 외래키 관계에 의해 자동으로 변경되는 경우 호출되지 않는다
  - 복제에 의해 레플리카 서버에 업데이트되는 데이터는 레코드 기반 복제에서는 레플리카 서버의 트리거를 기동시키지 않지만 문장 기반 복제에서는 레플리카 서버에서도 트리거를 기동시킨다
  - 명시적 또는 묵시적인 ROLLBACK/COMMIT을 유발하는 SQL 문장을 사용할 수 없다
  - RETURN 문장을 사용할 수 없으며, 트리거를 종료할 때는 LEAVE 명령을 사용한다
  - information_schema, performance_schema는 트리거를 생성할 수 없다

#### 3. 트리거 딕셔너리
- `information_schema` 데이터베이스의 `TRIGGERS` 뷰를 통해 트리거의 정보를 조회할 수 있다

### 5. 이벤트
- 특정한 시간에 스토어드 프로그램을 실행할 수 있는 스케줄러 기능을 의미한다
- 스케줄링을 전담하는 스레드를 활성화한 경우에만 이벤트가 실행된다
  - `event_scheduler` 시스템 변수가 ON이나 1로 설정해서 활성화해야 한다
  - `SHOW PROCESSLIST`에 `event_scheduler`가 나타나는지 확인할 수 있다

#### 1. 이벤트 생성
- 쿼리나 프로시저를 호출할 수도 있고 `BEGIN ... END` 블록을 사용할 수도 있다

```sql
-- 일회성 이벤트를 등록하려면 ON SCHEDULE AT을 사용하면 된다
CREATE EVENT ontime_job
  ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 DAY
  DO
    INSERT INTO daily_rank_log VALUES(NOW(), 'Done');

-- 반복성 이벤트
-- 하루단위로 해당 날짜까지 이벤트를 생성하는 예제이다
CREATE EVENT daily_ranking
  ON SCHEDULE EVERY 1 DAY START '2020-09-07 01:00:00' ENDS '2021-01-01 00:00:00'
  DO
    INSERT INTO daily_rank_log VALUES(NOW(), 'Done');
```

- 기본적으로는 완전히 종료된 이벤트는 삭제되지만 `ON COMPLETION PRESERVE`옵션과 함께 이벤트를 등록하면 삭제되지 않는다

### 6. 스토어드 프로그램 본문(Body) 작성

#### 1. BEGIN ... END 블록과 트랜잭션
- 본문은 BEGIN으로 시작해서 END로 끝나며, 하나의 블록은 여러 개의 블록을 가질 수 있다
- BEGIN ... END 블록 내에서 주의해야 할 것은 트랜잭션으로, 트랜잭션을 실행하는 명령어는 두 가지가 있다
  - BEGIN
  - START TRANSACTION
- 하지만 BEGIN ... END 블록 내에서의 `BEGIN`명령은 모두 해당 블록의 시작 키워드인 `BEGIN`으로 해석된다
- 트랜잭션을 시작할 때는 `START TRANSACTION`을 사용하고 종료할 때는 `COMMIT`이나 `ROLLBACK`을 사용한다

```sql
CREATE PROCEDURE sp_hello (IN name VARCHAR(50))
BEGIN
    START TRANSACTION;
    INSERT INTO tb_hello VALUES (name, CONCAT('Hello, ', name));
    COMMIT;
END;;
```

- 외부에서 트랜잭션을 실행하고 프로시저를 호출하더라도 프로시저 내부에서 실행되는 것이 아니므로 의미가 없다
- 함수와 트리거는 본문 내 트랜잭션을 사용할 수 없으므로 **프로시저 외부에서 트랜잭션 완료**와 같은 형태로만 처리된다

#### 2. 변수
- BEGIN ... END에서 사용하는 변수로, 로컬 변수라고 표현한다
- DECLARE 키워드로 정의되고 반드시 타입이 함께 명시돼야 한다
- 사용자 변수보다 빠르며 다른 쿼리나 스토어드 프로그램과의 간섭을 발생시키지 않는다
- 디폴트 값을 명시하지 않으면 NULL로 초기화된다

```sql
-- 로컬 변수 정의
DECLARE v_name VARCHAR(50) DEFAULT 'Matt';

-- 로컬 변수에 값을 할당한다
SET v_name = 'John';

-- SELECT ... INTO 구문을 이용한 값의 할당
SELECT emp_no, first_name, last_name INTO v_empno, v_firstname, v_lastname
FROM employees
WHERE emp_no = 1;
LIMIT 1
```

- `BEGIN ... END` 블록에서는 입력 파라미터와 로컬 변수, 테이블 칼럼명이 같은 이름을 가질 수 있다. 그럴 때 다음과 같은 우선순위를 가진다
  - 로컬 변수
  - 입력 파라미터
  - 테이블 칼럼명

#### 3. 제어문
- 조건 비교 및 반복문 같은 처리를 할 수 있다

#### 1). IF ... ELSEIF ... ELSE ... END IF
```sql
CREATE FUNCTION sf_greatest(p_value1 INT, p_value2 INT)
  RETURNS INT
  BEGIN
    IF p_value1 IS NULL THEN
      RETURN p_value2;
    ELSEIF p_value2 IS NULL THEN
      RETURN p_value1;
    ELSEIF p_value1 > p_value2 THEN
      RETURN p_value1;
    ELSE
      RETURN p_value2;
    END IF;
  END;;
```

#### 2). CASE WHEN ... THEN ... ELSE ... END CASE

```sql
CREATE FUNCTION sf_greatest(p_value1 INT, p_value2 INT)
  RETURNS INT
  BEGIN
    CASE
      WHEN p_value1 IS NULL THEN
        RETURN p_value2;
      WHEN p_value2 IS NULL THEN
        RETURN p_value1;
      WHEN p_value1 > p_value2 THEN
        RETURN p_value1;
      ELSE
        RETURN p_value2;
    END CASE;
  END;;
```

#### 3). 반복 루프
- LOOP, REPEAT, WHILE 구문을 사용할 수 있다
- LOOP
  - 반복 조건을 명시하지 못하고, 벗어나려면 `LEAVE`명령을 사용해야 한다
- REPEAT
  - 반복 조건을 명시할 수 있고, 본문을 먼저 처리하고 반복 조건을 체크한다
- WHILE
  - 반복 조건을 명시할 수 있고, 반복 조건을 먼저 체크하고 본문을 처리한다

```sql
CREATE FUNCTION sf_factorial1 (p_max INT)
  RETURNS INT
  BEGIN
    DECLARE v_factorial INT DEFAULT 1;

    factorial_loop : LOOP
      SET v_factorial = v_factorial * p_max;
      SET p_max = p_max - 1;
      IF p_max <=> 1 THEN
        LEAVE factorial_loop;
      END IF;
    END LOOP;
    RETURN v_factorial;
  END;;
```