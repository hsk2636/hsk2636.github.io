---
title: 제조 IT(MES·PDA·ERP) 실전 PL/SQL 케이스 분석
---

# 제조 IT(MES·PDA·ERP) 실전 PL/SQL 케이스 분석

이 문서는 실제 제조 현장에서 사용하는  
**LOT 공정 흐름, PDA 스캔 처리, 출고/입고, ERP 인터페이스 로직**을  
PL/SQL 중심으로 정리한 실전 포트폴리오 페이지입니다.

실제 업무에서 다루는 핵심 주제:

- LOT 공정 진행(시작/완료)
- PDA 스캔 기반 처리 흐름
- 출고/입고/재입고 업무 처리
- ERP(APPS) 인터페이스 반영
- 예외 처리(EXCEPTION) 패턴
- Data Flow + Process Flow 시각화

---

# 1. LOT 공정 시작(START) 처리

## 1-1. 업무 시나리오

PDA에서 작업자가 LOT 바코드를 스캔하면 다음과 같은 절차를 수행합니다.

1) LOT 존재 여부 확인  
2) HOLD(보류), 폐기, 완료 상태 여부 검증  
3) 공정 시작 가능한 상태인지 점검  
4) LOT 상태를 `START` 또는 `IN-PROCESS` 로 변경  
5) 작업 시작 시간 기록  
6) LOT 로그 테이블에 이력 저장

---

## 1-2. PL/SQL 프로시저 예제

```sql
CREATE OR REPLACE PROCEDURE USP_LOT_START (
    P_LOT_NO   IN  VARCHAR2,
    P_USER     IN  VARCHAR2,
    O_RESULT   OUT VARCHAR2,
    O_MESSAGE  OUT VARCHAR2
) IS
    v_cnt NUMBER;
BEGIN
    -- 1. LOT 존재 여부 확인
    SELECT COUNT(*)
      INTO v_cnt
      FROM TB_LOT
     WHERE LOT_NO = P_LOT_NO;

    IF v_cnt = 0 THEN
        O_RESULT  := 'N';
        O_MESSAGE := 'LOT 없음';
        RETURN;
    END IF;

    -- 2. LOT 상태 검증
    SELECT COUNT(*)
      INTO v_cnt
      FROM TB_LOT
     WHERE LOT_NO = P_LOT_NO
       AND STATUS IN ('HOLD','FINISH','SCRAP');

    IF v_cnt > 0 THEN
        O_RESULT  := 'N';
        O_MESSAGE := '작업 불가 상태';
        RETURN;
    END IF;

    -- 3. 작업 시작 처리
    UPDATE TB_LOT
       SET STATUS      = 'START',
           START_TIME  = SYSDATE,
           UPDATE_USER = P_USER,
           UPDATE_TIME = SYSDATE
     WHERE LOT_NO = P_LOT_NO;

    -- 4. 로그 기록
    INSERT INTO TB_LOT_LOG(LOT_NO, ACTION_TYPE, ACTION_TIME, ACTION_USER)
    VALUES(P_LOT_NO, 'START', SYSDATE, P_USER);

    O_RESULT  := 'Y';
    O_MESSAGE := '작업 시작 완료';

EXCEPTION
    WHEN OTHERS THEN
        O_RESULT  := 'N';
        O_MESSAGE := SQLERRM;
END;
/
```

---

## 1-3. 핵심 포인트

- PDA 스캔 로직에서 가장 먼저 호출되는 핵심 프로시저입니다.

- LOT의 존재 여부와 현재 상태를 검증하는 로직이 특히 중요합니다.

- LOT 로그(TB_LOT_LOG)는 공정 추적을 위해 반드시 기록해야 하는 정보입니다.

- 실패 시에는 OUT 파라미터를 통해 오류 메시지를 PDA 화면에 직접 전달할 수 있습니다.

---

# 2. LOT 공정 완료(FINISH) 처리

## 2-1. 업무 시나리오

작업 완료 시 PDA에서 “작업 완료” 버튼을 누르면 다음 절차를 수행합니다.

1. LOT 상태가 START 상태인지 확인

2. 이미 완료된 LOT인지 체크

3. 공정 완료 처리(STATUS='FINISH')

4. 작업 완료 시간 기록

5. 다음 공정 또는 인계 공정으로 데이터 이동 가능

---

## 2-2. PL/SQL 예제

