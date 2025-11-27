---
title: PL/SQL 예외 처리(Exception) 패턴
---

# PL/SQL 예외 처리(Exception) 패턴

이 문서는 Oracle PL/SQL에서 자주 등장하는  
**예외 처리(Exception Handling)** 패턴을 정리한 문서이다.

특히 다음 상황을 중심으로 예제를 구성했다.

- `SELECT INTO` 시 `NO_DATA_FOUND`, `TOO_MANY_ROWS`  
- `INSERT` 시 `DUP_VAL_ON_INDEX`  
- 공통 예외 처리용 블록 구조  
- `RAISE_APPLICATION_ERROR`를 활용한 사용자 정의 메시지  
- 커서(Cursor)와 예외 처리의 관계

---

# 1. 기본 예외 처리 구조

## 1-1. EXCEPTION 블록 개요

PL/SQL 블록 기본 구조는 다음과 같다.

```sql
DECLARE
    v_value NUMBER;
BEGIN
    -- 메인 로직
    v_value := 10 / 0;   -- 에러 발생 (0으로 나누기)

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('ERROR : ' || SQLERRM);
END;
/
```

---

# 1-2. 키워드 정리

- EXCEPTION
예외 처리 영역 시작 키워드

- WHEN ... THEN
특정 예외에 대한 처리 로직을 작성하는 구문

- WHEN OTHERS
명시되지 않은 예외 전체를 한 번에 처리하는 구문

- SQLERRM
발생한 오류 메시지를 문자열로 반환

- SQLCODE
오류 번호(코드)를 반환 (예: -1, -942 등)

---

# 2. NO_DATA_FOUND / TOO_MANY_ROWS

SELECT ... INTO 구문에서
결과 행 수가 기대와 다를 때 자주 발생하는 예외이다.

## 2-1. NO_DATA_FOUND
(1) 상황 정의

하나의 행만 조회된다고 가정하고 SELECT ... INTO를 수행했는데,
조건에 해당하는 행이 0건일 때 발생하는 예외.

(2) 예제 코드

```sql
DECLARE
    v_name  tb_customer.cust_name%TYPE;
BEGIN
    SELECT cust_name
      INTO v_name
      FROM tb_customer
     WHERE cust_id = 'C001';

    DBMS_OUTPUT.PUT_LINE('고객명: ' || v_name);

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('고객 정보를 찾을 수 없습니다.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('기타 오류: ' || SQLERRM);
END;
/
```

---

(3) 포인트

- 조회 결과가 0건이면 NO_DATA_FOUND 발생

- SELECT INTO가 사용하는 패턴에서는 거의 필수로 고려해야 하는 예외

---

## 2-2. TOO_MANY_ROWS
(1) 상황 정의

SELECT ... INTO는 1건만 가져올 수 있는데
조건에 해당되는 행이 2건 이상일 때 발생하는 예외.

(2) 예제 코드

```sql
DECLARE
    v_name  tb_customer.cust_name%TYPE;
BEGIN
    SELECT cust_name
      INTO v_name
      FROM tb_customer
     WHERE region = 'SEOUL';  -- 서울 지역에 고객이 여러 명이라고 가정

    DBMS_OUTPUT.PUT_LINE('고객명: ' || v_name);

EXCEPTION
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('조건에 해당하는 행이 너무 많습니다. (단일 행 조건 필요)');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('기타 오류: ' || SQLERRM);
END;
/
```

---

(3) 포인트

- SELECT INTO 사용 시 결과가 반드시 1건인지 확인 필요

- 그렇지 않을 가능성이 있으면 MAX, MIN, ROWNUM = 1 등을 사용하거나
  커서(Cursor) 기반 반복 구조로 바꾸는 것이 안전
  
---

# 3. DUP_VAL_ON_INDEX (중복 키 오류)

## 3-1. 상황 정의

PK 또는 UNIQUE 인덱스가 걸린 컬럼에
중복되는 값을 INSERT 또는 UPDATE 하려고 할 때 발생.

---

## 3-2. 예제 코드

```sql
DECLARE
    v_msg VARCHAR2(200);
BEGIN
    INSERT INTO tb_customer (
        cust_id,
        cust_name
    ) VALUES (
        'C001',        -- 이미 존재한다고 가정
        '중복고객'
    );

    v_msg := 'INSERT SUCCESS';

EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
        v_msg := 'ERROR : 중복된 고객 ID 입니다.';
    WHEN OTHERS THEN
        v_msg := 'ERROR : ' || SQLERRM;
END;
/
```

---

## 3-3. 포인트

- 회원가입, 코드 등록, 기준정보 관리 등에서 자주 발생

- 상황에 따라

  - 기존 데이터 UPDATE

  - 사용자에게 중복 알림 후 재입력 요청 등의 방식으로 분기 처리 필요

---

