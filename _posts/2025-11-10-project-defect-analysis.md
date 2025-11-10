---
layout: single
title: "프로젝트 1: 생산 불량률 분석"
date: 2025-11-10
categories: [Project, SQL]
tags: [불량률, 생산, Oracle, PL/SQL]
excerpt: "생산 공정 데이터를 기반으로 불량률을 분석하고, SQL과 PL/SQL을 활용해 품질 개선 인사이트를 도출하는 프로젝트."
toc: true
---

## 🎯 프로젝트 개요

- 주제: 생산 데이터 기반 공정별 불량률 분석
- 목표: LOT·공정 단위 불량률 계산 및 문제 구간 식별
- 사용 기술: Oracle SQL, PL/SQL, JOIN, GROUP BY

---

## 🧩 핵심 SQL 예시

```sql
SELECT process_name,
       ROUND(SUM(bad_qty) / NULLIF(SUM(total_qty), 0) * 100, 2) AS defect_rate
FROM production_data
GROUP BY process_name
ORDER BY defect_rate DESC;
