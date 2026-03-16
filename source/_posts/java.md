# 예외처리

## DTO + Validation 도입 필요한 코드

```sql
    public Map<String, Object> searchLasConfig(Map<String, Object> params) {
//        params가 null일 떄 터지는 걸 막는 안전장치
//        params null이면 request에 빈 맵 넣음
//        null일 때 params.get.. 할 때 NullPointerException 발생
//        아... null 처리를 이렇게 하는 구나
        Map<String, Object> request = params == null ? new HashMap<String, Object>() : params;
```

- pydantic처럼 처리할 수 있는 방안 필요
- DTO + Validation 도입하면 아래와 같이 처리 가능

```sql
public class LasConfigSearchRequest {
    private String libraryCode;
    private String lasType;

    public String getLibraryCode() {
        return libraryCode;
    }

    public void setLibraryCode(String libraryCode) {
        this.libraryCode = libraryCode;
    }

    public String getLasType() {
        return lasType;
    }

    public void setLasType(String lasType) {
        this.lasType = lasType;
    }
}

public class LasConfigSearchRequest {

    @NotBlank
    private String libraryCode;

    private String lasType;

    // getter/setter
}

# 컨트롤러는 간단하게
@PostMapping("/search")
public ResponseEntity<?> search(@Valid @RequestBody LasConfigSearchRequest request) {
    return ResponseEntity.ok(service.searchLasConfig(request));
}
```

- Controller에서 DTO로 받고 검증, Service 레이어는 검증된 DTO를 받는다
- 레거시 코드여서 지금 고칠 필요는 없으나, 저렇게 Null 값을 처리하는 것이 바람직하진 않음.

## DB transaction 추가하기

- 1개의 비지니스 로직에서 서로 다른 테이블에 접근하여 데이터를 저장할 때, transaction 추가 필요
- 수정 대상인 함수를 하나의 transaction 단위로 만들기
  - 함수 안의 B라는 구간에서 실패하면 A도 실패, 롤백 적용 대상아 너무
- 트랜잭션, 실패 시 단순 return이 아니라 실패 시 throw

````sql
// 예외 다시 던지기
  } catch (Exception e) {
      java.lang.System.out.println("e :::::: " + e.getMessage());
      throw new RuntimeException(e.getMessage(), e);
  }
- @Transaction이 롤백 판단
- 원래 코드는 여기에 없지만, 단순 에러를 발생시켜 return하는 것이 아니라 스프링으로 예외 처리를 던지는 것으로 변경함, 이렇게 되면 컨트롤러나 해당 함수의 상위에서 이 에러 메세지를 받아 처리하는 작업이 필요함

// 컨트롤러로 던저지는 에외를 컨트롤 할 수 있게 변경
  @PostMapping(value="/saveData")
  public Map<String, Object> saveData(@RequestBody Map<String, Object> params) throws Exception {
      try {
          return libraryService.saveData(params);
      } catch (Exception e) {
          Map<String, Object> result = new HashMap<>();
          result.put("resultMsg", e.getMessage());
          return result;
      }
  }

## MyBatis 설정 기초

### secretKey 동작 원리

- 프론트엔드에서 보낸 params 값에 백엔드가 계속 값을 추가하면서 사용을 하고 있음
- libraryMapper.insertLibData(params) 실행
- MyBatis selectKey가 동작하면서 params 객체 안에 libNoBySeq를 추가로 넣어줌, 실제로 찍어보면 객체 JSON 밖에 생성된 키가 들어와있음

- selectKey의 동작

```sql
  <selectKey order="BEFORE" keyProperty="libNoBySeq" resultType="int">
      select NLIB_SEQ.NEXTVAL from DUAL
  </selectKey>
````

- insertLibData(params) 호출할 때
- MyBatis가 먼저 select NLIB_SEQ.NEXTVAL from DUAL 실행
- 그 결과를 libNoBySeq라는 이름으로 parameter object에 넣음
- 지금 parameter object가 Map<String, Object> params 이므로 결국 params.put("libNoBySeq", 시퀀스값)처럼 동작

- 그래서 자바 백엔드 코드에서 params.get(”libNoBySeq”)로 접근이 가능, 방금 INSERT된 데이터에 대해서 접근 가능
- 클라이언트에서 백엔드로 넘겨주는 params 객체를 sql 실행 시 똑같이 parameter object로 실행을 하게됨
- 자바 메서드의 지역변수 이름: params
- 그 변수 안에 들어있는 실제 객체: Map<String, Object>
- MyBatis가 값을 써넣는 대상: 이 Map 객체 자체
- MyBatis가 넣는 항목 이름: keyProperty="libNoBySeq"

- MyBatis는 보통 **insert 성공하면 1 반환**
