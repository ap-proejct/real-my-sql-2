## 1. 전문 검색
- MySQL 서버는 용량이 큰 문서를 단어 수준으로 쪼개서 문서 검색으로 해주는 기능이 있었다
- 이를 전문 검색이라고 하고, 단어들을 분리해서 형태소를 찾고 그 형태소를 인덱싱하는 방법이 있었지만 서구권 언어에서만 적합했다
- 형태소나 어원과 관계없이 특정 길이로 잘라 인덱싱하는 `n-gram`파서도 도입됐다

### 1. 전문 검색 인덱스의 생성과 검색
- MySQL 2가지 알고리즘을 이용해 인덱싱할 토큰을 분리해낸다
  - 형태소 분석(서구권 언어의 경우 어근 분석)
  - n-gram 분석
- 형태소 분석은 공백과 같은 띄어쓰기 단위로 단어를 분리하고, 조사를 제거해 명사 또는 어근을 찾아서 인덱싱하는 알고리즘이다
  - 하지만 MySQL서버에서는 단순히 공백과 같은 띄어쓰기 기준으로 토큰을 분리해서 인덱싱한다
  - 즉 MySQL 서버에서는 형태소 분석이나 어근 분석 기능은 구현돼 있지 않다(플러그인 형태로 사용)
- n-gram은 공백과 같은 띄어쓰기 단위로 단어를 분리하고 특정 길이(n)으로 쪼개서 인덱싱하는 알고리즘이다
  - 잘려진 토큰들을 전문 검색 인덱스를 이용해 동등 비교로 검색한다
  - 검색된 결과는 도큐먼트 ID로 그루핑하고 그루핑된 결과에서 각 단어의 위치를 이용해 최종 검색어를 포함하는지 ㅅ ㅣㄱ별한다

### 2. 전문 검색 쿼리 모드
- 전문 검색은 자연어 검색 모드와 불리언 검색 모드를 지원한다
- 기본값은 지연어이고 자연어 검색 모드와 함께 쓸 수 있는 검색어 확장 기능도 지원한다

#### 1. 자연어 검색(NATUAL LANGUAGE MODE)
- 문장이 검색어로 사용되면 구분자로 단어를 분리하고 `n-gram`파서로 토큰을 생성하고 일치하는 단어의 개수로 일치율을 계산한다

```sql
SELECT id, title, body,
        MATCH(title, body) AGAINST ('MySQL' IN NATUAL LANGUAGE MODE) AS score
FROM articles
WHERE MATCH(title, body) AGAINST ('MySQL' IN NATUAL LANGUAGE MODE;
```

#### 2). 불리언 검색(BOOLEAN MODE)
- 두 단어의 존재 여부를 이용해 논리적 연산을 하는 검색 모드이다
- `+` : 반드시 포함해야 하는 단어
- `-` : 반드시 포함하지 않아야 하는 단어
- `"`: 하나의 단어인 것처럼 취급한다 (무조건 똑같은 게 아니라 MySQL doc 이라고 한다면 MySQL 뒤에 doc 이라는 단어가 나오면 일치하는 것으로 판단하다)
- 아무것도 사용하지 않으면 검색어에 포함된 단어 중 아무거나 하나라도 있으면 일치하는 것으로 판단한다

```sql
-- 아래 쿼리는 MySQL은 포함하지만 manual은 포함하지 않는 문서를 검색한다
SELECT id, title, body,
        MATCH(title, body) AGAINST ('+MySQL -manual' IN BOOLEAN MODE) AS score
FROM tb_bi_gram
WHERE MATCH(title, body) AGAINST ('+MySQL -manual' IN BOOLEAN MODE);
```

#### 3). 검색어 확장(QUERY EXPANSION)
- 검색된 결과에서 공통으로 발견된 단어들을 모아서 한번 더 검색하는 방식이다
- 좋아보이지만 원하지 않은 결과가 많고 이를 위해 전문 검색 쿼리를 불필요하게 많이 실행할 수 있다

```sql
-- 1. 먼저 database와 연관 있어 보이는 단어들을 뽑는다
-- 2. 그 단어들을 이용해 다시 검색한다
SELECT * FROM tb_bi_gram
WHERE MATCH(title, body) AGAINST ('database' WITH QUERY EXPANSION);
```

### 3. 전문 검색 인덱스 디버깅
- `SET GLOBAL innodb_ft_aux_table = 'test/tb_bi_gram';`
- 해당 시스템 변수로 테이블을 설정하면 `information_schema`를 통해 전문 검색 인덱스가 어떻게 저장 및 관리되는지 볼 수 있다
  - `information_schema.innodb_ft_config`, `information_schema.innodb_ft_index_table`

## 2. 공간 검색

### 1. 용어 설명
- OGC(Open Geospatial Consortium)
  - 위치 기반 데이터를에 대한 표준을 수립하는 단체로, 모든 기관이 자유롭게 가입할 수 있다
