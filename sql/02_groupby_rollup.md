---
title: GROUP BY / 집계함수 / ROLLUP
---

# GROUP BY / 집계함수 / ROLLUP

이 문서는 Oracle SQL의 핵심 집계 기능인  
**GROUP BY, 집계함수(SUM/COUNT/AVG), ROLLUP, GROUPING 함수**를  
실무 예제로 정리한 학습 페이입니다.

특히 다음과 같은 상황을 다루는 것을 목표로 합니다.

- 월별/일별 매출 집계  
- 전체 합계(TOTAL) 행 생성  
- 일자 상세 + 월별 합계가 함께 있는 보고서 SQL
- 
---

# 1. 기본 GROUP BY 집계

## 1-1. 문제 정의
`TB_ORDER` 테이블이 있다고 가정합니다.

- ORDER_ID  
- ORDER_DATE  
- ORDER_AMT  

요구사항:

> 월별 총 매출 금액을 구하는 SQL을 작성한다.

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
→ 주문일자를 “년-월” 형식 문자열로 변환하여 월 단위로 그룹핑합니다.

- SUM(ORDER_AMT)
→ 각 월별 주문 금액 합계를 계산합니다.

- GROUP BY
→ 월 기준으로 집계를 수행합니다.

이 패턴은 제조·판매·출하 시스템에서 월별 실적 집계를 만들 때 가장 기본이 되는 구조입니다.

---

# 2. ROLLUP으로 월별 + 전체 합계(TOTAL) 만들기

## 2-1. 문제 정의

👉 “월별 매출 + 전체 매출(TOTAL)까지 한 번에 보고 싶다.”

이 경우 GROUP BY ROLLUP 구문을 사용할 수 있습니다.

---

## 2-2. SQL 코드

```sql
SELECT
    TO_CHAR(ORDER_DATE, 'YYYY-MM') AS ORDER_MONTH,
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
GROUP BY ROLLUP (TO_CHAR(ORDER_DATE, 'YYYY-MM'))
ORDER BY ORDER_MONTH;
```

---

## 2-3. 설명

- GROUP BY ROLLUP(컬럼)
→ 각 그룹(월별) 집계 행 + 전체 합계 행을 함께 생성합니다.

- 이때 전체 합계 행의 ORDER_MONTH 값은 NULL로 표시됩니다.

예시:

| ORDER_MONTH | TOTAL_AMT |
|-------------|-----------|
| 2024-01     | 110,000   |
| 2024-02     | 98,000    |
| **TOTAL**   | **208,000** |

NULL 대신 “TOTAL”로 표시하는 것이 일반적입니다.
이를 위해 GROUPING 함수와 CASE 표현식을 함께 사용합니다.

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

## 3-2. 설명

- GROUPING(컬럼)
→ ROLLUP에 의해 생성된 합계(TOTAL) 행일 때 1, 일반 행일 때 0을 반환합니다.

- CASE WHEN GROUPING(...) = 1 THEN 'TOTAL'
→ TOTAL 행에서만 라벨을 “TOTAL”로 변경하고, 나머지는 월 값 그대로 사용합니다.

- ORDER BY 절에서
TOTAL 행이 마지막에 오도록 정렬 기준을 숫자로 변환하여 설정합니다.

이 패턴은 월별 데이터 + 마지막 TOTAL 행이 함께 필요한 보고서에서 매우 자주 사용됩니다.

---

# 4. 일자별 상세 + 월별 TOTAL 같이 출력하기

## 4-1. 문제 정의

요구사항:

“일별 매출과 월 전체 매출 합계를 한 번에 보고 싶다.”

예시 구조:

- 2024-01-01 매출

- 2024-01-02 매출

- …

- 2024-01 월 전체 합계(TOTAL)

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

## 4-3. 설명

- GROUP BY ROLLUP(월, 일)
→ (1) 월+일 기준 일별 행, (2) 월 기준 합계 행을 생성합니다.

- GROUPING(TO_CHAR(ORDER_DATE, 'YYYY-MM-DD')) 값이 1인 경우
→ 해당 행은 “월 전체 합계”에 해당합니다.

- ORDER_DATE_LABEL 컬럼에

    - 일자 데이터는 그대로 YYYY-MM-DD

    - 합계 행은 YYYY-MM TOTAL 형식으로 표시합니다.

이 패턴은 “일별 상세 + 월별 요약” 보고서를 하나의 SQL로 생성할 때 활용할 수 있습니다.

---

# 5. 실무 팁

1. GROUP BY + ROLLUP 조합

    - 월별, 분기별, 연도별 집계와 전체 합계를 동시에 만들 수 있습니다.

    - 품질 지표(불량률), 생산 실적, 출하 실적 보고서에서도 동일한 구조로 적용 가능합니다.

2. GROUPING 함수 활용

    - TOTAL 행을 일반 데이터와 구분하여 라벨을 붙이거나 정렬 순서를 조정할 수 있습니다.

3. 함수 사용과 인덱스 주의

    - TO_CHAR(ORDER_DATE, 'YYYY-MM')처럼 컬럼에 함수를 사용하는 경우,
      인덱스를 활용하기 위해 **함수 기반 인덱스(Function-Based Index)**를 고려해야 합니다.

---

# 6. 요약

- GROUP BY는 집계의 기본 구조입니다.

- ROLLUP을 사용하면 합계(TOTAL) 행을 자동으로 생성할 수 있습니다.

- GROUPING 함수는 TOTAL 행을 인식하여 라벨 처리와 정렬을 제어하는 데 사용됩니다.

- 제조·품질·출하 리포트에서 “상세 + 소계 + 총계” 구조를 SQL 레벨에서 구현할 때
  이 문서의 패턴을 그대로 응용할 수 있습니다.
  
