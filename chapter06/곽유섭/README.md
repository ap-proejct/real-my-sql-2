# 📘 곽유섭 - Chapter 16 - 복제

> Real MySQL 8.0 2권 | Chapter 16 - 복제

---

## 📝 정리 내용

# 16. 복제

데이터베이스를 사용하고 운영할 때 가장 중요한 두 가지 요소는 확장성과 가용성이다.

확장 : 서비스에서 발생하는 대용량 트래픽을 안정적으로 처리하기 위해서

가용 : 사용자가 언제든지 안정적인 서비스를 이용하게 하기 위해서

이 두 요소를 위해 가장 일반적으로 사용되는 기술이 `복제`다.

## 16-1. 개요

`복제` : 한 서버에서 다른 서버로 데이터가 동기화되는 것

    `소스 서버` : 원본 데이터를 가진 서버

    `레플리카 서버` : 복제된 데이터를 가지는 서버

    소스 서버에서 데이터 및 스키마에 대한 변경이 최초로 발생하며, 레플리카 서버에서는 이러한 변경 내역을 소스 서버로부터 전달받아 소스 서버에 저장된 데이터와 동기화시킨다.

**레플리카 서버를 구축하는 목적**

1. 스케일 아웃(Scale-out)

   서비스를 운영하다 보면 사용자가 늘어나 DB 서버로 유입되는 트래픽이 자연히 증가해 부하가 높아진다.

   스케일 업을 통해 조치를 취할 수 있지만, 서버 한 대가 처리할 수 있는 양에는 한계가 있다.

   동일한 데이터를 가진 DB 서버를 한 대 이상 늘리면 애플리케이션으로부터 실행되는 쿼리들을 분산할 수 있으며, 이를 스케일 아웃이라 한다.

   스케일 아웃은 스케일 업보다 갑자기 늘어나는 트래픽을 대응하는 데 더 유연하며, 복제를 사용해 DB 서버를 스케일 아웃할 수 있다.

2. 데이터 백업

   DBMS는 보통 백업 프로그램을 실행해 백업을 진행한다.

   동일한 서버에서 백업이 실행되는 경우 서버의 자원을 공유해서 사용하기 떄문에 백업으로 인해 실행 중인 쿼리들이 영향을 받을 수 있다.

   이를 방지하기 위해 복제를 사용해 레플리카 서버를 구축하고, 데이터 백업은 레플리카 서버에서 진행한다.

   이렇게 구축된 백업용 레플리카 서버는 소스 서버가 문제 생겼을 때를 대비한 대체 서버 역할을 하기도 한다.

3. 데이터 분석

   DB 서버에서는 가끔 서비스 발전을 위해 분석용 쿼리를 실행하며, 이러한 분석용 쿼리는 대량의 데이터를 조회하는 경우가 많아 리소슬르 많이 사용하게 된다.

   이로 인해 서비스에서 직접적으로 사용되는 다른 쿼리들이 영향을 받을 수 있으므로 복제를 사용해 여분의 레플리카 서버를 구축해 분석용 쿼리만 전용으로 실행될 수 있는 환경을 만드는 것이 좋다.

4. 데이터의 지리적 분산

   DB 서버와 애플리케이션 서버가 서로 떨어져 있는 경우 두 서버 간의 통신 시간은 떨어진 거리만큼 비례해서 늘어난다.

   만약 떨어져 있는 DB 서버의 위치를 이동시키지 못한다면 복제를 사용해 애플리케이션 서버가 위치한 곳에 DB 서버에 대한 레플리카 서버를 새로 구축해 응답 속도를 개선할 수 있다.

## 16-2. 복제 아키텍처

`바이너리 로그` :

    MySQL 서버에서 발생하는 모든 변경 사항을 기록하는 로그

    데이터의 변경 내역뿐만 아니라 데이터베이스나 테이블의 구조 변경과 계정, 권한의 변경 정보까지 모두 저장된다.

    `이벤트` : 바이너리 로그에 기록된 각 변경 정보

복제는 이 바이너리 로그를 기반으로 구현됐으며, 소스 서버에서 생성된 바이너리 로그가 레플리카 서버로 전송되고 레플리카 서버에서는 해당 내용을 로컬 디스크에 저장한 뒤 자신이 가진 데이터에 반영함으로써 동기화를 진행한다.

`릴레이 로그` : 레플리카 서버에서 소스 서버의 바이너리 로그를 읽어 들여 따로 로컬 디스크에 저장해둔 파일

**복제에 사용되는 세 개의 스레드**

    바이너리 로그 덤프 스레드 :

        소스 서버에서 레플리카 서버가 연결될 때 내부적으로 바이너리 로그 덤프 스레드를 생성해서 바이너리 로그의 내용을 레플리카 서버에 전송

    레플리케이션 I/O 스레드 :

        소스 서버의 바이너리 로그 덤프 스레드로부터 바이너리 로그 이벤트를 가져와 로컬 서버의 파일(릴레이 로그)로 저장하는 역할

    레플리케이션 SQL 스레드 :

        I/O 스레드에 의해 작성된 릴레이 로그 파일의 이벤트들을 읽고 실행하는 역할

레플리카 서버에서 소스 서버의 변경 사항들이 적용되는 것은 소스 서버가 동작하는 것과는 별개로 진행되므로 레플리카 서버에 문제가 생겨도 소스 서버는 전혀 영향을 받지 않는다.

