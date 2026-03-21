# 📘 곽유섭 - Chapter 15 - 데이터 타입

> Real MySQL 8.0 2권 | Chapter 15 - 데이터 타입

---

## 📝 정리 내용

# 15. 데이터 타입

## 15-1. 문자열(CHAR와 VARCHAR)

### 15-1-1. 저장 공간

- `고정 길이` :

  실제 입력되는 칼럼값의 길이에 따라 사용하는 저장 공간의 크기가 변하지 않는다.

- `가변 길이` :

  최대로 저장할 수 있는 값의 길이는 제한돼 있지만, 그 이하 크기의 값이 저장되면 그만큼 저장 공간이 줄어든다.

  VARCHAR 타입은 저장된 값의 유효 크기가 얼마인지를 별도로 저장해 둬야 하므로 1~2바이트의 저장 공간이 추가로 더 필요하다.

- `VARCHAR 타입의 길이` : 255바이트 이하면 1바이트만 사용하고, 256바이트 이상으로 설정되면 2바이트를 사용한다.

**CHAR타입과 VARCHAR타입을 결정하는 판단 기준**

- 저장되는 문자열의 길이가 대개 비슷한가?

- 칼럼의 값이 자주 변경되는가?

### 15-1-2. 저장 공간과 스키마 변경(Online DDL)

`OnLine DDL` : 데이터가 변경되는 도중에도 스키마 변경을 할 수 있도록 하는 기능

VARCHAR 타입의 칼럼이 가지는 길이 저장 공간의 크기에 따라 Online DDL을 지원하는 방식이 달라진다.

- `VARCHAR(63바이트 이하)` : 잠금 없이(LOCK=NONE) 매우 빠르게 변경이 가능하다.

- `VARCHAR(64바이트 이상)` : 문자열이 저장할 수 있는 크기는 최대 256바이트까지 가능하기 때문에 문자열 길이를 저장하는 공간의 크기가 2바이트로 바뀌어야한다. 이 경우 MySQL는 스키마 변경을 하는 동안 읽기 잠금(LOCK=SHARED)을 걸어서 아무도 데이터를 변경하지 못하도록 막고 테이블의 레코드를 복사한다.

### 15-1-3. 문자 집합(캐릭터 셋)

MySQL 서버에서 각 테이블의 칼럼은 모두 서로 다른 문자 집합을 사용해 문자열 값을 저장할 수 있으며, 문자 집합은 문자열을 저장하는 `CHAR`, `VARCHAR`, `TEXT` 타입의 칼럼에만 설정할 수 있다.

편의를 위해 서버, DB, 그리고 테이블 단위로 기본 문자 집합을 설정할 수 있는 기능을 제공한다.

`NCHAR타입` :

    ANSI 표준에서 다국어를 지원할 수 있도록 만든 문자열 타입

    MySQL은 사용할 필요가 없으며, 사용 시 UTF-8 문자 집합을 사용하는 CHAR 타입으로 생성된다.

```sql
--// MySQL에서 사용 가능한 문자 집합 조회
SHOW CHARACTER SET;
```

한국에선 대부분 `euckr`이나 `utf8mb4`를 사용한다.

- `latin계열 문자 집합` : 알파벳, 숫자, 키보드 특수 문자로만 구성된 문자열만 저장해도 될 때 저장공간을 절약하면서 사용할 수 있는 문자 집합(해시, 16진수로 구성된 'Hex String' 또는 단순 코드 값을 저장하는 용도)

- `euckr` : 한국어 전용 문자 집합, 모든 글자는 1~2바이트를 사용한다.

- `utf8mb4` : 다국어 문자를 포함할 수 있는 칼럼에 사용하기 적합하며, 1~4바이트까지 사용한다.

- `utf8` : `utf8mb4`와 부분 문자 집합이며, 1~3바이트까지만 지원한다.

**문자 집합을 설정하는 시스템 변수**

- `character_set_system` : 식별자를 저장할 때 사용하는 문자 집합 (기본값 : utf8)

- `character_set_server` : 서버 기본 문자 집합 (기본값 : utf8mb4)

- `character_set_database` : 데이터베이스 기본 문자 집합 (기본값 : utf8mb4)

- `character_set_filesystem` : 인자로 지정되는 파일의 이름을 해석할 때 사용되는 문자 집합 (기본값 : binary)

