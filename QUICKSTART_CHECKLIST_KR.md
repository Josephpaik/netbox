# NetBox 설치 빠른 체크리스트 (macOS)

## 📋 설치 전 준비사항

- [ ] macOS 10.15 (Catalina) 이상
- [ ] Homebrew 설치됨
- [ ] 약 45분의 시간 확보
- [ ] 인터넷 연결 양호
- [ ] 디스크 공간 2GB 이상

---

## 🚀 빠른 설치 (복사 & 붙여넣기)

### 1단계: 필수 소프트웨어 설치

```bash
# Homebrew로 필수 패키지 설치
brew install python@3.11 postgresql@15 redis git pkg-config

# PostgreSQL 및 Redis 시작
brew services start postgresql@15
brew services start redis

# 설치 확인
python3 --version  # Python 3.11.x
psql --version     # PostgreSQL 15.x
redis-cli ping     # PONG 출력 확인
```

---

### 2단계: 데이터베이스 설정

```bash
# PostgreSQL에 접속
psql postgres

# 다음 SQL 명령어 실행:
```

```sql
CREATE DATABASE netbox;
CREATE USER netbox WITH PASSWORD 'netbox123';
ALTER DATABASE netbox OWNER TO netbox;
GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;
\q
```

```bash
# 데이터베이스 연결 테스트
psql -U netbox -d netbox -h localhost
# 비밀번호: netbox123
# 연결되면 \q로 종료
```

---

### 3단계: NetBox 다운로드 및 설치

```bash
# 작업 디렉토리로 이동
cd ~/Documents

# NetBox 클론
git clone https://github.com/netbox-community/netbox.git
cd netbox

# 가상환경 생성 및 활성화
python3 -m venv venv
source venv/bin/activate

# 의존성 설치 (5-10분 소요)
pip install --upgrade pip
pip install -r requirements.txt
```

---

### 4단계: NetBox 설정

```bash
# netbox 디렉토리로 이동
cd netbox

# 설정 파일 복사
cp netbox/configuration_example.py netbox/configuration.py

# SECRET_KEY 생성
python3 generate_secret_key.py
# 출력된 키를 복사해둡니다!
```

**설정 파일 편집**:

```bash
# 에디터로 설정 파일 열기
nano netbox/configuration.py
# 또는
code netbox/configuration.py
```

**변경할 내용**:

```python
# 1. ALLOWED_HOSTS (11번째 줄 근처)
ALLOWED_HOSTS = ['localhost', '127.0.0.1', '::1']

# 2. DATABASES는 이미 기본값 사용 (변경 필요시만)
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

# 3. SECRET_KEY (69번째 줄 근처)
SECRET_KEY = '여기에_생성한_키_붙여넣기'

# 4. DEBUG (119번째 줄 근처)
DEBUG = True

# 5. TIME_ZONE (250번째 줄 근처) - 선택사항
TIME_ZONE = 'Asia/Seoul'
```

**저장**: `Ctrl+X` → `Y` → `Enter` (nano) 또는 `Cmd+S` (VS Code)

---

### 5단계: 데이터베이스 초기화 및 실행

```bash
# 설정 검증
python3 manage.py check

# 마이그레이션 (2-3분 소요)
python3 manage.py migrate

# 슈퍼유저 생성
python3 manage.py createsuperuser
# Username: admin
# Email: admin@example.com
# Password: admin123
# Password (again): admin123

# 정적 파일 수집
python3 manage.py collectstatic --noinput

# 서버 시작!
python3 manage.py runserver
```

---

### 6단계: 백그라운드 워커 시작 (새 터미널)

**새 터미널 창을 열고**:

```bash
cd ~/Documents/netbox/netbox
source ../venv/bin/activate
python3 manage.py rqworker
```

---

### 7단계: 접속!

브라우저에서 접속:
```
http://localhost:8000
```

**로그인**:
- Username: `admin`
- Password: `admin123`

---

## ✅ 설치 확인 체크리스트

완료했다면 체크하세요:

- [ ] PostgreSQL이 실행 중 (`brew services list`)
- [ ] Redis가 실행 중 (`brew services list`)
- [ ] NetBox 개발 서버 실행 중 (포트 8000)
- [ ] RQ 워커 실행 중
- [ ] 브라우저에서 `http://localhost:8000` 접속 가능
- [ ] `admin` 계정으로 로그인 성공
- [ ] 대시보드가 정상적으로 표시됨

