---
title: EXPLAIN PLAN / AUTOTRACE 성능 튜닝
---

# EXPLAIN PLAN / AUTOTRACE 기반 성능 튜닝

이 페이지는 Oracle SQL(Oracle SQL)에서  
**실행 계획(Execution Plan)** 과 **AUTOTRACE**를 활용하여  
SQL 성능 튜닝(Performance Tuning)을 수행하는 기본 과정을 정리한 페이입니다.

다음과 같은 흐름으로 설명합니다.

1. 느린 SQL 시나리오 정의  
2. EXPLAIN PLAN으로 실행 계획 확인  
3. 인덱스(Index) + 조건절 튜닝  
4. AUTOTRACE로 튜닝 전·후 비교  
5. WHERE 절/인덱스/통계정보 점검 체크리스트 

---

# 1. 느린 SQL 시나리오 정의

### 1-1. 문제 상황

`TB_ORDER` 테이블(수십만 ~ 수백만 건 수준)이 있다고 가정합니다.  
아래와 같은 쿼리가 보고서나 배치 작업에서 자주 실행되며, 성능 이슈가 발생한다고 합니다.

```sql
SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE BETWEEN DATE '2025-01-01'
                      AND DATE '2025-01-31';
```

증상 예시:

- 실행 시간이 수 초 이상 걸림

- AWR/Statspack 등에서 상위 리소스 사용 SQL로 자주 등장

- 월말/월초 배치 시간에 병목으로 작용

---

## 1-2. EXPLAIN PLAN 으로 실행 계획 확인

```sql
EXPLAIN PLAN FOR
SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE BETWEEN DATE '2025-01-01'
                      AND DATE '2025-01-31';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

---

예시 실행 계획(간단 버전):

<pre>
-------------------------------------------------------------
| Id  | Operation           | Name      | Rows | Cost | ... |
-------------------------------------------------------------
|   0 | SELECT STATEMENT    |           |    1 |  999 | ... |
|   1 |  SORT AGGREGATE     |           |    1 |      | ... |
|   2 |   TABLE ACCESS FULL | TB_ORDER  |   X  |  999 | ... |
-------------------------------------------------------------
</pre>

- TABLE ACCESS FULL TB_ORDER
→ TB_ORDER 전체를 스캔하는 Full Table Scan이 수행되고 있습니다.

- 데이터 건수가 많은 테이블에서 Full Scan은 I/O가 크게 증가하여 성능 저하로 이어질 수 있습니다.

---

# 2. 인덱스 설계 + 조건절 튜닝

## 2-1. 인덱스 생성

ORDER_DATE 기준으로 조회가 자주 발생한다고 가정하고,  
해당 컬럼에 인덱스를 생성합니다.

```sql
CREATE INDEX IDX_TB_ORDER_DATE
    ON TB_ORDER (ORDER_DATE);
```

---

실제 운영 환경에서는 다음 사항도 함께 고려해야 합니다.

- 이미 존재하는 인덱스와의 중복 여부

- INSERT / UPDATE / DELETE 부하

- 복합 인덱스 필요성(예: ORDER_DATE + STATUS 조합)

---

## 2-2. 날짜 조건 튜닝 (BETWEEN → >= AND <)

다음과 같이 조건절을 변경합니다.

```sql
SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE >= DATE '2025-01-01'
  AND ORDER_DATE <  DATE '2025-02-01';
```
변경 이유:

- DATE 'YYYY-MM-DD' 리터럴은 변환 비용 없이 직접 비교할 수 있습니다.

- BETWEEN을 사용할 경우, 종료일 포함 범위 처리 때문에
  파티션/인덱스 구조에 따라 최적화에 불리한 경우가 발생할 수 있습니다.

- >= 시작일 AND < 다음달 1일 패턴은
  월 단위 구간 처리와 파티션 범위 스캔에 잘 맞는 일반적인 작성 방식입니다.

---

## 2-3. 튜닝 후 EXPLAIN PLAN 다시 확인

```sql
EXPLAIN PLAN FOR
SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE >= DATE '2025-01-01'
  AND ORDER_DATE <  DATE '2025-02-01';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

---

예시 실행 계획(튜닝 후):

<pre>
-------------------------------------------------------------
| Id  | Operation           | Name      | Rows | Cost | ... |
-------------------------------------------------------------
|   0 | SELECT STATEMENT    |           |    1 |  100 | ... |
|   1 |  SORT AGGREGATE     |           |    1 |      | ... |
|   2 |   TABLE ACCESS FULL | TB_ORDER  |   X  |  100 | ... |
-------------------------------------------------------------
</pre>

---

- TABLE ACCESS FULL → INDEX RANGE SCAN으로 변경되었다면
  인덱스를 활용한 범위 조회로 개선된 것입니다.

- 이로 인해 I/O와 실행 시간이 감소할 것으로 예상할 수 있습니다.

---

# 3. AUTOTRACE 로 성능 전/후 비교

## 3-1. AUTOTRACE 설정

SQL*Plus 또는 유사 환경에서 다음과 같이 설정합니다.

```sql
SET AUTOTRACE ON STATISTICS;
-- 또는
SET AUTOTRACE ON EXPLAIN;
-- 또는
SET AUTOTRACE ON;
```