- `character_set_client` : 클라이언트가 보낸 SQL 문장을 인코딩해 서버로 전송할 때 사용되는 문자 집합 (기본값 : utf8mb4)

- `character_set_connection` : 서버가 클라이언트로부터 받은 SQL문장을 처리하기 위한 문자 집합 (기본값 : utf8mb4)

- `character_set_results` : 쿼리의 처리 결과를 클라이언트로 전송할 때 사용하는 문자 집합 (기본값 : utf8mb4)

#### 15-1-3-1. 클라이언트로부터 쿼리를 요청했을 때의 문자 집합 변환

서버는 클라이언트로부터 받은 SQL 문장과 변숫값을 정의된 문자 집합(`character_set_connection`)으로 변환한다. 이 때 SQL 문장에 별도의 문자 집합이 지정된 리터럴은 변환 대상에 포함하지 않는다.

`인트로듀서` : SQL문장에서 별도로 문자 집합을 설정하는 지정자.

문자열 앞에 \_와 문자 집합 이름을 붙여 사용한다.

```sql
mysql> SELECT emp_no, first_name FROM employees WHERE first_name = 'Matt';
mysql> SELECT emp_no, first_name FROM employees WHERE first_name = _latin1 'Matt';
```

#### 15-1-3-2. 처리 결과를 클라이언트로 전송할 때의 문자 집합 변환

```sql
--// character_set_client, character_set_results, character_set_connection 변수를 한번에 수정하는 방법
mysql> SET NAMES uft8mb4;   --// 현재 접속된 커넥션에서만 유효
mysql> CHARSET utf8mb4;     --// 재접속할 때도 유효
```

### 15-1-4. 콜레이션(Collation)

문자열 칼럼의 값에 대한 비교나 정렬 순서를 위한 규칙.

비교나 정렬 작업에서 영문 대소문자를 같은 것으로 처리할지, 아니면 더 크거나 작은 것으로 판단할지에 대한 규칙을 정의하는 것.

#### 15-1-4-1. 콜레이션 이해

- 문자 집합은 2개 이상의 콜레이션을 가지고 있다.

- 하나의 문자 집합에 속한 콜레이션은 다른 문자 집합과 공유해서 사용할 수 없다.

```sql
--// 사용가능한 콜레이션 목록 조회
mysql> SHOW COLLATION;
```

- 3개 파트로 구성된 콜레이션 이름
  - 1파트 : 문자 집합의 이름
  - 2파트 : 해당 문자 집합의 하위 분류
  - 3파트 : 대문자, 소문자의 구분 여부.
    - `ci` : 대소문자를 구분하지 않는 콜레이션
    - `cs` : 대소문자를 별도의 문자로 구분하는 콜레이션

- 2개 파트로 구성된 콜레이션 이름
  - 1파트 : 문자 집합의 이름
  - 2파트 : 항상 "bin" 키워드 사용. 이진 데이터를 의미하며, 이진 데이터로 관리되는 문자열 칼럼은 별도의 콜레이션을 가지지 않는다.

  **utf8mb4 문자 집합의 콜레이션**

- 버전이 올라갈수록 문자 간의 정렬 순서나 비교 규칙이 더 정교해지고 언어별 특성이 더 잘 반영된 버전.

- `as`, `ai` : 액센트를 가진 문자를 동일 문자로 판단할지 여부를 나타낸다.
  - `as` : 액센트를 가진 문자를 별도의 문자로 판단한다.
  - `ai` : 액센트를 가진 문자를 동일 문자로 판단한다.

`CHAR`와 `VARCHAR` 같은 타입은 타입의 이름, 길이, 문자 집합과 콜레이션까지 일치해야 똑같은 타입이라고 인식하며, 인덱스를 효율적으로 사용할 수 있다.

```sql
--// 문자 집합이나 콜레이션을 적용하는 방법
mysql> CREATE DATABASE db_test CHARACTER SET=utf8mb4;

mysql> CREATE TABLE tb_member (
    member_id VARCHAR(20) NOT NULL COLLATE latin1_general_cs,
    member_name VARCHAR(20) NOT NULL COLLATE utf8_bin,
    member_email VARCHAR(100) NOT NULL,
    ...
);
```

`SHOW CREATE TABLE` : 테이블의 구조를 확인하는 명령. 컬럼이 디폴트 문자 집합이나 콜레이션을 사용할 때는 별도로 표시하지 않아 분석이 어려울 수 있다.

