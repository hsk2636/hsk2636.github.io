# Oracle PL/SQL 포트폴리오

Oracle PL/SQL 기반으로  
**Procedure, Cursor, Bulk Processing, Exception, Package 구조** 등을  
실무 중심으로 정리한 포트폴리오입니다.

제조 IT(MES/PDA/ERP) 환경에서 자주 등장하는  
LOT 흐름, 출고/입고, ERP 인터페이스 로직을 이해하는 데 도움이 되도록 구성했습니다.

---

# 📚 전체 구성 (Contents)

## 1. PL/SQL 기본 문법 & 구조  
DECLARE / BEGIN / EXCEPTION / END  
변수, 조건문(IF), 반복문(LOOP), 기본 구조를 정리한 기초 문서입니다.

👉 바로가기  
**[01_plsql_basic.md](01_plsql_basic.md)**

---

## 2. 프로시저(Procedure) 실전 예제  
입고등록 / 출고처리 / LOT 등록 같은 실무형 프로시저 흐름을  
IN/OUT 파라미터, 예외 처리, DML 흐름 중심으로 설명합니다.

👉 바로가기  
**[02_procedure_examples.md](02_procedure_examples.md)**

---

## 3. Cursor / BULK COLLECT / FORALL  
명시적 커서, 묵시적 커서, BULK COLLECT, FORALL, LIMIT 처리 등  
대량 데이터 처리(DML 성능 최적화)의 핵심 패턴을 정리했습니다.

👉 바로가기  
**[03_cursor_examples.md](03_cursor_examples.md)**

---

## 4. 패키지(Package) 구조 분석  
PACKAGE SPEC / BODY 구성, 공통 상수 / 타입,  
PRIVATE FUNCTION 패턴 등 유지보수 관점 핵심 구조를 설명합니다.

👉 바로가기  
**[04_package_structure.md](04_package_structure.md)**

---

## 5. 예외 처리(Exception) 패턴  
NO_DATA_FOUND / TOO_MANY_ROWS / DUP_VAL_ON_INDEX 등  
PL/SQL에서 자주 등장하는 예외 처리 패턴과 주의점, 모범 사례를 정리했습니다.

👉 바로가기  
**[05_exception_patterns.md](05_exception_patterns.md)**

---

## 6. 실전 케이스: 제조 IT(ERP/MES/PDA)  
LOT 시작/완료, 입고/출고 처리, ERP 인터페이스 로직 등  
MES/PDA 기반 실무 흐름을 단계별로 분석한 케이스 스터디 문서입니다.

👉 바로가기  
**[06_realworld_cases.md](06_realworld_cases.md)**

---

# 🎯 작성 목적

- PL/SQL 문법 이해 + 실무 적용 능력 강화  
- Procedure / Cursor / Bulk / Exception / Package 구조 정리  
- 제조 IT(MES·PDA·ERP) 프로세스 분석 능력 향상  
- SQL 포트폴리오와 연계된 개발형 역량 구축  

---

# 📌 향후 확장 예정

- 트리거(TRIGGER) 실습  
- 사용자 정의 타입(VARRAY, TABLE TYPE)  
- PIPELINED FUNCTION  
- 동적 SQL(EXECUTE IMMEDIATE)  