반대로 소스 서버에 문제가 생겨 I/O 스레드가 정상적으로 동작하지 않는다면 복제가 중단된다. 이는 복제만 중단된 것이므로 레플리카가 쿼리를 처리하는 데는 문제가 없다.

**복제 시 레플리카 서버에서 생성, 관리하는 데이터**

    릴레이 로그 :

        소스 서버의 바이너리 로그에서 읽어온 이벤트 정보를 저장

        현재 존재하는 릴레이 로그 파일들의 목록이 담긴 인덱스 파일, 실제 이벤트 정보가 저장돼 있는 로그 파일들로 구성

    커넥션 메타데이터 :

        I/O 스레드에서 소스 서버에 연결할 때 사용하는 DB 계정 정보 및 현재 읽고 있는 소스 서버의 바이너리 파일명과 파일 내 위치 값 등이 담겨 있다.

        기본적으로 mysql.slave_master_info 테이블에 저장

    어플라이어 메타데이터 :

        SQL 스레드에서 릴레이 로그에 저장된 소스 서버의 이벤트들을 레플리카에 적용하는 컴포넌트를 "어플라이어"라고 한다.

        최근 적용된 이벤트에 대해 해당 이벤트가 저장돼 있는 릴레이 로그 파일명과 파일 내 위치 정보 등을 담고 있다.

        기본적으로 mysql.slave_relay_log_info 테이블에 저장

## 16-3. 복제 타입

### 16-3-1. 바이너리 로그 파일 위치 기반 복제

레플리카 서버에서 소스 서버의 바이너리 로그 파일명과 파일 내에서의 위치(Offset / Postion)로 개별 바이너리 로그 이벤트를 식별해서 복제가 진행되는 형태

바이너리 로그 파일 위치 기반 복제에서는 이벤트 하나하나를 소스 서버의 바이너리 로그 파일명과 파일 내에서의 위치 값(File Offset)의 조합으로 식별한다.

레플라키 서버에서 각 이벤트들을 식벼랗고 자신의 적용 내역을 추적함으로써 복제를 일시적으로 중단하거나 재개할 때도 마지막으로 적용한 이벤트 이후를 읽어올 수 있다.

복제에 참여한 MySQL 서버들이 모두 고유한 `server_id` 값을 가지고 있어야 한다.

바이너리 로그에 기록된 이벤트가 레플리카에 설정된 `server_id` 값과 동일한 `server_id`값을 가지는 경우 레플리카 서버는 자신의 서버에서 발생한 이벤트로 간주하고 무시한다.

#### 16-3-1-1. 바이너리 로그 파일 위치 기반의 복제 구축

##### 16-3-1-1-1. 설정 준비

- 소스 서버에서 반드시 바이너리 로그 활성화

- 복제 구성원이 되는 각 MySQL 서버가 고유한 `server_id` 값을 가져야 한다.

- 8.0부터 `binlog` 이름으로 바이너리 로그 파일이 자동으로 생성

- `server_id` 기본값은 1, 그러나 고유한 값을 가져야함으로 변경해주는게 좋다.

- `log_bin` 설정 시 바이너리 로그 파일 위치나 파일명을 따로 설정 가능

```
## 소스 서버 설정

[mysqld]
server_id=1
log_bin=/binary-log-dir-path/binary_log_name
sync_binlog=1
binlog_cache_size=5M
max_binlog_size=512M
binlog_expire_logs_seconds=1209600
...
```

- `SHOW MASTER STATUS` 명령을 통해 소스 서버에 바이너리 로그가 정상적으로 기록되고 있는지 확인 가능

- 레플리카 서버도 고유한 `server_id`를 설정해야 하며, 복제 설정 시 자동으로 릴레이 로그 파일이 데이터 디렉터리 밑에 생성된다.

- `relay_log` 변수를 통해 로그 파일 위치나 파일명을 따로 설정할 수 있다.

- 필요없어진 릴레이 로그 파일은 자동으로 삭제되며, 유지하고 싶다면 `relay_log_purge`를 OFF로 설정하면 된다.

- 레플리카 서버는 일반적으로 읽기 전용으로 사용되므로 `read_only` 설정을 하면 좋다.

- 추후 소스 서버의 장애로 레플리카 서버가 소스 서버로 승격될 수 있음을 고려하면 `log_slave_updates` 변수도 명시하는 게 좋다.
  - `log_slave_updates` 설정 시 복제의 의한 데이터 변경 내용도 바이너리 로그에 저장한다.

```
## 레플리카 서버 설정
[mysqld]
server_id=2
relay_log=/relay-log-dir-path/relay-log-name
relay_log_purge=ON
read_only
log_slave_updates
...
```

##### 16-3-1-1-2. 복제 계정 준비

**복제용 계정**

    레플리카 서버가 소스 서버로부터 바이너리 로그를 가져오려면 소스 서버에 접속해야 하므로 접속 시 사용할 DB 계정 필요

    복제에 사용되는 계정의 비밀번호가 레플리카 서버의 커넥션 메타데이터에 평문으로 저장되므로 보안 측면을 고려해 복제에 사용되는 권한만 주어진 별도의 계정을 생성해 사용하는 것이 좋다.

