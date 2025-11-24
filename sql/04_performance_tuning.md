---
title: EXPLAIN PLAN / AUTOTRACE 성능 튜닝
---

# EXPLAIN PLAN / AUTOTRACE 기반 성능 튜닝

이 문서는 Oracle SQL에서 **실행 계획(Execution Plan)** 과  
**AUTOTRACE**를 활용해서 성능 튜닝(Performance Tuning)을 진행하는 과정을  
“전/후 비교” 중심으로 정리한 문서야.

다루는 내용:

- 느린 SQL 문제 정의  
- EXPLAIN PLAN 으로 실행 계획 확인  
- 인덱스(Index) + 조건절 튜닝 전/후 비교  
- AUTOTRACE 로 Buffer Gets, Cost, Elapsed Time 비교  
- 실무에서 바로 써먹을 체크리스트  

---

# 1. 느린 SQL 시나리오 정의

## 1-1. 문제 상황

`TB_ORDER` 테이블(수십만~수백만 건)이 있고,  
**2025년 1월의 총 매출 합계**를 구하는 쿼리가 있다고 하자.

```sql
SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE BETWEEN DATE '2025-01-01'
                      AND DATE '2025-01-31';
```

---

증상:

- 실행 시간이 오래 걸림

- AWR/Statspack 에서도 이 쿼리가 상위 소비 SQL로 자주 등장

- DBA 또는 개발자 입장에서 튜닝 대상

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

-------------------------------------------------------------
| Id  | Operation           | Name      | Rows | Cost | ... |
-------------------------------------------------------------
|   0 | SELECT STATEMENT    |           |    1 |  999 | ... |
|   1 |  SORT AGGREGATE     |           |    1 |      | ... |
|   2 |   TABLE ACCESS FULL | TB_ORDER  |   X  |  999 | ... |
-------------------------------------------------------------

---

## 1-3. 실행 계획 해석
- TABLE ACCESS FULL TB_ORDER
→ TB_ORDER 전체를 스캔하는 Full Table Scan(FTS) 발생

- 데이터 건수가 많을수록 I/O 비용이 커져서 성능 저하

- HERE 절 조건 컬럼(ORDER_DATE) 에 인덱스가 없거나,
인덱스를 제대로 활용하지 못하는 상황으로 추정

👉 다음 단계:
인덱스를 설계하고, 조건절을 튜닝해서 Index Range Scan 으로 바꾸는 것이 목표.

---

# 2. 인덱스 설계 + 조건절 튜닝

## 2-1. 인덱스 생성

ORDER_DATE 기준으로 조회가 자주 발생한다고 가정하고,
ORDER_DATE 컬럼에 인덱스를 생성하자.

```sql
CREATE INDEX IDX_TB_ORDER_DATE
    ON TB_ORDER (ORDER_DATE);
```

---

실제 운영에서는:

- 인덱스 개수, DML(INSERT/UPDATE/DELETE) 부하

- 기존 인덱스 존재 여부

- 복합 인덱스 필요성 (ORDER_DATE + STATUS 등) 를 함께 검토해야 한다.

---

## 2-2. 날짜 조건 튜닝 (BETWEEN → >= AND <)

아래처럼 BETWEEN 대신 >= 시작일 AND < 다음달 1일 패턴으로 바꿔준다.

```sql
SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE >= DATE '2025-01-01'
  AND ORDER_DATE <  DATE '2025-02-01';
```

---

왜 이렇게 바꾸는가?

- DATE 'YYYY-MM-DD' 리터럴은 변환 비용 없이 사용 가능

- BETWEEN 은 끝 날짜 포함 때문에,
인덱스/파티션 구조에 따라 최적화가 불리한 경우가 있음

- >= 시작일 AND < 다음달 1일 패턴은:

  - 월 단위 구간을 표현하기 쉬움

  - 파티션 범위 스캔(Partition Range Scan)과도 잘 맞음

  - Range Scan(범위 스캔) 최적화에 유리
 
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

------------------------------------------------------------------------------
| Id  | Operation                    | Name              | Rows | Cost | ... |
------------------------------------------------------------------------------
|   0 | SELECT STATEMENT             |                   |    1 |  100 | ... |
|   1 |  SORT AGGREGATE              |                   |    1 |      | ... |
|   2 |   INDEX RANGE SCAN           | IDX_TB_ORDER_DATE |   X  |  100 | ... |
------------------------------------------------------------------------------

---

해석

- TABLE ACCESS FULL → INDEX RANGE SCAN 으로 변경

- Cost(비용) 감소, I/O 감소 기대

- 인덱스 + 조건절 패턴만으로도 성능이 크게 개선될 수 있음

---

# 3. AUTOTRACE 로 성능 전/후 비교

## 3-1. AUTOTRACE 설정

SQL*Plus 또는 비슷한 환경에서:

```sql
SET AUTOTRACE ON STATISTICS;
-- 또는
SET AUTOTRACE ON EXPLAIN;
-- 또는
SET AUTOTRACE ON;
```

