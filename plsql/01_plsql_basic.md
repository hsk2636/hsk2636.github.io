---
title: PL/SQL 기본 구조 & 문법
---

# PL/SQL 기본 구조 & 문법

이 문서는 Oracle **PL/SQL(Procedural Language/SQL)** 의  
가장 기본이 되는 블록 구조와 변수 선언, 제어문(IF, LOOP)까지  
실무에서 꼭 필요한 최소 구성을 정리한 페이지야.

특히 나중에 **입출고 프로시저, LOT 진행 프로시저** 같은 걸 이해할 때  
`DECLARE ~ BEGIN ~ EXCEPTION ~ END` 구조를 자연스럽게 읽을 수 있도록 하는 게 목표.

---

## 1. PL/SQL 블록 기본 구조

### 1-1. 기본 형태

```sql
DECLARE
    -- ① 변수 선언부 (선택)
    v_message   VARCHAR2(100);
BEGIN
    -- ② 실행부
    v_message := 'Hello PL/SQL';
    DBMS_OUTPUT.PUT_LINE(v_message);

EXCEPTION
    -- ③ 예외 처리부 (선택)
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('에러 발생: ' || SQLERRM);
END;
/
```

---

## 1-2. 파트별 설명

- DECLARE

변수/상수(Cursor, TYPE 등)를 선언하는 영역

필요 없으면 생략 가능 (바로 BEGIN ~ END부터 시작해도 됨)

- BEGIN ~ END

실제로 실행되는 로직(Statement)이 들어가는 메인 영역

INSERT / UPDATE / DELETE / SELECT … INTO 등 대부분 여기서 실행

- EXCEPTION

오류(예외)가 났을 때 처리하는 영역

실무 프로시저에서 로그 테이블에 오류 저장, 메시지 리턴 이런 부분이 자주 위치

- 마지막 /

SQL*Plus, SQL Developer 에서 PL/SQL 블록 실행을 의미하는 구분자

---

# 2. 변수(Variable) & 상수(Constant) 선언

## 2-1. 기본 변수 선언

```sql
DECLARE
    v_lot_no      VARCHAR2(30);
    v_job_qty     NUMBER;
    v_start_time  DATE;
BEGIN
    v_lot_no     := 'LOT2025-0001';
    v_job_qty    := 1000;
    v_start_time := SYSDATE;

    DBMS_OUTPUT.PUT_LINE('LOT_NO      : ' || v_lot_no);
    DBMS_OUTPUT.PUT_LINE('JOB_QTY     : ' || v_job_qty);
    DBMS_OUTPUT.PUT_LINE('START_TIME  : ' || TO_CHAR(v_start_time, 'YYYY-MM-DD HH24:MI:SS'));
END;
/
```

---

# 3. IF 문(조건 분기)

## 3-1. 기본 IF 예제

```sql
DECLARE
    v_job_flag  NUMBER := 0; -- 0: 완제, 1: 반제라고 가정
BEGIN
    IF v_job_flag = 0 THEN
        DBMS_OUTPUT.PUT_LINE('완제품 공정입니다.');
    ELSIF v_job_flag = 1 THEN
        DBMS_OUTPUT.PUT_LINE('반제품 공정입니다.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('정의되지 않은 공정 구분입니다.');
    END IF;
END;
/
```

---

## 3-2. 설명

- IF ~ ELSIF ~ ELSE ~ END IF; 구조

- 실무에서

  - 공정 구분(완제/반제)

  - 출고 유형(ETC/MOV/SAL 등)

  - LOT 상태(대기/진행/완료 등) 이런 분기 처리에 계속 등장

---

# 4. LOOP / WHILE / FOR 문

PL/SQL에서는 반복문을 크게 3가지로 많이 써:

- 기본 LOOP ~ EXIT WHEN ~ END LOOP

- WHILE LOOP

- FOR LOOP

---

## 4-1. 기본 LOOP + EXIT 문

