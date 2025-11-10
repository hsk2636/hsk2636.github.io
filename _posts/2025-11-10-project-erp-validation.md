---
layout: single
title: "프로젝트 3: ERP 연계 데이터 검증"
date: 2025-11-10
categories: [Project, SQL]
tags: [ERP, 데이터검증, Excel, API]
excerpt: "ERP 시스템과 SQL 데이터를 비교·검증하여 데이터 일관성 문제를 탐지합니다."
toc: true
---

## 🎯 프로젝트 개요
- 주제: ERP ↔ SQL 데이터 비교 검증  
- 목표: 수량, 단가, 재고 데이터의 불일치 탐지  
- 사용 기술: SQL, Excel, ERP API  

---

## 🧩 핵심 SQL 예시

```sql
SELECT a.item_code,
       a.qty AS erp_qty,
       b.qty AS sql_qty,
       (a.qty - b.qty) AS diff
FROM erp_inventory a
JOIN mes_inventory b
  ON a.item_code = b.item_code
WHERE a.qty <> b.qty;
```

**포인트**  
- ERP ↔ MES 데이터 불일치 탐색  
- ERP API 연동 전 로컬 DB 비교  
- PL/SQL로 자동 검증 프로시저 개발 예정  

---

## 📈 향후 개선 계획
- Excel export 기능 추가  
- ERP 연동 자동화 테스트 시나리오 구축
