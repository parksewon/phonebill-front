# 프론트엔드 컨테이너 이미지 작성 결과

## 작업 개요
- **서비스명**: `phonebill-front` (package.json name 필드에서 확인)
- **작업일시**: 2025-09-26
- **빌드 플랫폼**: Linux/amd64

## 수행한 작업

### 1. 서비스명 확인
```bash
# package.json에서 서비스명 확인
cat package.json | grep '"name"'
# 결과: "name": "phonebill-front"
```

### 2. 의존성 설치 및 일치성 확인
```bash
npm install
```
- package.json과 package-lock.json 일치 확인 완료
- 394개 패키지 설치 완료

### 3. 컨테이너 관련 파일 생성

#### 3.1 nginx.conf 생성
- 파일 위치: `deployment/container/nginx.conf`
- 포트: 8080
- Health check endpoint 포함 (/health)
- 정적 파일 캐싱 설정
- SPA 라우팅 지원 (try_files)

#### 3.2 Dockerfile-frontend 생성
- 파일 위치: `deployment/container/Dockerfile-frontend`
- 멀티 스테이지 빌드 구조:
  - Builder stage: Node.js 20-slim 기반으로 빌드
  - Runtime stage: nginx:stable-alpine 기반으로 실행
- 빌드 아티팩트: `/app/dist` → `/usr/share/nginx/html`
- 보안 설정: nginx 사용자로 실행

#### 3.3 .dockerignore 생성
```
node_modules
.git
.gitignore
README.md
.env
.env.*
dist
coverage
.nyc_output
.vscode
.idea
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.DS_Store
Thumbs.db
.eslintcache
.tsbuildinfo
claude/
design/
deployment/k8s/
!deployment/container/
*.md
!deployment/container/*.md
```

**중요**: deployment/container/ 디렉토리는 포함시키고, deployment/k8s/만 제외

### 4. 컨테이너 이미지 빌드 시도

#### 사용한 빌드 명령어
```bash
export DOCKER_FILE=deployment/container/Dockerfile-frontend

docker build \
  --platform linux/amd64 \
  --build-arg PROJECT_FOLDER="." \
  --build-arg BUILD_FOLDER="deployment/container" \
  --build-arg EXPORT_PORT="8080" \
  -f ${DOCKER_FILE} \
  -t phonebill-front:latest .
```

#### 빌드 상태
❌ **Docker 데몬 연결 실패**
- 오류: `error during connect: Head "http://%2F%2F.%2Fpipe%2FdockerDesktopLinuxEngine/_ping"`
- 원인: Docker Desktop이 실행되지 않음
- Docker 클라이언트 버전: 28.0.4 설치됨

## 생성된 파일 목록
```
deployment/
└── container/
    ├── nginx.conf              # Nginx 웹서버 설정
    ├── Dockerfile-frontend     # 프론트엔드 컨테이너 이미지 빌드 파일
    └── build-image.md          # 본 결과 파일
.dockerignore                   # Docker 빌드 제외 파일 목록
```

## 빌드 완료를 위한 다음 단계
1. **Docker Desktop 시작**: Windows에서 Docker Desktop 애플리케이션을 실행
2. **빌드 재실행**:
   ```bash
   export DOCKER_FILE=deployment/container/Dockerfile-frontend

   docker build \
     --platform linux/amd64 \
     --build-arg PROJECT_FOLDER="." \
     --build-arg BUILD_FOLDER="deployment/container" \
     --build-arg EXPORT_PORT="8080" \
     -f ${DOCKER_FILE} \
     -t phonebill-front:latest .
   ```
3. **이미지 확인**:
   ```bash
   docker images | grep phonebill-front
   ```

## 추가 정보
- **Node.js 버전**: 20 (slim)
- **웹서버**: Nginx stable-alpine
- **포트**: 8080
- **빌드 출력**: /app/dist (Vite 빌드 결과)
- **보안**: 비-루트 사용자(nginx)로 실행