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

- TB_CUSTOMER : 고객 정보  
- TB_ORDER : 주문 정보  

👉 **주문이 있는 고객만 대상으로 주문 건수 + 총 금액을 조회하고 싶다.**

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

- INNER JOIN → 주문이 있는 고객만 조회됨
- COUNT / SUM → 건수 및 총액 집계
- ORDER BY → 금액이 큰 순서 정렬
- 실무에서 매출 상위 고객 TOP N 조회할 때 자주 사용되는 구조

---

# 2. LEFT JOIN + NVL (주문 없는 고객 포함)

## 2-1. 문제 정의

👉 “주문을 한 적이 없는 고객도 목록에 포함하고 싶다.”

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

- LEFT JOIN → 고객은 모두 나오고, 주문이 없는 고객도 포함
- SUM() 결과가 NULL이므로 NVL 처리 필요
- 실무에서 “전체 고객 + 거래 여부” 보고서에 자주 등장

---

# 3. 실무 팁

- INNER JOIN → 활동 고객, 거래 고객만 보고 싶을 때
- LEFT JOIN → 전체 고객 기준으로 보되, 거래 데이터는 옵션일 때
- NVL/NVL2 → NULL 값 보정할 때 중요

---