```sql
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'repl_user_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';  --// 복제 권한 부여
```

##### 16-3-1-1-3. 데이터 복사

소스 서버의 데이터를 레플리카 서버로 가져와 적재할때, 백업이나 mysqldump 툴을 이용해 복사한다.

소스 서버의 데이터를 덤프할 때는 "--single-transaction"과 "--master-data" 두 옵션을 반드시 사용해야 한다.

    "--single-transaction" :

        데이터를 덤프할 때 하나의 트랜잭션을 사용해 덤프가 진행되게 해서 테이블이나 레코드에 잠금을 걸지 않고 InnoDB 테이블들에 대해 일관된 데이터를 덤프받을 수 있게 한다.

    "--master-data" :

        덤프 시작 시점의 소스 서버의 바이너리 로그 파일명과 위치 정보를 포함하는 복제 설정 구문(CHANGE REPLICATION SOURCE TO / CHANGE MASTER TO)이 덤프 파일 헤더에 기록될 수 있게 하는 옵션

        옵션 사용 시 mysqldump에서 글로벌 락(모든 테이블에 대한 읽기 잠금)을 건다.

        1 설정 시, 복제 설정 구문이 실행 가능한 형태로 저장

        2 설정 시, 주석처리되어 참조만 할 수 있는 형태로 저장

```
linux> mysqldump --uroot -p --single-transaction --master-date=2 \
        --opt --routines --triggers --hex-blob --all-databases > source_data.sql
```

덤프 완료 시 `source_data.sql`파일을 레플리카 서버로 옮겨 데이터 적재를 진행한다.

```
--// MySQL 서버에 직접 접소개 데이터 적재 명령을 실행
mysql> SOURCE /tmp/master_data.sql

## MySQL 서버에 로그인하지 않고 데이터 적재 명령 실행
## 다음 두 명령어 중 하나 사용
linux> mysql -uroot -p < /tmp/source_data.sql
linux> cat /tmp/source_data.sql | mysql -uroot -p
```

##### 16-3-1-1-4. 복제 시작

`CHANGE REPLICATION SOURCE TO(CHANGE MASTER TO)` : 복제를 설정하는 명령으로, mysqldump로 백업 받은 파일의 헤더 부분에서 해당 명령어를 참조할 수 있다.

```
linux> less /tmp/source_data.sql
...

-- CHANGE MASTER TO MASTER_LOG_FILE='binary-log.000002', MASTER_LOG_POS=2708;

--// MySQL 8.0.23 이상
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST='source_server_host',
    SOURCE_PORT=3306,
    SOURCE_USER='repl_user',
    SOURCE_PASSWORD='repl_user_password',
    SOURCE_LOG_FILE='binary-log.000002',
    SOURCE_LOG_POS=2708,
    GET_SOURCE_PUBLIC_KEY=1;    --// RSA 키 기반 비밀번호 교환 방식의 통신을 위해 공개키를 소스 서버에 요청할 것인지 여부

--// MySQL 8.0.23 미만
CHANGE MASTER TO
    MASTER_HOST='source_server_host',
    MASTER_PORT=3306,
    MASTER_USER='repl_user',
    MASTER_PASSWORD='repl_user_password',
    MASTER_LOG_FILE='binary-log.000002',
    MASTER_LOG_POS=2708,
    GET_MASTER_PUBLIC_KEY=1;
```

`SHOW REPLICA STATUS / SHOW SLAVE STATUS` : 복제 관련 정보가 레플리카 서버에 등록돼 있는 것을 확인하는 명령어

    Replica_IO_Running, Replica_SQL_Running 칼럼이 "No"로 돼 있다면 복제 관련 정보만 등록되고 동기화는 시작되지 않음을 나타냄.

    START REPLICA / START SLAVE : 복제 동기화 시작 명령어

#### 16-3-1-2. 바이너리 로그 파일 위치 기반의 복제에서 트랜잭션 건너뛰기

레플리카 서버에서 소스 서버로부터 넘어 온 트랜잭션이 제대로 실행되지 못하고 에러가 발생해 복제가 멈출 때도 있다. (ex : 중복 키 에러)

수동으로 복구가 불가능할 경우, 레플리카 서버의 데이터를 버린 후 다시 구축한 뒤 복제할 수도 있지만, 경우에 따라 레플리카 서버에서 문제되는 소스 서버의 트랜잭션을 무시하고 넘어가도록 처리해도 괜찮을 때가 있다.

`sql_slave_skip_counter` : 바이너리 로그 위치 기반 복제에서 문제되는 트랜잭션을 넘길 때 사용하는 변수

    값을 1로 지정해 레플리케이션 SQL 스레드를 재시작하면 에러가 발생하는 쿼리를 건너뛰고 정상적으로 복제를 재개한다.

    1로 설정되면 실제 DML 쿼리 문장 하나를 가지 이벤트 1개를 무시하는 것이 아니라 현재 이벤트를 포함한 이벤트 그룹을 무시하는 것이다.

    트랜잭션을 지원하는 테이블의 경우 트랜잭션이 하나의 이벤트 그룹이다.

