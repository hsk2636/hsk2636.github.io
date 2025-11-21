---
title: Subquery / Inline View / EXISTS 활용
---

# 서브쿼리 / 인라인 뷰 / EXISTS

이 문서는 Oracle SQL에서 자주 사용하는  
**서브쿼리(Subquery), 인라인 뷰(Inline View), EXISTS 최적화** 패턴을 정리한 페이지입니다.

특히 “최근 주문 고객”, “값 계산 후 JOIN”, “존재 여부 체크”처럼  
실무 시나리오 기반으로 설명합니다.

---

# 1. 인라인 뷰로 고객별 최근 주문일자 조회

## 1-1. 문제 정의

두 테이블이 있다고 하자.

- `TB_CUSTOMER` : 고객 정보  
- `TB_ORDER` : 주문 정보  

목표:

👉 **최근 3개월 동안 주문한 고객들의 ‘최근 주문일자(LAST_ORDER_DATE)’를 조회하기**

즉 아래 정보가 필요:

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

1-3. 설명

- 인라인 뷰 (SELECT ... recent)
→ 고객별로 최근 주문 날짜(MAX ORDER_DATE)를 계산

- WHERE ORDER_DATE >= ADD_MONTHS(...)
→ 최근 3개월만 필터링

- 바깥 SELECT 에서 고객 테이블과 JOIN
→ 고객 이름 + 최근 주문일자를 함께 조회

- 정렬
→ 최근 주문 고객이 위로 오게 DESC 정렬

👉 실무에서

- “고객별 마지막 활동일”

- “최근 접속일 / 최근 구매일”
  이런 데이터 뽑을 때 매우 많이 쓰는 패턴.

---

2. EXISTS로 최근 주문 고객 필터링

2-1. 문제 정의

이번 문제는 단순하다.

👉 “최근 3개월 동안 주문한 적이 있는 고객만 조회하고 싶다.”

최근 주문일자를 알고 싶지는 않다.
단지 “주문 이력이 있느냐?”만 중요하다.

이럴 때는 EXISTS가 가장 빠르고 효율적이다.

---

2-2. SQL 코드

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

2-3. 설명

- EXISTS (SELECT 1 FROM …)
→ 서브쿼리가 1건이라도 발견되면 TRUE

- 결과 값을 반환하는 목적이 아니라
→ “존재 여부 체크”를 위한 최적화 구조

- 특히 대용량 테이블에서는
→ IN, JOIN 보다 EXISTS가 더 빠른 경우 많음

👉 실무에서

- “최근 주문한 고객 목록”

- “유효한 데이터만 골라내기”

- “관련 테이블에 데이터가 있는 행만 조회”
  같은 상황에서 매우 자주 사용된다.

---

3. 서브쿼리 vs 인라인 뷰 vs EXISTS 비교
기능	목적	장점	단점
서브쿼리	SELECT 안에서 값 계산	단순함	복잡해지면 가독성 ↓
인라인 뷰	계산 결과를 다시 JOIN	구조적이고 직관적	VIEW가 길면 코드 길어짐
EXISTS	존재 여부만 체크	빠르고 효율적	값 반환에는 사용 불가

---

✔️ 요약

- 인라인 뷰 → 데이터를 계산해서 JOIN 하고 싶을 때

- EXISTS → 조건 만족하는 행이 존재하는지 여부가 중요할 때

- 대량 데이터에서는 EXISTS 가 매우 좋은 최적화 방법