```sql
DECLARE
    v_cnt NUMBER := 0;
BEGIN
    LOOP
        v_cnt := v_cnt + 1;

        DBMS_OUTPUT.PUT_LINE('현재 카운트: ' || v_cnt);

        IF v_cnt >= 5 THEN
            EXIT; -- 루프 탈출
        END IF;
    END LOOP;
END;
/
```

---

## 4-2. FOR LOOP 예제

```sql
BEGIN
    FOR i IN 1 .. 5 LOOP
        DBMS_OUTPUT.PUT_LINE('FOR LOOP i = ' || i);
    END LOOP;
END;
/
```

---

## 4-3. 실무에서 LOOP가 쓰이는 곳

- 특정 조건을 만족하는 레코드를 커서(Cursor)로 하나씩 읽으면서 처리

- 임시 테이블의 데이터를 순차적으로 스캔

- 생산지시 상세를 한 건씩 돌면서 PDA 쪽 테이블에 INSERT 하는 로직 등

※ 실제 프로시저에서는 커서(Cursor) + LOOP가 같이 등장하는 경우가 많다
(이건 /plsql/04_exception_cursor.md 에서 자세히 정리할 예정)

---

# 5. 간단한 “작업 시작 로그” PL/SQL 블록 예시

이제 지금까지 본 DECLARE / BEGIN / IF / DATE 를 합쳐서
“LOT 작업 시작 로그를 남기고, 상태를 출력하는 블록” 느낌으로 하나 만들어보자.

```sql
DECLARE
    v_lot_no      VARCHAR2(30) := 'LOT2025-0001';
    v_job_qty     NUMBER       := 1200;
    v_job_flag    NUMBER       := 0;   -- 0: 완제, 1: 반제
    v_start_time  DATE         := SYSDATE;
BEGIN
    -- 작업 시작 정보 출력 (실제 현장이라면 INSERT로 로그 테이블에 저장)
    DBMS_OUTPUT.PUT_LINE('=== 작업 시작 등록 ===');
    DBMS_OUTPUT.PUT_LINE('LOT_NO      : ' || v_lot_no);
    DBMS_OUTPUT.PUT_LINE('JOB_QTY     : ' || v_job_qty);
    DBMS_OUTPUT.PUT_LINE('START_TIME  : ' || TO_CHAR(v_start_time, 'YYYY-MM-DD HH24:MI:SS'));

    IF v_job_flag = 0 THEN
        DBMS_OUTPUT.PUT_LINE('공정 구분   : 완제');
    ELSIF v_job_flag = 1 THEN
        DBMS_OUTPUT.PUT_LINE('공정 구분   : 반제');
    ELSE
        DBMS_OUTPUT.PUT_LINE('공정 구분   : 미정의 코드');
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('작업 시작 처리 중 오류 발생: ' || SQLERRM);
END;
/
```

---

## 5-1. 실무 연계 상상

- 실제 프로시저에서는 INSERT INTO TB_JOB_START_LOG ... 같은 구문이 들어가고

- LOT 번호, 수량, 사용자 ID, 작업 시작 시간 등을 기록

- 예외 발생 시, ERROR_LOG 테이블에 INSERT 하거나, OUT 파라미터로 v_err_msg 같은 걸 반환

이 문서는 “PL/SQL 블록을 읽을 수 있는 눈”을 만드는 단계라고 보면 돼.

---

# 6. 요약

- PL/SQL은 기본적으로 DECLARE → BEGIN → EXCEPTION → END; 구조로 이해하면 편함

- 변수/상수 선언, IF 분기, LOOP 반복문은 나중에 프로시저(Procedure), 패키지(Package) 를 읽을 때 계속 반복적으로 등장

- 지금 단계에서는

  - 블록 구조

  - 변수 대입 :=

  - IF / LOOP 문법
 
이 3가지만 익숙해져도 큰 흐름 잡는 데 충분함




















