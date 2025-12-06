---
title: Oracle PL/SQL 포트폴리오
---

# Oracle PL/SQL 포트폴리오

본 문서는 Oracle PL/SQL 기반으로  
**Procedure, Cursor, Bulk Processing, Exception Handling, Package 구조** 등을  
실무 중심으로 정리한 포트폴리오입니다.

제조 IT(MES·PDA·ERP) 환경에서 빈번하게 사용되는  
LOT 흐름, 출고·입고 처리, ERP 인터페이스 로직을 이해하는 데 도움이 되도록 구성하였습니다.

---

## 📚 전체 구성 (Contents)

---

### 1. PL/SQL 기본 문법 & 구조

DECLARE / BEGIN / EXCEPTION / END 구조를 포함하여  
변수 선언, 조건문(IF), 반복문(LOOP) 등  
PL/SQL의 기본 문법을 정리한 문서입니다.

👉 **바로 보기**  
[01_plsql_basic.md](01_plsql_basic.md)

---

### 2. 프로시저(Procedure) 실전 예제

입고 등록, 출고 처리, LOT 등록 등  
실무 기반 흐름을 IN/OUT 파라미터, 예외 처리(Exception),  
DML(INSERT/UPDATE/DELETE) 중심으로 설명한 문서입니다.

👉 **바로 보기**  
[02_procedure_examples.md](02_procedure_examples.md)

---

### 3. Cursor / BULK COLLECT / FORALL

대용량 DML 처리 성능을 개선하기 위한  
Cursor, BULK COLLECT, FORALL, LIMIT 처리 방식 등을  
예제 중심으로 정리한 문서입니다.

👉 **바로 보기**  
[03_cursor_examples.md](03_cursor_examples.md)

---

### 4. 패키지(Package) 구조 분석

PACKAGE SPEC / BODY 구조와 역할을 비교하고,  
공통 상수·타입 선언, PRIVATE FUNCTION 구성,  
업무별 패키지 유지보수 포인트 등을 정리한 문서입니다.

👉 **바로 보기**  
[04_package_structure.md](04_package_structure.md)

---

### 5. 예외 처리(Exception) 패턴

PL/SQL에서 자주 등장하는 예외 상황을 정리하고,  
NO_DATA_FOUND / TOO_MANY_ROWS / DUP_VAL_ON_INDEX 등  
예외 처리 모범 사례를 담은 문서입니다.

👉 **바로 보기**  
[05_exception_patterns.md](05_exception_patterns.md)

---

### 6. 실전 케이스: 제조 IT(ERP/MES/PDA)

LOT 시작/완료 처리, 출고·입고 흐름, ERP 인터페이스 로직 등  
실제 제조 IT 업무에서 사용되는 PL/SQL 기반 처리 로직을  
단계별로 분석하여 정리한 문서입니다.

👉 **바로 보기**  
[06_realworld_cases.md](06_realworld_cases.md)

---

## 🎯 작성 목적

본 PL/SQL 포트폴리오는 다음과 같은 목적을 가지고 작성하였습니다.

- PL/SQL 문법을 **실무 흐름 중심**으로 이해하고 설명할 수 있는 능력 정리  
- Procedure, Cursor, Bulk, Exception, Package 구조를 **패턴 단위**로 정리  
- 제조 IT(MES·PDA·ERP) 흐름 분석 능력과 연계된 실무 역량 강화  
- SQL 포트폴리오와 함께 하나의 완성된 학습·업무 기반 자료로 제출하기 위함  

본 문서는 Oracle PL/SQL 학습 및 실무 적용을 위해  
완결된 형태로 정리하였습니다.
