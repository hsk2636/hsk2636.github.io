---
title: CURSOR / BULK / FORALL 실전 예제
---

# CURSOR / BULK COLLECT / FORALL 실전 예제

이 문서는 Oracle PL/SQL에서 자주 사용되는  
**Cursor(커서)**, **Cursor FOR LOOP**, **파라미터 커서**,  
그리고 대량 처리 성능을 위한 **BULK COLLECT**, **FORALL**까지  
실무 흐름에 맞춰 정리한 페이지입니다.

MES/PDA/ERP 배치 작업에서도 많이 사용되는 핵심 문법들이므로  
원리와 사용 시점을 함께 정리했습니다.

---

# 1. 기본 CURSOR 사용법

## 1-1. 문제 정의
`TB_CUSTOMER` 테이블의 고객 목록을 하나씩 가져와  
`DBMS_OUTPUT`으로 출력하는 기본 예제를 구성합니다.

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

- CURSOR 선언
→ SELECT 결과를 한 행씩 가져오기 위한 논리적 포인터입니다.

- OPEN → FETCH → CLOSE
→ 커서를 명시적으로 열고, 가져오고, 닫는 전형적인 구조입니다.

- %NOTFOUND
→ 더 이상 읽을 행이 없을 때 TRUE가 되어 반복을 종료합니다.

- 전통적인 CURSOR 구조로, 세밀한 제어가 필요할 때 사용합니다.

---

# 2. Cursor FOR LOOP (가장 실무적)

## 2-1. 문제 정의

위의 커서를 더 간단하게 작성하는 방법을 확인합니다.

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

- OPEN / FETCH / CLOSE를 자동으로 처리해 주는 구조입니다.

- 코드가 간결하고 가독성이 좋아 실무에서 가장 많이 사용하는 커서 패턴입니다.

- 개별 행에 대한 처리가 필요할 때 1차적으로 고려할 수 있는 방식입니다.

---

# 3. 파라미터 커서 (Parameterized Cursor)

## 3-1. 문제 정의

지역(REGION) 조건만 다르고 같은 SELECT를 반복 사용할 때 파라미터 커서를 사용하여 코드 가독성을 높입니다.

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

- 커서 호출 시 파라미터를 전달하면, 해당 조건에 맞는 SELECT가 실행됩니다.

- 공정 코드별, 작업장별, 창고별 조회 등 조건만 다른 동일 패턴의 조회에서 유용합니다.

- SELECT 문을 재사용할 수 있어 코드가 짧고 유지보수가 쉬워집니다.

---

# 4. BULK COLLECT (대량 조회)

## 4-1. 문제 정의

TB_ORDER 테이블에서 최근 주문 데이터를
여러 건 한 번에 메모리 배열로 가져와 처리하고자 합니다.
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

- BULK COLLECT
→ 여러 행을 한 번에 컬렉션(배열)로 가져오는 방식입니다.

- 일반적인 커서 + FETCH 방식보다 네트워크 왕복 횟수가 줄어들어 대량 데이터 처리 시 성능 이점이 큽니다.

- 배치(Batch) 작업, 인터페이스 처리 등에서 필수적으로 고려되는 기능입니다.

---

# 5. FORALL (대량 INSERT / UPDATE)

## 5-1. 문제 정의

BULK COLLECT로 조회한 데이터를 로그 테이블에 대량으로 INSERT 할 때 성능을 높이고자 합니다.

---

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

- FORALL
→ 반복적인 INSERT/UPDATE 문을 대량 처리 형태로 실행하여 성능을 최적화합니다.

- 응용 프로그램과 DB 사이의 왕복 횟수를 줄여 대량 DML에서 일반 FOR LOOP보다 훨씬 빠른 성능을 보입니다.

- ERP/MES 인터페이스 및 배치 작업에서 자주 활용됩니다.

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

✔ 요약

- 일반적인 반복 처리에는 CURSOR FOR LOOP를 우선적으로 고려합니다.

- 조건값만 다른 동일 SELECT는 파라미터 커서(Parameterized Cursor)를 활용하면 효율적입니다.

- **대량 데이터 처리(배치, 인터페이스)**에는 BULK COLLECT + FORALL 조합이 효과적입니다.

- 커서, 컬렉션, 반복문 패턴을 익히면 PL/SQL 처리 성능과 구조를 함께 개선할 수 있습니다.
