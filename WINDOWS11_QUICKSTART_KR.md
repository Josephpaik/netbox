# NetBox Windows 11 빠른 설치 가이드

## 🚀 한 눈에 보는 설치 순서

```
1. Chocolatey 설치 (5분)
   ↓
2. PostgreSQL + Memurai + Python 설치 (10분)
   ↓
3. 데이터베이스 생성 (2분)
   ↓
4. NetBox 다운로드 및 설정 (10분)
   ↓
5. 실행! (1분)
   ↓
총 소요 시간: 약 30분
```

---

## ⚡ 초고속 설치 (복사 & 붙여넣기)

### 사전 확인
- [ ] Windows 11 24H2 이상
- [ ] 관리자 권한 있음
- [ ] 인터넷 연결 양호

---

## 1️⃣ PowerShell 관리자 모드 실행

```
Win + X → "Windows PowerShell (관리자)" 클릭
```

---

## 2️⃣ Chocolatey 설치

```powershell
# 한 번에 복사해서 실행
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# 설치 확인
choco --version
```

---

## 3️⃣ 필수 소프트웨어 일괄 설치

```powershell
# 한 번에 모두 설치 (10-15분 소요)
choco install python311 git postgresql15 memurai-developer visualstudio2022buildtools visualstudio2022-workload-vctools -y --params="'/Password:postgres123'"

# PowerShell 재시작 (중요!)
exit
# 다시 관리자 모드로 PowerShell 실행
```

---

## 4️⃣ 서비스 시작 및 확인

```powershell
# PostgreSQL 서비스 시작
Start-Service postgresql-x64-15

# Memurai (Redis) 서비스 시작
Start-Service Memurai

# 서비스 상태 확인
Get-Service postgresql-x64-15, Memurai
# Status가 "Running"이면 성공!

# PostgreSQL PATH 추가
$env:Path += ";C:\Program Files\PostgreSQL\15\bin"
```

---

## 5️⃣ 데이터베이스 생성

```powershell
# 환경 변수 설정
$env:PGPASSWORD="postgres123"

# 데이터베이스 및 사용자 생성 (한 번에 실행)
psql -U postgres -c "CREATE DATABASE netbox;"
psql -U postgres -c "CREATE USER netbox WITH PASSWORD 'netbox123';"
psql -U postgres -c "ALTER DATABASE netbox OWNER TO netbox;"
psql -U postgres -c "GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;"

# 연결 테스트
$env:PGPASSWORD="netbox123"
psql -U netbox -d netbox -h localhost -c "\q"
# 에러 없이 종료되면 성공!
```

---

## 6️⃣ NetBox 다운로드

```powershell
# 문서 폴더로 이동
cd $env:USERPROFILE\Documents

# NetBox 클론
git clone https://github.com/netbox-community/netbox.git
cd netbox
```

---

## 7️⃣ Python 가상환경 설정

```powershell
# 가상환경 생성
python -m venv venv

# 실행 정책 변경 (처음 한 번만)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 가상환경 활성화
.\venv\Scripts\Activate.ps1
# 프롬프트가 (venv)로 시작하면 성공!

# pip 업그레이드
python -m pip install --upgrade pip

# 의존성 설치 (5-10분 소요)
pip install -r requirements.txt
```

---

## 8️⃣ NetBox 설정

```powershell
# netbox 디렉토리로 이동
cd netbox

# 설정 파일 복사
Copy-Item netbox\configuration_example.py netbox\configuration.py

# SECRET_KEY 생성
python generate_secret_key.py
# 출력된 키를 복사! (Ctrl+C로 복사)
```

**메모장으로 설정 파일 열기**:

```powershell
notepad netbox\configuration.py
```

**또는 VS Code**:

```powershell
code netbox\configuration.py
```

**다음 내용 수정** (Ctrl+F로 찾기):

```python
# 1. ALLOWED_HOSTS (약 11번째 줄)
ALLOWED_HOSTS = ['localhost', '127.0.0.1', '::1']

# 2. DATABASES - 비밀번호만 변경 (약 19번째 줄)
        'USER': 'netbox',               # 추가
        'PASSWORD': 'netbox123',        # 변경

# 3. SECRET_KEY (약 69번째 줄)
SECRET_KEY = '여기에_생성한_키_붙여넣기'

# 4. DEBUG (약 119번째 줄)
DEBUG = True

# 5. TIME_ZONE (약 250번째 줄)
TIME_ZONE = 'Asia/Seoul'
```

**저장**: `Ctrl+S` 후 메모장 닫기

---

## 9️⃣ 데이터베이스 초기화

