# 📘 곽유섭 - Chapter 12 - 확장 검색

> Real MySQL 8.0 2권 | Chapter 02 - 확장 검색

---

## 📝 정리 내용

# 12. 확장 검색

## 12-1. 전문 검색

`전문 검색` : MySQL서버에서 용량이 큰 문서를 단어 수준으로 잘게 쪼개어 문서 검색을 하게 해주는 기능

문서의 단어들을 분리해서 형태소를 찾고 그 형태소를 인덱싱하는 방법은 서구권 언어에만 적합.

### 12-1-1. 전문 검색 인덱스의 생성과 검색

**형태소 분석(서구권 : 어근 분석)**

- 문장의 공백과 같은 띄어쓰기 단위로 단어를 분리하고, 각 단어의 조사를 제거해 명사 또는 어근을 찾아서 인덱싱하는 알고리즘

**n-gram 파서**

- 문장 자에에 대한 이해 없이 공백과 같은 띄어쓰기 단위로 단어를 분리하고, 그 단어를 단순히 주어진 길이로 쪼개서 인덱싱하는 알고리즘
- `ngram_token_size` : n-gram의 길이를 설정하는 시스템 변수. 주로 bi-gram(2)나 tri-gram(3)으로 사용한다.
- **주의점** : 전문 검색 인덱스를 생성할땐 반드시 **"WITH PARSER ngram"** 옵션을 추가해야함. (그렇지 않으면 기본 파서(공백과 같은 구분자를 기준으로 단어 분리 후 인덱싱)을 통해 전문 검색 인덱스 생성)

```sql
--// my.cnf 설정 파일에서 다음 항목을 추가 후 재시작
--// ngram_token_size=2;

--// 인덱스 생성
mysql> CREATE TABLE tb_bi_gram (
    id BIGINT NOT NULL AUTO_INCREMENT,
    title VARCHAR(100),
    body TEXT,
    PRIMARY KEY(id),
    FULLTEXT INDEX fx_msg(title, body) WITH PARSER ngram
);

--// 인덱스 조회
mysql> SELECT COUNT(*) FROM tb_bi_gram
        WHERE MATCH(title, body) AGAINST ('단편' IN BOOLEAN MODE);
```

**전문 검색 인덱스 규칙**

- 검색어의 길이가 ngram_token_size보다 작은 경우 : 검색 불가능
- 검색어의 길이가 ngram_token_size보다 크거나 같은 경우 : 검색 가능

n-gram 전문 인덱스의 파서는 ngram_token_size=2보다 길이가 작은 단어는 모두 버려진다.

### 12-1-2. 전문 검색 쿼리 모드

#### 12-1-2-1. 자연어 검색(NATURAL LANGUAGE MODE)

`자연어 검색` : 검색어에 제시된 단어들을 많이 가지고 있는 순서대로 정렬해서 결과를 반환하는 것

```sql
mysql> SELECT id, title, body,
        MATCH(title, body) AGAINST ('MySQL' IN NATURAL LANGUAGE MODE) AS score
        FROM tb_bi_gram
        WHERE MATCH(title, body) AGAINST ('MySQL' IN NATURAL LANGUAGE MODE);

mysql> SELECT id, title, body,
        MATCH(title, body) AGAINST ('MySQL manual is true guide' IN NATURAL LANGUAGE MODE) AS score
        FROM tb_bi_gram
        WHERE MATCH(title, body) AGAINST ('MySQL manual is true guide' IN NATURAL LANGUAGE MODE);
```

검색어는 단어, 문장으로 사용할 수 있으며, 문장은 구분자(공백, 뉴 라인같은 띄어쓰기)로 단어를 분리하고 n-gram 파서로 토큰을 생성한 후 각 토큰에 대해 일치하는 단어의 개수를 확인해서 일치율을 계산한다.

검색어가 단일 단어 또는 문장인 경우 ".", ","등과 같은 문장 기호는 모두 무시된다.

#### 12-1-2-2. 불리언 검색(BOOLEAN MODE)

`불리언 검색` : 쿼리에 사용되는 검색어의 존재 여부에 대해 논리적 연산이 가능하다.

```sql
--// MySQL은 포함, manual은 포함하지 않는 레코드 검색 쿼리
mysql> SELECT id, title, body
        MATCH(title, body) AGAINST ('+MySQL -manual' IN BOOLEAN MODE) AS score
        FROM tb_bi_gram
        WHERE MATCH(title, body) AGAINST ('+MySQL -manual' IN BOOLEAN MODE);

--// 쌍따옴표(")내부의 단어가 순서대로 있는 레코드만 반환
mysql> SELECT id, title, body,
        MATCH(title, body) AGAINST ('+"MySQL man"' IN BOOLEAN MODE) AS score
        FROM tb_bi_gram
        WHERE MATCH(title, body) AGAINST ('+"MySQL man' IN BOOLEAN MODE);

--// "+","-"를 사용하지 않으면 검색어에 포함된 단어 중 아무거나 하나라도 일치하는 레코드가 있으면 반환
mysql> SELECT id, title, body,
        MATCH(title, body) AGAINST ('MySQL doc' IN BOOLEAN MODE) AS score
        FROM tb_bi_gram
        WHERE MATCH(title, body) AGAINST ('MySQL doc' IN BOOLEAN MODE);
```

