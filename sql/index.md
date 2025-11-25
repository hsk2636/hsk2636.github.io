---
title: Oracle SQL 포트폴리오
---

# Oracle SQL 포트폴리오

Oracle SQL(오라클 SQL)을 활용해서  
**분석(Analysis) · 집계(Aggregation) · 서브쿼리(Subquery) · 성능 튜닝(Performance Tuning)** 을  
실습 중심으로 정리한 포트폴리오입니다.

각 문서는 **독립된 학습 노트**이면서,  
실무에서 바로 쓸 수 있는 SQL 패턴 중심으로 구성했습니다.

---

## 📚 전체 구성 (Contents)

---

### 1. SELECT / JOIN 기본 및 응용

기초 SELECT 문법부터 INNER JOIN / LEFT JOIN,  
NVL, GROUP BY 집계까지 **가장 자주 사용하는 기본 패턴**을 정리한 문서입니다.

- 고객·주문 테이블을 가정한 예제
- INNER JOIN / LEFT JOIN 비교
- NVL을 사용한 NULL 보정
- 실무 보고서에서 자주 쓰는 집계 구조

👉 **바로 보기:**  
[01_select_join.md](01_select_join.md)

---

### 2. GROUP BY / 집계함수 / ROLLUP

월별/일별 집계, TOTAL 행 생성, GROUPING 함수까지  
**보고서·대시보드용 집계 SQL** 에서 필수인 패턴들을 정리했습니다.

- GROUP BY + 집계함수(SUM, COUNT 등)
- ROLLUP을 이용한 합계(TOTAL) 행 생성
- GROUPING 함수로 TOTAL 행 라벨링
- 월별/일별 + 합계가 동시에 나오는 패턴

👉 **바로 보기:**  
[02_groupby_rollup.md](02_groupby_rollup.md)

---

### 3. 서브쿼리 / 인라인 뷰 / EXISTS

서브쿼리(Subquery), 인라인 뷰(Inline View), EXISTS를 활용해서  
**최근 주문 고객, 마지막 주문일, 존재 여부 필터링** 같은  
실무 최적화 패턴을 정리한 문서입니다.

- 인라인 뷰로 고객별 최근 주문일(MAX ORDER_DATE) 계산
- 최근 3개월 주문 고객만 필터링하는 EXISTS 패턴
- IN vs EXISTS vs JOIN 비교표
- 대용량 데이터에서 EXISTS가 유리한 경우 정리

👉 **바로 보기:**  
[03_subquery_exists.md](03_subquery_exists.md)

---

### 4. EXPLAIN PLAN / AUTOTRACE 기반 성능 튜닝

EXPLAIN PLAN(실행 계획)과 AUTOTRACE를 활용해서  
**Full Table Scan → Index Range Scan** 으로 개선하는  
SQL 성능 튜닝(Performance Tuning) 과정을 정리한 문서입니다.

- 느린 SQL 시나리오 정의
- EXPLAIN PLAN으로 실행 계획 분석
- 인덱스(Index) 생성 + 날짜 조건 튜닝(BETWEEN → 범위 조건)
- AUTOTRACE로 튜닝 전/후 I/O · 시간 비교
- WHERE 절 함수 사용, 통계정보(Statistics), 인덱스 남발 주의점 정리

👉 **바로 보기:**  
[04_performance_tuning.md](04_performance_tuning.md)

---

## 🧩 문서 공통 구성

각 문서는 공통적으로 아래 흐름으로 작성되어 있습니다.

1. **문제 정의(Problem Definition)**  
   - 어떤 테이블 구조를 가정하는지  
   - 무엇을 구하고 싶은지(업무 요구사항)  

2. **SQL 코드(SQL Query)**  
   - Oracle SQL 기준 예제 쿼리  
   - SELECT / JOIN / GROUP BY / 서브쿼리 / 튜닝 쿼리 등  

3. **해설(Explanation)**  
   - 각 구문이 어떤 역할을 하는지  
   - 실무에서 자주 쓰이는 이유  

4. **실무 팁(Practical Tips)**  
   - 보고서 / 집계 / 로그 분석 / 제조(MES·PDA·ERP) 업무에  
     어떻게 응용할 수 있는지 정리

---

## 🎯 작성 목적

이 SQL 포트폴리오는 다음을 목표로 합니다.

- Oracle SQL 문법을 **실제 업무 흐름**에 맞게 이해  
- JOIN / 집계 / 서브쿼리 / 성능 튜닝을 **패턴 단위**로 정리  
- Oracle PL/SQL 프로시저 분석 능력과 연결  
- 제조 IT(MES/PDA/ERP) 환경에서 필요한 SQL 이해도 향상

앞으로 필요에 따라:

- 윈도우 함수(Window Function: ROW_NUMBER, RANK, LAG/LEAD)
- 분석 함수(Analytic Function)를 활용한 KPI 계산
- 추가적인 성능 튜닝 케이스

등을 새 문서로 확장해 나갈 예정입니다.
