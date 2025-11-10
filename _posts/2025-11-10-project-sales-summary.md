---
layout: single
title: "프로젝트 2: 매출 데이터 요약 자동화"
date: 2025-11-10
categories: [Project, SQL]
tags: [매출, 집계, Python, SQL]
excerpt: "SQL 기반 매출 데이터를 자동 요약하고, Python과 연동해 통계 리포트를 생성합니다."
toc: true
---

## 🎯 프로젝트 개요
- 주제: 매출 데이터 집계 자동화  
- 목표: 일별, 월별 매출 요약 리포트 생성  
- 사용 기술: SQL, Python, Pandas  

---

## 🧩 주요 SQL 예시

```sql
SELECT TO_CHAR(sales_date, 'YYYY-MM') AS month,
       SUM(sales_amount) AS total_sales
FROM sales_data
GROUP BY TO_CHAR(sales_date, 'YYYY-MM')
ORDER BY month;
```

**포인트**  
- GROUP BY를 통한 월별 집계  
- Python에서 pandas.read_sql()로 데이터 자동 호출  
- GitHub Actions로 스케줄링 예정  

---

## 📈 향후 개선 계획
- 그래프 시각화(Matplotlib, Plotly) 추가  
- PDF 리포트 자동 생성 기능 연동