#### 12-1-2-3. 검색어 확장(QUERY EXPANSION)

`검색어 확장` : 사용자가 쿼리에 사용한 검색어로 검색된 결과에서 공통으로 발견되는 단어들을 모아서 다시 한번 더 검색을 수행하는 방식

```sql
mysql> SELECT * FROM tb_bi_gram
        WHERE MATCH(title, body) AGAINST('database' WITH QUERY EXPANSION);
```

### 12-1-3. 전문 검색 인덱스 디버깅

```sql
--// MySQL에서 전문 검색 쿼리 오류 원인을 쉽게 찾기 위한 디버깅 기능
mysql> SET GLOBAL innodb_ft_aux_table = 'test/tb_bi_gram';

--// 전문 검색 인덱스의 설정 내용 조회
mysql> SELECT * FROM information_schema.innodb_ft_config;

--// 전문 검색 인덱스가 가지고 있는 인덱스 엔트리 목록 조회.
--// 토큰들이 어떤 레코드에 몇 번 사용됐는지, 레코드별로 문자 위치가 어디인지 등 정보 관리
mysql> SELECT * FROM information_schema.innodb_ft_index_table;

--// 메모리에 임시 저장된 인덱스 토큰 조회
mysql> SELECT * FROM information_schema.innodb_ft_index_cache;

--// 어떤 레코드가 삭제됐는지 조회
mysql> SELECT * FROM information_schema.innodb_ft_deleted;
```

## 12-2. 공간 검색

### 12-2-1. 용어 설명

- `OGC(Open Geospatial Consortium)`

  위치 기반 데이터에 대한 표준을 수립하는 단체

- `OpenGIS`

  OGC에서 제정한 지리 정보 시스템 표준, WKT나 WKB 같은 지리 정보 데이터를 표기하는 방법과 저장하는 방법, 그리고 SRID와 같은 표준을 포함한다.

- `SRS와 GCS, PCS`

  `SRS`: 좌표계

  `GCS(지리 좌표계)`: 지구 구체상의 특정 위치나 공간을 표현하는 좌표계, 위도와 경도와 같이 각도 단위의 숫자로 표시.

  `PCS(투영 좌표계)` : 지구를 평면으로 투영시킨 좌표계, 미터와 같은 선형적인 단위로 표시.

- `SRID와 SRS_ID`

  `SRID` : 특정 SRS를 지칭하는 고유 번호. MySQL에서 함수의 인자나 식별자로 사용

  `SRS_ID` : MySQL에서 SRS 고유 번호를 저장하는 칼럼의 이름

- `WKT와 WKB`

  OGC에서 제정한 OpenGIS에서 명시한 위치 좌표의 표현 방법

  `WKT` : 사람의 눈으로 쉽게 확인할 수 있는 텍스트 포멧

  `WKB` : 컴퓨터에 저장할 수 있는 형태의 이진 포맷의 저장 표준

- `MBR과 R-Tree`

  `MBR` : 어떤 도형을 감싸는 최소의 사각 상자. MySQL 공간 인덱스의 포함 관계에 사용됨.

  `R-Tree` : 도형들의 포함 관계를 이용해서 만들어진 인덱스

### 12-2-2. SRS(Spatial Reference System)

```sql
--// SRS에 대한 정보 조회
mysql> DESC information_schema.ST_SPATIAL_REFERENCE_SYSTEMS;
```

**중요한 컬럼**

`SRS_ID` : .해당 좌표계를 지칭하는 고유 번호

`DEFINITION` : 해당 좌표계가 어떤 좌표계인지에 대한 정의

`AXIS` : 위도 경도 값(첫 번째가 X축(위도), 두 번째가 Y축(경도))

````markdown
8.0부터 SRID를 지정할 수 있게 됐으며 지정하지 않았던 데이터는 모두 SRID가 0인 평면 좌표계로 인식된다.

```sql
--// 평면 좌표계(SRID=0)를 사용하는 공간 데이터
mysql> SELECT ST_Distance(ST_PointFromText('POINT(0 0)', 0),
ST_PointFromText('POINT(1 1)', 0)) AS distance;

--// 웹 기반 지도 좌표계(SRID=3857)를 사용하는 공간 데이터
mysql> SELECT ST_Distance(ST_PointFromText('POINT(0 0)', 3857),
ST_PointFromText('POINT(1 1)', 3857)) AS distance;

--// WGS 84 지리 좌표계(SRID=4326)를 사용하는 공간 데이터
mysql> SELECT ST_Distance(ST_PointFromText('POINT(0 0)', 4326),
ST_PointFromText('POINT(1 1)', 4326)) AS distance;
```
````

