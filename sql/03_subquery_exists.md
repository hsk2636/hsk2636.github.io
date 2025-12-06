---
title: Subquery / Inline View / EXISTS 활용
---

# 서브쿼리 / 인라인 뷰 / EXISTS

이 문서는 Oracle SQL에서 자주 사용하는  
**서브쿼리(Subquery), 인라인 뷰(Inline View), EXISTS 최적화** 패턴을 정리한 페이입니다.

다음과 같은 실무 시나리오를 중심으로 구성되어 있습니다.

- 고객별 최근 주문일 조회  
- 조건을 계산한 후 JOIN 하는 인라인 뷰 패턴  
- 존재 여부만 빠르게 확인하는 EXISTS  
- IN / EXISTS / JOIN 비교 관점

---

# 1. 인라인 뷰로 고객별 최근 주문일자 조회

## 1-1. 문제 정의

두 테이블이 있다고 하자.

- `TB_CUSTOMER` : 고객 정보 테이블  
- `TB_ORDER` : 주문 정보 테이블  

요구사항:

> 최근 3개월 동안 주문한 고객에 대해  
> “고객 ID, 고객 이름, 최근 주문일(LAST_ORDER_DATE)”을 조회한다.

---

## 1-2. SQL 코드

```sql
SELECT
    c.CUST_ID,
    c.CUST_NAME,
    recent.LAST_ORDER_DATE
FROM TB_CUSTOMER c
JOIN (
    SELECT
        CUST_ID,
        MAX(ORDER_DATE) AS LAST_ORDER_DATE
    FROM TB_ORDER
    WHERE ORDER_DATE >= ADD_MONTHS(TRUNC(SYSDATE, 'MM'), -3)
    GROUP BY CUST_ID
) recent
  ON c.CUST_ID = recent.CUST_ID
ORDER BY recent.LAST_ORDER_DATE DESC;
```

---

## 1-3. 설명

- 내부 서브쿼리(인라인 뷰)

    - 고객별로 최근 주문일(MAX ORDER_DATE)을 계산합니다.

    - 최근 3개월만 대상으로 필터링합니다.

- 외부 SELECT

    - 고객 마스터(TB_CUSTOMER)와 인라인 뷰를 JOIN하여 고객 이름과 최근 주문일을 함께 조회합니다.

이 패턴은 다음과 같은 상황에서 활용할 수 있습니다.

- 고객별 마지막 구매일, 최근 방문일 조회

- 최근 활동 이력을 기준으로 한 “휴면 고객(비활동 고객)” 분석

- 제조/출하 시스템에서 고객별 마지막 출하일 조회

---

# 2. EXISTS로 최근 주문 고객 필터링

## 2-1. 문제 정의

이번에는 “최근 주문일” 자체는 필요 없고,
최근 3개월 이내에 주문한 적이 있는 고객만 알고 싶은 경우를 가정합니다.

요구사항:

최근 3개월 동안 주문한 적이 있는 고객의
고객 ID, 고객 이름, 지역을 조회한다.

## 2-2. SQL 코드

```sql
SELECT
    c.CUST_ID,
    c.CUST_NAME,
    c.REGION
FROM TB_CUSTOMER c
WHERE EXISTS (
    SELECT 1
      FROM TB_ORDER o
     WHERE o.CUST_ID = c.CUST_ID
       AND o.ORDER_DATE >= ADD_MONTHS(TRUNC(SYSDATE, 'MM'), -3)
)
ORDER BY c.CUST_ID;
```
---

## 2-3. 설명

- EXISTS (서브쿼리)
→ 서브쿼리에서 조건을 만족하는 행이 1건이라도 존재하면 TRUE를 반환합니다.
→ 실제 값이 무엇인지보다는 “존재 여부”만 중요할 때 사용하는 패턴입니다.

- 이 예제에서는

    - TB_CUSTOMER의 각 고객에 대해

    - TB_ORDER에 최근 3개월 주문이 존재하는지 확인합니다.

실무 예:

- 최근 주문 이력이 있는 고객 목록

- 재고가 실제로 존재하는 품목 목록

- 최근 점검 이력이 있는 설비/장비 목록

---

# 3. IN vs EXISTS 성능 비교

## 3-1. IN 서브쿼리 예제

```sql
SELECT
    c.CUST_ID,
    c.CUST_NAME
FROM TB_CUSTOMER c
WHERE c.CUST_ID IN (
    SELECT DISTINCT CUST_ID
      FROM TB_ORDER
     WHERE ORDER_DATE >= ADD_MONTHS(TRUNC(SYSDATE, 'MM'), -3)
);
```

- IN (SELECT …) 패턴 역시
“최근 주문이 있는 고객만”을 조회하는 방식입니다.

---

## 3-2. IN vs EXISTS 비교 관점

| 방식     | 동작 방식           | 장점          | 단점               |
| ------ | --------------- | ----------- | ---------------- |
| IN     | 서브쿼리 전체 결과와 비교  | 코드 단순       | 대용량 서브쿼리에서 느림    |
| EXISTS | 조건 충족 시 즉시 TRUE | 빠르고 최적화됨    | 값 반환할 때는 사용 불가   |
| JOIN   | 양쪽 테이블 결합       | 다중 컬럼 비교 가능 | 불필요한 JOIN 시 오버헤드 |

일반적인 가이드라인은 다음과 같습니다.

- 존재 여부만 확인할 때 → EXISTS 선호

- 서브쿼리 결과가 상대적으로 작고 코드 단순성이 중요할 때 → IN

- 실제 조인 후 컬럼을 함께 조회해야 할 때 → JOIN

---

# 4. 인라인 뷰 + EXISTS 혼합 예시

## 4-1. 문제 정의

요구사항:

고객별 최근 주문일을 구하되,
최근 6개월 이내에 주문이 전혀 없는 고객은 제외한다.

---

4-2. SQL 코드

```sql
SELECT
    c.CUST_ID,
    c.CUST_NAME,
    recent.LAST_ORDER_DATE
FROM TB_CUSTOMER c
     INNER JOIN (
        SELECT
            CUST_ID,
            MAX(ORDER_DATE) AS LAST_ORDER_DATE
        FROM TB_ORDER
        GROUP BY CUST_ID
     ) recent
        ON c.CUST_ID = recent.CUST_ID
WHERE EXISTS (
    SELECT 1
      FROM TB_ORDER o
     WHERE o.CUST_ID   = c.CUST_ID
       AND o.ORDER_DATE >= ADD_MONTHS(TRUNC(SYSDATE, 'MM'), -6)
)
ORDER BY
    recent.LAST_ORDER_DATE DESC;
```

- 인라인 뷰로 고객별 최종 주문일을 계산하고,

- WHERE EXISTS로 최근 6개월 이내 주문이 있는 고객만 필터링하는 구조입니다.

---

# 5. 요약

- 서브쿼리는 다른 SELECT 결과를 활용하여 필터링하거나,
  집계 결과를 재사용할 때 쓰이는 중요한 도구입니다.

- 인라인 뷰(Inline View)는 복잡한 집계/조건 로직을
  FROM 절의 임시 테이블처럼 분리하여 가독성과 재사용성을 높이는 방법입니다.

- EXISTS는 존재 여부만 확인하는 조건 필터링에서
  성능과 가독성을 모두 고려할 때 자주 선택되는 패턴입니다.

- IN / EXISTS / JOIN은 서로 대체 가능한 경우가 많지만,
  목적(값 조회 vs 존재 여부)과 데이터 양(대용량 여부)에 따라 적절한 방식을 선택하는 것이 중요합니다.