```sql
--// MySQL 8.0.22 미만
mysql_Replica> STOP SLAVE SQL_THREAD;
mysql_Replica> SET GLOBAL sql_slave_skip_counter=1;
mysql_Replica> START SLAVE SQL_THREAD;

--// MySQL 8.0.22 이상
mysql_Replica> STOP REPLICA SQL_THREAD;
mysql_Replica> SET GLOBAL sql_slave_skip_counter=1;
mysql_Replica> START REPLICA SQL_THREAD;
```

### 16-3-2. 글로벌 트랜잭션 아이디(GTID) 기반 복제

복제에서 각각의 이벤트들이 바이너리 로그 파일명과 파일 내 위치 값의 조합으로 식별되는 것.

    이는 바이너리 로그 파일이 저장돼 있는 소스 서버에서만 유효함.

    동일한 이벤트가 레플리카 서버에서 동일한 파일명의 동일한 위치에 저장된다는 보장이 없다.

    복제에 투입된 서버들마다 동일한 이벤트에 대해 서로 다른 식별 값을 갖게된다.

    서로 호환되지 않는 정보를 이용해 복제를 진행함으로써 복제의 토폴로지를 변경하느 작업이 불가능할 때가 많았다.

동일한 고유 식별값을 가진다면, 장애가 발생해도 쉽게 복제 토폴로지를 변경할 수 있으며, 장애 복구에 소요되는 시간도 줄어들게 된다.

`글로벌 트랜잭션 아이디(GTID)` : 소스 서버에서만 유효한 고유 식별 값이 아닌 복제에 참여한 전체 MySQL 서버들에서 고유하도록 각 이벤트에 부여된 식별 값

#### 16-3-2-1. GTID의 필요성

**장애 상황**

소스 서버 A와 레플리카 서버 B(SELECT 쿼리 분산용), 레플리카 서버 C(배치나 통계용)이 있다고 가정해보자.

    A에서 장애가 발생해 B를 소스 서버로 승격하면, 복제는 끊어지고 B 서버로 트래픽이 유입된다.

    B 서버가 새로운 소스 서버로 승격되면서 SELECT 쿼리 처리와 기본 서비스 서버의 역할이 겹쳐 과부하 상태가 될 것이다.

    C 서버는 동기화되어 있지 않기 때문에 C를 SELECT 용도로 사용할 수 없다.

    물론 레플리카 B의 릴레이 로그가 남아있다면 그 로그를 가져와서 필요한 부분만 실행해 복구가 가능하다.

    그러나 릴레이 로그는 불필요한 시점에 자동으로 삭제되고, 수동으로 확인하는 방법도 쉽지 않다.

**글로벌 트랜잭션 아이디로 복제하는 상황**

현재 GTID는 '120'이고 B는 '120', C는 '98'까지 동기화된 상태라고 가정해보자.

    A에서 장애 발생 시 B 서버를 C 서버의 소스 서버가 되도록 `CHANGE REPLICATION SOURCE TO SOURCE_HOST='B', SOURCE_PORT=3306;` 명령을 실행한다.

    이때 B 서버에서 아무것도 입력할 필요가 없으며, 자동으로 '98' 이후의 바이너리 로그 이벤트를 가져와서 동기화한다.

GTID를 통해 복제 토폴로지 변경 뿐만 아니라 레플리카 서버 확장, 축소 또는 통합과 같은 여러 문제를 해결할 수 있다.

#### 16-3-2-2. 글로벌 트랜잭션 아이디

GTID는 서버에서 커밋된 각 트랜잭션과 연결된 고유 식별자로, 해당 트랜잭션이 발생한 서버에서 고유할뿐만 아니라 그 서버가 속한 복제 토폴로지 내 모든 서버에서 고유하다.

     GTID는 커밋되어 바이너리 로그에 기록된 트랜잭션에 한해서만 할당된다.

     데이터 읽기만 수행한느 SELECT 쿼리나 sql_log_bin 설정이 비활성화돼 있는 상태에서 발생한 트랜잭션은 바이너리에 기록되지 않으므로 GTID가 할당되지 않는다.

```
GTID = [source_id]:[transaction_id]
```

`source_id` : 트랜잭션이 발생된 소스 서버를 식별하기 위한 값.

    MySQL 서버의 server_uuid 값을 사용

    server_uuid는 "auto.cnf"파일의 "[auto]"라는 섹션에 있다.

`transaction_id` : 서버에서 커밋된 트랜잭션 순서대로 부여되는 값으로 1씩 단조 증가하는 형태로 발급된다.

**GTID 값 확인 방법**

```
mysql> SELECT * FROM mysql.gtid_executed;

mysql> SHOW GLOBAL VARIABLES 'gtid_executed';

mysql> SHOW MASTER STATUS \G
```

`GTID 셋` : 하나 이상의 GTID 값으로 구성돼 있는 것

    기본적으로 동일한 서버에서 생성된 연속하는 GTID 값은 축소시켜 범위로 보여지며, 범위 값과 단일 값이 하나의 표현식으로 나타날 수 있다.

    서로 다른 UUID를 가지는 GTID 값도 포함될 수있는데 UUID 값이 변경됐거나 여러 서버에서 데이터를 복제해오는 경우 등이 있다.