SRID가 0이면 사용자가 의도한 값을 계산하지 못할 수 있기 때문에 주의가 필요하다.

### 12-2-3. 투영 좌표계와 평면 좌표계

`투영 좌표계` : 지구 구체 전체 또는 일부를 평면으로 투영해서 표현한 좌표계

`평면 좌표계` : SRID=0인 좌표계로 단위를 가지지 않으며 X축과 Y축의 값이 제한을 가지지 않는 좌표계(=무한 평면 좌표계)

```sql
--// 평면 좌표계 사용 예제
mysql> CREATE TABLE plain_coord (
    id INT NOT NULL AUTO_INCREMENT,
    location POINT SRID 0,
    PRIMARY KEY(id)
);

mysql> INSERT INTO plain_coord VALUES(1, ST_PointFromText('POINT(0 0)'));

mysql> INSERT INTO plain_coord VALUES(2, ST_PointFromText('POINT(5 5)', 0));

--// 투영 좌표계 사용 예제
mysql> CREATE TABLE projection_coord (
    id INT NOT NULL AUTO_INCREMENT,
    location POINT SRID 3857,
    PRIMARY KEY(id)
);

mysql> INSERT INTO projection_coord VALUES(1, ST_PointFromText('POINT(14133791.0666622 4509381.876958)', 3857));
```

테이블 생성 시 SRID를 지정하지 않으면 모든 SRID를 저장할 수 있지만, 인덱스를 이용할 수 없게 된다.

```sql
--// 평면 좌표계 조회
mysql> SELECT id, location, ST_AsWKB(location) FROM plain_coord \G

mysql> SELECT id,
            ST_AsText(location) AS location_wkt,
            ST_X(location) AS location_x,
            ST_Y(location) AS location_y
        FROM plain_coord;

--// 투영 좌표계 조회
mysql> SELECT id, location, ST_AsWKB(location) FROM projection_coord \G

mysql> SELECT id,
            ST_AsText(location) AS location_wkt,
            ST_X(location) AS location_x,
            ST_Y(location) AS location_y
        FROM projection_coord;

```

`ST_Distance_Sphere()` : 지구(구면체)상의 두 점 간의 거리를 계산하는 함수(SRID=4326과 같은 지리 좌표에서만 사용 가능)

### 12-2-4. 지리 좌표계

#### 12-2-4-1. 지리 좌표계 데이터 관리

1. ST_Distance_Sphere() 함수를 통한 검색
   - 다른 RDBMS는 인덱스를 이용한 반경 검색이 있지만, MySQL에서는 아직 없다.

2. MBR을 이용한 ST_Contains(), ST_Within() 함수를 통한 검색

   ```sql
   --// 두 쿼리 모두 공간인덱스에서 range 접근 방법으로 데이터를 읽는다.
   mysql> SELECT id, name
           FROM sphere_coord
           WHERE ST_Contains(
               getDistanceMBR(ST_PointFromText('POINT(37.547027 127.047337)', 4326), 1), location);

   mysql> SELECT id, name
           FROM sphere_coord
           WHERE ST_Within(location,
           getDistanceMBR(ST_PointFromText('POINT(37.5547027 127.047337)', 4326), 1));
   ```

#### 12-2-4-2. 지리 좌표계 주의 사항

##### 12-2-4-2-1. 정확성 주의 사항

지리 좌표계(SRID=4326)에서 ST_Contains() 함수가 정확하지 않은 결과를 반환하기도 한다. (MySQL 8.0.25버전 기준)

**회피 방법**

```sql
--// POLYGON의 중심 지점에 대한 POINT 객체 생성
mysql> SET @center:= ST_GeomFromText('POINT(37.398899 126.966064)', @SRID);

mysql> SELECT (ST_Contains(@polygon, @point)
                AND ST_Distance_Sphere(@point, @center)<=100) AS within_polygon;
```

##### 12-2-4-2-2. 성능 주의 사항

지리 좌표계 데이터의 경우 일반적으로 SRID=4236인 좌표계를 많이 사용하는데 ST_Contains() 함수 등으로 포함 관계를 비교하는 경우 투영 좌표계보다는 느린 성능을 보인다.

##### 12-2-4-2-3. 좌표계 변환

MySQL에서 ST_Transform() 함수를 이용해 SRID 간에 좌푯값을 변환할 수 있지만 아직 SRID 3857에 대한 변환은 지원하지 않는다.
