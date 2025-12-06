---
title: PL/SQL Package 구조 분석
---

# PL/SQL Package 구조 분석

이 문서는 Oracle PL/SQL에서 가장 강력한 구성 요소 중 하나인  
**PACKAGE (명세서 SPEC + 구현 BODY)** 구조를 이해하기 쉽게 정리한 페이지입니다.

Package는 실무에서 다음 이유로 매우 중요합니다:

- 여러 Procedure / Function을 하나의 모듈로 묶어 관리할 수 있습니다.
- SPEC과 BODY 분리 덕분에 유지보수 및 API 관리가 용이합니다.
- MES/PDA/ERP 등 여러 시스템에서 공통 API로 활용할 수 있습니다.
- 공통 상수, 타입, Private 함수 등을 제공하여 개발 품질을 향상시킬 수 있습니다.

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

- SPEC 영역 = API 명세서 역할

    - 어떤 Procedure / Function을 외부에서 사용할 수 있는지 정의합니다.

    - 내부 구현 내용은 포함하지 않습니다.

- 상수(Constant) 정의

    - 성공/실패 코드 등 시스템 전체에 공통으로 적용되는 규칙을 정의할 수 있습니다.

- 사용자 정의 타입(Type)

    - 여러 컬럼을 하나의 RECORD 타입으로 묶어 반환해야 할 때 유용합니다.

    - ERP/MES 통합 보고서, 공정 정보 전달 등에서 활용될 수 있습니다.

- PUBLIC PROCEDURE 목록

    - SPEC에 선언된 Procedure / Function만 외부(C#, Python, MES/PDA 등)에서 호출 가능합니다.

    - 외부에 제공할 API 목록을 SPEC이 문서처럼 보여주는 역할을 합니다.
 
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

- FN_CHECK_LOT_EXISTS (Private Function)

    - SPEC에 선언되어 있지 않으므로 외부에서 직접 호출할 수 없습니다.

    - PACKAGE 내부에서만 사용하는 검증용 함수입니다.

- START_LOT, FINISH_LOT (Public Procedure)

    - SPEC에 선언되어 있으므로 외부에서 호출할 수 있습니다.

    - MES/PDA/ERP 시스템에서 LOT 시작/완료 처리 시 공통 API로 사용할 수 있습니다.

- 예외 처리(WHEN OTHERS)

    - 오류 발생 시 O_RESULT에 공통 실패 코드(C_FAIL)를 설정하는 방식으로 결과 코드 규칙을 일관되게 유지할 수 있습니다.

- 공통 상수 사용

    - C_SUCCESS, C_FAIL과 같은 상수를 사용하면 결과 상태를 시스템 전반에서 일관성 있게 관리할 수 있습니다.
 
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
1) 유지보수 용이성
    - SPEC이 바뀌지 않는 한, BODY 내부 로직을 변경해도 외부 프로그램(C#, MES, PDA 등)은 수정 없이 그대로 사용할 수 있습니다.

2) 공통 모듈화
    - 여러 프로그램이 동일한 Package API를 호출함으로써 중복 코드를 줄이고, 규칙을 일관되게 적용할 수 있습니다.

3) Private Function을 통한 안정성 향상
    - 외부에서 호출하면 안 되는 로직은 BODY 내부 Private 함수로 감춰 잘못된 사용을 방지할 수 있습니다.

4) SPEC = API 문서 역할
    - SPEC 파일 자체가 “이 패키지가 제공하는 기능 목록”이 되므로, 타 시스템 개발자(C#, Python, 웹 등)에게 API 문서처럼 제공할 수 있습니다.

---

✔ 요약

- PACKAGE SPEC은 공개 API(Procedure, Function)를 정의하는 영역입니다.

- PACKAGE BODY는 실제 구현 로직을 담고 있으며, Private Function을 통해 내부 로직을 캡슐화할 수 있습니다.

- 공통 상수와 타입을 Package로 관리하면, 시스템 전반의 규칙을 일관되게 유지할 수 있습니다.

- MES/PDA/ERP 등 제조 IT 환경에서는 “LOT 시작/완료, 출고, 입고, ERP 연계”와 같은 기능 단위로 Package를 설계하면 유지보수성과 확장성이 크게 향상됩니다.