- OpenGIS
  - OGC에서 제정한 지리 정보 시스템(GIS, Geographic Information System) 표준으로, WKT나 WKB 같은 지리 정보 데이터를 표기하는 방법과 저장하는 방법, 그리고 SRID 같은 표준을 포함한다
  - OpenGIS 표준을 준수한 응용 프로그램의 위치 기반 데이터는 상호 변환 없이 교환 가능하도록 설계돼 있다
- SRS와 GCS, PCS
  - SRS(Spatial Reference System, 공간 참조 시스템): 흔히 좌표계를 의미하고 SRS는 크게 GCS와 PCS로 구분된다
  - GCS(Global Coordinate System, 지리 좌표계): 지구 구체상 특정 위치나 공간을 표현하는데, 흔히 위도와 경도 같은 각도 단위의 숫자로 표시된다
    - 지구와 같은 구체 표면에서 특정 위치를 정의한다
  - PCS(Projected Coordinate System, 투영 좌표계): 구체 형태의 지구를 종이 지도같은 평면으로 투영시킨 좌표를 의미하고 주로 미터같은 선형적 단위로 표시된다
    - 2차원 평면인 종이에 어떻게 표현할지를 정의한다. PCS 좌표는 GCS 좌표도 같이 포함한다
- SRID와 SRS-ID
  - Spatial Reference ID로 특정 SRS를 지칭하는 고유 번호를 의미한다. SRS-ID와 SRID는 동의어이다
- WKT와 WKB
  - 둘 다 OpenGIS명시한 위치 좌표의 표현 방법이다
  - MySQL 내부적으로는 해당 포맷으로 저장하는지 않는다
  - WKT(Well-Known Text format): 사람 눈으로 쉽게 확인할 수 있는 텍스트 포맷이며 `POINT(15 20)` 또는 `LINESTRING(15 20, 20 25)`등 점이나 도형의 위치 정보를 정의하는 표준이다
  - WKB(Well-Known Binary format): 컴퓨터에 저장할 수 있는 형태의 이진 포맷의 저장 표준이다
- MBR과 R-Tree
  - MBR(Minimum Bounding Rectangle, 최소 경계 사각형): MySQL는 공간 인덱스를 도형 포함 관계를 이용해서 만들고, 이렇게 만들어진 인덱스를 R-Tree라고 한다

### 2. SRS(Spatial Reference System)
- SRS(좌표계)는 GCS(지리 좌표계)와 PCS(투영 좌표계)로 구분되고 MySQL 지원하는 SRS는 5000개가 넘고, `information_schema.ST_SPATIAL_REFERENCE_SYSTEMS` 테이블을 통해 확인할 수 있다
  - 여기서 중요한 것은 SRS_ID 칼럼과 DEFINITION 칼럼으로 어떤 좌표계인지에 대한 정의가 저장돼 있다
  - DEFINITION 칼럼 값은 항상 GEOGCS(지리 좌표계(GCS)) 또는 PROJCS(투영 좌표계(PCS))로 시작된다
  - `AXIX`도 중요한 필드로 위도와 경도를 나타내고, 첫 번째가 `X`축이고 두 번째가 `Y`축이다

| Field                     | Type             | Null | Key | Default | Extra |
|---------------------------|------------------|------|-----|---------|-------|
| SRS_NAME                  | varchar(80)      | NO   |     |         |       |
| SRS_ID                    | int unsigned     | NO   |     |         |       |
| ORGANIZATION              | varchar(256)     | YES  |     |         |       |
| ORGANIZATION_COORDSYS_ID  | int unsigned     | YES  |     |         |       |
| DEFINITION                | varchar(4096)    | NO   |     |         |       |
| DESCRIPTION               | varchar(2048)    | YES  |     |         |       |

| SRS_NAME | SRS_ID | ORGANIZATION | ORGANIZATION_COORDSYS_ID | DEFINITION | DESCRIPTION |
|---|---|---|---|---|---|
| WGS 84 | 4326 | EPSG | 4326 | GEOGCS["WGS 84",DATUM["World Geodetic System 1984",SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.017453292519943278,AUTHORITY["EPSG","9122"]],AXIS["Lat",NORTH],AXIS["Lon",EAST],AUTHORITY["EPSG","4326"]] | |

- 투영 좌표계는 반대로 `POINT(경도 위도)` 순서로 명시해야 한다

```sql
-- 두 점 (0, 0)과 (1, 1) 사이의 거리를 구하는 예제이다
-- 평면 좌표계(SRID=0)를 사용한 공간 데이터이다
-- 결과값은 아무런 단위가 없고 단순히 피타고라스 정의에 의해 수식으로 계산된 거리 값이다
SELECT ST_Distance(ST_GeomFromText('POINT(0 0)', 0), 
                   ST_GeomFromText('POINT(1 1)', 0)) AS distance;

-- 웹 기반 지도 좌표계(SRID=3857)를 사용한 공간 데이터이다
SELECT ST_Distance(ST_GeomFromText('POINT(0 0)', 3857), 
                   ST_GeomFromText('POINT(1 1)', 3857)) AS distance;
```

