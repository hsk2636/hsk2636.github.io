---
title: Oracle SQL 포트폴리오
---

# Oracle SQL 포트폴리오

Oracle SQL을 활용한 **데이터 분석 + 성능 튜닝(SQL Performance Tuning)** 실습 내용을 정리한 페이지입니다.  
각 예제는 아래 흐름을 기준으로 작성했습니다.

1. 문제 정의(Problem Definition)  
2. 초기 SQL 코드(Initial Query)  
3. 실행 결과 또는 실행 계획(Result / Plan)  
4. 성능/로직 상 이슈 분석(Issue Analysis)  
5. 개선 쿼리 및 정리(Improved Query & Summary)  

---

## 🗂 목차 (Contents)

1. [SELECT / JOIN 기본 및 응용](#1-select--join-기본-및-응용)  
2. [GROUP BY / 집계함수 / ROLLUP](#2-group-by--집계함수--rollup)  
3. [서브쿼리 / 인라인 뷰 / EXISTS](#3-서브쿼리--인라인-뷰--exists)  
4. [EXPLAIN PLAN / AUTOTRACE 기반 성능 튜닝](#4-explain-plan--autotrace-기반-성능-튜닝)  

---

## 1. SELECT / JOIN 기본 및 응용

### 1-1. 문제 정의

두 테이블 `TB_CUSTOMER`, `TB_ORDER`가 있다고 가정합니다.

- `TB_CUSTOMER` : 고객 정보 (CUST_ID, CUST_NAME, REGION 등)  
- `TB_ORDER` : 주문 정보 (ORDER_ID, CUST_ID, ORDER_AMT, ORDER_DATE 등)  

👉 **고객별 주문 건수와 총 주문 금액을 구하고, 금액이 큰 순서로 정렬**하고자 합니다.

---

### 1-2. 초기 SQL 쿼리 (INNER JOIN + GROUP BY)

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
ORDER BY
    ORDER_SUM DESC;
