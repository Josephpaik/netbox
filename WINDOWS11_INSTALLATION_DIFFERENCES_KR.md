# NetBox Windows 11 설치 가이드 (macOS와의 차이점)

## 📌 개요

이 문서는 **Windows 11 24H2**에서 NetBox를 설치할 때 macOS 설치 가이드와 **달라지는 부분만** 설명합니다.

> **참고**: 기본 개념과 NetBox 기능 사용법은 `MACOS_INSTALLATION_TUTORIAL_KR.md`를 참고하세요.

---

## 🔄 주요 차이점 요약

| 항목 | macOS | Windows 11 |
|------|-------|------------|
| **패키지 관리자** | Homebrew | Chocolatey / winget |
| **쉘** | Bash / Zsh | PowerShell / CMD |
| **경로 구분자** | `/` | `\` |
| **가상환경 활성화** | `source venv/bin/activate` | `venv\Scripts\activate` |
| **서비스 관리** | `brew services` | Windows 서비스 / NSSM |
| **Redis** | Native 지원 | WSL 또는 Memurai 필요 |
| **PostgreSQL** | Native 지원 | Native 지원 (공식 인스톨러) |

---

## 1단계: 사전 준비 (차이점)

### 1.1 관리자 권한으로 PowerShell 실행

Windows에서는 관리자 권한이 필요합니다.

```powershell
# PowerShell을 관리자 권한으로 실행
# 시작 메뉴에서 "PowerShell" 검색 → 우클릭 → "관리자 권한으로 실행"
```

### 1.2 패키지 관리자 설치

#### 옵션 A: Chocolatey 사용 (권장)

```powershell
# PowerShell 관리자 모드에서 실행
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# 설치 확인
choco --version
```

#### 옵션 B: winget 사용 (Windows 11에 기본 포함)

```powershell
# winget 버전 확인 (Windows 11에 기본 설치되어 있음)
winget --version
```

### 1.3 Python 설치

#### Chocolatey 사용:
```powershell
choco install python311 -y
```

#### winget 사용:
```powershell
winget install Python.Python.3.11
```

#### 수동 설치:
1. https://www.python.org/downloads/ 접속
2. "Download Python 3.11.x" 클릭
3. 설치 시 **"Add Python to PATH"** 체크 ✅
4. "Install Now" 클릭

**설치 확인**:
```powershell
# PowerShell 재시작 후
python --version
# 출력: Python 3.11.x

# pip 확인
pip --version
```

---

## 2단계: PostgreSQL 설치 (Windows 방식)

### 2.1 PostgreSQL 15 설치

#### 옵션 A: Chocolatey
```powershell
choco install postgresql15 --params '/Password:postgres123' -y
```

#### 옵션 B: winget
```powershell
winget install PostgreSQL.PostgreSQL.15
```

#### 옵션 C: 공식 인스톨러 (권장)

1. https://www.postgresql.org/download/windows/ 접속
2. "Download the installer" 클릭
3. PostgreSQL 15.x 다운로드
4. 설치 마법사 실행:
   - **Password**: `postgres123` (기억해두세요!)
   - **Port**: `5432` (기본값)
   - **Locale**: `Korean, Korea` 또는 `Default`
5. "Stack Builder" 실행은 건너뛰기

### 2.2 PostgreSQL 서비스 시작

```powershell
# Windows 서비스에서 자동으로 시작됨
# 수동으로 시작하려면:
Start-Service postgresql-x64-15

# 서비스 상태 확인
Get-Service postgresql-x64-15
```

### 2.3 환경 변수 설정 (PATH 추가)

PostgreSQL 명령어를 사용하려면 PATH에 추가합니다.

```powershell
# 일반적인 PostgreSQL 설치 경로
$env:Path += ";C:\Program Files\PostgreSQL\15\bin"