```sql
CREATE OR REPLACE PROCEDURE USP_LOT_FINISH (
    P_LOT_NO   IN  VARCHAR2,
    P_USER     IN  VARCHAR2,
    O_RESULT   OUT VARCHAR2,
    O_MESSAGE  OUT VARCHAR2
) IS
    v_status VARCHAR2(20);
BEGIN
    -- LOT 상태 조회
    SELECT STATUS
      INTO v_status
      FROM TB_LOT
     WHERE LOT_NO = P_LOT_NO;

    IF v_status <> 'START' THEN
        O_RESULT  := 'N';
        O_MESSAGE := '작업 시작 상태가 아닙니다.';
        RETURN;
    END IF;

    -- 완료 처리
    UPDATE TB_LOT
       SET STATUS      = 'FINISH',
           FINISH_TIME = SYSDATE,
           UPDATE_USER = P_USER,
           UPDATE_TIME = SYSDATE
     WHERE LOT_NO = P_LOT_NO;

    INSERT INTO TB_LOT_LOG(LOT_NO, ACTION_TYPE, ACTION_TIME, ACTION_USER)
    VALUES(P_LOT_NO, 'FINISH', SYSDATE, P_USER);

    O_RESULT := 'Y';
    O_MESSAGE := '작업 완료 처리되었습니다.';

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        O_RESULT  := 'N';
        O_MESSAGE := 'LOT 없음';
    WHEN OTHERS THEN
        O_RESULT  := 'N';
        O_MESSAGE := SQLERRM;
END;
/
```

---

## 2-3. 핵심 포인트

- 시작되지 않은 LOT은 완료할 수 없으므로, 상태 검증 로직이 반드시 필요합니다.

- PDA에서는 “작업 완료 불가 사유”를 O_MESSAGE로 내려주어 작업자가 즉시 이유를 확인할 수 있도록 하는 것이 중요합니다.

- LOT 로그는 공정 이력을 추적하기 위한 핵심 데이터입니다.

---

# 3. 출고(ETC / MOV / SAL) 처리

## 3-1. 업무 시나리오

PDA 출고 메뉴에서는 다음 흐름으로 작업이 진행됩니다.

1. PDA 출고 메뉴 흐름:

2. LOT 또는 자재 바코드 스캔

  - 출고 유형 선택

  - ETC = 기타 출고

  - MOV = 이동 출고

  - SAL = 매각/판매 출고

3. 재고량/단위/상태 검증

4. 출고 테이블 INSERT

5. 재고(TB_STOCK) 감소

6. ERP 인터페이스 필요 시 APPS 테이블 INSERT

---

```sql
CREATE OR REPLACE PROCEDURE USP_ITEM_OUT (
    P_ITEM_CODE IN VARCHAR2,
    P_QTY       IN NUMBER,
    P_TYPE      IN VARCHAR2,   -- ETC / MOV / SAL
    P_USER      IN VARCHAR2,
    O_RESULT    OUT VARCHAR2,
    O_MESSAGE   OUT VARCHAR2
) IS
    v_stock NUMBER;
BEGIN
    -- 1. 재고 체크
    SELECT QTY
      INTO v_stock
      FROM TB_STOCK
     WHERE ITEM_CODE = P_ITEM_CODE;

    IF v_stock < P_QTY THEN
        O_RESULT  := 'N';
        O_MESSAGE := '재고 부족';
        RETURN;
    END IF;

    -- 2. 출고 이력 기록
    INSERT INTO TB_OUT (
        ITEM_CODE, OUT_QTY, OUT_TYPE, OUT_TIME, OUT_USER
    ) VALUES (
        P_ITEM_CODE, P_QTY, P_TYPE, SYSDATE, P_USER
    );

    -- 3. 재고 감소
    UPDATE TB_STOCK
       SET QTY = QTY - P_QTY
     WHERE ITEM_CODE = P_ITEM_CODE;

    -- 4. ERP 인터페이스 (예: 출고 기록 전송)
    INSERT INTO ERP_INV_INTERFACE(
        ITEM_CODE, OUT_QTY, OUT_TYPE, CREATE_TIME
    ) VALUES (
        P_ITEM_CODE, P_QTY, P_TYPE, SYSDATE
    );

    O_RESULT  := 'Y';
    O_MESSAGE := '출고 완료';

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        O_RESULT  := 'N';
        O_MESSAGE := '품목 없음';
    WHEN OTHERS THEN
        O_RESULT  := 'N';
        O_MESSAGE := SQLERRM;
END;
/
```

---

## 3-3. 핵심 포인트

- PDA 출고 모듈에서 가장 중요한 부분은 재고량 검증입니다.

