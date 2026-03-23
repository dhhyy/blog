# 윈도우에서 wsl 환경 + codex 셋팅

## 윈도우에서 cli로 코덱스 사용 시 문제

- 한글 인코딩이 깨짐
- 파일 수정 권한 오류
- 그 밖에 뭔가.. 쾌적하지 않음

## 환경 체크

- WSL에서 `codex`를 실행했더니 `/mnt/c/Users/NICOM/AppData/Roaming/npm/codex`를 잡고 있었다.
- 그 결과 WSL 내부에서 실행되는 것처럼 보이지만 실제로는 Windows에 설치된 npm 글로벌 스크립트를 호출하고 있었다.
- 그 스크립트 안에서 `node`를 찾는데 WSL 내부에는 그 시점에 Node가 없어 `node: not found`가 발생했다.
- `source ~/.bashrc`를 실행했는데 `source: not found`가 나왔고, 이것으로 현재 셸이 `bash`가 아니라 `sh` 계열임을 알 수 있었다.
- `echo $SHELL`결과가 `/bin/sh`였고, 일반 사용자 계정이 `ubuntu`임도 확인되었다.
- 이후 `ubuntu` 사용자로 전환하고, WSL 내부에 `nvm`, `node`, `codex`를 다시 설치했다.
- 최종적으로 `type -a codex` 결과가 `/home/ubuntu/.nvm/versions/node/v22.22.1/bin/codex` 하나만 나오면서 WSL 내부 Codex를 정상적으로 잡는 상태가 확인되었다.

# 파일시스템

- Windows는 주로 NTFS 기반이며 드라이브 문자(C:, D:)를 기준으로 파일을 구분한다
- .Linux는 단일 루트(/) 아래에 모든 파일 시스템을 붙이는 방식이다.

### WSL 사용 시 연결

- WSL 내부 Linux 파일은 WSL 가상 디스크(VHDX) 안에 저장된다.
- Windows 드라이브는 WSL에서 DrvFs를 통해 /mnt/c, /mnt/d 같은 경로로 노출된다. 즉, WSL에서 /mnt/c/...는 “리눅스 파일이 저장된 곳”이 아니라 “Windows 파일 시스템을 WSL이 연결해서 보여주는 곳”이다.
- WSL2 기준으로 보면 Linux 루트 파일 시스템은 VHDX 기반의 ext4 계열 환경에 놓이고, Windows 볼륨은 DrvFs를 통해 /mnt/<drive>에 브리지된다. 이 브리지 레이어는 POSIX-like 접근을 제공하지만 Windows NT object model/ACL/metadata semantics를 완전히 동일하게 재현하는 것은 아니다. Microsoft는 DrvFs가 Windows 파일 시스템과의 상호운용을 위한 플러그인이라고 설명한다.

# 샌드박스 권한

- Codex는 현재 디렉터리를 기준으로 작업하는 도구지만, 안전을 위해 플랫폼별 샌드박싱/승인 모델을 사용한다. OpenAI는 Codex의 샌드박싱이 OS별 네이티브 방식으로 구현되며, macOS, Linux, WSL, native Windows에서 구현이 다르다고 설명한다.

## 윈도우에서의 샌드박스 권한

- Codex가 현재 프로젝트 폴더는 읽고 쓸 수 있음
- 그러나 시스템 폴더나 다른 민감한 위치는 기본적으로 접근이 제한될 수 있음
- 어떤 작업은 사용자 승인(approval)이 필요할 수 있음, 즉 AI 에이전트가 실수로 시스템 전체를 건드리거나 중요한 파일을 수정하지 못하게 경계를 치는 것이다.
- 실무적으로는 아래와 같은 범위까지 권한의 의미가 확대됨
    - filesystem scope: 어떤 루트/서브트리까지 접근 허용인가
    - network scope: 외부 네트워크 허용 여부
    - execution scope: 어떤 명령 실행 허용인가
    - approval model: 특정 작업 전 명시적 사용자 승인 필요 여부

# bash, sh 차이

- `sh`는 역사적으로 Bourne shell 계열의 인터페이스를 뜻하고, 현대 Linux에서는 `/bin/sh`가 실제로는 `dash` 또는 다른 POSIX shell을 가리키는 경우가 많다. 반면 `bash`는 GNU Bash로, POSIX shell 호환성을 갖추면서도 더 많은 확장 기능을 제공한다. GNU Bash는 Bash를 features-rich shell로 설명한다.
- `source`는 Bash에서 쓰지만 POSIX shell에서는 `.` 를 쓴다.
- 배열, 고급 문자열 처리, completion, 편의 기능은 Bash에서 더 풍부하다.
- `.bashrc`는 Bash 인터랙티브 셸용 설정 파일이다.

# locale 설정

- 문자 인코딩, 언어, 날짜/시간 형식, 숫자 형식, 정렬 규칙 등 지역화 동작에 영향을 주는 환경 설정이다.
- `LANG`
- `LC_ALL`
- `LC_CTYPE`
- `LC_TIME`
- `LC_NUMERIC`

## wsl 설치

## bash 설치 및 변경

### ✔ 자동완성 패키지 설치

```
sudo apt update
sudo apt install bash-completion-y
```

### ✔ 권한 변경

```sql
echo $SHELL

cut -d: -f1,6 /etc/passwd | grep /home

su - 계정
```

### ✔ 활성화

```
echo'if [ -f /etc/bash_completion ]; then . /etc/bash_completion; fi' >> ~/.bashrc
source ~/.bashrc
```

## 윈도우에 설치된 codex 찾기

```sql
which codex
which node
echo $PATH

> 
/mnt/c/Users/NICOM/AppData/Roaming/npm/codex
```

## locale 설치

```sql
sudo apt update
sudo apt install locales -y

sudo locale-gen ko_KR.UTF-8

sudo update-locale LANG=ko_KR.UTF-8 LC_ALL=ko_KR.UTF-8

source ~/.bashrc

locale
```