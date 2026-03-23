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

## 오라클 데이터베이스 복구, 복원 시 주의

### dump

- impdb 방식과 imp 방식이 있음
- 덤프가 이뤄진 동일한 방식으로 복원을 해야함
- dump 파일 내부 DDDL에 `TABLESPACE “SOMETHING”` 이 명시가 되어 있는 경우 tablespace도 만들어야함
- 복구 사용자와 tablespace가 달라도 되는 것 같음, 파일만 만들어주면 동작함

### tablespace 확인

```sql
SELECT NAME FROM V$DATABASE

  - `C:\APP\ADMINISTRATOR\ORADATA\K3\~~~~USERS01.DBF`
  - `C:\APP\ADMINISTRATOR\ORADATA\K3\~~~~UNDOTBS01.DBF`
  - `C:\APP\ADMINISTRATOR\ORADATA\K3\~~~~SYSAUX01.DBF`
  - `C:\APP\ADMINISTRATOR\ORADATA\K3\~~~~SYSTEM01.DBF`
  - `C:\APP\ADMINISTRATOR\ORADATA\K3\~~~~EXAMPLE01.DBF`
  - `C:\APP\ADMINISTRATOR\ORADATA\K3.DBF`
```

### 오라클은 객체 소유자와 저장공간을 분리해서 관리함

- 소유자와 저장공간이 다를 수가 있음
- 이런 경우에는 각각 권한을 맞춰줘야함

## 오라클 디비 구조

- 데이터베이스 > 인스턴스 > 사용자 > 스키마
- 데이터베이스
    - 현재 프로젝트의 경우에는 k3가 데이터베이스
- 인스턴스
    - 메모리와 백그라운드 프로세스 집합
- 사용자
    - 디비 객체를 소유하는 계정
- 스키마
    - 테이블, 시퀀스, 뷰, 프로시저 등
- 테이블 스페이스
    - oracle 객체가 저장되는 논리 저장공간
    - 사용자와 테이블 스페이스는 다르다
- 데이터 파일
    - 실제 os 파일
    - `.DBF`
    - 테이블스페이스는 논리 단위이고, 데이터파일은 실제 디스크 파일

```sql
Database
  -> Tablespace
    -> Datafile(.dbf)
  -> User/Schema
    -> Table / Index / Sequence / View
```

## 정리

- Oracle dump는 단순 SQL 파일이 아니다 ⇒ dump 생성 세대와 import 도구 세대가 다르면 예외가 발생한다, 새 스키마로 restore하더라도 저장공간 참조는 원래 이름을 유지한다.
- Oracle은 객체 정의 안에 tablespace 지정이 들어가 있을 수 있다.
- 최종적으로 `PORTAL01.DBF` 같은 새 파일명으로 우회 생성하는 것이 안전했다

## 명령어 정리

### 오라클 환경변수 설정

```sql
set ORACLE_HOME=D:\oracle\product\11.2.0\dbhome_1
set NLS_LANG=KOREAN_KOREA.KO16MSWIN949
set PATH=%ORACLE_HOME%\bin;%PATH%
exp PORTAL/PORTAL123##@NICOMCS file=D:\backup2026-03-20.dmp log=D:\backup2026-03-20.log owner=PORTAL
```

### sqlplus 찾기

```sql
where sqlplus
where exp
where imp

# 오라클이 서버에 2개가 설치가 되어 있을 수가 있다
C:\app\Administrator\product\11.2.0\dbhome_1\BIN\sqlplus / as sysdba

# 백업 시 버전 명시
C:\app\Administrator\product\11.2.0\dbhome_1\BIN\imp "'/ as sysdba'" file=D:\backup2026-03-20.dmp log=D:\import_portal_test.log fromuser=PORTAL touser=PORTAL_TEST
```

### 데이터베이스 찾기

```sql
SELECT NAME FROM V$DATABASE;
```

### 데이터 파일 경로 찾기

```sql
SELECT FILE_NAME FROM DBA_DATA_FILES;
```

### 테이블스페이스 만들기

```sql
CREATE TABLESPACE PORTAL_TEST
DATAFILE 'C:\APP\ADMINISTRATOR\ORADATA\K3\PORTAL_TEST.DBF'
SIZE 500M
AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED;
```

### 사용자 생성 후 권한 부여

```sql
// 사용자 삭제
DROP USER PORTAL_TEST CASCADE;

// 사용자 생성
CREATE USER PORTAL_TEST IDENTIFIED BY PORTAL123##
DEFAULT TABLESPACE PORTAL_TEST
TEMPORARY TABLESPACE TEMP;

// 권한 셋팅
GRANT CONNECT, RESOURCE TO PORTAL_TEST;
GRANT CREATE VIEW TO PORTAL_TEST;
GRANT CREATE SEQUENCE TO PORTAL_TEST;
GRANT CREATE SYNONYM TO PORTAL_TEST;
GRANT CREATE TRIGGER TO PORTAL_TEST;
GRANT UNLIMITED TABLESPACE TO PORTAL_TEST;
ALTER USER PORTAL_TEST QUOTA UNLIMITED ON PORTAL_TEST;
ALTER USER PORTAL_TEST QUOTA UNLIMITED ON PORTAL;
ALTER USER PORTAL_TEST QUOTA UNLIMITED ON USERS;
```

## 에러 정리

### IMP-00010: 엑스포트 파일이 유효하지 않고, 헤더가 검증에 실패했습니다

- 상황: 10g XE `imp` 사용
- 원인: 11g dump를 10g `imp`로 읽으려 했음
- 조치: 11g `imp`로 변경

### IMP-00002: dump file open failed

- 상황: 파일명을 잘못 입력하거나 전체 명령을 잘못 입력했을 때 발생
- 원인: 경로 오타 또는 shell 입력 오류
- 조치: dump 경로를 직접 `dir`로 확인하고 다시 실행

### ORA-01435: 사용자가 존재하지 않습니다

- 상황: import 대상 계정이 없는 상태
- 원인: `PORTAL_TEST`가 현재 접속한 11g DB에 생성되지 않았음
- 조치: 해당 DB에 `PORTAL_TEST` 생성 후 재실행

### ORA-00959: 테이블스페이스 'PORTAL'이(가) 존재하지 않습니다

- 상황: 일부 테이블 생성 중 실패
- 원인: dump 내부 DDL이 `TABLESPACE "PORTAL"` 직접 참조
- 조치: `PORTAL` tablespace 생성

### IMP-00015: 객체가 이미 존재하므로 ... 실패

- 상황: 부분 실패 후 같은 사용자로 재실행
- 원인: 시퀀스, 테이블, 뷰가 이미 생성된 상태
- 조치: `DROP USER PORTAL_TEST CASCADE` 후 깨끗하게 다시 생성

### ORA-01119 / ORA-27038 / ORA-27086

- 상황: datafile 생성 시 실패
- 원인: 기존 파일 존재, OS 잠금, 파일명 충돌
- 조치: 다른 파일명 사용

### IMP-00008: unrecognized statement

- 상황: 구형 dump와 최신 환경 조합에서 일부 발생 가능
- 원인: export/import 세대 차이, dump 포맷 차이
- 조치: 가능하면 원본 세대와 맞는 11g 환경 사용