---

- STATISTICS : 실제 실행 후 논리/물리 I/O, CPU, 시간 등을 출력합니다.

- EXPLAIN : 실행 계획만 출력합니다.

- ON : 실행 계획 + 통계를 동시에 출력합니다.

---

## 3-2. 튜닝 전/후 쿼리 실행
1. 튜닝 전 쿼리

```sql
SET AUTOTRACE ON STATISTICS;

SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE BETWEEN DATE '2025-01-01'
                      AND DATE '2025-01-31';
```

2. 튜닝 후 쿼리

```sql
SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE >= DATE '2025-01-01'
  AND ORDER_DATE <  DATE '2025-02-01';
```

---

## 3-3. AUTOTRACE 결과 비교 예시

예시 지표(실제 숫자는 환경에 따라 상이함):

- Consistent Gets (논리적 블록 읽기)

- Physical Reads (물리적 디스크 읽기)

- Elapsed Time (경과 시간)

- CPU Time

- Optimizer Cost

튜닝 전 대비 튜닝 후에 다음과 같은 변화가 있는지 확인합니다.

- 논리/물리 I/O 감소

- 경과 시간(Elapsed Time) 감소

- Access Path가 Full Scan에서 Index Range Scan으로 변경

보고서나 포트폴리오에는
“튜닝 전/후 AUTOTRACE 결과 비교표”를 함께 제시하면
성능 개선 근거를 명확하게 설명할 수 있습니다.

---

# 4. WHERE 절/인덱스/통계정보 관련 추가 포인트

## 4-1. WHERE 절에서 함수 사용 주의
 
예를 들어 다음과 같은 조건은 인덱스 사용에 불리할 수 있습니다.

```sq;
WHERE TO_CHAR(ORDER_DATE, 'YYYY-MM-DD') = '2025-01-15'
```
- ORDER_DATE 컬럼에 인덱스가 있어도
  컬럼에 함수를 적용하면 인덱스를 사용하지 못하는 경우가 많습니다.
  
튜닝 예시:
```sql
WHERE ORDER_DATE >= DATE '2025-01-15'
  AND ORDER_DATE <  DATE '2025-01-16';
```
또는 필요하다면 **함수 기반 인덱스(Function-Based Index)**를 고려할 수 있습니다.

---

# 4-2. 통계정보(Statistics) 최신화

Oracle 옵티마이저(Optimizer)는 테이블/인덱스에 대한 통계정보를 기반으로
실행 계획을 선택합니다. 오래된 통계정보는 비효율적인 실행 계획의 원인이 될 수 있습니다.

예제:
```sql
EXEC DBMS_STATS.GATHER_TABLE_STATS(
    ownname => 'SCOTT',
    tabname => 'TB_ORDER',
    cascade => TRUE
);
```
- 정기적으로 통계정보를 수집하여 옵티마이저가 최신 분포를 반영할 수 있도록 하는 것이 좋습니다.

---

## 4-3. 인덱스 남발에 대한 주의

인덱스는 SELECT 성능을 개선하지만,
INSERT / UPDATE / DELETE 시에는 인덱스도 함께 갱신해야 하므로 부하가 증가합니다.

실무에서는 다음을 함께 고려해야 합니다.

- 실제로 자주 사용되는 조회 조건 컬럼인지

- 사용되지 않는 인덱스는 정리 가능한지

- 복합 인덱스(Composite Index) 설계 시 컬럼 순서가 적절한지

---

# 5. 실무용 체크리스트

성능 튜닝을 진행할 때 다음 순서를 기준으로 점검하는 것이 좋습니다.

1. 문제 정의

    - 어떤 SQL이 느린지, 언제 느린지(배치 시간, 피크 타임 등)를 명확히 파악합니다.

2. EXPLAIN PLAN / AUTOTRACE 확인

    - Access Path(Full Scan / Index Scan)

    - I/O, Cost, Elapsed Time 등 핵심 지표를 확인합니다.

3. 인덱스 / WHERE 절 튜닝

    - 컬럼에 함수 사용 여부 확인

    - 날짜 조건 표현 방식 개선

    - 인덱스 설계(단일/복합 인덱스) 재검토

3. 튜닝 전·후 비교

    - AUTOTRACE 결과 수치 비교

    - 실행 계획 변화 확인

5. 부작용 점검

    - 다른 쿼리에 미치는 영향

    - DML 성능 저하 여부

    - 인덱스 추가로 인한 공간·관리 비용

---

# 6. 요약

- EXPLAIN PLAN과 AUTOTRACE는
  SQL 성능 문제를 분석할 때 가장 기본이 되는 도구입니다.

- WHERE 절 작성 방식과 인덱스 설계만으로도
  Full Table Scan을 Index Range Scan으로 전환하여
  성능을 크게 개선할 수 있습니다.

- 통계정보, 인덱스 개수, 함수 사용 여부를 함께 고려하면서
  튜닝 전·후 성능을 수치로 비교하는 것이 중요합니다.

- 이 문서는 실제 제조·물류·출하 시스템에서
  자주 발생하는 기간 조건 기반 집계 SQL의 성능 이슈를
  체계적으로 분석·개선하는 흐름을 정리한 예시로 사용할 수 있습니다.

---