`information_schema.COLUMNS` : 각 칼럼의 문자 집합이나 콜레이션을 정확히 확인하는 뷰

#### 15-1-4-2. utf8mb4 문자 집합의 콜레이션

콜레이션 이름에 Locale(로캘) 포함 여부에 따라 언어에 종속적인 콜레이션인지 비종속적인 콜레이션인지 구분된다.

    - 비종속 : `utf8mb4_0900_ai_ci`
    - 종속 : `utf8mb4_0900_as_cs`, `utf8mb4_0900_ai_ci` ...

`utf8mb4_0900` 콜레이션은 `NO PAD` 옵션으로 문자열 뒤에 존재하는 공백도 유효 문자로 취급하고 비교하니 주의하자.

만약 5.7 버전에서 도중에 8.0 버전으로 업그레이드할 경우, utf8mb4 콜레이션은 기본값이 달라져 새로 생성되는 db나 테이블은 `utf8mb4_general_ci`이 아닌 `utf8mb4_0900_ai_ci` 콜레이션을 사용하니 주의하자.

**해결책**

1. `default_collation_for_utf8mb4`을 이용해 `utf8bm4_general_ci`로 고정

2. MySQL 설정 파일(my.cnf) 콜레이션 관련 시스템 변수를 `utf8bm4_general_ci`로 고정

3. JDBC드라이버의 경우, 연결 문자열에 `connectionCollation` 속성을 추가해 `utf8bm4_general_ci`로 고정

### 15-1-5. 비교 방식

`CHAR`타입의 칼럼에 SELECT를 실행했을 때는 기본적으로 다른 DBMS처럼 사용되지 않는 공간에 공백 문자가 채워지지 않는다. 그러나 `utf8mb4_bin` 콜레이션을 사용하면 공백 문자가 채워져서 비교된다.

`information_schema.COLLATIONS 뷰에서 PAD_ATTRIBUTE 칼럼` : 문자열 뒤의 공백이 비교 결과에 영향을 미치는지 판단

    - `PAD SPACE` : 공백이 비교 결과에 영향을 미침
    - `NO PAD` : 공백이 비교 결과에 영향을 미치지 않음

예외적으로 LIKE절에선 공백 문자가 항상 무시된다.

### 15-1-6. 문자열 이스케이프 처리

| 이스케이프 표기 | 의미                   |
| --------------- | ---------------------- |
| \0              | 아스키 NULL 문자(0x00) |
| \\'             | 홑따옴표               |
| \\"             | 쌍따옴표               |
| \\b             | 백스페이스             |
| \\n             | 개행문자               |
| \\r             | 캐리지 리턴            |
| \\t             | 탭                     |
| \\\             | 백 슬래시 문자         |
| \\%             | 퍼센트                 |
| \\\_            | 언더스코어             |

## 15-2. 숫자

**숫자를 저장하는 타입**

- `참값` : 소수점 이하 값의 유무와 관계없이 정확히 그 값을 그대로 유지하는 것. (`INTEGER`를 포함한 `INT`, `DECIMAL`)

- `근삿값` : 부동 소수점이라고 불리는 값, 처음 칼럼에 저장한 값과 조회된 값이 정확하게 일치하지 않고 최대한 비슷한 값으로 관리하는 것.
  (`FLOAT`, `DOUBLE`)

**값이 저장되는 포맷**

- `이진 표기법` : 프로그래밍 언어에서 사용하는 정수나 실수 타입.
  (MySQL의 `INTEGER`, `BIGINT` 등 대부분의 숫자 타입이 이진 표기법을 사용한다.)

- `십진 표기법` : 숫자 값의 각 자릿값을 표현하기 위해 4비트나 한 바이트를 사용해서 표기하는 방법.
  (`DECIMAL`만이 십진 표기법을 사용한다.)

근삿값은 저장, 조회할 때의 값이 정확히 일치하지 않아 데이터 차이가 발생할 수 있어 잘 사용되지 않는다.

십진 표기법을 사용하는 `DECIMAL` 타입은 다른 숫자 타입보다 저장 공간을 2배 이상 필요로 한다.

### 15-2-1. 정수

**정수에 사용할 수 있는 데이터 타입**