MySQL 서버는 주기적으로 mysql.gtid_executed 테이블에 쌓여있는 데이터를 하나로 압축하며, 이는 여러 레코드를 연속된 것들끼리 모아 1건의 레코드로 만드는 것이다.

    압축은 바이너리 로그 활성화 여부에 따라 수행하는 조건이 달라진다.

        활성화 : 바이너리 로그 파일이 로테이션될 때 자동으로 압축이 수행된다.

        비활성화 : "thread/sql/compress_gtid_table"이라는 포그라운드 스레드에 의해 수행되며, 서버에서 실행된 트랜잭션 수가 "gtid_executed_compression_period" 변수에 지정된 수까지 도달하면 스레드에서 압축을 수행하고, 다음 주기까지 슬립 모드를 유지한다.

            값이 0일 경우, 필요에 따라 자동으로 압축이 실행된다.

#### 16-3-2-3. 글로벌 트랜잭션 아이디 기반의 복제 구축

소스 서버에서 GTID 복제를 하기 위해선 GTID를 활성화해야한다.

##### 16-3-2-3-1. 설정 준비

GTID 기반의 복제를 하려면 복제에 참여하는 모든 MySQL 서버들의 GTID가 활성화돼 있어야 하며, `server_id`와 `server_uuid` 복제 그룹 내에서 고유해야 한다.

```
## 소스 서버 설정
[mysqld]
gtid_mode=ON                    --// gtid 복제 시 필수
enforce_gtid_consistency=ON     --// gtid 복제 시 필수
server_id=1111
log_bin=/binary-log-dir-path/binary-log-name

## 레플리카 서버 설정
[mysqld]
gtid_mode=ON
enforce_gtid_consistency=ON
server_id=2222
relay_log=/relay-log-dir-path/relay-log-name
relay_log_purge=ON
read_only
log_slave_updates
```

##### 16-3-2-3-2. 복제 계정 준비

```sql
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'repl_user_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
```

##### 16-3-2-3-3. 데이터 복사

```
--// mysqldump를 통한 복사
linux> mysqldump -uroot -p --single-transaction --master-data=2 --set-gtid-purged=ON \
        --opt --routines --triggers --hex-blob --all-databases > source_data.sql
```

GTID가 활성화된 소스 서버에서 mysqldump로 데이터를 덤프받아 레플리카 서버를 구축하려는 경우, 덤프가 시작된 시점의 소스 서버 GTID 값을 레플리카 서버에서 2개의 시스템 변수를 설정해야 복제를 시작할 수 있다.

    - gtid_executed : MySQL 서버에서 실행되어 바이너리 로그 파일에 기록된 모든 트랜잭션들의 GTID 셋

    - gtid_purged : 현재 MySQL 서버의 바이너리 로그 파일에 존재하지 않는 모든 트랜잭션들의 GTID 셋

GTID 기반 복제에서 레플리카 서버는 `gtid_executed` 값을 기반으로 다음 복제 이벤트를 소스 서버로부터 가져온다.

복제를 시작하기 위해서는 소스 서버에서 데이터 덤프가 시작된 시점의 소스 서버의 GTID 값을 레플리카 서버의 `gtid_purged` 시스템 변수에 지정해 `gtid_executed` 시스템 변수에도 그 값이 설정되게 해야한다.

`--set-gtid-purged` : 옵션 활성화 시 덤프가 시작된 시점의 GTID가 덤프 파일에 기록된다.

    sql_log_bin 시스템 변수를 비활성화하는 구문도 함께 기록되며, 레플리카 서버에서 덤프 파일을 적재하는 작업이 바이너리 로그에 기록되지 않아 GTID가 생성되지 않는다.

| 옵션      | 설명                                                                                                                                                                                              |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AUTO      | 덤프를 받는 서버에서 GTID가 활성화돼 있으면 덤프를 시작하는 시점의 GTID 값 및 sql_log_bin 비활성화 구문을 덤프 파일에 기록하며, 만약 GTID가 비활성상태인 서버의 경우 기록하지 않는다.             |
| OFF       | 덤프 시작 시점이 GTID 값 및 sql_log_bin 비활성화 구문을 덤프 파일에 기록하지 않는다.                                                                                                              |
| ON        | 덤프 시작 시점이 GTID 값 및 sql_log_bin 비활성화 구문을 덤프 파일에 기록한다. 만약 GTID가 활성화돼 있지 않은 서버에서 이 옵션갑을 사용하는 경우 에러가 발생한다.                                  |
| COMMENTED | 8.0.17 이상부터 사용할 수 있는 값, 이 값이 설정되면 ON 값이랑 동일하게 동작하되, 덤프 시작 시점이 GTID 값이 주석으로 처리되어 기록된다. 비활성화 구문은 주석으로 처리되지 않고 동일하게 기록된다. |

> 참고 : 단순 데이터 마이그레이션 용도로 mysqldump를 사용하는 경우 "OFF" 옵션을 명시해 비활성화 구문이 덤프 파일에 기록되지 않도록 해야 한다. 그렇지 않으면 덤프 파일 적용 시 비활성화 구문으로 인해 적재한 데이터가 바이너리 로그에 기록되지 않아, 해당 DB 서버와 연결된 레플리카 서버에 데이터가 복제가 안될수 있다.

```
linux> less /tmp/source_data.sql

...

--// '+' 기호는 혀재 gtid_purged 시스템 변수에 설정돼 있는 값에 새로운 값을 덧붙이는 것
SET @@GLOBAL.GTID_PURGED=/*!80000 '+'*/ 'ed22da00-e052-11ea-ae88-ee4baf89a396:1-30';
```

