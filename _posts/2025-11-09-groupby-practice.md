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