# 영구적으로 추가 (시스템 환경 변수)
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\PostgreSQL\15\bin", "Machine")
```

또는 **GUI로 추가**:
1. `Win + R` → `sysdm.cpl` 입력
2. "고급" 탭 → "환경 변수"
3. "시스템 변수"에서 "Path" 선택 → "편집"
4. "새로 만들기" → `C:\Program Files\PostgreSQL\15\bin` 추가

### 2.4 데이터베이스 생성 (Windows 방식)

```powershell
# psql 실행 (비밀번호: postgres123)
psql -U postgres

# 또는 pgAdmin 4 사용 (PostgreSQL 설치 시 같이 설치됨)
```

**psql 프롬프트에서**:
```sql
-- NetBox 데이터베이스 생성
CREATE DATABASE netbox;

-- NetBox 사용자 생성
CREATE USER netbox WITH PASSWORD 'netbox123';

-- 권한 부여
ALTER DATABASE netbox OWNER TO netbox;
GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;

-- 종료
\q
```

**연결 테스트**:
```powershell
$env:PGPASSWORD="netbox123"
psql -U netbox -d netbox -h localhost
# 연결되면 \q로 종료
```

---

## 3단계: Redis 설치 (Windows는 복잡함!)

### ⚠️ Redis는 Windows를 공식 지원하지 않습니다!

세 가지 옵션이 있습니다:

### 옵션 A: Memurai 사용 (권장) ⭐

Memurai는 Redis의 Windows 호환 버전입니다.

```powershell
# Chocolatey로 설치
choco install memurai-developer -y

# 서비스 시작
Start-Service Memurai

# 연결 테스트
memurai-cli ping
# 출력: PONG
```

**설정 파일 위치**: `C:\Program Files\Memurai\memurai.conf`

### 옵션 B: WSL2에서 Redis 실행

Windows Subsystem for Linux를 사용하는 방법입니다.

```powershell
# 1. WSL2 설치
wsl --install

# 2. Ubuntu 설치
wsl --install -d Ubuntu

# 3. WSL Ubuntu 쉘에서 Redis 설치
wsl
sudo apt update
sudo apt install redis-server -y
sudo service redis-server start

# 4. Redis 테스트
redis-cli ping
# 출력: PONG
```

**주의**: WSL2를 사용하면 localhost 대신 WSL IP 주소를 사용해야 할 수 있습니다.

```powershell
# WSL IP 확인
wsl hostname -I
```

### 옵션 C: Docker 사용

```powershell
# Docker Desktop 설치 후
docker run -d -p 6379:6379 --name redis redis:latest

# 테스트 (redis-cli를 별도로 설치해야 함)
docker exec -it redis redis-cli ping
```

---

## 4단계: NetBox 다운로드 (경로 차이)

### 4.1 Git 설치

```powershell
# Chocolatey
choco install git -y

# winget
winget install Git.Git

# 또는 https://git-scm.com/download/win 에서 다운로드
```

### 4.2 NetBox 클론

```powershell
# 작업 디렉토리로 이동 (예: C:\Users\사용자명\Documents)
cd $env:USERPROFILE\Documents

# NetBox 클론
git clone https://github.com/netbox-community/netbox.git
cd netbox
```

**경로 예시**: `C:\Users\YourName\Documents\netbox`

---

## 5단계: Python 가상환경 (Windows 방식)

### 5.1 가상환경 생성

```powershell
# netbox 디렉토리에서
python -m venv venv
```

### 5.2 가상환경 활성화 (중요한 차이!)

#### PowerShell:
```powershell
# 실행 정책 확인 (처음 한 번만)
Get-ExecutionPolicy

# Restricted라면 변경 필요
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 가상환경 활성화
.\venv\Scripts\Activate.ps1

# 프롬프트가 (venv)로 시작하면 성공
```

#### CMD (명령 프롬프트):
```cmd
venv\Scripts\activate.bat
```

### 5.3 의존성 설치

```powershell
# pip 업그레이드
python -m pip install --upgrade pip

