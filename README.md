### Beellage Docker Compose 프로젝트

이 프로젝트는 Beellage 애플리케이션의 최상위 레포지토리로, 백엔드와 프론트엔드 컴포넌트를 포함합니다. Docker Compose를 사용하여 서비스를 함께 관리하고 실행합니다.

## 레포지토리 구조

```
Beellage-docker-compose/
├── beellage-backend-server/ (Git 서브모듈)
├── beellage-frontend-server/ (Git 서브모듈)
├── data/
│   └── postgres/
├── docker-compose.yml
├── .gitignore
└── README.md
```

## 초기 설정

### 레포지토리 클론

이 레포지토리와 서브모듈을 함께 클론하려면 다음 명령어를 사용하세요:

```
git clone --recurse-submodules <레포지토리-URL>
cd Beellage-docker-compose
```

### 서브모듈 초기화 및 업데이트

이미 클론한 경우, 서브모듈을 초기화하고 업데이트하려면 다음 명령어를 사용하세요:

```
git submodule update --init --recursive
```

## 원격 저장소 설정

서브모듈의 원격 저장소를 설정하려면 각 서브모듈 디렉토리로 이동하여 설정할 수 있습니다.

```
# 백엔드 서브모듈 디렉토리로 이동
cd beellage-backend-server

# 원격 저장소 설정 (origin)
git remote add origin <backend-repo-url>

# 원격 저장소 설정 (upstream, 필요 시)
git remote add upstream <upstream-backend-repo-url>

# 프론트엔드 서브모듈 디렉토리로 이동
cd ../beellage-frontend-server

# 원격 저장소 설정 (origin)
git remote add origin <frontend-repo-url>

# 원격 저장소 설정 (upstream, 필요 시)
git remote add upstream <upstream-frontend-repo-url>
```

### 서브모듈 업데이트

서브모듈을 업데이트할 때는 일반적으로 `origin`을 사용합니다.

```
# 최상위 프로젝트 디렉토리로 이동
cd Beellage-docker-compose

# 모든 서브모듈의 최신 변경 사항을 가져오기
git submodule foreach git pull origin main
```

필요한 경우, `upstream`에서 변경 사항을 가져올 수 있습니다.

```
# 백엔드 서브모듈 디렉토리로 이동
cd beellage-backend-server

# upstream의 최신 변경 사항을 가져오기
git pull upstream main

# 프론트엔드 서브모듈 디렉토리로 이동
cd ../beellage-frontend-server

# upstream의 최신 변경 사항을 가져오기
git pull upstream main
```

## Docker Compose 설정

### 데이터 디렉토리 생성

프로젝트 루트 디렉토리에서 데이터베이스 데이터를 저장할 디렉토리를 생성합니다.

```
mkdir -p data/postgres
```

### Docker Compose 서비스 시작

Docker Compose를 사용하여 모든 서비스를 빌드하고 시작합니다.

```
docker-compose up --build
```

## CI/CD 설정

이 프로젝트는 GitHub Actions를 사용하여 CI/CD 파이프라인을 설정합니다. Docker 이미지를 빌드하고 GitHub Packages에 푸시합니다.

### Docker 이미지를 빌드하고 GitHub Packages에 푸시

백엔드 및 프론트엔드 Docker 이미지를 빌드하고 GitHub Packages에 업로드하는 GitHub Actions 워크플로우를 설정합니다.

### 백엔드 GitHub Actions 설정

```
name: Build and Push Backend Docker Image

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Log in to GitHub Docker Registry
      run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

    - name: Build and push Docker images
      uses: docker/build-push-action@v2
      with:
        context: ./beellage-backend-server
        push: true
        tags: ghcr.io/${{ github.repository }}/beellage-backend:latest
        file: ./beellage-backend-server/Dockerfile
```

### 프론트엔드 GitHub Actions 설정

```
name: Build and Push Frontend Docker Image

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Log in to GitHub Docker Registry
      run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

    - name: Build and push Docker images
      uses: docker/build-push-action@v2
      with:
        context: ./beellage-frontend-server
        push: true
        tags: ghcr.io/${{ github.repository }}/beellage-frontend:latest
        file: ./beellage-frontend-server/Dockerfile
```

### Docker 이미지 사용하기

GitHub Packages에 업로드된 Docker 이미지를 사용하려면 다음 명령어를 실행합니다.

#### 백엔드 Docker 이미지 사용

```
docker pull ghcr.io/<사용자명>/beellage-backend:latest
```

#### 프론트엔드 Docker 이미지 사용

```
docker pull ghcr.io/<사용자명>/beellage-frontend:latest
```