| 데이터 타입 | 저장공간(Bytes) | 최솟값(Signed) | 최솟값(Unsigned) | 최댓값(Signed) | 최댓값(Unsigned) |
| ----------- | --------------- | -------------- | ---------------- | -------------- | ---------------- |
| TINYINT     | 1               | -128           | 0                | 127            | 255              |
| SMALLINT    | 2               | -32768         | 0                | 32767          | 65535            |
| MEDIUMINT   | 3               | -8388608       | 0                | 8388607        | 16777215         |
| INT         | 4               | -2147483648    | 0                | 2147483647     | 4294967295       |
| BIGINT      | 8               | -2^63          | 0                | 2^63-1         | 2^64-1           |

`UNSIGNED` : 0보다 큰 양의 정수만 저장할 수 있으며 저장 가능한 최댓값은 `SIGNED` 최댓값의 2배.

### 15-2-2. 부동 소수점

`FLOAT` : 정밀도를 명시하지 않으면 4바이트를 사용해 유효 자릿수를 8개까지 유지. 최대 8바이트까지 저장 공간 사용 가능.

`DOUBLE` : 8바이트의 저장 공간을 필요로 하며 최대 유효 자릿수를 16개까지 유지 가능.

**복제 시 주의점**

바이너리 로그 포맷이 `STATEMENT` 타입인 경우, 복제에서 소스 서버와 레플리카 서버 간의 데이터가 달라질 수 있다.

### 15-2-3. DECIMAL

`FLOAT`와 `DOUBLE`타입과 달리 고정 소수점 타입으로 지원한다.

`DECIMAL`은 저장하는 (숫자의 자릿수) / 2의 결괏값을 올림 처리한 만큼의 바이트 수를 필요로 한다.

### 15-2-5. 자동 증가(AUTO_INCREMENT) 옵션 사용

`AUTO_INCREMENT` 옵션을 사용한 칼럼은 반드시 그 테이블에서 프라이머리 키나 유니크 키의 일부로 정의해야 한다.

`auto_increment_increment` : 자동 증가 값의 증가 폭

`auto_increment_offset` : 자동 증가 값의 시작 값

**프라이머리 키나 유니크 키가 여러 개일 경우 각 엔진의 동작 차이**

- MyISAM의 경우, 자동 증가 컬럼이 프라이머리 키나 유니크 키 아무 위치에 사용될 수 있다.

- InnoDB의 경우, 자동 증가 칼럼으로 시작되는 인덱스(프라이머리 키나 일반 인덱스)를 생성해야한다.

`AUTO_INCREMENT` 칼럼은 테이블 당 하나만 사용할 수 있다.

스키마 복사 시 AUTO_INCREMENT의 초깃값에 주의하라.

## 15-3. 날짜와 시간

```sql
mysql> CREATE TABLE tb_datetime (current DATETIME(6));

mysql> INSERT INTO tb_datetime VALUES (NOW());  --// 밀리초 0으로 반환

mysql> INSERT INTO tb_datetime VALUES (NOW(6));  --// 밀리초 6자리까지 반환
```

`DATE`와 `DATETIME`은 칼럼 자체에 타임존 정보가 저장되지 않으므로 커넥션의 타임존과 관계없이 클라이언트로부터 입력된 값을 그대로 저장하고 조회할 때 변환 없이 출력한다.

`TIMESTAMP`는 항상 UTC 타임존으로 저장되므로 값이 자동으로 보정된다.

서버의 타임존(`system_time_zone`)을 변경해야 한다면 테이블 뿐만 아니라 DATETIME 타입 칼럼이 가지고 있는 값도 CONVERT_TZ() 같은 함수를 이용해 변환해야 한다.

### 15-3-1. 자동 업데이트

`DEFAULT CURRENT_TIMESTAMP` : 레코드가 INSERT될 때 시점을 자동으로 업데이트해주는 옵션

`ON UPDATE CURRENT_TIMESTAMP` : 해당 레코드가 UPDATE될 때의 시점을 자동으로 업데이트해주는 옵션

## 15-4. ENUM과 SET

### 15-4-1. ENUM

`ENUM 타입` : 테이블의 구조(메타 데이터)에 나열된 목록 중 하나의 값을 가질 수 있다. 코드화된 값을 관리하는 것이 가장 큰 목적.

