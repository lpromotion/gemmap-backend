# Gemmap Backend (젬맵 백엔드)

> 사진의 메타데이터(EXIF)를 활용하여 사용자들에게 숨겨진 사진 명소(Spot)를 공유하고 탐색할 수 있게 해주는 위치 기반 서비스의 백엔드 API 서버입니다.

## ✨ Key Features (주요 기능)


  * **🔐 OAuth2 인증 및 인가**
      * Kakao 소셜 로그인 지원 (Native App SDK 연동 방식)
      * JWT(Access/Refresh Token) 기반의 보안 인증 및 Token Rotation 적용
      * GUEST -\> USER 권한 승격 프로세스 (회원가입 절차)
  * **📸 사진 및 스팟(Spot) 관리**
      * 사진 업로드 시 **EXIF 메타데이터(GPS 위/경도, 촬영 일시, 카메라 정보 등) 자동 추출 및 저장**
      * S3 호환 Object Storage(OCI/NHN)를 이용한 이미지 파일 관리
      * 보상 트랜잭션 패턴 적용: DB 저장 실패 시 업로드된 S3 객체 자동 롤백
  * **🗺️ 위치 기반 서비스**
      * 사용자별 스팟 생성, 조회 및 삭제 (Soft Delete 지원)
      * 지도 표시를 위한 전체 스팟 마커 데이터 조회 (N+1 문제 해결을 위한 최적화 쿼리 적용)
  * **⚙️ DevOps & Infrastructure**
      * Docker & Kubernetes(Kustomize) 기반의 배포 환경 구성
      * GitHub Actions를 활용한 CI/CD 파이프라인 구축 (Oracle Cloud 자동 배포)

## 🛠 Tech Stack (기술 스택)

### Backend

  * **Language:** Java 17
  * **Framework:** Spring Boot 3.5.4
  * **Build Tool:** Gradle (w/ Version Catalog `libs.versions.toml`)
  * **Database:** MySQL (Production/Dev), H2 (Test)
  * **ORM:** Spring Data JPA, Hibernate Spatial
  * **Security:** Spring Security, JWT (Json Web Token)

### Infra & Tools

  * **Infrastructure:** Oracle Cloud Infrastructure (OCI), Kubernetes
  * **Storage:** S3 Compatible Object Storage (OCI/NHN)
  * **CI/CD:** GitHub Actions
  * **Container:** Docker

## 📂 Directory Structure (폴더 구조)

핵심 도메인 중심의 패키지 구조로 설계되었습니다.

```bash
gemmap-backend
├── k8s/                     # Kubernetes 배포 설정 (Kustomize)
├── src/main/java/com/gemmap/gemmap
│   ├── auth/                # 인증 도메인 (Kakao Login, JWT, User)
│   │   ├── application/     # 비즈니스 로직 (Service, DTO)
│   │   ├── domain/          # 엔티티 및 레포지토리
│   │   ├── infrastructure/  # 구현체 (JWT Provider, OAuth Client)
│   │   └── presentation/    # 인증 컨트롤러
│   ├── image/               # 이미지 처리 도메인
│   │   ├── application/     # EXIF 추출기, 이미지 서비스
│   │   ├── domain/          # Image 엔티티
│   │   ├── infrastructure/  # Object Storage(S3) 연동
│   │   └── presentation/    # 이미지 컨트롤러
│   ├── spot/                # 스팟(장소) 도메인
│   │   ├── application/     # 스팟 생성/조회 로직
│   │   ├── domain/          # Spot, SpotPhoto 엔티티
│   │   └── presentation/    # 스팟 컨트롤러
│   └── shared/              # 공통 모듈 (Config, Exception, Util)
└── .github/workflows        # CI/CD 파이프라인 설정
```

## 🚀 Getting Started (설치 및 실행)

로컬 개발 환경에서 프로젝트를 실행하기 위한 가이드입니다.

### Prerequisites

  * Java 17+
  * Docker & Docker Compose (Optional for DB)
  * MySQL 8.0+

### 1\. Repository Clone

```bash
git clone https://github.com/YOUR_USERNAME/gemmap-backend.git
cd gemmap-backend
```

### 2\. Environment Variables Setup

프로젝트 루트 또는 IDE 설정에 환경 변수를 등록해야 합니다. (예: `.env` 또는 `application-local.yml` 활용)

```properties
# Database
DB_HOST=localhost
DB_PORT=3306
DB_NAME=gemmap
DB_USERNAME=root
DB_PASSWORD=your_password

# JWT
JWT_SECRET_KEY=your_256bit_secret_key
JWT_ACCESS_TOKEN_EXPIRATION=3600000
JWT_REFRESH_TOKEN_EXPIRATION=604800000

# OAuth2 (Kakao)
KAKAO_CLIENT_ID=your_kakao_rest_api_key
KAKAO_CLIENT_SECRET=your_kakao_client_secret
KAKAO_REDIRECT_URI=http://localhost:8080/api/v1/auth/kakao/callback
KAKAO_APP_ID=your_kakao_app_id

# Object Storage (S3 Compatible)
S3_ENDPOINT=https://your-object-storage-endpoint
S3_REGION=ap-seoul-1
S3_BUCKET=your_bucket_name
S3_ACCESS_KEY=your_access_key
S3_SECRET_KEY=your_secret_key
S3_BASE_PATH=local/spots/
```

### 3\. Run Application

Gradle Wrapper를 사용하여 애플리케이션을 실행합니다.

```bash
# Linux/macOS
./gradlew bootRun --args='--spring.profiles.active=local'

# Windows
.\gradlew.bat bootRun --args='--spring.profiles.active=local'
```

### 4\. Build with Docker

```bash
docker build -t gemmap-backend .
docker run -p 8080:8080 --env-file .env gemmap-backend
```

## 📡 API Documentation (API 문서 예시)

주요 API 엔드포인트에 대한 요약입니다. 자세한 스펙은 Swagger UI (`/swagger-ui.html`)를 참고하세요.

### Auth API

| Method | Endpoint | Description | Auth Required |
| :--- | :--- | :--- | :---: |
| `POST` | `/api/v1/auth/kakao/login` | 카카오 Access Token으로 로그인/회원가입 시도 | ❌ |
| `POST` | `/api/v1/auth/register` | 추가 정보(닉네임, 프로필) 입력 후 정회원(User) 승격 | ✅ |
| `POST` | `/api/v1/auth/refresh` | Refresh Token을 이용한 Access Token 갱신 | ❌ |

### Spot API

| Method | Endpoint | Description | Auth Required |
| :--- | :--- | :--- | :---: |
| `POST` | `/api/v1/spots` | 사진과 메타데이터를 포함한 신규 스팟 등록 | ✅ |
| `GET` | `/api/v1/spots` | 지도 표시용 전체 스팟 마커 조회 | ✅ |
| `GET` | `/api/v1/spots/my` | 내가 등록한 스팟 목록 조회 | ✅ |
| `GET` | `/api/v1/spots/{spotId}` | 스팟 상세 정보 조회 (사진, EXIF 정보 포함) | ✅ |

### Image API

| Method | Endpoint | Description | Auth Required |
| :--- | :--- | :--- | :---: |
| `POST` | `/api/v1/images/upload` | 다중 이미지 업로드 및 URL 반환 | ✅ |
| `DELETE` | `/api/v1/images` | 이미지 URL 기반 삭제 | ✅ |

-----

**License**
Copyright © 2025 Gemmap Team. All rights reserved.