# NetBox 의존성 설치
pip install -r requirements.txt
```

**주의**: Windows에서 `psycopg` 설치 시 문제가 발생할 수 있습니다.

**문제 해결**:
```powershell
# Visual C++ Build Tools 설치 필요
# https://visualstudio.microsoft.com/visual-cpp-build-tools/

# 또는 Chocolatey로 설치
choco install visualstudio2022buildtools -y
choco install visualstudio2022-workload-vctools -y
```

---

## 6단계: NetBox 설정 (경로 표기 차이)

### 6.1 설정 파일 생성

```powershell
cd netbox

# PowerShell에서 복사
Copy-Item netbox\configuration_example.py netbox\configuration.py

# 또는 CMD에서
copy netbox\configuration_example.py netbox\configuration.py
```

### 6.2 SECRET_KEY 생성

```powershell
python generate_secret_key.py
# 출력된 키 복사
```

### 6.3 설정 파일 편집

```powershell
# 메모장으로 열기
notepad netbox\configuration.py

# 또는 VS Code
code netbox\configuration.py
```

**수정 내용** (macOS와 동일):

```python
# 1. ALLOWED_HOSTS
ALLOWED_HOSTS = ['localhost', '127.0.0.1', '::1']

# 2. DATABASE
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'netbox',
        'USER': 'netbox',
        'PASSWORD': 'netbox123',
        'HOST': 'localhost',
        'PORT': '',
        'CONN_MAX_AGE': 300,
    }
}

# 3. REDIS (Memurai 사용 시)
REDIS = {
    'tasks': {
        'HOST': 'localhost',
        'PORT': 6379,
        'PASSWORD': '',
        'DATABASE': 0,
        'SSL': False,
    },
    'caching': {
        'HOST': 'localhost',
        'PORT': 6379,
        'PASSWORD': '',
        'DATABASE': 1,
        'SSL': False,
    }
}

# WSL Redis 사용 시 HOST를 WSL IP로 변경
# 'HOST': '172.x.x.x',  # wsl hostname -I로 확인한 IP

# 4. SECRET_KEY
SECRET_KEY = '여기에_생성한_키_붙여넣기'

# 5. DEBUG
DEBUG = True

# 6. TIME_ZONE
TIME_ZONE = 'Asia/Seoul'
```

---

## 7단계: 데이터베이스 초기화 (명령어 동일)

```powershell
# 가상환경이 활성화된 상태에서

# 설정 검증
python manage.py check

# 마이그레이션
python manage.py migrate

# 슈퍼유저 생성
python manage.py createsuperuser
# Username: admin
# Email: admin@example.com
# Password: admin123

# 정적 파일 수집
python manage.py collectstatic --noinput
```

---

## 8단계: NetBox 실행 (Windows 방식)

### 8.1 개발 서버 실행

```powershell
# PowerShell 창 1 (NetBox 서버)
cd $env:USERPROFILE\Documents\netbox\netbox
.\venv\Scripts\Activate.ps1
python manage.py runserver 0.0.0.0:8000
```

### 8.2 백그라운드 워커 실행

```powershell
# PowerShell 창 2 (새 창 열기)
cd $env:USERPROFILE\Documents\netbox\netbox
.\..\venv\Scripts\Activate.ps1
python manage.py rqworker
```

### 8.3 접속

브라우저에서: `http://localhost:8000`

---

## 💡 Windows 전용 편의 스크립트

### start_netbox.bat 생성

```powershell
# netbox 루트 디렉토리에 파일 생성
@"
@echo off
cd /d %~dp0\netbox
call ..\venv\Scripts\activate.bat
echo NetBox 서버를 시작합니다...
echo 접속: http://localhost:8000
echo 종료: Ctrl+C
python manage.py runserver
pause
"@ | Out-File -FilePath start_netbox.bat -Encoding ASCII
```

### start_worker.bat 생성

```powershell
@"
@echo off
cd /d %~dp0\netbox
call ..\venv\Scripts\activate.bat
echo RQ 워커를 시작합니다...
echo 종료: Ctrl+C
python manage.py rqworker
pause
"@ | Out-File -FilePath start_worker.bat -Encoding ASCII
```