- ERP 인터페이스 수행 여부는 회사 정책에 따라 ON/OFF가 가능하며, 인터페이스 테이블만 적재하고 배치는 별도 프로세스에서 수행하기도 합니다.

- SAL(판매), MOV(이동), ETC(기타) 등 출고 유형별로 보고서/회계 처리 방식이 달라질 수 있으므로 유형 관리가 중요합니다.

---

# 4. 재입고(Re-Inbound) 처리

## 4-1. 업무 시나리오

이미 출고된 LOT/자재를 다시 입고 처리하는 기능입니다.

1. 기존 출고 이력 존재 여부 확인

2. 출고 취소 가능 상태인지 검증

3. 재고 증가 처리

4. 재입고 로그 기록

5. ERP 인터페이스는 선택 사항 (취소 처리에 따라 다름)

---

## 4-2. PL/SQL 예제

```sql
CREATE OR REPLACE PROCEDURE USP_ITEM_REINBOUND (
    P_ITEM_CODE IN VARCHAR2,
    P_QTY       IN NUMBER,
    P_USER      IN VARCHAR2,
    O_RESULT    OUT VARCHAR2,
    O_MESSAGE   OUT VARCHAR2
) IS
    v_cnt NUMBER;
BEGIN
    -- 1. 기존 출고 이력 확인
    SELECT COUNT(*)
      INTO v_cnt
      FROM TB_OUT
     WHERE ITEM_CODE = P_ITEM_CODE
       AND OUT_QTY   = P_QTY;

    IF v_cnt = 0 THEN
        O_RESULT  := 'N';
        O_MESSAGE := '출고 이력 없음 → 재입고 불가';
        RETURN;
    END IF;

    -- 2. 재입고 처리
    UPDATE TB_STOCK
       SET QTY = QTY + P_QTY
     WHERE ITEM_CODE = P_ITEM_CODE;

    INSERT INTO TB_REINBOUND_LOG(
        ITEM_CODE, QTY, ACTION_TIME, ACTION_USER
    ) VALUES (
        P_ITEM_CODE, P_QTY, SYSDATE, P_USER
    );

    O_RESULT  := 'Y';
    O_MESSAGE := '재입고 완료';

EXCEPTION
    WHEN OTHERS THEN
        O_RESULT  := 'N';
        O_MESSAGE := SQLERRM;
END;
/
```

---

# 5. ERP 인터페이스(APPS) 실전 구조

Oracle ERP(특히 E-Business Suite, EBS) 인터페이스는 일반적으로 다음 구조를 가집니다.

1. 인터페이스 테이블 INSERT

    - 예: APPS.MTL_TRANSACTIONS_INTERFACE 등

2. 인터페이스 프로세스 실행

    - 배치(Job)로 자동 실행

    - 또는 사용자가 수동으로 인터페이스 프로그램 실행

3. 결과 상태 확인

    - PROCESSED, ERROR, 재처리(RETRY) 대상 등 상태값 확인

4. 오류 시 메시지 조회

    - INTERFACE_ERRORS 또는 유사 에러 테이블에서 상세 메시지 확인

---

# 5-1. 예시 코드

```sql
INSERT INTO APPS.MTL_TRANSACTIONS_INTERFACE(
      SOURCE_CODE,
      ITEM_NUMBER,
      TRANSACTION_TYPE,
      TRANSACTION_QUANTITY,
      CREATION_DATE
) VALUES (
      'MES',
      P_ITEM_CODE,
      'ISSUE',
      P_QTY,
      SYSDATE
);
```

---

# 6. 전체 공정 Flow 정리

PDA 스캔
    ↓
LOT 검증
    ↓
작업 시작(START)
    ↓
작업 진행(PROCESS)
    ↓
작업 완료(FINISH)
    ↓
출고 / 입고 / 재입고
    ↓
ERP 인터페이스(APPS)

---

✔ 요약

- PL/SQL + PDA + ERP를 연결하는 실전 업무 흐름은 “LOT 검증 → 작업 처리 → 로그 기록 → 상태 변경 → ERP 전송” 패턴으로 구성됩니다.

- PDA는 주로 스캔 및 입력 역할을 수행하며, 실제 업무 로직은 대부분 PL/SQL 프로시저에서 처리됩니다.

- ERP 인터페이스는 보통 APPS 인터페이스 테이블 INSERT 후 배치 프로그램 실행 구조로 설계됩니다.

- 본 페이지는 제조 IT(MES·PDA·ERP) 환경에서 PL/SQL을 어떻게 활용할 수 있는지에 대한 대표적인 케이스를 정리한 자료입니다.

---
