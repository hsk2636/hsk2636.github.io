# 🧪 통합 품질 정보 SQL 분석 (Quality SQL Analysis)

Oracle SQL을 활용하여 **LOT별 / 버전별 품질 데이터(GOOD / BAD / M2)** 를 집계하고  
**불량률(Defect Rate)** 및 **TOTAL ROW 생성 로직**을 분석한 실습 기반 프로젝트입니다.

제조 IT(MES/PDA/ERP) 환경에서 자주 등장하는  
품질 지표 산출 로직을 SQL 중심으로 재구성하여 포트폴리오 분석 자료로 만들었습니다.

---

## 📌 프로젝트 목표

이 프로젝트에서는 다음 내용을 SQL 중심으로 정리합니다.

- LOT 단위 품질 데이터(GOOD/BAD) 집계
- 버전(Rev)별 품질 지표 산출
- NVL / DECODE / GROUP BY 를 활용한 TOTAL ROW 구성
- 월별 품질 현황 집계 구조
- 실무 품질 분석 쿼리 패턴 익히기

---

## 📂 파일 구성

```
quality-sql-analysis/
│
├── 01_quality_summary.sql # 기본 품질 집계 쿼리
├── 02_quality_version.sql # 버전(REV)별 GOOD/BAD 집계
├── 03_quality_total_row.sql # TOTAL ROW 생성용 SQL 패턴
├── 04_monthly_quality.sql # 월별 품질 현황 집계
└── README.md # 프로젝트 소개 문서
```

---

## 🏷 사용된 Oracle SQL 기능

| 기능 | 설명 |
|------|------|
| `NVL` | NULL 값 치환 |
| `DECODE` | 조건 분기 기반 값 변환 |
| `GROUP BY` | 그룹 집계 |
| `ROLLUP` | TOTAL 행 자동 생성 |
| `CASE WHEN` | 조건 기반 컬럼 생성 |
| `SUM`, `COUNT` | 기본 집계 함수 |

---

## 📊 1. 기본 품질 집계 SQL 예제

```sql
SELECT
    LOT_ID,
    SUM(GOOD_QTY) AS GOOD,
    SUM(BAD_QTY)  AS BAD,
    SUM(GOOD_QTY + BAD_QTY) AS TOTAL
FROM TB_QUALITY
GROUP BY LOT_ID
ORDER BY LOT_ID;
```

---

## 📌 2. 버전(REV)별 품질 지표 집계

```sql
SELECT
    LOT_ID,
    REV_NO,
    SUM(GOOD_QTY) AS GOOD,
    SUM(BAD_QTY)  AS BAD,
    ROUND(SUM(BAD_QTY) / NULLIF(SUM(GOOD_QTY + BAD_QTY), 0) * 100, 2) AS DEFECT_RATE
FROM TB_QUALITY
GROUP BY LOT_ID, REV_NO
ORDER BY LOT_ID, REV_NO;
```

---

## 📐 3. TOTAL ROW 생성 패턴
실무 품질 SQL에서 가장 자주 등장하는 패턴:

```sql
SELECT
    LOT_ID,
    SUM(GOOD_QTY) AS GOOD,
    SUM(BAD_QTY) AS BAD,
    SUM(GOOD_QTY + BAD_QTY) AS TOTAL
FROM TB_QUALITY
GROUP BY ROLLUP(LOT_ID)
ORDER BY LOT_ID;
```

---

➕ 출력 예시

| LOT_ID          | GOOD     | BAD    | TOTAL    |
| --------------- | -------- | ------ | -------- |
| LOT001          | 900      | 30     | 930      |
| LOT002          | 850      | 50     | 900      |
| **NULL(TOTAL)** | **1750** | **80** | **1830** |

---

## 📅 4. 월별 품질 현황 집계

```sql
SELECT
    TO_CHAR(WORK_DATE, 'YYYY-MM') AS WORK_MONTH,
    SUM(GOOD_QTY) AS GOOD,
    SUM(BAD_QTY) AS BAD
FROM TB_QUALITY
GROUP BY TO_CHAR(WORK_DATE, 'YYYY-MM')
ORDER BY WORK_MONTH;
```

---

🔍 실무 응용 포인트

- LOT 상태 관리와 결합하면 공정 수율 분석으로 확장 가능

- 버전(REV) 단위 집계는 엔지니어링 변경(ECN) 영향 분석에 활용

- NO_DATA 처리 시 NVL + NULLIF 조합 필수

- 월 단위 집계는 생산/품질 Dashboard 자료로 바로 사용 가능

---

📁 향후 추가 예정

- 파티션 기반 품질 집계

- 수율(Yield) 계산 공식 종류 비교

- 제조사(Fab/Module) 단위 품질 비교 예시

---

🙋‍♂️ Author

권한수 (Hansoo Kwon)
SQL Developer · PL/SQL Engineer · Manufacturing IT
🔗 https://hsk2636.github.io