### 3. 투영 좌표계와 평면 좌표계
- MySQL에는 투영 좌표계나 지리 좌표계에 속하지 않은 평면 좌표계가 있는데, 투평 좌표계와 비슷한 특성을 가진다
- 지구 구체 전체 또는 일부를 평면으로 투영해서 표현화 좌표계이다
- 평면 좌표계와 투영 좌표계를 이용하기 위해서는 다음과 같은 테이블을 생성하면 된다
- 평면 좌표계(SRID=0) 에서는 `ST_PointFromText()` 함수를 이용해 `POINT(X, Y)` 문자열(WKT)을 Geometry 타입 값으로 변환할 때 특별히 SRID를 설정하지 않아도 자동으로 SRID=0으로 설정된다
- 하지만 투영 좌표계(SRID=3857)은 `ST_PointFromText()`를 이용해 `POINT(경도, 위도)` 문자열을 Geometry 타입 값으로 변환할 때는 명시적으로 설정해야 한다
- 테이블을 생성할 때 SRID를 명시적으로 정의하지 않으면 모든 SRID를 저장할 수 있다
  - 하지만 하나의 칼럼에 여러 SRID이면 인덱스를 이용한 빠른 검색을 수행할 수 없다(마치 여러 콜레이션을 넣은 느낌)
- 응용 프로그램에서 평면 좌표계는 그다지 사용되지 않지만 투영 좌표계는 지도나 화면에 지도를 표현하는 경우 자주 사용된다
  - 지리 좌표계도 사용 방법이 크게 다르지 않아 데이터를 저장하고 조회할 때는 동일하게 사용한다

```sql
-- 평면 좌표계 사용 예제
CREATE TABLE plain_coord(
    location POINT SRID 0,
);

INSERT INTO plain_coord VALUES(ST_PointFromText('POINT(0 0)'));
INSERT INTO plain_coord VALUES(ST_PointFromText('POINT(5 5)', 0));

-- 조회
SELECT ST_AsText(location) AS location_wkt,
        ST_X(location) AS location_x,
        ST_Y(location) AS location_y
FROM plain_coord;

-- 투영 좌표계 사용 예제
CREATE TABLE projection_coord(
    location POINT SRID 3857
);

INSERT INTO projection_coord VALUES(ST_PointFromText('POINT(1413.066 4509381.87)', 3857));

-- 조회
SELECT ST_AsText(location) AS location_wkt,
        ST_X(location) AS location_x,
        ST_Y(location) AS location_y
FROM projection_coord;
```

### 4. 지리 좌표계
- 인덱스를 이용한 검색을 위해서는 MBR을 이용한 `ST_Within` 함수를 이용해야 한다
  - 1KM 반경을 구하기 위해서는 1KM의 원을 감싸는 사각형(MBR) 만들어야 하는데, WGS 84 공간 좌표계의 우치의 단위는 각도(Degree)이기 때문에 1KM의 거리가 각도로는 얼마인지 계산해야 한다
  - 문제는 지구가 구면체라서 위도에 따라서 경도 1도에 해당하는 거리가 달라진다
#### 1). 지리 좌표계 데이터 관리
- 공간 인덱스는 반드시 `NOT NULL`이어야 한다
```sql
CREATE TABLE sphere_coord(
    location POINT NOT NULL SRID 4326, -- WGS84 좌표계
    SPATIAL INDEX sc_location(location)
)

INSERT INTO sphere_coord VALUES(
    ST_PointFromText('POINT(37.544 127.945)', 4326)
)

-- 아래 쿼리는 인덱스를 사용할 수 없어 풀 테이블 스캔을 이용한다
SELECT ST_AsText(location) AS location,
       ST_Distance_Sphere(location, ST_PointFromText('Point(37.54 127.04)' 4326))) AS distance_meters
FROM sphere_coord
WHERE ST_Distance_Sphere(location,
    ST_PointFromText('POINT(37.54 127.04)', 4326)) < 1000;

```


#### 2). 지리 좌표계 주의 사항
- 지리 좌표계나 SRS 관리 기능이 도입된건 8.0이 처음이라 살짝 미숙한 부분이 있다

#### (1). 정확성 주의 사항
- 지리 좌표계에서 `ST_Contains()`함수가 정확하지 않은 결과를 반환하기도 한다
- `ST_Distance_Sphere()`는 공간 인덱스를 활용하지 못하기 때문에 `ST_Contains()`함수와 같이 써서 정확성을 높이는 것이 좋다

#### (2). 성능 주의사항
- 지리 좌표계의 경우 SRID가 4326을 많이 사용하는데 `ST_Contains()` 등으로 포함 관계를 비교하는 경우 투영 좌표계보다 느린 성능을 보인다

#### (3). 좌표계 변환
- 휴대폰같은 장치로부터 수신받는 GPS 좌표는 WGS 84이며, SRID 4326인 지리 좌표계로 표현된다
- SRID 3857은 WGS 84 좌표를 평면으로 투영한 좌표시스템이기 때문에 상호 변환이 가능하다
  - MySQL에서도 ST_Transform()로 좌푯값을 변환할 수 있지만 아직 SRID 3857에 대한 변환은 지원하지 않는 상태다
  - 또한 SRID 3857은 평면으로 투영된 좌표계이기 때문에 거리 계산 시 상당한 오차를 보인다