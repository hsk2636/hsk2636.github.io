---
title: CURSOR / BULK / FORALL 실전 예제
---

# CURSOR / BULK COLLECT / FORALL 실전 예제

이 문서는 Oracle PL/SQL에서 자주 사용되는  
**Cursor(커서)**, **Cursor FOR LOOP**, **파라미터 커서**,  
그리고 대량 처리 성능을 위한 **BULK COLLECT**, **FORALL**까지  
실무 흐름에 맞춰 정리한 문서입니다.

MES/PDA/ERP 배치 작업에서도 많이 사용되는 핵심 문법들이므로  
원리와 사용 시점을 함께 정리했습니다.

---

# 1. 기본 CURSOR 사용법

## 1-1. 문제 정의
`TB_CUSTOMER` 테이블의 고객 목록을 하나씩 가져와  
DBMS_OUTPUT으로 출력하는 기본 예제를 만든다.

---

## 1-2. SQL 코드

```sql
DECLARE
    CURSOR cur_cust IS
        SELECT cust_id, cust_name
          FROM tb_customer;

    v_id   tb_customer.cust_id%TYPE;
    v_name tb_customer.cust_name%TYPE;
BEGIN
    OPEN cur_cust;

    LOOP
        FETCH cur_cust INTO v_id, v_name;
        EXIT WHEN cur_cust%NOTFOUND;

        DBMS_OUTPUT.PUT_LINE(v_id || ' : ' || v_name);
    END LOOP;

    CLOSE cur_cust;
END;
/
```

---

## 1-3. 설명

- CURSOR 선언 → SELECT 결과를 한 행씩 가져옴

- OPEN → FETCH → CLOSE → 명시적 제어

- %NOTFOUND → 더 이상 행이 없을 때 반복 종료

- 전통적 CURSOR 구조이며, 세밀한 제어가 필요한 경우 사용

# 2. Cursor FOR LOOP (가장 실무적)

## 2-1. 문제 정의

위의 커서를 더 간단하게 작성하는 방법을 알아본다.

## 2-2. SQL 코드

```sql
BEGIN
    FOR rec IN (
        SELECT cust_id, cust_name
          FROM tb_customer
    ) LOOP
        DBMS_OUTPUT.PUT_LINE(rec.cust_id || ' : ' || rec.cust_name);
    END LOOP;
END;
/
```

## 2-3. 설명

- OPEN / FETCH / CLOSE 자동 처리

- 구조가 간단하고 가독성이 좋음

- 실무에서 가장 많이 사용하는 커서 패턴

# 3. 파라미터 커서 (Parameterized Cursor)

## 3-1. 문제 정의

지역(REGION) 조건만 다르고 같은 SELECT를 반복 사용할 때 파라미터 커서를 사용하여 코드 가독성을 높인다.

## 3-2. SQL 코드

```sql
DECLARE
    CURSOR cur_cust(p_region VARCHAR2) IS
        SELECT cust_id, cust_name
          FROM tb_customer
         WHERE region = p_region;
BEGIN
    FOR rec IN cur_cust('SEOUL') LOOP
        DBMS_OUTPUT.PUT_LINE('[서울] ' || rec.cust_name);
    END LOOP;

    FOR rec IN cur_cust('BUSAN') LOOP
        DBMS_OUTPUT.PUT_LINE('[부산] ' || rec.cust_name);
    END LOOP;
END;
/
```

---

## 3-3. 설명

- 커서 호출 시 값을 넘기면 → 해당 조건으로 SELECT 실행

- 공정코드별 / 작업장별 / 창고별 조회에 자주 사용

- SELECT 재사용성이 좋아지고 코드가 짧아짐

---

# 4. BULK COLLECT (대량 조회)

## 4-1. 문제 정의

TB_ORDER 테이블에서 최근 주문 데이터를
여러 건 한 번에 메모리 배열로 가져오고 싶다.

---

## 4-2. SQL 코드

```sql
DECLARE
    TYPE t_order_amt IS TABLE OF tb_order.order_amt%TYPE;
    v_list t_order_amt;
BEGIN
    SELECT order_amt
      BULK COLLECT INTO v_list
      FROM tb_order
     WHERE order_date >= SYSDATE - 30;

    DBMS_OUTPUT.PUT_LINE('총 로우 수 : ' || v_list.COUNT);
END;
/
```

---

## 4-3. 설명

- BULK COLLECT → 여러 행을 한 번에 가져오는 방식

- FOR LOOP로 한 건씩 FETCH하는 것보다 수십~수백 배 빠름

- 배치(Batch) 작업에서 필수 기능

---

# 5. FORALL (대량 INSERT / UPDATE)

## 5-1. 문제 정의

대량 조회(BULK COLLECT)한 데이터를 빠르게 로그 테이블에 INSERT 하고 싶다.

## 5-2. SQL 코드

```sql
DECLARE
    TYPE t_order_list IS TABLE OF tb_order%ROWTYPE;
    v_orders t_order_list;
BEGIN
    -- 1) BULK 조회
    SELECT *
      BULK COLLECT INTO v_orders
      FROM tb_order
     WHERE order_date >= SYSDATE - 30;

    -- 2) FORALL 로 빠른 INSERT
    FORALL i IN 1 .. v_orders.COUNT
        INSERT INTO tb_order_log (
            log_id,
            order_id,
            order_amt,
            reg_time
        ) VALUES (
            seq_order_log.NEXTVAL,
            v_orders(i).order_id,
            v_orders(i).order_amt,
            SYSDATE
        );

    DBMS_OUTPUT.PUT_LINE('총 INSERT 건수 : ' || v_orders.COUNT);
END;
/
```

---

## 5-3. 설명

- FORALL → 반복 INSERT 시 성능 최적화

- 네트워크 왕복이 1회이기 때문에 매우 빠름

- ERP/MES 인터페이스·배치 작업에서 필수적

---

# 6. CURSOR / BULK / FORALL 비교 요약

| 기능              | 목적                  | 장점                  |
| --------------- | ------------------- | ------------------- |
| 명시적 CURSOR      | 한 행씩 제어 필요          | 정교한 제어 가능           |
| CURSOR FOR LOOP | 반복 구조 단순화           | 가장 읽기 쉽고 실무에서 많이 사용 |
| 파라미터 커서         | 조건값만 바뀌는 SELECT 재사용 | 유지보수성 높음            |
| BULK COLLECT    | 대량 조회               | 성능 대폭 향상            |
| FORALL          | 대량 INSERT/UPDATE    | 배치 처리 속도 극대화        |

---

✔ 결론

- 일상적인 반복 로직 → CURSOR FOR LOOP

- 조건 변형이 많을 때 → PARAMETERIZED CURSOR

- 대량 처리(Batch, ERP/MES 인터페이스) → BULK COLLECT + FORALL

- 커서/배열/반복 패턴을 익히면 PL/SQL 처리 성능을 높일 수 있음
