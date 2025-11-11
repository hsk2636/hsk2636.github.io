---
layout: single
title: "GROUP BY와 HAVING 실습 정리"
date: 2025-11-09
categories: [SQL]
tags: [GROUP BY, HAVING, 집계함수]
excerpt: "GROUP BY와 HAVING을 이용해 데이터 집계와 조건 필터링을 실습한 노트."
toc: true
---

## 1️⃣ 실습 목적

- 집계 함수(SUM, AVG, COUNT 등)와 GROUP BY의 관계 이해  
- HAVING 절로 그룹 조건 필터링 실습  
- 중복 데이터 처리와 집계 효율화  

---

## 2️⃣ 기본 GROUP BY 예제

```sql
SELECT dept_id,
       COUNT(*) AS 인원수,
       AVG(salary) AS 평균급여
FROM employees
GROUP BY dept_id;
```

**포인트**  
- GROUP BY는 SELECT 절의 비집계 컬럼과 일치해야 함  
- 부서별, 제품별, 월별 등 “단위별 집계” 작성 시 필수  

---

## 3️⃣ HAVING 절 적용

```sql
SELECT dept_id,
       COUNT(*) AS 인원수
FROM employees
GROUP BY dept_id
HAVING COUNT(*) >= 5;
```

**포인트**  
- WHERE는 행(row) 기준, HAVING은 그룹(group) 기준  
- “특정 부서 인원이 5명 이상인 경우만” 필터링 가능  

---

## 4️⃣ 실습 결과 요약

| 조건 | 사용 위치 | 의미 | 예시 |
|------|------------|------|------|
| WHERE | GROUP BY 이전 | 개별 행 필터 | `salary > 3000` |
| HAVING | GROUP BY 이후 | 그룹 필터 | `HAVING COUNT(*) > 5` |

---

## 5️⃣ 실무 팁  
- WHERE와 HAVING을 혼용할 때는 **WHERE → GROUP BY → HAVING** 순서  
- “월별 매출 TOP 5” 같은 문제에서 두 절의 차이를 명확히 알아야 함  

---

## 6️⃣ GROUP BY 심화: ROLLUP / CUBE / GROUPING SETS

```sql
SELECT region,
       product,
       SUM(sales_amount) AS total_sales
FROM sales_data
GROUP BY ROLLUP(region, product);
```

---

**포인트**

ROLLUP: 상위 그룹의 합계를 자동 계산 (예: 지역별 → 제품별 → 전체)  
CUBE: 가능한 모든 조합의 집계를 계산  
GROUPING SETS: 여러 GROUP BY 결과를 하나로 결합  

---

🔍 비교 요약
| 구분 | 기능 | 결과 범위 | 특징 |
|-------------|---------|--------|----------|
| ROLLUP | 계층형 집계 | 상위 합계 포함 | `총계` 자동 추가 |
| CUBE | 다차원 조합 집계 | 모든 조합 계산 | 분석용 통계에 적합 |
| GROUPING SETS | 선택적 집계 | 지정된 집합만 | 성능 효율적 |

---

7️⃣ 실행 순서 복습

| 단계 | 절(Clause)   | 설명         |
| -- | ----------- | ---------- |
| 1  | FROM / JOIN | 데이터 소스 지정  |
| 2  | WHERE       | 행(Row) 필터링 |
| 3  | GROUP BY    | 그룹 생성      |
| 4  | HAVING      | 그룹 필터링     |
| 5  | SELECT      | 최종 컬럼 선택   |
| 6  | ORDER BY    | 결과 정렬      |

핵심 요약
- WHERE는 개별 데이터, HAVING은 집계 결과 기준  
- 실행 순서를 이해하면 집계 성능과 필터링 효율이 확실히 달라짐  
- 실제 리포트 SQL 작성 시 HAVING COUNT(*) > 0 같은 조건 자주 등장함  

---

8️⃣ 마무리 정리

- GROUP BY → HAVING → ORDER BY 흐름은 모든 분석 쿼리의 기본  
- 실무에서는 ROLLUP + HAVING 조합으로 “총합 제외 / 포함” 제어도 가능  
- 다음 실습: “월별 매출 TOP 5 추출 + 누적합(Analytic Function)”  

---