```sql
--// 복제한 데이터를 레플리카에 적재하는 명령
mysql_Replica> SOURCE /tmp/source_data.sql;
```

```
--// XtraBackup 툴을 사용해 데이터를 백업받아 복구하는 경우 생성되는 파일
linux> cat xtrabackup_binlog_info
mysql-bin.000003 8886 ed22da00-e052-11ea-ae88-ee4baf89a396:1-30
```

해당 파일의 값은 복구 시 mysql.gtid_executed 테이블이 GTID값이 된다.

##### 16-3-2-3-4. 복제 시작

**소스 서버와 레플리카 서버 간의 복제를 시작하는 명령**

```
--// MySQL 8.0.23 이상
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST='source_server_host',
    SOURCE_PORT=3306,
    SOURCE_USER='repl_user',
    SOURCE_PASSWORD='repl_user_password',
    SOURCE_AUTO_POSITION=1,
    GET_SOURCE_PUBLIC_KEY=1;

--// MySQL 8.0.23 미만
CHANGE MASTER TO
    MASTER_HOST='source_server_host',
    MASTER_PORT=3306,
    MASTER_USER='repl_user',
    MASTER_PASSWORD='repl_user_password',
    MASTER_AUTO_POSITION=1,
    GET_MASTER_PUBLIC_KEY=1;
```

`SOURCE_AUTO_POSITION` : 바이너리 로그 파일 위치 기반 복제와의 차이점으로, 해당 옵션으로 레플리카 서버는 자신의 `gtid_executed` 값을 참조해 해당 시점부터 소스 서버와 복제를 연결해서 데이터를 동기화하게 된다.

#### 16-3-2-4. 글로벌 트랜잭션 아이디 기반 복제에서 트랜잭션 건너뛰기

GTID를 사용하는 복제 환경의 레플리카 서버에선 `sql_slave_skip_counter` 변수를 사용해 이벤트 그룹을 건너뛸 수 없다.

만약 레플리카 서버에서 소스 서버로부터 넘어온 트랜잭션을 무시하고 싶다면 레플리카 서버에서 수동으로 빈 트랜잭션을 생성해 GTID 값을 만들어야 한다.

```sql
--// 복제 중단
mysql_Replica> STOP REPLICA;

--// gtid_next 값을 문제가 발생한 트랜잭션의 GTID 값으로 설정
mysql_Replica> SET gtid_next='af995d80-939e-11eb-bb37-ba122a9a8ae3:7';

--// 아무런 DML도 없는 빈 트랜잭션을 생성
mysql_Replica> BEGIN; COMMIT;

--// gtid_next 변수 값이 자동으로 초기화될 수 있도록 설정
mysql_Replica> SET gtid_next='AUTOMATIC';

--// 복제 시작
mysql_Replica> START REPLICA;
```

#### 16-3-2-5. Non_GTID 기반 복제에서 GTID 기반 복제로 온라인 변경

8.0에서 서비스가 현재 동작하고 있는 상태에서 MySQL 서버가 GTID를 사용하도록 혹은 사용하지 않도록 GTID 모드를 온라인으로 전환할 수 있는 기능을 제공한다.

    기존의 바이너리 로그 위치 기반의 복제를 GTID 기반의 복제로 변경할 수 있으며, 그 반대로 가능하다.

GTID 모드를 전환하는 작업은 GTID와 관련된 두 시스템 변수의 값만 순차적으로 변경하면 된다.

`enforce_gtid_consistency` : GTID 기반의 복제에서 소스 서버와 레플리카 서버 간의 데이터 일관성을 해칠 수 있는 쿼리들이 MySQL 서버에서 실행되는 것을 허용할지를 제어하는 변수.

**일관성을 해칠 수 있는 쿼리**

- 트랜잭션을 지원하는 테이블과 지원하지 않는 테이블을 함께 변경하는 쿼리 혹은 트랜잭션

- CREATE TABLE ... SELECT ... 구문

- 트랜잭션 내에서 CREATE TEMPORARY TABLE, DROP TEMPORARY TABLE 구문 사용

GTID가 트랜잭션 단위로 올바르게 할당돼야 복제가 정상적으로 동작하는 데 이 같은 쿼리들은 GTID 기반의 복제에서 문제가 될 수있기 때문에 설정을 통해 실행 가능 여부를 제어할 수 있다.

> 참고 : 8.0.13부터 바이너리 로그 포맷이 ROW, MIXED일 경우 CREATE TEMPORARY TABLE 및 DROP TEMPORARY TABLE 구문을 사용할 수 있다.
>
> 8.0.21부턴 Atomic DDL 기능을 지원하는 InnoDB 스토리지 엔진 테이블에 한해 CREATE TABLE ... SELECT 구문을 사용할 수 있다.

| 옵션 | 설명                                                                                   |
| ---- | -------------------------------------------------------------------------------------- |
| OFF  | GTID 일관성을 해칠 수 있는 쿼리들을 허용                                               |
| ON   | GTID 일관성을 해칠 수 있는 쿼리들을 허용하지 않음(GTID가 활성화 된 경우 설정)          |
| WARN | GTID 일관성을 해칠 수 있는 쿼리들을 허용하지만 쿼리들이 실행될 때 경고 메시지가 발생함 |

