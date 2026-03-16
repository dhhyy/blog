# 데이터베이스 (프로시저)

### CREATE PROCEDURE

```sql
CREATE PROCEDURE TEST_PROC
@ID INT
AS
BEGIN
    SELECT @ID
END
```

```sql
CREATE PROCEDURE [dbo].[프로시저 이름]
@ROOM_NO INT

==> 실행 EXEC WEB_PROC_EZ5500_BATCH_ISSUE 1
```

- 입력값으로 `@ROOM_NO` 하나를 받음
- 타입은 `INT`
- 프로시저는 **DB 안에 저장해두고 실행하는 SQL 프로그램**

### 파라미터

```sql
@USER_NO INT
```

- 프로시저를 실행할 때 외부에서 파라미터를 하나 받음
- 프로시저 파라미터는 **외부에서 전달받는 입력값**

### BEGIN ... END

```sql
BEGIN
    SELECT 1
END
```

- `BEGIN ... END`는 **여러 SQL 문장을 하나의 묶음으로 처리**할 때 사용

### SET NOCOUNT OFF

```sql
SET NOCOUNT OFF;
```

- `NOCOUNT`는 **영향받은 행 수 메시지를 출력할지 말지** 정하는 옵션

### BEGIN TRAN

```sql
BEGIN TRAN
UPDATE USER_INFO SET NAME = 'A' WHERE ID = 1
COMMIT TRAN
```

- 이 뒤에서 수행하는 작업들을 **하나의 거래 단위**로 묶고, 즉, 중간에 에러가 나면 전부 취소하고 끝까지 성공하면 전부 반영.
- 트랜잭션은 **전부 성공하거나 전부 실패하게 만드는 묶음**.

### BEGIN TRY ... END TRY

```sql
BEGIN TRY
    SELECT 1 / 0
END TRY
```

- `TRY`는 **정상 실행을 시도하는 구간**

### BEGIN CATCH ... END CATCH

```sql
BEGIN CATCH
    PRINT '에러 발생'
END CATCH
```

```sql
BEGIN CATCH
    IF @@TRANCOUNT > 0
    BEGIN
        ROLLBACK TRAN
    END
END CATCH
```

- 에러가 발생하면 트랜잭션이 살아 있는지 확인하고, 살아 있으면 롤백
- 에러 메시지도 추가 가능
- CATCH 구문이라고 생각하면 된다

### DECLARE

```sql
DECLARE @NUMBER INT
DECLARE @USER_ID VARCHAR(20)
```

- 프로시저 안에서 사용할 변수를 선언

### SET

```sql
SET @NAME = 'KIM'
```

```sql
SET @USER_IDS = ''
SET @ROOM_NOS = ''
SET @NUMBERS = ''
```

- 변수에 값을 대입할 때 사용

## SELECT

### SELECT 변수 = 변수 + 값

```sql
SELECT @LIST = @LIST + NAME + ','
FROM USER_INFO
```

```sql
SELECT @USER_IDS = @USER_IDS + CONVERT(VARCHAR, ROOM_NO) + '/' + CONVERT(VARCHAR, NUMBER) + N',',
       @ROOM_NOS = @ROOM_NOS + CONVERT(VARCHAR, ROOM_NO) + N',',
       @NUMBERS = @NUMBERS + CONVERT(VARCHAR, NUMBER) + N','
FROM [EZ5500_ROOM]
WHERE ROOM_NO = @ROOM_NO AND USE_TYPE = 0

----
@ROOM_NOS = '1,1,1,'
@NUMBERS = '10,11,12,'
@USER_IDS = '1/10,1/11,1/12,'
```

- 조회 결과를 문자열로 이어 붙이는 방식

### CONVERT

```sql
SELECT CONVERT(VARCHAR, 100)
```

```sql
CONVERT(VARCHAR, ROOM_NO)
CONVERT(VARCHAR, NUMBER)
CONVERT(INT, SUBSTRING(...))
```

- 숫자를 문자열로 바꾸거나, 문자열을 숫자로 바꾸는 함수

### WHILE

```sql
WHILE @I < 10
BEGIN
    SET @I = @I + 1
END
```

```sql
WHILE CHARINDEX(',', @ROOM_NOS) <> 0
BEGIN
    ...
END
```

- `@ROOM_NOS` 안에 콤마가 있는 동안 계속 반복, 콤마로 끝나는 문자열을 하나씩 잘라서 처리하는 방식으로 사용
- 조건이 참인 동안 반복 실행

### CHARINDEX

```sql
SELECT CHARINDEX(',', '10,20,30')
```

- 문자열 안에서 , 의 위치를 찾음
- 문자열 안에서 특정 문자 위치를 찾는 함수

### EXEC

```sql
EXEC TEST_PROC 1

EXEC ROOM_MANAGE_SEAT_ISSUE @USER_ID, @ROOM_NO, @NUMBER
```

- 반복문에서 잘라낸 값을 가지고 다른 프로시저를 호출

### IF

```sql
IF @A > 0
BEGIN
    PRINT 'OK'
END

IF @@TRANCOUNT > 0
BEGIN
    COMMIT TRAN
END
```

- 트랜잭션이 실제로 열려 있을 때만 처리하도록 조건을 걸 수 있음

### @@TRANCOUNT

- 열려 있는 트랜잭션 갯수 확인

### ROLLBACK TRAN

- 트랜잭션 작업 취소

## 프로시저 주의 사항

1. 구조
   1. 프로시저 생성
   2. 파라미터 입력
   3. 트랜잭션 시작
   4. 대상 데이터 조회
   5. 문자열 누적
   6. 반복문
   7. (하위 프로시저 실행)
   8. 성공 시 커밋, 실패 시 롤백

---

# 데이터베이스(oracle)

### NVL

```sql
SELECT NVL(phone, '없음')
FROM user;
```

- 값이 `NULL`이면 **대체 값을 반환**한다.

### MAX

```sql
SELECT MAX(score)
FROM exam;
```

- 컬럼에서 **가장 큰 값**을 반환하는 **집계 함수**

### KEEP (DENSE_RANK FIRST / LAST)

- KEEP (DENSE_RANK FIRST ORDER BY ...)

```sql
SELECT MAX(price)
KEEP (DENSE_RANK FIRST ORDER BY created_date)
FROM product;
```

- 정렬 기준에서 **가장 앞(또는 뒤)에 있는 행의 값을 가져온다**

### DENSE_RANK

```sql
SELECT
    name,
    DENSE_RANK() OVER (ORDER BY score DESC)
FROM student;
```

- 정렬 기준으로 **순위를 매긴다 (동점은 같은 순위)**

### ORDER BY

```sql
SELECT *
FROM user
ORDER BY name;
```

- 결과를 **정렬**한다.

### 서브쿼리 (Subquery)

```sql
SELECT
    name,
    (SELECT MAX(score) FROM exam)
FROM student;

```

- SELECT 안에 또 SELECT를 사용하는 쿼리

### 상관 서브쿼리 (Correlated Subquery)

- 외부 컬럼 참조

```sql
SELECT *
FROM library L
WHERE EXISTS (
    SELECT *
    FROM book
    WHERE book.lib_no = L.lib_no
);
```

- 서브쿼리가 바깥 쿼리의 컬럼을 참조하는 쿼리

## 문자열

### 문자열 연결

```sql
SELECT 'Hello' || 'World'
FROM dual;
```

## JOIN

### LEFT JOIN

```sql
SELECT *
FROM user U
LEFT JOIN order O
ON U.id = O.user_id;
```

- 왼쪽 테이블 기준으로 **모든 행을 유지하면서 조인**