---

## 🎯 첫 번째 데이터 생성 (5분 실습)

### 빠른 테스트 시나리오

```
1. 사이트 생성
   Organization → Sites → Add
   - Name: Seoul DC1
   - Status: Active
   - Create 클릭

2. 제조사 생성
   Devices → Device Types → Manufacturers → Add
   - Name: Cisco
   - Create 클릭

3. 장비 역할 생성
   Devices → Device Roles → Add
   - Name: Core Router
   - Color: 빨강 선택
   - Create 클릭

4. 장비 타입 생성
   Devices → Device Types → Add
   - Manufacturer: Cisco
   - Model: ASR 1000
   - U Height: 2
   - Create 클릭

5. 장비 생성
   Devices → Devices → Add
   - Name: seoul-core-rt01
   - Device Role: Core Router
   - Device Type: Cisco ASR 1000
   - Site: Seoul DC1
   - Status: Active
   - Create 클릭

6. IP 프리픽스 생성
   IPAM → Prefixes → Add
   - Prefix: 10.0.0.0/24
   - Status: Active
   - Site: Seoul DC1
   - Create 클릭

7. IP 주소 생성
   IPAM → IP Addresses → Add
   - IP Address: 10.0.0.1/24
   - Status: Active
   - Create 클릭
```

**축하합니다!** 🎉 NetBox의 기본 기능을 체험했습니다!

---

## 🔧 일일 사용 명령어

### 서버 시작

```bash
# 터미널 1: NetBox 서버
cd ~/Documents/netbox/netbox
source ../venv/bin/activate
python3 manage.py runserver

# 터미널 2: RQ 워커
cd ~/Documents/netbox/netbox
source ../venv/bin/activate
python3 manage.py rqworker
```

### 서버 종료

```bash
# 각 터미널에서:
Ctrl + C

# 가상환경 비활성화:
deactivate
```

---

## 🆘 빠른 문제 해결

### PostgreSQL 연결 오류
```bash
brew services restart postgresql@15
psql -U netbox -d netbox -h localhost
```

### Redis 연결 오류
```bash
brew services restart redis
redis-cli ping
```

### 포트 이미 사용 중
```bash
# 포트 8000을 사용 중인 프로세스 종료
lsof -ti:8000 | xargs kill -9

# 또는 다른 포트 사용
python3 manage.py runserver 8080
```

### 정적 파일 안보임
```bash
cd ~/Documents/netbox/netbox
python3 manage.py collectstatic --clear --noinput
```

### 모든 서비스 재시작
```bash
brew services restart postgresql@15
brew services restart redis
# 그런 다음 NetBox 서버 재시작
```

---

## 📚 다음 단계

설치를 완료했다면:

1. **상세 가이드 읽기**: `MACOS_INSTALLATION_TUTORIAL_KR.md`
2. **API 테스트**: http://localhost:8000/api/docs/
3. **GraphQL 테스트**: http://localhost:8000/graphql/
4. **공식 문서**: https://docs.netbox.dev
5. **데모 사이트 탐색**: https://demo.netbox.dev

---

## 💡 유용한 팁

### API 토큰 빠른 생성
```
우측 상단 아이콘 → API Tokens → Add Token → Write enabled 체크 → Create
```

### 데이터 백업
```bash
# PostgreSQL 백업
pg_dump -U netbox -h localhost netbox > netbox_backup.sql

# 복원
psql -U netbox -h localhost netbox < netbox_backup.sql
```

### 개발 서버를 백그라운드에서 실행
```bash
# 백그라운드 실행
nohup python3 manage.py runserver > netbox.log 2>&1 &

# 프로세스 확인
ps aux | grep runserver

# 종료
pkill -f runserver
```

---

## 🎓 학습 경로

1. **초급**: 사이트, 장비, IP 주소 관리
2. **중급**: 케이블 연결, VLAN, 태그
3. **고급**: API 통합, 커스텀 필드, 웹훅
4. **전문가**: 플러그인 개발, 대규모 배포

---

**설치 소요 시간**: 30-45분
**학습 곡선**: 중간
**난이도**: ⭐⭐⭐☆☆

**행운을 빕니다!** 🚀
