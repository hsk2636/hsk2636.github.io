---
title: PL/SQL Package 구조 분석
---

# PL/SQL Package 구조 분석

이 문서는 Oracle PL/SQL에서 가장 강력한 구성 요소 중 하나인  
**PACKAGE (명세서 SPEC + 구현 BODY)** 구조를 이해하기 쉽게 정리한 문서입니다.

Package는 실무에서 다음 이유로 매우 중요합니다:

- 여러 Procedure/Function을 한 모듈로 묶어 관리
- SPEC과 BODY 분리 덕분에 유지보수·API 관리가 쉬움
- MES/PDA/ERP 같은 여러 시스템에서 공통 API로 사용 가능
- 공통 상수/타입/함수(Private) 제공 → 개발 품질 향상

---

# 1. PACKAGE SPEC 구조

## 1-1. 예제 SPEC 코드

```sql
CREATE OR REPLACE PACKAGE PKG_LOT_PROCESS IS

    ----------------------------------------
    -- 1) 공통 상수(Constant)
    ----------------------------------------
    C_SUCCESS  CONSTANT VARCHAR2(1) := 'Y';
    C_FAIL     CONSTANT VARCHAR2(1) := 'N';

    ----------------------------------------
    -- 2) 사용자 정의 타입(Type)
    ----------------------------------------
    TYPE LOT_INFO_REC IS RECORD (
        LOT_NO     VARCHAR2(30),
        PROD_CODE  VARCHAR2(20),
        STATUS     VARCHAR2(10)
    );

    ----------------------------------------
    -- 3) 외부에 공개되는 PUBLIC PROCEDURE
    ----------------------------------------
    PROCEDURE START_LOT(
        P_LOT_NO IN VARCHAR2,
        P_USER   IN VARCHAR2,
        O_RESULT OUT VARCHAR2
    );

    PROCEDURE FINISH_LOT(
        P_LOT_NO IN VARCHAR2,
        P_USER   IN VARCHAR2,
        O_RESULT OUT VARCHAR2
    );

END PKG_LOT_PROCESS;
/
```

---

## 1-2. 설명

- SPEC 영역은 API 명세서

  - 어떤 프로시저/함수를 외부에서 사용할 수 있는지 정의

  - 내부 구현은 전혀 없음

- 상수(Constant) 정의

  - 성공/실패 코드 등 시스템 전체에서 동일 규칙 적용 가능

- 사용자 정의 타입(Type)

  - 여러 컬럼을 묶어서 반환해야 할 때 유용

  - ERP/MES 통합 보고서에서 자주 활용

- PUBLIC PROCEDURE 목록

  - 외부(C#, Python, MES/PDA 등)에서 호출 가능한 API만 여기에 선언
 
---

# 2. PACKAGE BODY 구조

## 2-1. 예제 BODY 코드

```sql
CREATE OR REPLACE PACKAGE BODY PKG_LOT_PROCESS IS

    ----------------------------------------------------
    -- 1) 내부 전용 함수 (Private Function)
    --    SPEC 에 정의되어 있지 않음 = 외부 호출 불가
    ----------------------------------------------------
    FUNCTION FN_CHECK_LOT_EXISTS(P_LOT_NO VARCHAR2) RETURN NUMBER IS
        V_CNT NUMBER;
    BEGIN
        SELECT COUNT(*)
          INTO V_CNT
          FROM TB_LOT
         WHERE LOT_NO = P_LOT_NO;

        RETURN V_CNT;
    END FN_CHECK_LOT_EXISTS;


    ----------------------------------------------------
    -- 2) LOT 시작 처리 Procedure
    ----------------------------------------------------
    PROCEDURE START_LOT(
        P_LOT_NO IN VARCHAR2,
        P_USER   IN VARCHAR2,
        O_RESULT OUT VARCHAR2
    ) IS
    BEGIN
        IF FN_CHECK_LOT_EXISTS(P_LOT_NO) = 0 THEN
            O_RESULT := 'LOT_NOT_FOUND';
            RETURN;
        END IF;

        UPDATE TB_LOT
           SET STATUS      = 'START',
               UPDATE_USER = P_USER,
               UPDATE_TIME = SYSDATE
         WHERE LOT_NO      = P_LOT_NO;

        O_RESULT := C_SUCCESS;
    EXCEPTION
        WHEN OTHERS THEN
            O_RESULT := C_FAIL;
    END START_LOT;


    ----------------------------------------------------
    -- 3) LOT 완료 처리 Procedure
    ----------------------------------------------------
    PROCEDURE FINISH_LOT(
        P_LOT_NO IN VARCHAR2,
        P_USER   IN VARCHAR2,
        O_RESULT OUT VARCHAR2
    ) IS
    BEGIN
        UPDATE TB_LOT
           SET STATUS      = 'FINISH',
               UPDATE_USER = P_USER,
               UPDATE_TIME = SYSDATE
         WHERE LOT_NO      = P_LOT_NO;

        O_RESULT := C_SUCCESS;
    EXCEPTION
        WHEN OTHERS THEN
            O_RESULT := C_FAIL;
    END FINISH_LOT;

END PKG_LOT_PROCESS;
/
```

---

## 2-2. 설명

- FN_CHECK_LOT_EXISTS → Private Function

  - SPEC에 없음 → 외부 호출 불가

  - BODY 내부에서만 사용하는 검증용 함수

- START_LOT / FINISH_LOT → Public Procedure

  - SPEC에 선언되어 있으므로 외부에서 직접 호출 가능

  - MES/PDA/ERP 시스템도 호출할 수 있는 대표 API

- 예외 처리(WHEN OTHERS)

  - 에러 발생 시 O_RESULT에 실패 코드 입력

- 공통 상수 사용

  - C_SUCCESS / C_FAIL 로 결과 상태 일원화
 
---

# 3. SPEC vs BODY 비교 정리

| 구분         | SPEC             | BODY             |
| ---------- | ---------------- | ---------------- |
| 역할         | API 명세서          | 실제 구현부           |
| 외부 노출      | 됨                | 안됨               |
| PRIVATE 함수 | 불가능              | 가능               |
| 타입/상수 선언   | 가능               | 가능 (보통 SPEC에 정의) |
| 코드 길이      | 짧음               | 길어짐              |
| 변경 시 영향    | API 변경 시 외부 영향 큼 | 내부 변경만이면 영향 없음   |

---

# 4. 실무에서 Package를 사용하는 이유
✔ 1) 유지보수 쉬움

SPEC은 바뀌지 않고 BODY만 바뀌면 외부 시스템은 영향 없이 로직만 개선 가능.

✔ 2) 공통 모듈화

여러 프로그램이 같은 패키지 API를 호출하여 중복 코드 없이 통일된 규칙 적용 가능.

✔ 3) PRIVATE 함수로 안정성 ↑

외부에서 호출하면 안 되는 로직을 숨길 수 있음.

✔ 4) SPEC = API 문서 역할

C#, Python, MES/PDA 시스템 개발자에게 “이 패키지에서 제공하는 기능 목록”을 전달할 수 있음.

---

✔ 요약

- PACKAGE SPEC = 공개 API 정의

- PACKAGE BODY = 실제 구현

- Private Function 으로 내부 기능 캡슐화

- 공통 상수/타입 관리에 유리

- MES/PDA/ERP 같은 제조 IT에서는
  “LOT 시작/완료 / 출고 / 입고 / ERP 연계” 같은 기능 단위로 Package 설계하면 유지보수 최적화됨