```powershell
# 설정 검증
python manage.py check
# "System check identified no issues" 나오면 성공!

# 마이그레이션 (2-3분)
python manage.py migrate

# 슈퍼유저 생성
python manage.py createsuperuser
# Username: admin
# Email: admin@example.com
# Password: admin123
# Password (again): admin123

# 정적 파일 수집
python manage.py collectstatic --noinput
```

---

## 🔟 NetBox 실행!

### PowerShell 창 1 (현재 창)

```powershell
# NetBox 서버 시작
python manage.py runserver 0.0.0.0:8000

# 성공 메시지 확인:
# "Starting development server at http://0.0.0.0:8000/"
```

### PowerShell 창 2 (새 창 열기)

```
Win + X → "Windows PowerShell (관리자)" 클릭
```

```powershell
# NetBox 디렉토리로 이동
cd $env:USERPROFILE\Documents\netbox\netbox

# 가상환경 활성화
.\..\venv\Scripts\Activate.ps1

# RQ 워커 시작
python manage.py rqworker

# "Listening on default..." 메시지 확인
```

---

## 🌐 브라우저에서 접속

```
http://localhost:8000
```

**로그인**:
- Username: `admin`
- Password: `admin123`

**성공!** 🎉

---

## 💾 편의 스크립트 생성 (선택사항)

다음에 쉽게 시작할 수 있도록 배치 파일을 생성합니다.

### start_netbox.bat

```powershell
# NetBox 루트 디렉토리로 이동
cd $env:USERPROFILE\Documents\netbox

# start_netbox.bat 파일 생성
@"
@echo off
echo ======================================
echo   NetBox 서버 시작
echo ======================================
cd /d %~dp0\netbox
call ..\venv\Scripts\activate.bat
echo.
echo 접속 주소: http://localhost:8000
echo 종료: Ctrl+C
echo.
python manage.py runserver
pause
"@ | Out-File -FilePath start_netbox.bat -Encoding ASCII
```

### start_worker.bat

```powershell
@"
@echo off
echo ======================================
echo   NetBox RQ 워커 시작
echo ======================================
cd /d %~dp0\netbox
call ..\venv\Scripts\activate.bat
echo.
echo 종료: Ctrl+C
echo.
python manage.py rqworker
pause
"@ | Out-File -FilePath start_worker.bat -Encoding ASCII
```

**사용법**:
1. 탐색기에서 `start_netbox.bat` 더블클릭 → 서버 시작
2. `start_worker.bat` 더블클릭 → 워커 시작

---

## 🔄 일상적인 시작/종료

### 시작

#### 방법 1: 배치 파일 사용
```
1. start_netbox.bat 더블클릭
2. start_worker.bat 더블클릭
3. 브라우저에서 http://localhost:8000 접속
```

#### 방법 2: PowerShell 사용
```powershell
# 창 1
cd $env:USERPROFILE\Documents\netbox\netbox
.\venv\Scripts\Activate.ps1
python manage.py runserver

# 창 2
cd $env:USERPROFILE\Documents\netbox\netbox
.\..\venv\Scripts\Activate.ps1
python manage.py rqworker
```

### 종료

```
각 PowerShell 창에서:
Ctrl + C
```

---

## 🆘 문제 해결 체크리스트

### ❌ 서비스가 시작되지 않음

```powershell
# PostgreSQL
Get-Service postgresql-x64-15
Start-Service postgresql-x64-15

# Memurai
Get-Service Memurai
Start-Service Memurai
```

### ❌ 가상환경 활성화 오류

```powershell
# 실행 정책 변경
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### ❌ psycopg 설치 오류

```powershell
# Visual C++ Build Tools 재설치
choco install visualstudio2022buildtools -y --force
choco install visualstudio2022-workload-vctools -y --force

# PowerShell 재시작 후 다시 시도
pip install -r requirements.txt
```

### ❌ PostgreSQL 연결 오류

```powershell
# 연결 테스트
$env:PGPASSWORD="netbox123"
psql -U netbox -d netbox -h localhost

# PATH 확인
$env:Path -split ';' | Select-String postgres
```

### ❌ 포트 8000 이미 사용 중

```powershell
# 포트 사용 중인 프로세스 확인
netstat -ano | findstr :8000

# 프로세스 종료 (PID 확인 후)
taskkill /PID [프로세스ID] /F

