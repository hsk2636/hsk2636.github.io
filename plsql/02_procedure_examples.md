---
title: PL/SQL 절차형 프로그래밍 예제
---

# PL/SQL Procedure 실전 예제 모음

이 문서는 Oracle PL/SQL에서 가장 많이 사용하는  
**IN/OUT 파라미터**, **예외 처리(WHEN OTHERS)**,  
**SQLERRM 기반 오류 반환**, 그리고  
**실제 업무 흐름에 가까운 LOT 작업 등록 프로시저**를 정리한 페이지이다.

---

# 1. PL/SQL 프로시저 기본 구조 이해

## 1-1. 가장 단순한 형태의 프로시저 예제

```sql
CREATE OR REPLACE PROCEDURE PR_HELLO_PROC (
    p_name     IN  VARCHAR2,
    p_message  OUT VARCHAR2
)
IS
BEGIN
    p_message := 'Hello, ' || p_name;
EXCEPTION
    WHEN OTHERS THEN
        p_message := 'ERROR : ' || SQLERRM;
END PR_HELLO_PROC;
/
```

---

## 1-2. 실행 예시

```sql
DECLARE
    v_msg VARCHAR2(200);
BEGIN
    PR_HELLO_PROC('HANSOO', v_msg);
    DBMS_OUTPUT.PUT_LINE(v_msg);
END;
/
```

---

1-3. 설명

- IN
프로시저에 값을 넘겨주는 입력 파라미터

- OUT
프로시저가 호출자에게 반환하는 값 (결과 메시지 등)

- WHEN OTHERS
PL/SQL에서 가장 많이 사용하는 예외 처리 패턴
→ 모든 예외를 한 번에 처리할 때 사용

- SQLERRM
Oracle 내부 오류 메시지를 문자열로 반환하는 함수
(예: ORA-00001: unique constraint 오류 등)

---

# 2. LOT 작업 시작 등록 (작업지시 기반)

제조 현장에서 가장 많이 쓰는 시나리오를 기준으로 구성.

---

## 2-1. 문제 정의

입력 파라미터:

- LOT 번호

- 작업 수량

- 작업자 ID

해야 하는 일:

- LOT 상태를 “작업 시작”으로 변경

- LOT 작업 로그 기록

- OUT 메시지 반환

---

# 2-2. SQL 코드

```sql
CREATE OR REPLACE PROCEDURE PR_JOB_START (
    p_lot_no      IN  VARCHAR2,
    p_job_qty     IN  NUMBER,
    p_user_id     IN  VARCHAR2,
    p_return_msg  OUT VARCHAR2
)
IS
BEGIN
    -- 1) LOT 상태 업데이트
    UPDATE TB_LOT
       SET LOT_STATUS = 'JOB_START'
         , START_QTY  = p_job_qty
         , START_USER = p_user_id
         , START_TIME = SYSDATE
     WHERE LOT_NO = p_lot_no;

    -- LOT 존재 여부 체크
    IF SQL%ROWCOUNT = 0 THEN
        p_return_msg := 'ERROR : LOT NOT FOUND';
        RETURN;
    END IF;

    -- 2) LOT 작업 로그 기록
    INSERT INTO TB_LOT_LOG (
        LOG_ID,
        LOT_NO,
        LOG_TYPE,
        LOG_QTY,
        REG_USER,
        REG_TIME
    ) VALUES (
        SEQ_LOT_LOG.NEXTVAL,
        p_lot_no,
        'JOB_START',
        p_job_qty,
        p_user_id,
        SYSDATE
    );

    -- 3) 성공 메시지
    p_return_msg := 'SUCCESS : JOB STARTED';

EXCEPTION
    WHEN OTHERS THEN
        p_return_msg := 'ERROR : ' || SQLERRM;
END PR_JOB_START;
/
```

---

## 2-3. 설명

- UPDATE TB_LOT … WHERE LOT_NO = p_lot_no
→ LOT이 존재하는지 판단하는 가장 직관적인 방식
→ SQL%ROWCOUNT = 0 이면 “LOT 없음” 에러

- INSERT INTO TB_LOT_LOG
→ 이력 테이블에 남기는 방식 (현장에서 가장 일반적)

- RETURN 메시지
→ PDA, MES, ERP 등 호출하는 쪽에서 쉽게 처리 가능

---

# 3. LOT 작업 완료 등록 (Finish)

(원한다면 추가로 작성 가능!
작업 완료 → LOT 상태 변경 + 완료 로그 기록 + OUT 메시지 구조)

---

# 4. 요약

- IN/OUT 파라미터 구조는 PL/SQL의 기본

- WHEN OTHERS + SQLERRM 조합은 실무에서 가장 많이 사용

- LOT 상태 업데이트 → 로그 기록 → 메시지 반환은 PDA/MES/ERP 공통 구조

- PR_JOB_START 예제는 실제 제조 시스템에서도 거의 그대로 쓰는 형태