---

- STATISTICS : 실행 후 통계 정보(버퍼 읽기, CPU 등) 출력

- EXPLAIN : 실행 계획만 출력

- 그냥 ON : 실행 계획 + 통계 동시에 출력

---

## 3-2. 튜닝 전/후 쿼리 실행
1) 튜닝 전 쿼리

```sql
SET AUTOTRACE ON STATISTICS;

SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE BETWEEN DATE '2025-01-01'
                      AND DATE '2025-01-31';
```

2) ) 튜닝 후 쿼리

```sql
SELECT
    SUM(ORDER_AMT) AS TOTAL_AMT
FROM TB_ORDER
WHERE ORDER_DATE >= DATE '2025-01-01'
  AND ORDER_DATE <  DATE '2025-02-01';
```

---

## 3-3. AUTOTRACE 결과 비교 예시

(실제 숫자는 환경마다 다르지만, 예시로 이해하면 돼)

튜닝 전:

| 항목               | 값 예시            |
| ---------------- | --------------- |
| Consistent Gets  | 120,000         |
| Physical Reads   | 15,000          |
| Elapsed Time     | 2.5초            |
| Cost (Optimizer) | 999             |
| Access Path      | TABLE FULL SCAN |

---

해석 포인트

- Consistent Gets / Physical Reads
→ 논리/물리 I/O 감소 여부 확인

- Elapsed Time
→ 실제 걸린 시간 비교

- Access Path
→ Full Scan → Index Scan 으로 개선됐는지 확인

- Cost
→ Optimizer 관점의 상대적인 비용 (절대 수치는 아니라 참고용)

👉 포트폴리오에 “튜닝 전/후 AUTOTRACE 비교표” 넣으면
“분석 + 개선 + 근거 제시” 흐름이 보여서 매우 좋다.

---

# 4. 추가 튜닝 포인트 (WHERE, INDEX, 통계정보)
## 4-1. WHERE 절에서 함수 사용 주의

예를 들어, 아래와 같은 쿼리는:

```sq;
WHERE TO_CHAR(ORDER_DATE, 'YYYY-MM-DD') = '2025-01-15'
```
- ORDER_DATE 컬럼에 인덱스가 있어도
→ 함수(TO_CHAR) 를 감싸버리면 인덱스를 못 타는 경우 많음

튜닝 버전:
```sql
WHERE ORDER_DATE >= DATE '2025-01-15'
  AND ORDER_DATE <  DATE '2025-01-16';
```
또는 함수 기반 인덱스(Function-based Index)를 고려할 수 있다.

---

# 4-2. 통계정보(Statistics) 최신화

옵티마이저는 통계정보(Statistics)를 기반으로
실행 계획을 선택한다.
```sql
EXEC DBMS_STATS.GATHER_TABLE_STATS(
    ownname => 'SCOTT',
    tabname => 'TB_ORDER',
    cascade => TRUE
);
```

- 오래된 통계정보 → 왜곡된 실행 계획 → 성능 문제

- 대용량 테이블은 정기적인 통계 수집 필요

---

## 4-3. 인덱스 남발 주의

인덱스는 읽기(SELECT) 성능을 올려주지만,
INSERT/UPDATE/DELETE 시에는 인덱스도 같이 갱신해야 해서 부하가 생긴다.

실무에서 고려할 점:

- 조회가 매우 자주 일어나는 컬럼 위주로 인덱스 설계

- 사용되지 않는 인덱스는 과감히 제거

- 복합 인덱스(Composite Index) 설계 시 컬럼 순서 중요

---

# 5. 실무용 체크리스트

성능 튜닝(Performance Tuning)할 때 아래 순서로 점검하면 좋다.

1. 문제 정의

- 어떤 SQL이 느린가?

- 언제 느린가? (배치 시간, 피크 타임 등)

2. EXPLAIN PLAN / AUTOTRACE 확인

- Access Path: Full Scan? Index Scan?

- Cost, I/O, Buffer Gets, Elapsed Time 확인

3. 인덱스/조건절 튜닝

- WHERE 절에서 함수 사용 여부 확인

- BETWEEN → >= AND < 패턴 고려

- 인덱스 컬럼 조합 및 순서 재검토

4. 전/후 비교

- AUTOTRACE 로 수치 비교

- 실행 계획 변화 확인

5. 부작용 체크

- 다른 쿼리에 영향 여부

- DML 성능에 미치는 영향

- 인덱스 증가로 인한 저장공간/관리 비용

---

✔ 요약

- EXPLAIN PLAN 으로 현재 실행 계획을 먼저 정확히 이해한다.

- 인덱스 + WHERE 조건 튜닝으로 Access Path 를 개선한다.

- AUTOTRACE 로 튜닝 전/후 성능 차이를 수치로 증명하면 설득력이 크다.

- 통계정보, 인덱스 설계, 함수 사용 여부까지 함께 보는 것이 진짜 성능 튜닝이다.

---
