<div align="center">

<img src="https://img.shields.io/badge/Real%20MySQL%208.0-2권-4479A1?style=for-the-badge&logo=mysql&logoColor=white"/>
&nbsp;
<img src="https://img.shields.io/badge/Members-2명-brightgreen?style=for-the-badge&logo=github&logoColor=white"/>
&nbsp;
<img src="https://img.shields.io/badge/Status-진행중-orange?style=for-the-badge"/>

<br/>
<br/>

# 📗 Real MySQL 8.0 스터디 — 2권

### *쿼리 최적화부터 InnoDB 클러스터까지, 함께 깊이 파고드는 MySQL 스터디*

</div>

<br/>

---

## 👥 스터디 멤버

<table align="center">
  <tr>
    <td align="center" width="200">
      <b>정주연</b><br/>
      <a href="https://github.com/">@정주연</a>
    </td>
    <td align="center" width="200">
      <b>곽유섭</b><br/>
      <a href="https://github.com/">@곽유섭</a>
    </td>
  </tr>
</table>

---

## 📚 스터디 커리큘럼

> Real MySQL 8.0 **2권** (원서 기준 **11장 ~ 18장**)을 스터디 편의상 **Chapter 01 ~ 08** 로 표기합니다.
> 책 챕터와 스터디 챕터는 **1:1** 로 대응됩니다.

| 스터디 챕터 | 원서 장 | 주제 | 핵심 키워드 |
|:-----------:|:-------:|------|------------|
| [**Chapter 01**](./chapter01/) | 11장 | 쿼리 작성 및 최적화 | `SELECT 최적화` `GROUP BY` `ORDER BY` `서브쿼리` `INSERT/UPDATE/DELETE` |
| [**Chapter 02**](./chapter02/) | 12장 | 확장 검색 | `전문 검색` `공간 검색` `멀티 밸류 인덱스` |
| [**Chapter 03**](./chapter03/) | 13장 | 파티션 | `파티션 종류` `파티션 프루닝` `파티션 주의사항` |
| [**Chapter 04**](./chapter04/) | 14장 | 스토어드 프로그램 | `프로시저` `함수` `트리거` `이벤트` |
| [**Chapter 05**](./chapter05/) | 15장 | 준비된 트랜잭션 (XA) | `XA 트랜잭션` `분산 트랜잭션` `2PC` |
| [**Chapter 06**](./chapter06/) | 16장 | 복제 (Replication) | `Binary Log` `GTID` `준동기 복제` `복제 지연` |
| [**Chapter 07**](./chapter07/) | 17장 | InnoDB 클러스터 | `Group Replication` `MySQL Shell` `고가용성` |
| [**Chapter 08**](./chapter08/) | 18장 | Performance 스키마 | `performance_schema` `sys 스키마` `모니터링` |

---

## 📁 레포지토리 구조

```
real-my-sql-2/
├── README.md                   ← 스터디 소개 & 규칙 (이 파일)
│
├── chapter01/                  ← 11장: 쿼리 작성 및 최적화
│   ├── 정주연/
│   │   ├── README.md           ← 개인 정리 노트
│   │   └── images/             ← 캡처 이미지 저장 폴더
│   └── 곽유섭/
│       ├── README.md
│       └── images/
│
└── chapter02/ ~ chapter08/     ← 동일 구조 반복
```

---

## 📋 스터디 규칙

### ✅ 기본 규칙

1. **매주 1챕터** 분량을 읽고, 각자 본인 폴더의 `README.md` 에 정리합니다.
2. 정리본은 **스터디 전날 자정** 까지 PR을 올립니다.
3. 스터디 시간에는 서로의 정리를 **리뷰**하고, 질문 & 토론을 진행합니다.
4. 이해 안 되는 내용은 솔직하게 질문하고, 아는 내용은 적극적으로 공유합니다.
5. **결석/지각** 시 사전에 반드시 알립니다.

### 📝 정리 방법

- 본인 이름 폴더(`정주연/`, `곽유섭/`) 안의 `README.md` 에 자유 형식으로 작성합니다.
- 이미지(캡처, 다이어그램)는 `images/` 폴더에 넣고 README 에 삽입합니다.
  ```md
  ![설명](./images/파일명.png)
  ```
- 정리 형식은 자유이나, 핵심 개념과 본인 생각은 반드시 포함합니다.

### 🔀 PR & 머지 규칙

| 항목 | 규칙 |
|------|------|
| **브랜치 네이밍** | `chapter01/정주연` |
| **PR 제목** | `[Chapter 01] 정주연 - 정리 완료` |
| **머지 조건** | 상대방 **Approve** 필수 |
| **머지 시점** | 스터디 당일 종료 후 `main` 브랜치에 머지 |

### ⏰ 스터디 일정

| 항목 | 내용 |
|------|------|
| 📅 진행 주기 | 매주 1회 |
| 🕐 소요 시간 | 약 1 ~ 2시간 |
| 📡 진행 방식 | 온 / 오프라인 병행 |
| 📣 공지 채널 | Discord / 카카오톡 |

---

## 📈 진행 현황

| 챕터 | 원서 | 정주연 | 곽유섭 |
|:----:|:----:|:------:|:------:|
| Chapter 01 | 11장 | ⬜ | ⬜ |
| Chapter 02 | 12장 | ⬜ | ⬜ |
| Chapter 03 | 13장 | ⬜ | ⬜ |
| Chapter 04 | 14장 | ⬜ | ⬜ |
| Chapter 05 | 15장 | ⬜ | ⬜ |
| Chapter 06 | 16장 | ⬜ | ⬜ |
| Chapter 07 | 17장 | ⬜ | ⬜ |
| Chapter 08 | 18장 | ⬜ | ⬜ |

> ✅ 완료 &nbsp;|&nbsp; 🔄 진행 중 &nbsp;|&nbsp; ⬜ 예정

---

## 💡 참고자료

- 📖 [Real MySQL 8.0 2권 — 교보문고](https://product.kyobobook.co.kr/detail/S000001766483)
- 🌐 [MySQL 8.0 공식 문서](https://dev.mysql.com/doc/refman/8.0/en/)
- 🐋 [MySQL Docker Hub](https://hub.docker.com/_/mysql)
- 🛠️ [MySQL Workbench 다운로드](https://www.mysql.com/products/workbench/)

---

<div align="center">

*차근차근, 꼼꼼하게 — 함께라서 더 빠릅니다* 🚀

</div>
