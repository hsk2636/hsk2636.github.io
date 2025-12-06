---
title: Oracle SQL 포트폴리오
---

# Oracle SQL 포트폴리오

본 문서는 Oracle SQL(오라클 SQL)을 활용하여  
**분석(Analysis), 집계(Aggregation), 서브쿼리(Subquery), 성능 튜닝(Performance Tuning)** 을  
실습 중심으로 정리한 포트폴리오입니다.

각 문서는 서로 독립적인 학습 노트이면서,  
실무에서 바로 사용할 수 있는 SQL 패턴을 중심으로 구성하였습니다.

---

## 📚 전체 구성 (Contents)

---

### 1. SELECT / JOIN 기본 및 응용

기초 SELECT 문법부터 INNER JOIN / LEFT JOIN,  
NVL, GROUP BY 집계까지 **자주 사용하는 기본 패턴**을 정리한 문서입니다.

- 고객·주문 테이블을 가정한 예제
- INNER JOIN / LEFT JOIN 비교
- NVL을 사용한 NULL 보정 방식
- 실무 보고서에서 자주 사용하는 집계 구조

👉 **바로 보기:**  
[01_select_join.md](01_select_join.md)

---

### 2. GROUP BY / 집계함수 / ROLLUP

월별·일별 집계와 TOTAL 행 생성, GROUPING 함수까지 포함하여  
**보고서·대시보드용 집계 SQL**에 필요한 패턴을 정리한 문서입니다.

- GROUP BY + 집계함수(SUM, COUNT 등) 활용
- ROLLUP을 이용한 합계(TOTAL) 행 생성
- GROUPING 함수를 활용한 TOTAL 행 라벨링
- 월별·일별 데이터와 합계를 동시에 조회하는 패턴

👉 **바로 보기:**  
[02_groupby_rollup.md](02_groupby_rollup.md)

---

### 3. 서브쿼리 / 인라인 뷰 / EXISTS

서브쿼리(Subquery), 인라인 뷰(Inline View), EXISTS를 활용하여  
**최근 주문 고객, 마지막 주문일, 존재 여부 필터링**과 같은  
실무 최적화 패턴을 정리한 문서입니다.

- 인라인 뷰를 활용한 고객별 최근 주문일(MAX ORDER_DATE) 계산
- 최근 3개월 주문 고객만 필터링하는 EXISTS 패턴
- IN vs EXISTS vs JOIN 비교 정리
- 대용량 데이터 환경에서 EXISTS가 유리한 경우 정리

👉 **바로 보기:**  
[03_subquery_exists.md](03_subquery_exists.md)

---

### 4. EXPLAIN PLAN / AUTOTRACE 기반 성능 튜닝

EXPLAIN PLAN(실행 계획)과 AUTOTRACE를 활용하여  
**Full Table Scan을 Index Range Scan으로 개선**하는  
SQL 성능 튜닝(Performance Tuning) 과정을 정리한 문서입니다.

- 느린 SQL 시나리오 정의
- EXPLAIN PLAN을 이용한 실행 계획 분석
- 인덱스(Index) 생성 및 날짜 조건 튜닝 예시
- AUTOTRACE를 활용한 튜닝 전·후 I/O 및 수행 시간 비교
- WHERE 절 함수 사용, 통계정보(Statistics), 인덱스 남용에 대한 주의 사항 정리

👉 **바로 보기:**  
[04_performance_tuning.md](04_performance_tuning.md)

---

## 🧩 문서 공통 구성

각 문서는 공통적으로 아래와 같은 흐름으로 작성하였습니다.

1. **문제 정의 (Problem Definition)**  
   - 가정한 테이블 구조  
   - 해결하고자 하는 업무 요구사항 정의  

2. **SQL 코드 (SQL Query)**  
   - Oracle SQL 기준 예제 쿼리 제시  
   - SELECT / JOIN / GROUP BY / 서브쿼리 / 튜닝 쿼리 등 포함  

3. **해설 (Explanation)**  
   - 각 구문이 수행하는 역할 설명  
   - 실무에서 해당 패턴을 사용하는 이유 정리  

4. **실무 팁 (Practical Tips)**  
   - 보고서·집계·로그 분석·제조(MES·PDA·ERP) 업무에  
     어떻게 응용할 수 있는지에 대한 코멘트 정리  

---

## 🎯 작성 목적

본 Oracle SQL 포트폴리오는 다음과 같은 목적을 가지고 작성하였습니다.

- Oracle SQL 문법을 **실제 업무 흐름**에 맞게 이해하고 설명할 수 있는 능력 정리  
- JOIN, 집계, 서브쿼리, 성능 튜닝을 **재사용 가능한 패턴** 단위로 정리  
- Oracle PL/SQL 프로시저 분석 및 제조 IT(MES/PDA/ERP) 환경 이해와의 연계  
- 제조·품질·물류 데이터를 다루는 실무에서 활용 가능한 SQL 역량을 문서화

위의 네 개 문서를 중심으로 본 Oracle SQL 포트폴리오를 구성하였으며,  
제출용 포트폴리오로서 하나의 완결된 자료가 되도록 정리하였습니다.