# 또는 다른 포트 사용
python manage.py runserver 8080
```

### ❌ 화면에 CSS가 안 나옴

```powershell
cd $env:USERPROFILE\Documents\netbox\netbox
.\..\venv\Scripts\Activate.ps1
python manage.py collectstatic --clear --noinput
```

---

## 📋 설치 완료 체크리스트

- [ ] Chocolatey 설치됨 (`choco --version`)
- [ ] Python 3.11 설치됨 (`python --version`)
- [ ] PostgreSQL 실행 중 (`Get-Service postgresql-x64-15`)
- [ ] Memurai 실행 중 (`Get-Service Memurai`)
- [ ] Git 설치됨 (`git --version`)
- [ ] NetBox 클론 완료
- [ ] 가상환경 생성 및 활성화
- [ ] 의존성 설치 완료
- [ ] 데이터베이스 마이그레이션 완료
- [ ] 슈퍼유저 생성 완료
- [ ] 정적 파일 수집 완료
- [ ] http://localhost:8000 접속 성공
- [ ] admin 로그인 성공

---

## 🎯 다음 단계

설치가 완료되었다면:

1. ✅ **첫 번째 사이트 생성**
   - Organization → Sites → Add
   - Name: Seoul DC1

2. ✅ **API 토큰 생성**
   - 우측 상단 사용자 아이콘 → API Tokens → Add

3. ✅ **기능 탐색**
   - `MACOS_INSTALLATION_TUTORIAL_KR.md`의 "주요 기능 사용 가이드" 참고
   - 20가지 기능을 하나씩 따라해보세요!

4. ✅ **API 문서 확인**
   - http://localhost:8000/api/docs/

5. ✅ **GraphQL 인터페이스**
   - http://localhost:8000/graphql/

---

## 📊 설치 시간 예상

| 단계 | 소요 시간 |
|------|-----------|
| Chocolatey 설치 | 3분 |
| 소프트웨어 설치 | 10-15분 |
| 데이터베이스 설정 | 2분 |
| NetBox 클론 | 2분 |
| Python 의존성 설치 | 5-10분 |
| 설정 및 초기화 | 5분 |
| **총계** | **30-40분** |

---

## 🔗 추가 문서

| 문서 | 설명 |
|------|------|
| `WINDOWS11_INSTALLATION_DIFFERENCES_KR.md` | Windows와 macOS의 상세 차이점 |
| `MACOS_INSTALLATION_TUTORIAL_KR.md` | 전체 기능 사용 가이드 (20개 기능) |
| `QUICKSTART_CHECKLIST_KR.md` | macOS 빠른 설치 (참고용) |

---

## 💡 프로 팁

### 1. 서비스 자동 시작 설정

```powershell
# PostgreSQL과 Memurai는 이미 자동 시작으로 설정됨
# 확인:
Get-Service postgresql-x64-15 | Select-Object Name, StartType
Get-Service Memurai | Select-Object Name, StartType

# StartType이 "Automatic"이면 OK
```

### 2. Windows 시작 시 NetBox 자동 실행

**작업 스케줄러** 사용:

```powershell
# 작업 스케줄러 열기
taskschd.msc

# 기본 작업 만들기:
# - 이름: NetBox Server
# - 트리거: 로그온 시
# - 작업: 프로그램 시작
# - 프로그램: C:\Users\[사용자명]\Documents\netbox\start_netbox.bat
```

### 3. 네트워크에서 접속 허용

```powershell
# 방화벽 규칙 추가
New-NetFirewallRule -DisplayName "NetBox" -Direction Inbound -Protocol TCP -LocalPort 8000 -Action Allow

# 이제 같은 네트워크의 다른 PC에서:
# http://[내PC의IP주소]:8000
```

### 4. 성능 향상 팁

```powershell
# PostgreSQL 설정 최적화
# C:\Program Files\PostgreSQL\15\data\postgresql.conf 편집

shared_buffers = 256MB          # RAM의 25%
effective_cache_size = 1GB      # RAM의 50%
work_mem = 16MB
maintenance_work_mem = 128MB
```

---

## 🎓 학습 리소스

### 공식 문서
- https://docs.netbox.dev
- https://demo.netbox.dev (데모 사이트)

### 커뮤니티
- https://github.com/netbox-community/netbox/discussions
- https://netdev.chat/ (Slack)

### Windows 관련
- Memurai: https://www.memurai.com/
- WSL2: https://learn.microsoft.com/ko-kr/windows/wsl/

---

## 🎉 축하합니다!

Windows 11에서 NetBox 설치를 완료했습니다!

이제 `MACOS_INSTALLATION_TUTORIAL_KR.md`를 열어서 20가지 주요 기능을 배워보세요!

```powershell
# VS Code로 열기
code $env:USERPROFILE\Documents\netbox\MACOS_INSTALLATION_TUTORIAL_KR.md

# 또는 메모장
notepad $env:USERPROFILE\Documents\netbox\MACOS_INSTALLATION_TUTORIAL_KR.md
```

**Happy NetBox-ing on Windows!** 🚀
