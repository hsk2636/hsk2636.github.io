---
layout: single
title: "프로젝트 1: 생산 불량률 분석"
date: 2025-11-10
categories: [Project, SQL]
tags: [불량률, 생산, Oracle, PLSQL]
excerpt: "생산 공정 데이터를 기반으로 불량률을 분석하고, SQL과 PL/SQL을 활용한 품질 개선 인사이트를 도출합니다."
toc: true
---

## 🎯 프로젝트 개요
- 주제: 생산 데이터 기반 불량률 통계  
- 목표: LOT 단위 불량률 계산, 공정별 원인 파악  
- 사용 기술: Oracle SQL, PL/SQL  

---

## 🧩 핵심 SQL 구조 예시

```sql
SELECT process_name,
       ROUND(SUM(bad_qty)/SUM(total_qty)*100,2) AS defect_rate
FROM production_data
GROUP BY process_name
ORDER BY defect_rate DESC;
```

**포인트**  
- 공정별로 불량률을 계산해 품질 문제 우선순위 도출  
- OUTER JOIN을 통해 누락 데이터 방지  
- PL/SQL 커서(cursor)를 사용해 자동 리포트 생성 예정  

---

## 📈 향후 개선 계획
- 불량 유형별 상세 분석 페이지 추가  
- KPI 대시보드 형태로 시각화 연결