```sql
--// enum 타입 생성
mysql> CREATE TABLE tb_enum (
    fd_enum ENUM('PROCESSING', 'FAILURE', 'SUCCESS') );

--// 숫자 값으로 조회
mysql> SELECT * FROM tb_enum WHERE fd_enum=1;

mysql> SELECT * FROM tb_enum WHERE fd_enum='PROCESSING';
```

문자열처럼 비교가 가능하지만 실제로 값을 디스크나 메모리에 저장할땐 매핑된 정숫값을 사용한다.

아이템 개수가 255개 미만이면 `ENUM`타입은 공간으로 1바이트만 사용하며, 그 이상은 2바이트까지 사용한다.

**단점** : 칼럼에 저장되는 새로운 문자열이 추가될 때마다 테이블의 구조를 변경해야 한다.

5.6버전부턴 이를 개선되어 새로 추가되는 아이템이 마지막에 추가된다면 테이블의 구조(메타데이터) 변경만으로 즉시 완료된다.

### 15-4-2. SET

`SET 타입` : 테이블의 구조(메타 데이터)에 정의된 아이템을 정숫값으로 매핑하며, 하나의 칼럼에 1개 이상의 값을 저장할 수 있다.

서버에서 내부적으로 `BIT-OR`연산을 거쳐 1개 이상의 선택된 값을 저장하며, 각 아이템 값에 매핑되는 정숫값은 1씩 증가하는 정숫값이 아닌 2^n의 값을 가진다.

아이템이 8개 이하라면 1바이트, 16개 이하라면 2바이트 사용하며 같은 방식으로 최대 8바이트까지 저장 공간을 사용한다.

```sql
--// set 타입 생성
mysql> CREATE TABLE tb_set (
    fd_set SET('TENNIS', 'SOCCER', 'GOLF', 'TABLE-TENNIS', 'BASKETBALL', 'BILLIARD')
);

mysql> INSERT INTO tb_set (fd_set) VALUES ('SOCCER'), ('GOLF, TENNIS');

--// set 타입 조회, 해당 종류의 쿼리들은 인덱스를 사용할 수 없다.
mysql> SELECT * FROM tb_set WHERE FIND_IN_SET('GOLF',fd_set);

mysql> SELECT * FROM tb_set WHERE fd_set LIKE '%GOLF%';

--// 동등 비교를 수행하려면 칼럼에 저장된 값과 동일한 순서로 값을 전달해야한다.
mysql> SELECT * FROM tb_set WHERE fd_set = 'TENNIS,GOLF';
```

## 15-5. TEXT와 BLOB

`TEXT타입` : 문자열을 저장하는 대용량 칼럼. 문자 집합과 콜레이션을 가짐.

`BLOB타입` : 이진 데이터 타입. 별도의 문자 집합이나 콜레이션을 가지지 않음.

**사용하는 상황**

- 칼럼 하나에 저장되는 문자열이나 이진 값의 길이가 예측할 수 없이 클 때

- 레코드의 전체 크기가 64KB를 넘어서서 더 큰 칼럼을 추가할 수 없을 때

BLOB과 TEXT 타입의 칼럼에 인덱스를 생성할 때는 칼럼값의 몇 바이트까지 인덱스를 생성할 것인지 명시해야 하며, 최대 제한 크기를 넘어서는 인덱스는 생성할 수 없다.

`tmp_mem_storage_engine` : 설정값이 "TempTable"일 때만 BLOB, TEXT 타입을 포함하는 결과도 메모리를 사용할 수 있다.

`max_allowed_packet` : 설정된 값보다 큰 SQL 문장은 서버로 전송되지 못하고 오류가 발생할 수 있다.

BLOB과 TEXT 타입의 칼럼은 가능한 레코드에 같이 포함해서 저장하려고 한다.

    - 레코드의 전체 길이가 최대 용량을 넘어설 경우, 용량이 큰 칼럼 순으로 외부 페이지로 옮겨서 레코드의 크기를 맞춘다.

    - `COMPACT`와 `REDUNDANT` 레코드 포맷은 외부 페이지에 저장된 TEXT나 BLOB 칼럼의 BLOB 프리픽스를 같이 키 페이지에 저장한다.

## 15-6. 공간 데이터 타입

OpenGIS에서 제공하는 WKT(Well Known Text) 또는 WKB(Well Known Binary)를 이용해 공간 데이터를 관리한다.

**공간 정보 저장용 데이터 타입**

- `POINT` : 하나의 점 정보

- `LINESTRING` : 하나의 라인

