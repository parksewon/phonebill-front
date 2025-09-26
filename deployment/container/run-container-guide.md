# 프론트엔드 컨테이너 실행 가이드

## 실행 정보
- **시스템명**: `tripgen`
- **ACR명**: `acrdigitalgarage01`
- **서비스명**: `phonebill-front`
- **VM 정보**:
  - KEY파일: `~/home/bastion-P82265782`
  - USERID: `azureuser`
  - IP: `20.196.95.67`

## 1. VM 접속 방법

### 1.1 터미널 실행
- **Linux/Mac**: 기본 터미널 실행
- **Windows**: Windows Terminal 실행

### 1.2 Private Key 파일 권한 설정 (최초 한 번)
```bash
chmod 400 ~/home/bastion-P82265782
```

### 1.3 VM 접속
```bash
ssh -i ~/home/bastion-P82265782 azureuser@20.196.95.67
```

## 2. Git Repository 클론

### 2.1 작업 디렉토리 생성 및 이동
```bash
mkdir -p ~/home/workspace
cd ~/home/workspace
```

### 2.2 소스 코드 클론
```bash
git clone https://github.com/cna-bootcamp/phonebill-front.git
```

### 2.3 프로젝트 디렉토리로 이동
```bash
cd phonebill-front
```

## 3. 컨테이너 이미지 생성

### 3.1 이미지 빌드
`deployment/container/build-image.md` 파일의 가이드에 따라 컨테이너 이미지를 생성합니다.

주요 명령어:
```bash
# 의존성 설치
npm install

# Docker 이미지 빌드
export DOCKER_FILE=deployment/container/Dockerfile-frontend

docker build \
  --platform linux/amd64 \
  --build-arg PROJECT_FOLDER="." \
  --build-arg BUILD_FOLDER="deployment/container" \
  --build-arg EXPORT_PORT="8080" \
  -f ${DOCKER_FILE} \
  -t phonebill-front:latest .
```

## 4. ACR 연동

### 4.1 ACR 인증 정보 확인
```bash
az acr credential show --name acrdigitalgarage01
```

예시 출력:
```json
{
  "passwords": [
    {
      "name": "password",
      "value": "{암호}"
    },
    {
      "name": "password2",
      "value": "{암호2}"
    }
  ],
  "username": "acrdigitalgarage01"
}
```

### 4.2 ACR 로그인
```bash
docker login acrdigitalgarage01.azurecr.io -u {ID} -p {암호}
```

### 4.3 이미지 태깅 및 푸시
```bash
# 이미지 태깅
docker tag phonebill-front:latest acrdigitalgarage01.azurecr.io/tripgen/phonebill-front:latest

# 이미지 푸시
docker push acrdigitalgarage01.azurecr.io/tripgen/phonebill-front:latest
```

## 5. 런타임 환경 설정

### 5.1 런타임 환경 파일 생성
VM IP(20.196.95.67)에 맞춰 `~/phonebill-front/public/runtime-env.js` 파일을 생성합니다:

```javascript
// 런타임 환경 설정
window.__runtime_config__ = {
  // API 서버 설정
  USER_HOST: 'http://20.196.95.67:8080',
  BILL_HOST: 'http://20.196.95.67:8080',
  PRODUCT_HOST: 'http://20.196.95.67:8080',
  KOS_MOCK_HOST: 'http://20.196.95.67:8080',
  API_GROUP: '/api/v1',

  // 환경 설정
  NODE_ENV: 'production',

  // 기타 설정
  APP_NAME: '통신요금 관리 서비스',
  VERSION: '1.0.0'
};
```

파일 생성 명령:
```bash
# 디렉토리 생성
mkdir -p ~/phonebill-front/public

# 환경 파일 생성
cat > ~/phonebill-front/public/runtime-env.js << 'EOF'
// 런타임 환경 설정
window.__runtime_config__ = {
  // API 서버 설정
  USER_HOST: 'http://20.196.95.67:8080',
  BILL_HOST: 'http://20.196.95.67:8080',
  PRODUCT_HOST: 'http://20.196.95.67:8080',
  KOS_MOCK_HOST: 'http://20.196.95.67:8080',
  API_GROUP: '/api/v1',

  // 환경 설정
  NODE_ENV: 'production',

  // 기타 설정
  APP_NAME: '통신요금 관리 서비스',
  VERSION: '1.0.0'
};
EOF
```

## 6. 컨테이너 실행

### 6.1 컨테이너 실행 명령
```bash
SERVER_PORT=3000

docker run -d --name phonebill-front --rm -p ${SERVER_PORT}:8080 \
-v ~/phonebill-front/public/runtime-env.js:/usr/share/nginx/html/runtime-env.js \
acrdigitalgarage01.azurecr.io/tripgen/phonebill-front:latest
```

### 6.2 실행 확인
```bash
# 컨테이너 실행 상태 확인
docker ps | grep phonebill-front

# 서비스 접속 확인
curl http://20.196.95.67:3000/health
```

## 7. 서비스 접속
- **프론트엔드 서비스**: http://20.196.95.67:3000
- **Health Check**: http://20.196.95.67:3000/health

## 8. 재배포 방법

### 8.1 로컬에서 소스 수정 후 푸시
```bash
# 로컬 개발 환경에서
git add .
git commit -m "수정 사항 설명"
git push origin main
```

### 8.2 VM 접속 및 소스 업데이트
```bash
# VM 접속
ssh -i ~/home/bastion-P82265782 azureuser@20.196.95.67

# 작업 디렉토리로 이동
cd ~/home/workspace/phonebill-front

# 최신 소스 가져오기
git pull
```

### 8.3 컨테이너 이미지 재생성
`deployment/container/build-image.md` 파일의 가이드에 따라 이미지를 다시 빌드합니다.

### 8.4 이미지 재푸시
```bash
docker tag phonebill-front:latest acrdigitalgarage01.azurecr.io/tripgen/phonebill-front:latest
docker push acrdigitalgarage01.azurecr.io/tripgen/phonebill-front:latest
```

### 8.5 기존 컨테이너 중지 및 이미지 삭제
```bash
# 컨테이너 중지
docker stop phonebill-front

# 기존 이미지 삭제
docker rmi acrdigitalgarage01.azurecr.io/tripgen/phonebill-front:latest
```

### 8.6 컨테이너 재실행
```bash
SERVER_PORT=3000

docker run -d --name phonebill-front --rm -p ${SERVER_PORT}:8080 \
-v ~/phonebill-front/public/runtime-env.js:/usr/share/nginx/html/runtime-env.js \
acrdigitalgarage01.azurecr.io/tripgen/phonebill-front:latest
```

## 트러블슈팅

### Docker 관련 문제
```bash
# Docker 서비스 상태 확인
sudo systemctl status docker

# Docker 서비스 시작
sudo systemctl start docker
```

### 포트 충돌 문제
```bash
# 포트 사용 확인
sudo netstat -tlnp | grep :3000

# 프로세스 종료 (필요시)
sudo kill -9 {PID}
```

### 로그 확인
```bash
# 컨테이너 로그 확인
docker logs phonebill-front

# 실시간 로그 확인
docker logs -f phonebill-front
```