`gtid_mode` : 바이너리 로그에 트랜잭션들이 GTID 기반으로 로깅될 수 있는지 여부와 트랜잭션 유형별로 MySQL 서버에서의 처리 가능 여부를 제어한다.

    바이너리 로그에 기록되는 유형에는 익명 트랜잭션과 GTID 트랜잭션이 있다.

        익명 트랜잭션은 GTID가 부여되지 않은 트랜잭션으로 바이너리 로그 파일명과 위치로 식별된다.

        GTID 트랜잭션은 고유한 식별값이 GTID가 부여된 트랜잭션을 지칭한다.

|                | 신규 트랜잭션 | 복제된 트랜잭션 |
| -------------- | ------------- | --------------- |
| OFF            | 익명 트랜잭션 | 익명 트랜잭션   |
| OFF_PERMISSIVE | 익명 트랜잭션 | 모두 처리 가능  |
| ON_PERMISSIVE  | GTID 트랜잭션 | 모두 처리 가능  |
| ON             | GTID 트랜잭션 | GTID 트랜잭션   |

위 표의 적혀진 값 순서를 기주으로 한 번에 한 단계씩만 변경할 수 있다.

\*\* gtid_mode별 소스 서버와 레플리카 서버 간 복제 가능 여부 및 자동 포지션(SOURCE_AUTO_POSITION) 사용 가능 여부

- O : 복제 가능
- X : 복제 불가능
- A : 복제 설정 시 자동 포지션 옵션 사용 가능

|                              | 소스 서버 OFF | 소스 서버 OFF_PERMISSIVE | 소스 서버 ON_PERMISSIVE | 소스 서버 ON |
| ---------------------------- | ------------- | ------------------------ | ----------------------- | ------------ |
| 레플리카 서버 OFF            | O             | O                        | X                       | X            |
| 레플리카 서버 OFF_PERMISSIVE | O             | O                        | O                       | O + A        |
| 레플리카 서버 ON_PERMISSIVE  | O             | O                        | O                       | O + A        |
| 레플리카 서버 ON             | X             | X                        | O                       | O + A        |

**전환 순서**

1. 각 서버에서 `enforce_gtid_consistency` 시스템 변수 값을 WARN으로 변경

   ```sql
   mysql> SET GLOBAL enforce_gtid_consistency = WARN;
   ```

   GTID 사용 시 일관성을 해치는 트랜잭션들을 감지해 에러 로그에 경고 메시지를 남기므로, 모니터링을 위해 설정한다.

2. 각 서버에서 `enforce_gtid_consistency` 시스템 변수 값을 ON으로 변경

   ```sql
   mysql> SET GLOBAL enforce_gtid_consistency = ON;
   ```

3. 각 서버에서 gtid_mode 시스템 변수 값을 OFF_PERMISSIVE로 변경

   ```sql
   mysql> SET GLOBAL gtid_mode = OFF_PERMISSIVE;
   ```

4. 각 서버에서 gtid_mode 시스템 변수 값을 ON_PERMISSIVE로 변경

   ```sql
   mysql> SET GLOBAL gtid_mode = ON_PERMISSIVE;
   ```

5. 잔여 익명 트랜잭션 확인

   ```sql
   mysql> SHOW GLOBAL STATUS LIKE 'Ongoing_anonymous_transaction_count';
   ```

6. 각 서버에서 gtid_mode 시스템 변수 값을 ON으로 변경

   ```sql
   mysql> SET GLOBAL gtid_mode = ON;
   ```

7. my.cnf 파일 변경

   ```
   [mysqld]
   gtid_mode=ON
   enforce_gtid_consistency=ON
   ```

   재시작 시 해당 설정들이 유지될 수 있도록 my.cnf 파일 설정

8. GTID 기반 복제를 사용하도록 복제 설정을 변경

   ```sql
   --// 해당 명령 실행 시 GTID 기반의 복제로 설정
   mysql> STOP REPLICA;
   mysql> CHANGE REPLICATION SOURCE TO SOURCE_AUTO_POSITION=1;
   mysql> START REPLICA;
   ```

GTID를 비활성화하는 작업은 역순으로 진행하면 된다.

```sql
--// 1. 바이너리 로그 위치 기반 복제로 변경
mysql> STOP REPLICA;
mysql> CHANGE REPLICATION SOURCE TO
        SOURCE_LOG_FILE='xxxxx', SOURCE_LOG_POS=xxxxx,
        SOURCE_AUTO_POSITION=0;
mysql> START REPLICA;

--// 2. 모든 서버의 gtid_mode를 ON_PERMISSIVE로 변경(적용 순서 무관)
mysql> SET GLOBAL gtid_mode = ON_PERMISSIVE;

--// 3. 모든 서버의 gtid_mode를 OFF_PERMISSIVE로 변경(적용 순서 무관)
mysql> SET GLOBAL gtdi_mode = OFF_PERMISSIVE;

--// 4. 모든 서버에서 잔여 GTID 트랜잭션 확인
mysql> SHOW GLOBAL VARIABLES LIKE 'gtid_owned';

--// 5. 모든 서버의 gtid_mode를 OFF로 변경
mysql> SET GLOBAL gtid_mode = OFF;
--// 필요 시 enforce_gtid_consistency도 OFF로 설정
mysql> SET GLOBAL enforce_gtid_consistency = OFF;

--// 6. my.cnf 파일 변경
[mysqld]
gtid_mode=OFF
enforce_gtid_consistency=OFF
```

