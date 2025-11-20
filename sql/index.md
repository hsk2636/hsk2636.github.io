---
title: Oracle SQL 포트폴리오 (Main)
---

# Oracle SQL 포트폴리오

Oracle SQL을 활용해 분석·집계·튜닝을 수행한 실습과 정리 자료입니다.  
각 파트는 **독립된 학습 문서**로 구성되어 있으며,  
실무형 SQL 패턴들을 단계별로 정리해 두었습니다.

---

# 📚 전체 구성 (Contents)

## 1. SELECT / JOIN 기본 및 응용  
기초 SELECT 문법부터 INNER JOIN / LEFT JOIN,  
NVL, GROUP BY 응용까지 실무에서 가장 많이 사용하는 패턴 정리.

👉 바로 보기  
**[01_select_join.md](01_select_join.md)**  

---

## 2. GROUP BY / 집계함수 / ROLLUP  
월별/일별 집계, TOTAL 행 생성, GROUPING 함수 사용법 등  
보고서·대시보드 제작 시 필요한 집계 SQL 실습.

👉 바로 보기  
**[02_groupby_rollup.md](02_groupby_rollup.md)**  

---

## 3. 서브쿼리 / 인라인 뷰 / EXISTS  
최근 주문 고객 조회, 고객별 마지막 주문일 계산,  
조건 필터링 최적화(EXISTS) 등 실무 최적화 패턴 정리.

👉 바로 보기  
**[03_subquery_exists.md](03_subquery_exists.md)**  

---

## 4. EXPLAIN PLAN / AUTOTRACE 기반 성능 튜닝  
실행 계획 해석, FULL SCAN 원인 분석,  
인덱스 생성 효과, AUTOTRACE를 통한 성능 비교 실습.

👉 바로 보기  
**[04_performance_tuning.md](04_performance_tuning.md)**  

---

# ✔️ 목적

이 SQL 포트폴리오는 다음을 목표로 작성되었습니다:

- 실무에서 자주 사용하는 SQL 구조와 패턴 이해  
- JOIN / 집계 / 서브쿼리 / 성능 튜닝 능력 강화  
- Oracle PL/SQL 프로그래밍과 연계된 SQL 분석 능력 향상  
- 데이터 기반 제조(MES/PDA/ERP) 업무에 필요한 SQL 능력 정리  

---

# ✔️ 앞으로 확장될 문서

- 05. 윈도우 함수(ROW_NUMBER, RANK, LAG/LEAD)  
- 06. 분석함수 + 대시보드용 KPI 계산  
- 07. 대용량 테이블 성능 튜닝 추가 케이스  

필요하면 언제든지 새 문서로 확장할 수 있습니다.
