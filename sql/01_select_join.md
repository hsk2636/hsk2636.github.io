---
title: SELECT / JOIN 기본 및 응용
---

# SELECT / JOIN 기본 및 응용

이 문서는 Oracle SQL에서 가장 기본이면서 실무에서 가장 자주 등장하는  
**JOIN + GROUP BY 집계** 관련 내용을 정리한 페이지입니다.

---

# 1. INNER JOIN 기본 예제

## 1-1. 문제 정의

두 테이블 `TB_CUSTOMER`, `TB_ORDER`가 있다고 가정합니다.

- `TB_CUSTOMER` : 고객 정보 테이블  
- `TB_ORDER` : 주문 정보 테이블   

요구사항:

> 주문이 있는 고객만 대상으로 **주문 건수**와 **총 주문 금액**을 조회한다.

---

## 1-2. SQL 코드

```sql
SELECT
    c.CUST_ID,
    c.CUST_NAME,
    COUNT(o.ORDER_ID) AS ORDER_CNT,
    SUM(o.ORDER_AMT)  AS ORDER_SUM
FROM TB_CUSTOMER c
JOIN TB_ORDER o
  ON c.CUST_ID = o.CUST_ID
GROUP BY
    c.CUST_ID,
    c.CUST_NAME
ORDER BY ORDER_SUM DESC;
```

---

## 1-3. 설명

- INNER JOIN
→ 양쪽 테이블(TB_CUSTOMER, TB_ORDER)에 모두 존재하는 데이터만 조회합니다.
→ 즉, “주문 이력이 있는 고객”만 대상이 됩니다.

- COUNT(o.ORDER_ID)
→ 고객별 주문 건수를 계산합니다.

- SUM(o.ORDER_AMT)
→ 고객별 총 주문 금액을 계산합니다.

- GROUP BY
→ 고객 ID, 고객 이름 기준으로 집계를 수행합니다.

- ORDER BY ORDER_SUM DESC
→ 총 주문 금액이 큰 고객부터 내림차순으로 정렬합니다.

---

# 2. LEFT JOIN + NVL (주문 없는 고객 포함)

## 2-1. 문제 정의

이번에는 다음 요구사항을 가정합니다.

“주문을 한 적이 없는 고객도 목록에 모두 포함하고 싶다.”

즉, 전체 고객 목록을 기준으로 주문 정보를 붙이되, 주문이 없으면 0으로 보정하는 형태가 필요합니다.

---

## 2-2. SQL 코드

```sql
SELECT
    c.CUST_ID,
    c.CUST_NAME,
    COUNT(o.ORDER_ID)         AS ORDER_CNT,
    NVL(SUM(o.ORDER_AMT), 0)  AS ORDER_SUM
FROM TB_CUSTOMER c
LEFT JOIN TB_ORDER o
       ON c.CUST_ID = o.CUST_ID
GROUP BY
    c.CUST_ID,
    c.CUST_NAME
ORDER BY ORDER_SUM DESC;
```

## 2-3. 설명

- LEFT JOIN
→ 왼쪽 테이블(TB_CUSTOMER)을 기준으로 모든 고객이 조회됩니다.
→ 주문이 없는 고객의 경우, 오른쪽 테이블(TB_ORDER) 컬럼이 NULL이 됩니다.

- COUNT(o.ORDER_ID)
→ 주문이 없으면 0건으로 집계됩니다.

- SUM(o.ORDER_AMT)
→ 주문이 없는 고객의 경우 합계가 NULL이 되므로 보고서에서 사용하기 위해 NVL(SUM(...), 0)으로 0으로 보정합니다.

이 패턴은 다음과 같은 상황에서 자주 사용됩니다.

- 전체 고객 리스트 + 거래 여부 표시

- 전체 품목 리스트 + 판매 실적 유무 확인

- “휴면 고객”, “거래 중단 고객”을 찾는 분석 기준으로 활용

---

# 3. INNER JOIN vs LEFT JOIN 비교

| 구분         | 특징                                     | 대표적인 사용 목적                      |
| ---------- | -------------------------------------- | ------------------------------- |
| INNER JOIN | 양쪽 테이블 모두에 존재하는 행만 조회                  | 거래가 있는 고객만, 재고가 있는 품목만 조회할 때    |
| LEFT JOIN  | 왼쪽 테이블의 모든 행을 기준으로, 없는 쪽은 NULL로 채워서 조회 | 전체 고객/품목 기준으로 거래 여부를 함께 보고 싶을 때 |

실무에서는

활동 고객 / 실적 있는 품목만 보고 싶을 때 → INNER JOIN

전체 기준(고객·품목·거래처)을 모두 포함해서 보고 싶을 때 → LEFT JOIN

과 같이 선택하는 것이 일반적입니다.

---

# 4. NVL / NVL2를 이용한 NULL 처리

## 4-1. NVL 기본 사용 예제

```sql
SELECT
    c.CUST_ID,
    c.CUST_NAME,
    NVL(SUM(o.ORDER_AMT), 0) AS ORDER_SUM
FROM TB_CUSTOMER c
     LEFT JOIN TB_ORDER o
        ON c.CUST_ID = o.CUST_ID
GROUP BY
    c.CUST_ID,
    c.CUST_NAME;
```

- NVL(expr, value)
→ expr가 NULL인 경우 value로 대체합니다.
→ 합계·건수 등 숫자형 컬럼을 보고서에 표시할 때 NULL 대신 0으로 보정하는 데 자주 사용합니다.

---

## 4-2. NVL2 간단 예제

```sql
SELECT
    c.CUST_ID,
    c.CUST_NAME,
    NVL2(o.ORDER_ID, '주문 있음', '주문 없음') AS ORDER_YN
FROM TB_CUSTOMER c
     LEFT JOIN TB_ORDER o
        ON c.CUST_ID = o.CUST_ID;
```

---

- NVL2(expr, value_if_not_null, value_if_null)
→ expr가 NULL이 아니면 두 번째 인자, NULL이면 세 번째 인자를 반환합니다.

- 주문 존재 여부를 “주문 있음 / 주문 없음”으로 표시할 때 활용할 수 있습니다.

---

# 5. 실무 관점 정리

1. JOIN 기준 키(조인 키)를 먼저 파악하는 것이 중요합니다.

    - 고객–주문: CUST_ID

    - 품목–재고: ITEM_ID

    - LOT–공정이력: LOT_NO

2. INNER JOIN과 LEFT JOIN의 차이를 명확히 이해해야 합니다.

    - “누가 기준인지”를 먼저 정한 후, 기준이 되는 테이블을 FROM/LEFT에 두는 방식으로 설계합니다.

3. NVL을 이용해 NULL 값을 명시적으로 처리합니다.

    - 집계 결과나 보고서 숫자 컬럼에서는 NULL이 그대로 남아 있으면  
      해석하기 어렵기 때문에, 대부분 0 또는 지정된 기본값으로 보정합니다.

# 6. 요약

- INNER JOIN은 교집합(공통 데이터) 을 조회하는 패턴입니다.

- LEFT JOIN은 기준 테이블의 전체 데이터를 유지하면서 부족한 부분을 NULL로 채우는 패턴입니다.

- NVL/NVL2를 함께 사용하면 거래 여부, 주문 유무, 실적 유무를 명확하게 표현할 수 있습니다.

- 제조·물류·판매 환경에서 고객, 품목, 재고, LOT 데이터를 다룰 때
  이 문서의 JOIN + GROUP BY 패턴을 그대로 응용하여 사용할 수 있습니다.
