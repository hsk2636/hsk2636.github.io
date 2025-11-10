---
layout: single
title: "SQL 학습 및 실습 프로젝트"
permalink: /projects/
toc: true
---

# 🧩 SQL 학습 및 실습 로드맵

SQL 개발자로 성장하기 위해 진행 중이거나 예정인 실습들을 정리합니다.

## 1. 기본 SQL 문법 실습

- SELECT, WHERE, ORDER BY, GROUP BY
- 단일 테이블 조회, 기본 집계 함수
- 목표: 안정적인 기본기 확보

## 2. 조인(Join) & 서브쿼리 심화

- INNER / LEFT / RIGHT / FULL OUTER JOIN
- 서브쿼리, IN / EXISTS 활용
- 목표: 다중 테이블 기반 조회 로직 설계 능력

## 3. PL/SQL 프로그래밍

- Stored Procedure, Function
- Trigger, Exception Handling
- 목표: 업무 로직을 DB 레벨에서 구현하는 경험

## 4. 성능 튜닝 & 실행 계획 분석

- EXPLAIN PLAN, AUTOTRACE 실습
- 인덱스 적용 전/후 성능 비교
- 목표: 느린 쿼리 원인 분석 및 개선 능력

## 5. 미니 프로젝트 (예정)

- 판매/생산 데이터 기반 KPI 분석
- ERP 데이터 흐름을 가정한 시나리오 설계
- 결과를 개별 포스트로 공개 예정

---

## 📊 SQL 기반 프로젝트 계획표

## 📊 SQL 기반 프로젝트 계획표

| 구분 | 프로젝트명 | 주요 기술 | 진행 상태 |
|------|-------------|------------|-------------|
| 1 | [생산 불량률 분석](/sql/2025/11/10/project-defect-analysis.html) | Oracle SQL / PL/SQL | 진행 중 |
| 2 | [매출 데이터 요약 자동화](/sql/2025/11/10/project-sales-summary.html) | SQL / Python | 준비 중 |
| 3 | [ERP 연계 데이터 검증](/sql/2025/11/10/project-erp-validation.html) | SQL / Excel / ERP API | 예정 |


---

### 🔍 프로젝트 진행 방향
- 실제 ERP 테이블 구조와 유사한 샘플 데이터 사용  
- SQL 튜닝(EXPLAIN PLAN, AUTOTRACE) 중심 성능 실험  
- PL/SQL 기반 데이터 정제 및 배치 로직 구현  
- 시각화 결과는 추후 `/posts` 형태로 공개 예정  

> 💡 이 표는 프로젝트 현황을 간단히 보여주기 위한 **개요 대시보드** 역할을 합니다.  
> 각 프로젝트가 완료되면 세부 내용은 별도 포스트로 연결될 예정입니다.
