# Gemmap Backend

> Gemmap(젬맵)은 사진의 EXIF 메타데이터(GPS, 촬영 정보)를 자동으로 분석하여 숨겨진 사진 명소(Spot)를 공유하고 탐색할 수 있게 해주는 위치 기반 서비스의 백엔드 API 서버입니다.

현재 이 프로젝트는 **Oracle Cloud Infrastructure (OCI)** 환경에서 Docker 컨테이너 기반으로 배포 및 운영되고 있습니다.

## 🔄 Infrastructure Migration (인프라 변경 사항)

본 프로젝트는 초기 **NHN Cloud (Kubernetes Service)** 환경에서 개발되었으나, 배포 환경의 제약 사항으로 인해 **Oracle Cloud Infrastructure (OCI)** 로 마이그레이션을 진행했습니다. 주요 변경 사항은 다음과 같습니다.

| 구분 | 변경 전 (NHN Cloud) | 변경 후 (Oracle Cloud Infrastructure) |
| :--- | :--- | :--- |
| **Compute / Orchestration** | NHN Kubernetes Service (NKS) | OCI Compute Instance (Docker Host) |
| **CI/CD Pipeline** | GitHub Actions ➔ NHN Cloud Pipeline (Anchor API) | GitHub Actions ➔ SSH Remote Docker Command |
| **Object Storage** | NHN Cloud Object Storage (Tenant ID 기반 URL) | OCI Object Storage (S3 호환 Path Style URL) |
| **Deployment Strategy** | Kubernetes Manifest 배포 | Docker Container 직접 실행 (`docker run`) |

> **Note**: 인프라 변경에 따라 `S3UrlGenerator` 로직이 OCI의 S3 호환 URL 구조(`{endpoint}/{bucket}/{key}`)에 맞게 재작성되었으며,
> 배포 방식이 간소화된 Docker 배포 파이프라인으로 변경되었습니다.

## ✨ Key Features (주요 기능)

  * **📸 EXIF 데이터 자동 추출**: 이미지 업로드 시 GPS 위/경도, 촬영 일시, 카메라 모델, 렌즈 정보(초점 거리, 조리개, ISO 등)를 자동으로 추출하여 DB에 저장합니다.
  * **☁️ OCI Object Storage 연동**: Oracle Cloud의 S3 호환 Object Storage를 사용하여 이미지를 저장하며, **보상 트랜잭션(Rollback)** 을 통해 DB 저장 실패 시 업로드된 파일을 자동 삭제하여 데이터 정합성을 보장합니다.
  * **🔐 Kakao OAuth2 인증**: 카카오 모바일 SDK와 연동되는 인증 방식을 지원하며, 회원가입 시 `GUEST`에서 `USER`로 권한이 승격되는 로직을 포함합니다.
  * **🗺️ 위치 기반 스팟 조회**: 지도에 표출될 마커 데이터를 최적화된 쿼리로 조회하여 대량의 데이터 처리 시에도 성능을 유지합니다.

## 🛠 Tech Stack (기술 스택)

| Category | Technology |
| :--- | :--- |
| **Language** | Java 17 |
| **Framework** | Spring Boot 3.5.4 |
| **Database** | MySQL 8.0 (Prod/Dev), H2 (Test) |
| **ORM** | Spring Data JPA, Hibernate Spatial |
| **Security** | Spring Security, JWT |
| **Infrastructure** | Oracle Cloud Infrastructure (OCI) |
| **Storage** | OCI Object Storage (S3 Compatible) |
| **DevOps** | Docker, GitHub Actions |

## 🚀 Getting Started (설치 및 실행)

로컬 개발 환경에서 프로젝트를 실행하기 위한 가이드입니다.

### 1\. Prerequisites

  * Java 17+
  * MySQL 8.0+
  * Oracle Cloud (또는 S3 호환 스토리지) 계정 설정

### 2\. Clone Repository

```bash
git clone https://github.com/YOUR_USERNAME/gemmap-backend.git
cd gemmap-backend
```

### 3\. Environment Variables

`application-local.yml` 또는 환경 변수를 통해 다음 설정을 완료해야 합니다.

```yaml
# Database
DB_HOST: localhost
DB_PORT: 3306
DB_NAME: gemmap
DB_USERNAME: root
DB_PASSWORD: your_password

# JWT
JWT_SECRET_KEY: your_super_secret_key_base64_encoded

# OAuth2 (Kakao)
KAKAO_APP_ID: your_kakao_app_id

# OCI Object Storage (S3 Compatible)
S3_ENDPOINT: https://your-namespace.compat.objectstorage.ap-seoul-1.oraclecloud.com
S3_REGION: ap-seoul-1
S3_BUCKET: your_bucket_name
S3_ACCESS_KEY: your_oci_customer_secret_key_id
S3_SECRET_KEY: your_oci_customer_secret_key
S3_BASE_PATH: local/spots/
```

### 4\. Run Application

```bash
# MacOS/Linux
./gradlew bootRun --args='--spring.profiles.active=local'

# Windows
.\gradlew.bat bootRun --args='--spring.profiles.active=local'
```

## 📂 Directory Structure (폴더 구조)

```bash
gemmap-backend
├── k8s/                     # (Legacy) Kubernetes Manifests
├── src/main/java/com/gemmap/gemmap
│   ├── auth/                # 인증 도메인 (Kakao Login, JWT)
│   ├── image/               # 이미지 도메인 (S3 Upload, EXIF)
│   ├── spot/                # 스팟 도메인 (Spot CRUD, Geo Logic)
│   └── shared/              # 공통 모듈 (Config, Exception, Utils)
└── .github/workflows        # CI/CD (Docker Build & OCI Deploy)
```

## 📡 API Documentation (주요 엔드포인트)

| Domain | Method | URL | Description | Auth |
| :--- | :---: | :--- | :--- | :---: |
| **Auth** | `POST` | `/api/v1/auth/kakao/login` | 카카오 Access Token 로그인 | ❌ |
| | `POST` | `/api/v1/auth/register` | 회원가입 (User 권한 승격) | ✅ |
| | `POST` | `/api/v1/auth/refresh` | 토큰 갱신 | ❌ |
| **Spot** | `POST` | `/api/v1/spots` | 스팟 등록 (이미지+메타데이터) | ✅ |
| | `GET` | `/api/v1/spots` | 전체 스팟 마커 조회 | ✅ |
| | `GET` | `/api/v1/spots/{spotId}` | 스팟 상세 조회 | ✅ |
| **Image** | `POST` | `/api/v1/images/upload` | 이미지 업로드 | ✅ |

-----

**License**
Copyright © 2025 Gemmap Team. All rights reserved.