- `POLYGON` : 하나의 다각형

- `GEOMETRY` : `POINT`, `LINESTRING`, `POLYGON`의 수퍼 타입

- `MULTIPOINT`, `MULTILINESTRING`, `MULTIPOLYGON` : 종류별로 여러 개의 객체를 저장

- `GEOMETRYCOLLECTION` : `MULTIPOINT`, `MULTILINESTRING`, `MULTIPOLYGON`의 수퍼 타입

GEOMETRY 타입과 모든 자식 타입은 서버 메모리에서 BLOB 객체로 관리되고, 클라이언트로 전송될 때도 BLOB으로 전송된다.

JDBC에서는 공간 데이터를 지원하지 않기 때문에, 오픈소스 공간 데이터 라이브러리나 공간 함수(ST_AsText() 또는 ST_X(), ST_Y()함수 등)를 이용해 JDBC에서 지원하는 데이터 타입으로 변환 후 조회하는 방법도 생각해 봐야 한다.

### 15-6-1. 공간 데이터 생성

```sql
--// WKT 포맷 변환을 통한 생성
WKT 포맷 : POINT(x y)

객체 생성 : ST_PointFromText('POINT(x y)')

WKT 포맷 : LINESTRING(x0 y0, x1 y1, x2 y2, x3 y3, ...)

객체 생성 : ST_LineStringFromText('LINESTRING(x0 y0, x1 y1, x2 y2, x3 y3, ...)')

WKT 포맷 : POLYGON((x0 y0, x1 y1, x2 y2, x3 y3, x0 y0))

객체 생성 : ST_PolygonFromText('POLYGON((x0 y0, x1 y1, x2 y2, x3 y3, x0 y0))')
```

`FromText` 대신 `FromWKB`를 사용하면 WKT대신 WKB를 이용한 공간 데이터 객체를 생성할 수 있다.

SRID를 별도로 명시하지 않으면 기본값으로 0이 설정된다.

```sql
ST_PointFromText('POINT(10 20)')
ST_PointFromText('POINT(10 20)', 0)
```

### 15-6-2. 공간 데이터 조회

**데이터 조회 방법**

- 이진 데이터 조회(WKB 포맷 또는 MySQL 이진 포맷)

- 텍스트 데이터 조회(WKT 포맷)

- 공간 데이터의 속성 함수를 이용한 조회

```sql
mysql> SELECT id,
            location AS internal_format,    --// 서버 내부의 이진 데이터 그대로 조회
            ST_AsText(location) AS wkt_format,
            ST_AsBinary(location) AS wkb_format
        FROM plain_coord \G

--// 속성 함수
mysql> SET @poi:=ST_PointFromText('POINT(37.544738 127.039074)', 4326);

mysql> SELECT
        ST_SRID(@poi) AS srid,
        ST_X(@poi) AS coord_x,  --// 첫 번째 축 값
        ST_Y(@poi) AS coord_y,  --// 두 번째 축 값
        ST_Latitude(@poi) AS coord_latitude,    --// 첫 번째 축 값, 위도 경도를 사용할때만 사용 가능
        ST_Longitude(@poi) AS coord_longitude;  --// 두 번째 축 값, 위도 경도를 사용할때만 사용 가능
```

## 15-7. JSON 타입

`JSON` 타입의 칼럼은 JSON 데이터를 문자열로 저장하는 것이 아닌 바이너리 포맷의 `BSON`으로 변환해서 저장한다.

### 15-7-1. 저장 방식

MySQL 서버 내부적으로는 JSON 타입을 BLOB 타입에 저장한다.

JSON 데이터를 BSON타입으로 변환해서 저장하기 때문에, BLOB이나 TEXT타입의 칼럼에 저장하는 것보다 공간 효율이 높은 편이다.

```sql
mysql> CREATE TABLE tb_json (id INT, fd JSON);
mysql> INSERT INT tb_json VALUES
        (1, '{"user_id":1234567890}'),
        (2, '{"user_id":"1234567890"}');    --// 7바이트 공간 차이 발생

mysql> SELECT id, fd,
            JSON_TYPE(fd->"$.user_id") AS field_type,
            JSON_STORAGE_SIZE(fd) AS byte_size
        FROM tb_json;
```

JSON값은 이진 데이터 값으로 저장되는데 이는 24개의 필드로 구성되어있다.