**사용법**:
- `start_netbox.bat` 더블클릭 → NetBox 서버 시작
- `start_worker.bat` 더블클릭 → RQ 워커 시작

---

## 🔧 Windows 전용 문제 해결

### 문제 1: psycopg 설치 오류

**증상**:
```
error: Microsoft Visual C++ 14.0 or greater is required
```

**해결책**:
```powershell
# Visual C++ Build Tools 설치
choco install visualstudio2022buildtools -y
choco install visualstudio2022-workload-vctools -y

# 또는 수동 다운로드
# https://visualstudio.microsoft.com/visual-cpp-build-tools/
```

### 문제 2: PowerShell 스크립트 실행 정책 오류

**증상**:
```
cannot be loaded because running scripts is disabled on this system
```

**해결책**:
```powershell
# 현재 사용자에 대해 실행 정책 변경
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 확인
Get-ExecutionPolicy
```

### 문제 3: Redis 연결 오류 (Memurai)

**증상**:
```
redis.exceptions.ConnectionError
```

**해결책**:
```powershell
# Memurai 서비스 상태 확인
Get-Service Memurai

# 서비스 시작
Start-Service Memurai

# 연결 테스트
memurai-cli ping

# 방화벽 확인 (필요 시)
New-NetFirewallRule -DisplayName "Memurai" -Direction Inbound -Protocol TCP -LocalPort 6379 -Action Allow
```

### 문제 4: PostgreSQL 포트 충돌

**증상**:
```
port 5432 is already in use
```

**해결책**:
```powershell
# 포트 사용 프로세스 확인
netstat -ano | findstr :5432

# PostgreSQL 서비스 재시작
Restart-Service postgresql-x64-15
```

### 문제 5: 한글 경로 문제

**증상**: 경로에 한글이 포함되면 오류 발생

**해결책**:
```powershell
# 영문 경로 사용 (예시)
# 나쁨: C:\Users\홍길동\Documents\netbox
# 좋음: C:\Users\gildong\Documents\netbox

# 또는 C:\netbox 처럼 루트에 생성
```

---

## 🚀 Windows 서비스로 등록 (선택사항)

개발 서버를 Windows 서비스로 등록하여 자동 시작할 수 있습니다.

### NSSM 사용 (Non-Sucking Service Manager)

```powershell
# NSSM 설치
choco install nssm -y

# NetBox 서비스 등록
$netboxPath = "$env:USERPROFILE\Documents\netbox\netbox"
$pythonPath = "$netboxPath\..\venv\Scripts\python.exe"

nssm install NetBox $pythonPath "$netboxPath\manage.py" runserver 0.0.0.0:8000
nssm set NetBox AppDirectory $netboxPath
nssm set NetBox DisplayName "NetBox Web Server"
nssm set NetBox Description "NetBox IPAM/DCIM Web Application"
nssm set NetBox Start SERVICE_AUTO_START

# RQ 워커 서비스 등록
nssm install NetBoxWorker $pythonPath "$netboxPath\manage.py" rqworker
nssm set NetBoxWorker AppDirectory $netboxPath
nssm set NetBoxWorker DisplayName "NetBox RQ Worker"
nssm set NetBoxWorker Start SERVICE_AUTO_START

# 서비스 시작
Start-Service NetBox
Start-Service NetBoxWorker

# 서비스 상태 확인
Get-Service NetBox*
```

**서비스 제거**:
```powershell
nssm remove NetBox confirm
nssm remove NetBoxWorker confirm
```

---

## 📊 성능 비교

| 항목 | Windows 11 | macOS |
|------|-----------|-------|
| 설치 복잡도 | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Redis 설치 | 복잡 (Memurai/WSL) | 간단 (brew) |
| PostgreSQL | 간단 (GUI 설치) | 간단 (brew) |
| 성능 | 양호 | 양호 |
| 안정성 | 양호 | 우수 |