# 4. 사용자 정의 예외 & RAISE_APPLICATION_ERROR

단순히 Oracle 기본 메시지 대신
업무에 맞는 한글 메시지 / 코드를 사용하고 싶을 때
RAISE_APPLICATION_ERROR를 많이 사용한다.

---

## 4-1. 사용자 정의 예외 선언 + RAISE

```sql
DECLARE
    e_invalid_qty EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_invalid_qty, -20001);

    v_qty NUMBER := -10;
BEGIN
    IF v_qty < 0 THEN
        RAISE e_invalid_qty;
    END IF;

EXCEPTION
    WHEN e_invalid_qty THEN
        DBMS_OUTPUT.PUT_LINE('ERROR : 수량이 0보다 작을 수 없습니다.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('기타 오류: ' || SQLERRM);
END;
/
```

---

## 4-2. RAISE_APPLICATION_ERROR 사용

```sql
DECLARE
    v_qty NUMBER := -5;
BEGIN
    IF v_qty < 0 THEN
        RAISE_APPLICATION_ERROR(-20010, '수량은 0 이상이어야 합니다.');
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('에러코드 : ' || SQLCODE);
        DBMS_OUTPUT.PUT_LINE('에러메시지 : ' || SQLERRM);
END;
/
```

---

## 4-3. 포인트

- 에러코드는 -20000 ~ -20999 범위를 주로 사용

- ERP/MES/PDA 등 외부 시스템에서

  - 에러코드로 분기 처리

  - 에러메시지는 사용자 출력용으로 사용

- 공통 에러 규칙을 정해두면 운영 및 디버깅에 유리

---

# 5. 커서(Cursor)와 예외 처리

커서를 사용할 때도 예외 처리가 중요하다.
특히 SELECT INTO와 다르게,
커서 FOR LOOP는 NO_DATA_FOUND를 발생시키지 않는 점이 포인트다.

---

## 5-1. Cursor FOR LOOP 와 NO_DATA_FOUND

```sql
BEGIN
    FOR rec IN (
        SELECT cust_id, cust_name
          FROM tb_customer
         WHERE region = 'NO_REGION'  -- 존재하지 않는 지역
    ) LOOP
        DBMS_OUTPUT.PUT_LINE(rec.cust_name);
    END LOOP;

    DBMS_OUTPUT.PUT_LINE('정상 종료');
END;
/
```

---

- 위 예제에서 조회 결과가 0건이어도 예외가 발생하지 않는다.

- 단순히 LOOP가 한 번도 돌지 않고 끝날 뿐이다.

---

## 5-2. SELECT INTO vs CURSOR FOR LOOP 비교

SELECT INTO

- 0건 → NO_DATA_FOUND

- 2건 이상 → TOO_MANY_ROWS

CURSOR FOR LOOP

- 0건 → 에러 없음, LOOP 진입 안 함

- n건 → n번 반복

---

# 6. 상황별 예외 처리 패턴 정리

아래는 자주 등장하는 케이스별 예외 처리 추천 패턴이다.

상황	예외 유형	권장 패턴

| 상황                  | 예외 유형               | 권장 패턴                             |
| ------------------- | ------------------- | --------------------------------- |
| SELECT 1건 기대        | NO_DATA_FOUND       | 예외 잡아서 "데이터 없음" 처리                |
| SELECT 1건 기대, 조건 모호 | TOO_MANY_ROWS       | 조건 재검토 또는 ROWNUM = 1 활용           |
| PK/UNIQUE 컬럼 INSERT | DUP_VAL_ON_INDEX    | 중복 안내 후 재입력 또는 UPDATE 분기          |
| 수량/상태 등 비즈니스 규칙 위반  | 사용자 정의 예외 / -20000대 | RAISE_APPLICATION_ERROR 로 명확한 메시지 |
| 배치/인터페이스 공통 에러 처리   | WHEN OTHERS         | 에러 로그 테이블 기록 + 공통 코드 반환           |

---

# 7. 요약

- NO_DATA_FOUND, TOO_MANY_ROWS, DUP_VAL_ON_INDEX는
  실무에서 가장 자주 보는 예외 유형이다.

- SELECT INTO 는 반드시 1건이라는 전제가 필요하다.
  애매하면 CURSOR 또는 집계함수(MAX 등)를 고려해야 한다.

- RAISE_APPLICATION_ERROR를 활용하면
  업무 로직에 맞는 에러코드/메시지를 정의할 수 있다.

- 커서 FOR LOOP는 결과가 0건이어도 예외를 발생시키지 않는다.
  반복이 0번 수행될 뿐이므로, 별도의 로우 카운트 체크가 필요할 수 있다.

- 공통 예외 처리 패턴과 에러 로그 테이블을 함께 설계하면
  장애 분석과 운영이 훨씬 수월해진다.
