# NetBox macOS 설치 가이드 요약

## 🚀 빠른 시작

NetBox를 macOS에 설치하는 세 가지 방법:

### 방법 1: 자동 설치 스크립트 (추천) ⭐

가장 쉽고 빠른 방법입니다!

```bash
# NetBox 디렉토리로 이동
cd /path/to/netbox

# 설치 스크립트 실행
./install_netbox_macos.sh
```

스크립트는 자동으로:
- PostgreSQL 15 설치
- Redis 설치
- Python 가상환경 생성
- 모든 의존성 설치
- 데이터베이스 설정
- 슈퍼유저 생성 (admin/admin123)
- 정적 파일 수집

**설치 완료 후**:

```bash
# 터미널 1: NetBox 서버
./start_netbox.sh

# 터미널 2: RQ 워커 (새 터미널에서)
./start_worker.sh
```

브라우저에서 `http://localhost:8000` 접속!

---

### 방법 2: 단계별 수동 설치

체크리스트를 따라 수동으로 설치하고 싶다면:

```bash
# 빠른 체크리스트 참고
cat QUICKSTART_CHECKLIST_KR.md
```

또는 파일을 열어서 단계별로 따라하세요.

---

### 방법 3: 상세 가이드

모든 기능을 자세히 이해하면서 설치하고 싶다면:

```bash
# 상세 튜토리얼 참고
cat MACOS_INSTALLATION_TUTORIAL_KR.md
```

이 가이드는 다음을 포함합니다:
- 상세한 설치 과정 설명
- 20가지 주요 기능 사용법
- 문제 해결 가이드
- API 사용 예제

---

## 📁 문서 파일 목록

이 저장소에는 다음 한국어 문서가 포함되어 있습니다:

| 파일명 | 설명 | 난이도 | 소요 시간 |
|--------|------|--------|-----------|
| `README_MACOS_KR.md` | 이 파일 (빠른 참조) | ⭐ | 5분 |
| `QUICKSTART_CHECKLIST_KR.md` | 빠른 설치 체크리스트 | ⭐⭐ | 30분 |
| `MACOS_INSTALLATION_TUTORIAL_KR.md` | 완전한 설치 및 사용 가이드 | ⭐⭐⭐ | 60분+ |
| `install_netbox_macos.sh` | 자동 설치 스크립트 | ⭐ | 15분 |

---

## 🎯 설치 후 할 일

1. **로그인**
   - URL: http://localhost:8000
   - Username: `admin`
   - Password: `admin123`

2. **첫 번째 사이트 생성**
   - Organization → Sites → Add
   - 이름: Seoul DC1
   - Status: Active

3. **API 토큰 생성**
   - 우측 상단 → API Tokens → Add Token
   - Write enabled 체크

4. **API 문서 확인**
   - http://localhost:8000/api/docs/

5. **GraphQL 인터페이스**
   - http://localhost:8000/graphql/

---

## 💡 주요 명령어

### 서버 시작

```bash
# 방법 1: 편의 스크립트 사용
./start_netbox.sh          # 터미널 1
./start_worker.sh          # 터미널 2

# 방법 2: 수동 실행
cd netbox
source ../venv/bin/activate
python3 manage.py runserver              # 터미널 1
python3 manage.py rqworker              # 터미널 2
```

### 서버 종료

```bash
# 각 터미널에서
Ctrl + C

# 가상환경 비활성화
deactivate
```

### 서비스 상태 확인

```bash
# PostgreSQL 및 Redis 상태
brew services list

# PostgreSQL 재시작
brew services restart postgresql@15

# Redis 재시작
brew services restart redis
```

---

## 🆘 자주 묻는 질문 (FAQ)

### Q: 서버가 시작되지 않아요

```bash
# 1. PostgreSQL 확인
brew services start postgresql@15
psql -U netbox -d netbox -h localhost

# 2. Redis 확인
brew services start redis
redis-cli ping

# 3. 설정 파일 확인
cd netbox
python3 manage.py check
```

### Q: 화면에 CSS가 적용되지 않아요

```bash
cd netbox
python3 manage.py collectstatic --clear --noinput
```

### Q: 포트 8000이 이미 사용 중이에요

```bash
# 포트를 사용 중인 프로세스 종료
lsof -ti:8000 | xargs kill -9

# 또는 다른 포트 사용
python3 manage.py runserver 8080
```

### Q: 비밀번호를 잊어버렸어요

```bash
cd netbox
source ../venv/bin/activate
python3 manage.py changepassword admin
```

### Q: 데이터베이스를 초기화하고 싶어요

