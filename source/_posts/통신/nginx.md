# proxy_set_header Host $host vs $http_host의 차이

## 대략적인 의미

- `$host`는 호스트명 중심
- `$http_host`는 클라이언트가 보낸 Host 헤더 원문, 원문에는 보통 포트 정보까지 같이 들어옴

## 문제상황

- 설정값에 포트를 적어 놨는데도 불구하고 계속 포트 부분이 삭제가 되는 이상한 현상 때문에 해당 문제 정리함

## 통신 구성

- app 1 -> 서버로 통신 -> nginx -> reverse proxy -> 서버 도달
- 문제가 생긴 지점은 nginx가 백엔드로 요청을 넘길 때
- 원래 내가 받은 nginx conf 파일에는 `$host`만 되어 있었고, 그래서 백엔드가 계속 포트 정보 없이 받고 있었음
- 백엔드에서는 `req.get('host')` 이런 식으로 nginx가 넘겨준 header 값만 보고 있었음

## $host

- nginx가 내부적으로 정규화하여 쓰는 host 값
- 요청라인, Host 헤더를 먼저 찾고 server_name 매칭값 찾고 없으면 로컬 주소로 정규화한다고 함

## 로컬에서는 왜 포트가 생략되지 않았나

- nginx가 없기 때문에, 백엔드 서버가 직접 받는 헤더는 생략이 되지 않음

## node의 new URL()

- 기본 포트만 생략 가능
