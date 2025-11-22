---
title: Subquery / Inline View / EXISTS 활용
---

# 서브쿼리 / 인라인 뷰 / EXISTS

이 문서는 Oracle SQL에서 자주 사용하는  
**서브쿼리(Subquery), 인라인 뷰(Inline View), EXISTS 최적화** 패턴을 정리한 문서입니다.

특히 아래 실무 시나리오를 중심으로 구성했습니다.

- “고객별 최근 주문일 조회”
- “조건 계산 후 JOIN”
- “존재 여부만 빠르게 체크(EXISTS)”
- “IN vs EXISTS 성능 차이”

---

# 1. 인라인 뷰로 고객별 최근 주문일자 조회

## 1-1. 문제 정의

두 테이블이 있다고 하자.

- `TB_CUSTOMER` : 고객 정보  
- `TB_ORDER` : 주문 정보  

목표 👉  
**최근 3개월 동안 주문한 고객들의 ‘최근 주문일자(LAST_ORDER_DATE)’를 조회하기**

필요한 데이터:

- 고객 ID  
- 고객 이름  
- 최근 주문일자  

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

- 인라인 뷰에서 고객별 최근 주문일(MAX ORDER_DATE) 계산

- WHERE 절에서 최근 3개월만 필터링

- 외부 SELECT 에서 고객 테이블과 JOIN
→ 고객 이름 + 최근 주문일을 함께 조회

👉 실무 활용 예

- 고객별 마지막 활동일 조회

- 최근 방문일 / 최근 구매일

- 휴먼 고객(Hibernation 고객) 분석

---

# 2. EXISTS로 최근 주문 고객 필터링

## 2-1. 문제 정의

이번 목표 👉
최근 3개월 동안 주문한 적이 있는 고객만 조회하고 싶다.

최근 주문일자까지는 필요 없음.
“존재 여부”만 중요할 때 최적화되는 방식이 EXISTS 이다.

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

- EXISTS → 서브쿼리가 1건이라도 존재하면 TRUE

- 반환 값이 목적이 아니라, 행 존재 여부만 체크

- 대용량 테이블에서 JOIN 또는 IN 보다 빠른 경우 많음

👉 실무 활용 예

- 최근 주문한 고객 목록

- 재고가 존재하는 제품 조회

- 이력 테이블에서 “최신 이력 존재하는 데이터만” 조회

---

# 3. IN vs EXISTS 성능 비교

## 3-1. 문제 정의

둘 다 조건 필터링으로 사용 가능하지만, 성능 차이가 존재한다.

- IN → 서브쿼리 결과를 먼저 메모리에 적재

- EXISTS → 조건 충족하는 순간 TRUE 반환 (빠름)

## 3-2. 비교표

| 방식     | 동작 방식           | 장점          | 단점               |
| ------ | --------------- | ----------- | ---------------- |
| IN     | 서브쿼리 전체 결과와 비교  | 코드 단순       | 대용량 서브쿼리에서 느림    |
| EXISTS | 조건 충족 시 즉시 TRUE | 빠르고 최적화됨    | 값 반환할 때는 사용 불가   |
| JOIN   | 양쪽 테이블 결합       | 다중 컬럼 비교 가능 | 불필요한 JOIN 시 오버헤드 |

---

# 4. 요약

✔ 인라인 뷰 → “계산 → JOIN” 구조에 적합  

✔ EXISTS → “존재 여부만 중요”한 조건 필터링 최강  

✔ 대량 데이터일수록 EXISTS 성능 효과가 큼  

✔ IN, JOIN과 혼용 가능하지만 목적에 맞게 선택해야 함  
