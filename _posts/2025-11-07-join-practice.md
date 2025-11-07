---
layout: single
title: "조인(Join) 문법 심화 실습 정리"
date: 2025-11-07
categories: [SQL]
tags: [JOIN, INNER JOIN, LEFT JOIN, SELF JOIN]
excerpt: "INNER/LEFT/SELF JOIN을 활용해 실제 테이블 관계를 설계하고 비교한 실습 노트."
toc: true
---

## 1️⃣ 실습 목적

- 여러 테이블을 연결하는 조인 패턴을 정확히 이해  
- 잘못된 조인으로 인한 중복/누락 데이터를 방지  
- 향후 리포트·통계 쿼리 작성의 기반 마련  

---

## 2️⃣ 기본 INNER JOIN 예제

```sql
SELECT e.emp_id,
       e.emp_name,
       d.dept_name
FROM employees e
JOIN departments d
  ON e.dept_id = d.dept_id;
```

포인트
ㆍ조건 누락 시 카티전 곱(Cartesian product) 발생  
ㆍ별칭(alias) 사용으로 가독성 향상  
ㆍ가장 기본적이면서도 실무에서 가장 자주 쓰이는 형태  

---

3️⃣ LEFT JOIN vs INNER JOIN 비교

```sql
SELECT d.dept_name,
       e.emp_name
FROM departments d
LEFT JOIN employees e
  ON d.dept_id = e.dept_id;
```

비교 요약
| 구분 | 기준 테이블 | 결과 범위 | 특징 |
|------|--------------|------------|------|
| INNER JOIN | 두 테이블 모두 존재하는 데이터만 | 교집합 | 누락된 데이터 발생 가능 |
| LEFT JOIN  | 왼쪽 테이블(departments) 기준 | 왼쪽 전체 + 매칭되는 오른쪽 | NULL 포함 가능 |


해석 포인트

ㆍ직원이 없는 부서도 포함해야 할 때 LEFT JOIN 사용  
ㆍ반면, INNER JOIN은 직원이 없는 부서는 제외됨  
ㆍ실무 리포트에서 “부서별 인원 수” 계산 시 LEFT JOIN이 필수적임  

---

4️⃣ SELF JOIN 예제 (관리자-사원 관계)

```
sql
코드 복사
SELECT e.emp_name AS employee,
       m.emp_name AS manager
FROM employees e
LEFT JOIN employees m
  ON e.manager_id = m.emp_id;
```

핵심 포인트

ㆍ동일한 테이블을 두 번 사용하므로 별칭(alias) 필수  
ㆍ관리자가 없는 경우 manager 컬럼에 NULL 표시  
ㆍ조직도, 상하 관계 분석 등에 자주 사용됨  

---

5️⃣ 실습 후기

핵심 포인트

ㆍ조인 조건을 명시적으로 작성하지 않으면 데이터가 꼬이기 쉬움  
ㆍERD와 함께 생각하며 쿼리를 짜면 구조 이해가 훨씬 빠름  
ㆍINNER JOIN → LEFT JOIN → SELF JOIN 순서로 연습하니 자연스럽게 구조적 사고가 생김  

---
