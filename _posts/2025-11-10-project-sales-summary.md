---
layout: single
title: "프로젝트 2: 매출 데이터 요약 자동화"
date: 2025-11-10
categories: [Project, SQL]
tags: [매출, 집계, 자동화, Python, SQL]
excerpt: "매출 데이터를 SQL로 집계하고, Python과 연계해 리포트를 자동 생성하는 프로젝트."
toc: true

header:
  overlay_image: /assets/images/projects/sales-summary-banner.png
  overlay_filter: 0.3
---

## 🎯 프로젝트 개요

- 주제: 일/월/분기 매출 데이터 자동 요약
- 목표: 반복 리포트를 SQL + 스크립트로 자동화
- 사용 기술: Oracle SQL, GROUP BY, Python(pandas) 연동

---

## 🧩 주요 SQL 예시

```sql
SELECT TO_CHAR(sales_date, 'YYYY-MM') AS month,
       SUM(sales_amount) AS total_sales
FROM sales_data
GROUP BY TO_CHAR(sales_date, 'YYYY-MM')
ORDER BY month;
```

---