#### 16-3-2-6. GTID 기반 복제 제약 사항

- GITD가 활성화된 MySQL에선 "enforce_gtid_consistency=ON"옵션으로 인해 GTID 일관성을 해칠 수 있는 일부 유형의 쿼리들은 실행할 수 없다.

- GTID 기반 복제가 설정된 레플리카 서버에서는 `sql_slave_skip_counter` 변수를 사용해 복제된 트랜잭션을 건너뛸 수 없다.

- GTID 기반 복제에서 `CHANGE REPLICATION SOURCE TO` 구문의 `IGNORE_SERVER_IDS` 옵션은 사용되지 않는다.

  `IGNORE_SERVER_IDS`옵션은 순환 복제 구조에서 한 서버가 장애로 인해 복제 토폴로지에서 제외했을 떄 장애 서버에서 발생한 이벤트가 중복으로 적용되지 않게 할 때 유용하게 사용할 수 있는데, GTID 복제에선 자동으로 무시된다.

## 16-4. 복제 데이터 포맷

레플리카 서버가 소스 서버의 바이너리 로그 이벤트를 내부적으로 가공하지 않고 가져온 그대로 실행해 자신의 데이터에 적용하므로 복제에서 어떤 바이너리 로그 포맷을 사용하느냐는 중요한 부분이다.

### 16-4-1. Statement 기반 바이너리 로그 포맷

`Statement 기반 포맷` : 변경 이벤트에 대해 이벤트를 발생시킨 SQL문을 바이너리 로그에 기록하는 방식

    Statement 포맷에선 SQL문만 기록되, 바이너리 로그 파일의 용령이 작아지므로 사용자 입장에선 저장 공간에 대한 부담을 덜 수 있다.

    원격으로 바이너리 로그를 백업하거나 원격에 위치한 레플리카 서버와 복제할 때도 좀 더 빠르게 처리될 수 있다.

**단점**

비확정적(Non-Deterministic)으로 처리될 수 있는 쿼리가 실행된 경우 Statement 포맷에서는 복제 시 소스 서버와 레플리카 서버 간에 데이터가 달라질 수 있다.

**비확정적 쿼리 유형**

- DELETE / UPDATE 쿼리에서 ORDER BY 절 없이 LIMIT 사용

- SELECT ... FOR UPDATE 및 SELECT ... FOR SHARE 쿼리에서 NOWAIT이나 SKIP LOCKED 옵션 사용

- LOAD_FILE(), UUID(), UUID_SHORT(), USER(), FOUND_ROWS(), RAND(), VERSION() 등과 같은 함수를 사용하는 쿼리

- 동일한 파라미터 값을 입력하더라도 결괏값이 달라질 수 있는 사용자 정의 함수나 스토어드 프로시저를 사용하는 쿼리

Row 포맷으로 복제될 때마다 데이터에 락을 더 많이 건다는 점이 또 다른 단점이다.

    INSERT INTO ... SELECT 구문이나 풀 테이블 스캔을 유발하는 UPDATE 쿼리, INSERT with AUTO_INCREMENT 쿼리 등이 실행된 경우, 오랜 시간 동안 락을 걸 가능성이 있어 주의해야한다.

Statement 기반 바이너리 로그 포맷을 사용할 떈 반드시 트랜잭션 격리 수준이 "REPEATABLE-READ" 이상이어야 한다.

### 16-4-2. Row 기반 바이너리 로그 포맷

데이터 변경이 발생했을 때 변경된 값 자체가 바이너리 로그에 기록되는 방식.

소스 서버와 레플리카 서버의 데이터를 일관되게 하는 가장 안전한 방식으로 5.7.7부터 기본 포맷으로 지정되었다.

MySQL 서버에서 실행된 쿼리가 많은 데이터를 변경한 경우 변경된 데이터가 모두 바이너리 로그 파일에 저장되 크기가 단시간에 매우 커질 수 있다. 또한 변경된 데이터 수가 적더라도 BLOB 형태의 큰 값이 새로 저장되거나 변경되는 경우에도 파일 크기가 많이 커지니 주의해야 한다.

Row 포맷은 실행 중인 쿼리를 확인할 수 없기 때문에, 릴레이 로그나 바이너리 로그를 mysqlbinlog 프로그램을 사용해 변환해야 한다.

    -v(--verbose) : mysqlbinlog는 변경된 데이터를 유사 SQL 형태로 변환해서 보여준다.

    -vv(--verbose --verbose) : 소스 서버에서 binlog_rows_query_log_events 시스템 변수를 활성화하면 SQL문을 그대로 볼 수 있다.

    결과 파일에서 Base64 문자열로 인코딩된 변겨 데이터를 제외하고 싶다면 "--base64-output=DECODE-ROWS" 옵션을 사용하면 된다.

Row 포맷은 모든 트랜잭션 격리 수준에서 사용 가능하며, 사용자 계정 생성과 권한 부여 및 회수, 그리고 테이블과 뷰, 트리거 생성 등과 같은 DDL문은 모두 Statement 포맷 형태로 바이너리 로그에 기록된다.

### 16-4-3. Mixed 포맷

---
