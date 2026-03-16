# IIS (Internet Information Service)

- 마이크로소프트의 윈도우에서 무료로 지원되는 웹 서버

### 포트 찾기

```sql
netstat -ano | findstr :8080

---
PID : 4
```

- IIS로 서비스 되고 있는 프로세스의 이름은 나오지 않음.
- HTTP.sys 아니면 커널 레벨에서 서비스 되고 있는 프로세스가 포트를 잡고 있는 것을 확인 가능
- 예
  - IIS
  - Windows HTTP Service (HTTP.sys)
  - Web Deployment
  - WinRM
  - BranchCache
  - SQL Reporting Service
  - 일부 백신 / 프록시

### HTTP.sys 예약 목록 확인

```sql
netsh http show servicestate | findstr 8081

netsh http show urlacl

---
http://+:8081/
```

- HTTP.sys가 8081 포트를 사용하고 있는 것을 확인

### 호출 구조 정리 1

- 브라우저 → HTTP.sys(Window Kernel) → 어떤 특정 서비스
- netstat → PID 4
- servicestate → http
- `netsh http show urlacl` 명령어의 결과 없음
- 브라우저 → HTTP.sys (PID 4) → HTTP.sys Request Queue → 실제 서비스 프로세스

### 커널 단위 호출

```sql
Get-NetTCPConnection -LocalPort 8081 | Format-List
```

- 브라우저 → HTTP.sys (Kernel, PID 4) → HTTP Request Queue → User-mode 서비스

## request queue 찾기

```sql
netsh http show servicestate view=requestq

---
Request queue name: something
Process IDs: 1234
URLs:
    http://+:8081/
```

### 컨트롤러 프로세스와 프로세스의 차이

**Controller Process IDs**

- HTTP 요청 큐 관리하는 프로세스
- 리스너를 등록하고 제어하는 주체, 즉 IIS, HTTP.sys 주체, 앱 풀 관리자 역할

**Process IDs**

- 실제로 요청을 처리하는 워커 프로세스
- 실제 구동이 되고 있는 서비스 자치

### 호출 구조 정리 2

- 브라우저 → HTTP.sys (PID 4) → IIS → w3wp.exe → .NET 앱

## 터미널에서 IIS 서비스 되고 있는 경로 찾기는 번거로움

```sql
%windir%\system32\inetsrv\appcmd list site /text:name,physicalPath
```

- 이 명령어로 찾을 수 있으나 나의 경우에는 포트까지는 못찾음

### 터미널로만 찾으려면?

포트 8081 → w3wp.exe → appcmd list wp → ApplicationPool → appcmd list app → Site → appcmd list site → PhysicalPath