```bash
cd netbox
source ../venv/bin/activate

# 주의: 모든 데이터가 삭제됩니다!
python3 manage.py flush
python3 manage.py migrate
python3 manage.py createsuperuser
```

---

## 📚 추가 리소스

### 공식 리소스
- **공식 문서**: https://docs.netbox.dev
- **GitHub**: https://github.com/netbox-community/netbox
- **공식 데모**: https://demo.netbox.dev
- **Discussion Forum**: https://github.com/netbox-community/netbox/discussions
- **Slack**: https://netdev.chat/

### 학습 자료
- **REST API 문서**: http://localhost:8000/api/docs/
- **GraphQL 문서**: http://localhost:8000/graphql/
- **플러그인 개발**: https://docs.netbox.dev/en/stable/plugins/

---

## 🔄 업데이트

NetBox를 최신 버전으로 업데이트하려면:

```bash
cd /path/to/netbox

# Git으로 최신 코드 가져오기
git fetch --all
git checkout master  # 또는 원하는 버전
git pull

# 가상환경 활성화
source venv/bin/activate

# 의존성 업데이트
pip install --upgrade -r requirements.txt

# 데이터베이스 마이그레이션
cd netbox
python3 manage.py migrate

# 정적 파일 재수집
python3 manage.py collectstatic --clear --noinput

# 서버 재시작
```

---

## 🛡️ 보안 주의사항

### 개발/테스트 환경

현재 설정은 **개발 및 테스트 전용**입니다:

- ✅ `DEBUG = True` (디버그 모드 활성화)
- ✅ 기본 비밀번호 사용 (`admin123`)
- ✅ 개발 서버 사용 (`runserver`)

### 운영 환경 배포 시

운영 환경에서는 다음을 변경하세요:

1. **DEBUG 모드 비활성화**
   ```python
   DEBUG = False
   ```

2. **강력한 비밀번호 사용**
   ```bash
   python3 manage.py changepassword admin
   ```

3. **새 SECRET_KEY 생성**
   ```bash
   python3 generate_secret_key.py
   ```

4. **Gunicorn + Nginx 사용**
   - 공식 문서 참고: https://docs.netbox.dev/en/stable/installation/

5. **HTTPS 설정**
   - SSL 인증서 설치
   - Nginx 리버스 프록시 설정

---

## 🎓 학습 경로

### 1단계: 기초 (1-2시간)
- [x] NetBox 설치
- [ ] 사이트 생성
- [ ] 장비 추가
- [ ] IP 주소 할당

### 2단계: 중급 (2-4시간)
- [ ] 랙 관리
- [ ] 케이블 연결
- [ ] VLAN 설정
- [ ] 태그 사용

### 3단계: 고급 (4-8시간)
- [ ] REST API 사용
- [ ] GraphQL 쿼리
- [ ] 커스텀 필드 생성
- [ ] 웹훅 설정

### 4단계: 전문가 (8시간+)
- [ ] 대량 데이터 가져오기
- [ ] 플러그인 개발
- [ ] 운영 환경 배포
- [ ] 외부 시스템 통합

---

## 📊 시스템 요구사항

### 최소 사양
- **OS**: macOS 10.15+
- **CPU**: 2 cores
- **RAM**: 4GB
- **Disk**: 10GB

### 권장 사양
- **OS**: macOS 12+
- **CPU**: 4 cores
- **RAM**: 8GB
- **Disk**: 20GB SSD

---

## 🤝 기여하기

NetBox는 오픈소스 프로젝트입니다!

- **버그 리포트**: https://github.com/netbox-community/netbox/issues
- **기능 제안**: https://github.com/netbox-community/netbox/discussions
- **코드 기여**: https://github.com/netbox-community/netbox/pulls

---

## 📝 라이선스

NetBox는 **Apache 2.0 라이선스** 하에 배포됩니다.

---

## 👨‍💻 도움말

문제가 발생했나요?

1. **문서 확인**: `MACOS_INSTALLATION_TUTORIAL_KR.md`의 "문제 해결" 섹션
2. **로그 확인**: NetBox 서버 터미널의 에러 메시지
3. **커뮤니티 질문**: https://github.com/netbox-community/netbox/discussions
4. **Slack**: https://netdev.chat/

---

**설치에 성공하셨나요?** 🎉

이제 `MACOS_INSTALLATION_TUTORIAL_KR.md`를 열어 20가지 주요 기능을 체험해보세요!

```bash
# 튜토리얼 열기
cat MACOS_INSTALLATION_TUTORIAL_KR.md

# 또는 VS Code로 열기
code MACOS_INSTALLATION_TUTORIAL_KR.md
```

**Happy NetBox-ing!** 🚀