---

## 🎯 Windows 전용 빠른 체크리스트

```powershell
# 1. 관리자 PowerShell 열기
# Win + X → "Windows PowerShell (관리자)"

# 2. Chocolatey 설치
Set-ExecutionPolicy Bypass -Scope Process -Force
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# 3. 필수 소프트웨어 설치
choco install python311 git visualstudio2022buildtools -y
choco install postgresql15 --params '/Password:postgres123' -y
choco install memurai-developer -y

# 4. 서비스 시작
Start-Service postgresql-x64-15
Start-Service Memurai

# 5. 환경 변수 추가
$env:Path += ";C:\Program Files\PostgreSQL\15\bin"

# 6. 데이터베이스 생성
$env:PGPASSWORD="postgres123"
psql -U postgres -c "CREATE DATABASE netbox;"
psql -U postgres -c "CREATE USER netbox WITH PASSWORD 'netbox123';"
psql -U postgres -c "ALTER DATABASE netbox OWNER TO netbox;"

# 7. NetBox 클론
cd $env:USERPROFILE\Documents
git clone https://github.com/netbox-community/netbox.git
cd netbox

# 8. 가상환경 설정
python -m venv venv
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
.\venv\Scripts\Activate.ps1

# 9. 의존성 설치
pip install --upgrade pip
pip install -r requirements.txt

# 10. 설정 파일
cd netbox
Copy-Item netbox\configuration_example.py netbox\configuration.py
python generate_secret_key.py
# SECRET_KEY를 configuration.py에 추가

# 11. 데이터베이스 초기화
python manage.py migrate
python manage.py createsuperuser
python manage.py collectstatic --noinput

# 12. 서버 시작
python manage.py runserver
```

---

## 🔗 추가 참고 자료

### Windows 전용 도구
- **Memurai**: https://www.memurai.com/
- **NSSM**: https://nssm.cc/
- **WSL2**: https://learn.microsoft.com/ko-kr/windows/wsl/install
- **pgAdmin**: https://www.pgadmin.org/

### PostgreSQL Windows 가이드
- https://www.postgresql.org/docs/15/install-windows.html

### Visual Studio Build Tools
- https://visualstudio.microsoft.com/visual-cpp-build-tools/

---

## 📝 요약: macOS vs Windows 명령어 비교

| 작업 | macOS | Windows (PowerShell) |
|------|-------|---------------------|
| **패키지 설치** | `brew install package` | `choco install package -y` |
| **Python 가상환경 생성** | `python3 -m venv venv` | `python -m venv venv` |
| **가상환경 활성화** | `source venv/bin/activate` | `.\venv\Scripts\Activate.ps1` |
| **서비스 시작** | `brew services start redis` | `Start-Service Memurai` |
| **서비스 상태** | `brew services list` | `Get-Service postgresql*` |
| **경로** | `/Users/name/netbox` | `C:\Users\name\netbox` |
| **경로 이동** | `cd ~/Documents` | `cd $env:USERPROFILE\Documents` |
| **환경 변수** | `export PATH=$PATH:/path` | `$env:Path += ";C:\path"` |
| **복사** | `cp file1 file2` | `Copy-Item file1 file2` |

---

## ✅ 최종 확인 사항

Windows 11에서 NetBox 설치 완료 후:

- [ ] PostgreSQL 서비스 실행 중
- [ ] Memurai (Redis) 서비스 실행 중
- [ ] Python 가상환경 활성화됨
- [ ] `http://localhost:8000` 접속 가능
- [ ] admin 계정으로 로그인 성공
- [ ] 대시보드 정상 표시

---

**이 가이드는 macOS 가이드의 보완 문서입니다.**
기본 개념과 NetBox 사용법은 `MACOS_INSTALLATION_TUTORIAL_KR.md`를 참고하세요.

**Windows 11 24H2에서 성공적인 설치를 기원합니다!** 🚀