각 필드는 저장되는 값의 특성에 맞게 1개 이상의 바이트를 차지한다.

매우 큰 JSON 토큐먼트는 16KB 단위로 데이터 페이지가 나뉘어 저장된다.

이 떄 나누어진 BLOB 페이지들은 인덱스를 통해 관리되어지고, 실제 BLOB 데이터를 가진 페이지들의 링크를 갖도록 개선됐다.

이런 인덱스를 통해 JSON필드의 부분 업데이트가 필요할 경우, 필요한 부분만 업데이트할 수 있게 됐다.

### 15-7-2. 부분 업데이트 성능

`JSON_SET()`, `JSON_REPLACE()`, `JSON_REMOVE()` 함수를 이용해 JSON 도큐먼트의 특정 필드 값을 변경하거나 삭제할 수 있다.

```sql
mysql> UPDATE tb_json
        SET fd=JSON_SET(fd, '$.user_id', "12345")
        WHERE id=2;
```

JSON 칼럼은 최대 4GB까지의 값을 가질 수 있다.

초기 할당됐던 공간으로 부족할 경우 부분 업데이트가 안된다.

부분 업데이트가 안된다면 모든 데이터 페이지를 다시 디스크에 기록해야된다.

`binlog_row_value_options`와 `binlog_row_image`의 설정값을 변경하면 JSON 칼럼의 부분 업데이트의 성능을 더 빠르게 만들 수 있다.

### 15-7-3. JSON 타입 콜레이션과 비교

JSON 칼럼에 저장되는 데이터와 가공되어 나온 결과값은 모두 `utf8bm4` 문자 집합과 `utf8mb4_bin` 콜레이션을 가진다.

바이너리 콜레이션이기 때문에 대소문자, 액센트 문자 등 구분해서 비교한다.

### 15-7-4. JSON 칼럼 선택

JSON 타입은 JSON 데이터를 이진 포맷으로 컴팩션해서 저장할 뿐만 아니라 부분 업데이트 기능, JSON 데이터 가공 등 여러 기능을 제공하기 때문에 JSON 데이터를 저장할 땐 TEXT, BLOB 칼럼보다 JSON 칼럼이 더 좋다.

## 15-8. 가상 칼럼(파생 칼럼)

```sql
--// 가상 칼럼
mysql> CREATE TABLE tb_virtual_column (
        id INT NOT NULL AUTO_INCREMENT,
        price DECIMAL(10,2) NOT NULL DEFAULT '0.00',
        quantity INT NOT NULL DEFAULT 1,
        total_price DECIMAL(10,2) AS (quantity * price) VIRTUAL, --// VIRTUAL 생략 가능
        PRIMARY KEY (id)
);

--// 스토어드 칼럼
mysql> CREATE TABLE tb_stored_column (
        id INT NOT NULL AUTO_INCREMENT,
        price DECIMAL(10,2) NOT NULL DEFAULT '0,00',
        quantity INT NOT NULL DEFAULT 1,
        total_price DECIMAL(10,2) AS (quantity * price) STORED,
        PRIMARY KEY (id)
);
```

가상 칼럼은 다른 칼러믜 값을 참조해서 계산된 값을 관리하기 때문에 항상 AS절 뒤에 계산식이나 데이터 가공을 위한 표현식을 정의한다.

가상 칼럼은 표현식은 입력이 동일하면 시점 관계없이 항상 동일한(DETERMINISTIC) 표현식만 사용할 수 있다.

`가상 칼럼`

    - 칼럼의 값이 디스크에 저장되지 않음
    - 칼럼의 구조 변경은 테이블 리빌드를 필요로 하지 않음
    - 칼럼의 값은 레코드가 읽히기 전 또는 BEFORE 트리거 실행 직후에 계산되어 만들어짐

`스토어드 칼럼`

    - 칼럼의 값이 물리적으로 디스크에 저장됨
    - 칼럼의 구조 변경은 다른 일반 테이블과 같이 필요 시 테이블 리빌드 방식으로 처리됨
    - INSERT와 UPDATE 시점에만 칼럼의 값이 계산됨

가상 칼럼에 인덱스를 생성하게 되면 해당 인덱스는 계산된 값을 저장하게 되며, 변경이 필요하다면 리빌드 작업도 필요하다.

함수 기반 인덱스는 가상 칼럼에 인덱스를 생성하는 방식으로 작동한다.

---
