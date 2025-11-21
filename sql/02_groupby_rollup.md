---
title: GROUP BY / 집계함수 / ROLLUP
---

# GROUP BY / 집계함수 / ROLLUP

이 문서는 Oracle SQL의 핵심 집계 기능인  
**GROUP BY, 집계함수(SUM/COUNT/AVG), ROLLUP, GROUPING 함수**를  
실무 예제로 정리한 학습 문서입니다.

특히 “월별/일별 집계”, “TOTAL 행 생성”, “보고서용 집계 SQL” 제작에 자주 사용되는 패턴을 중심으로 설명합니다.

---

# 1. 기본 GROUP BY 집계

## 1-1. 문제 정의
`TB_ORDER` 테이블이 있다고 가정합니다.

- ORDER_ID  
- ORDER_DATE  
- ORDER_AMT  

👉 **월별 총 매출 금액을 구하는 SQL을 작성하라.**

---

## 1-2. SQL 코드

```sql
SELECT
    TO_CHAR(ORDER_DATE, 'YYYY-MM') AS ORDER_MONTH,
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
GROUP BY TO_CHAR(ORDER_DATE, 'YYYY-MM')
ORDER BY ORDER_MONTH;
```

---

## 1-3. 설명

- TO_CHAR(ORDER_DATE, 'YYYY-MM')
주문일자 → 월 단위로 기준 맞춤

- SUM(ORDER_AMT)
월별 총 매출 계산

- GROUP BY
월 기준으로 집계 실행

👉 실무에서 가장 자주 등장하는 "월별 집계" 기본 패턴.

---

# 2. ROLLUP으로 월별 + 전체 합계(TOTAL) 만들기

## 2-1. 문제 정의

👉 “월별 매출 + 전체 매출(TOTAL)까지 한 번에 보고 싶다.”

그럴 때 ROLLUP을 사용한다.

---

# 2-2. SQL 코드

```sql
SELECT
    TO_CHAR(ORDER_DATE, 'YYYY-MM') AS ORDER_MONTH,
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
GROUP BY ROLLUP (TO_CHAR(ORDER_DATE, 'YYYY-MM'))
ORDER BY ORDER_MONTH;
```

---

# 2-3. 설명

- ROLLUP(컬럼)
→ 그룹별 집계 + 전체 TOTAL 행 생성

예)
| ORDER_MONTH   | TOTAL_AMT   |
| ------------- | ----------- |
| 2024-01       | 110,000     |
| 2024-02       | 98,000      |
| **NULL (전체)** | **208,000** |

---

# 3. GROUPING 함수로 TOTAL 행 라벨링

ROLLUP을 사용하면 TOTAL 행은 ORDER_MONTH = NULL로 표시된다.
보고서에서는 보통 “TOTAL”로 바꿔줘야 한다.

---

## 3-1. SQL 코드
```sql
SELECT
    CASE
        WHEN GROUPING(TO_CHAR(ORDER_DATE, 'YYYY-MM')) = 1
             THEN 'TOTAL'
        ELSE TO_CHAR(ORDER_DATE, 'YYYY-MM')
    END AS ORDER_MONTH,
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
GROUP BY ROLLUP (TO_CHAR(ORDER_DATE, 'YYYY-MM'))
ORDER BY
    CASE
        WHEN GROUPING(TO_CHAR(ORDER_DATE, 'YYYY-MM')) = 1 THEN 9999
        ELSE TO_NUMBER(TO_CHAR(ORDER_DATE, 'YYYYMM'))
    END;
```

---

# 3-2. 설명

- GROUPING(컬럼)
→ ROLLUP에서 합계행(TOTAL)이면 1, 일반행이면 0

- CASE WHEN ... THEN 'TOTAL'
→ NULL 대신 TOTAL로 표시

- ORDER BY
→ TOTAL 행을 마지막으로 보내기 위해 숫자로 변환 정렬

👉 실무 보고서/대시보드에서 매우 자주 사용하는 패턴.

---

# 4. 일자별 상세 + 월별 TOTAL 같이 출력하기

## 4-1. 문제 정의

👉 “일별 매출 + 월 전체 합계를 같은 보고서에 넣고 싶다.”

예시:

일별
2024-01-01
2024-01-02
…
월별 TOTAL
2024-01 TOTAL

---

## 4-2. SQL 코드
```sql
SELECT
    CASE
        WHEN GROUPING(TO_CHAR(ORDER_DATE, 'YYYY-MM-DD')) = 1
             THEN TO_CHAR(ORDER_DATE, 'YYYY-MM') || ' TOTAL'
        ELSE TO_CHAR(ORDER_DATE, 'YYYY-MM-DD')
    END AS ORDER_DATE_LABEL,
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
GROUP BY ROLLUP (
    TO_CHAR(ORDER_DATE, 'YYYY-MM'),
    TO_CHAR(ORDER_DATE, 'YYYY-MM-DD')
)
ORDER BY
    TO_CHAR(ORDER_DATE, 'YYYY-MM'),
    TO_CHAR(ORDER_DATE, 'YYYY-MM-DD');
```

---

# 4-3. 설명

- ROLLUP(월, 일)
→ (1) 월+일(일자별), (2) 월 총합 생성

- GROUPING으로 구분
→ 월 TOTAL인지 일자 데이터인지 구분 가능

👉 실무에서 “일자 상세 + 월별 요약” 보고서 만들 때 자주 사용.

---

# 5. 실무 팁

- GROUP BY + ROLLUP
→ 월/일/분기/반기 보고서에서 가장 많이 쓰는 패턴

- GROUPING 함수
→ TOTAL 라벨 처리 필수

- 숫자로 변환하여 정렬 (TO_NUMBER)
→ 날짜 문자열 정렬 문제 해결

- 인덱스 컬럼 변환 주의
→ TO_CHAR(ORDER_DATE, 'YYYY-MM') 사용 시 함수 기반 인덱스 추천

---

# ✔ 요약

- GROUP BY → 기본 집계

- ROLLUP → TOTAL / 합계 행 자동 생성

- GROUPING → TOTAL 여부 판단

- 보고서·대시보드·BI SQL에서 필수 기술
