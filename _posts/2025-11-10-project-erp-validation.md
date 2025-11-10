---
layout: single
title: "프로젝트 3: ERP 연계 데이터 검증"
date: 2025-11-10
categories: [Project, SQL]
tags: [ERP, 데이터검증, 재고, SQL]
excerpt: "ERP 시스템과 내부 DB 데이터를 비교하여 수량/금액 불일치를 탐지하는 데이터 검증 프로젝트."
toc: true
---

## 🎯 프로젝트 개요

- 주제: ERP ↔ 내부 시스템 간 데이터 일관성 검증
- 목표: 재고 수량, 매출, 발주 데이터 불일치 자동 탐지
- 사용 기술: Oracle SQL, JOIN, 비교 쿼리, Excel 연동

---

## 🧩 핵심 SQL 예시

```sql
SELECT a.item_code,
       a.qty  AS erp_qty,
       b.qty  AS system_qty,
       (a.qty - b.qty) AS diff_qty
FROM erp_inventory a
JOIN system_inventory b
  ON a.item_code = b.item_code
WHERE a.qty <> b.qty;
