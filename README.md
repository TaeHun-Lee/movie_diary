# Movie Diary (영화 다이어리 프로젝트)

영화 검색, 개인 감상 기록, 그리고 다른 사용자들과 다이어리를 공유할 수 있는 통합 영화 다이어리 플랫폼입니다.

## 📌 프로젝트 개요

이 프로젝트는 사용자가 좋아하는 영화를 검색하고 평점과 함께 자신만의 다이어리를 작성할 수 있는 서비스입니다. 개인 기록 공간뿐만 아니라, 다른 사용자들이 공개한 다이어리를 볼 수 있는 커뮤니티 기능도 함께 제공됩니다.

## 🛠️ 기술 스택 (Tech Stack)

### 프론트엔드 (Frontend)
- **Framework**: Flutter (v3.19.0 이상 권장)
- **State Management**: Riverpod, Provider
- **Design System**: 사용자 정의 "The Ethereal Archive" 시스템 적용

### 백엔드 (Backend)
- **Framework**: NestJS (v10 이상, Node.js v18+ 권장)
- **Database**: MySQL (v8.0 이상 권장), TypeORM
- **Authentication**: JWT 기반 인증 체계

## ⚙️ 개발 환경 셋업 가이드 (Development Setup Guide)

### 1. 필수 도구 (Prerequisites)
본 프로젝트를 로컬에서 실행하기 위해 아래 도구들이 설치되어 있어야 합니다.
- **Node.js**: v18.x 이상 ([다운로드](https://nodejs.org/))
- **Flutter SDK**: v3.19.0 이상 ([설치 가이드](https://docs.flutter.dev/get-started/install))
- **MySQL Server**: v8.0 이상 ([다운로드](https://dev.mysql.com/downloads/installer/))
- **Git**: 소스 코드 관리를 위해 필요

### 2. 데이터베이스 초기화 (Database Setup)
MySQL에 접속하여 프로젝트에서 사용할 데이터베이스를 생성합니다.
```sql
-- 데이터베이스 생성
CREATE DATABASE movie_diary_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- (선택 사항) root 비밀번호 설정이 필요한 경우 (mysql-init.txt 참고)
ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_password_here';
```

### 3. 백엔드 설정 및 서버 실행
`movie-diary-backend` 디렉토리로 이동하여 설정을 진행합니다.
```bash
cd movie-diary-backend

# 의존성 설치
npm install

# 환경 변수 설정 (.env.development 생성)
cp .env.development.example .env.development
# .env.development 파일을 열어 DB_USER, DB_PASSWORD 등을 본인의 환경에 맞게 수정하세요.

# DB 마이그레이션 실행 (테이블 생성)
npm run typeorm:run-migrations

# 서버 실행 (개발 모드)
npm run start:dev
```
백엔드 서버가 `http://localhost:3000`에서 정상적으로 구동되는지 확인하세요.

### 4. 프론트엔드 설정 및 앱 실행
`movie_diary_app` 디렉토리로 이동하여 설정을 진행합니다.
```bash
cd ../movie_diary_app

# 패키지 설치
flutter pub get

# 환경 변수 설정 (.env 생성)
cp .env.example .env
# .env 파일을 열어 API_BASE_URL이 백엔드 주소와 일치하는지 확인하세요.
# 예: API_BASE_URL=http://localhost:3000/api/v1

# 앱 실행
flutter run
```

## 📁 디렉토리 구조 (Directory Structure)

- `movie_diary_app/` : Flutter 기반의 프론트엔드 애플리케이션
- `movie-diary-backend/` : NestJS 기반의 백엔드 서버 및 API (MySQL)

## 🚀 빠른 시작 (Quick Start)

본 프로젝트는 프론트엔드와 백엔드가 분리되어 있습니다. 각 프로젝트의 디렉토리에서 상세한 설정 및 실행 방법을 확인하세요.

1. **백엔드 서버 설정 및 실행**
   - 백엔드 실행 가이드는 [movie-diary-backend/README.md](./movie-diary-backend/README.md)를 참조하세요.
   - MySQL 데이터베이스 초기화 및 환경 변수 설정 과정이 포함되어 있습니다.

2. **프론트엔드 앱 설정 및 실행**
   - 프론트엔드 실행 가이드는 [movie_diary_app/README.md](./movie_diary_app/README.md)를 참조하세요.
   - 백엔드 서버가 구동 중인 상태에서 `.env` 파일을 설정하고 앱을 빌드합니다.

## 📝 주요 기능

- **회원가입 및 로그인**: JWT 기반 사용자 인증
- **영화 검색**: 외부 API를 연동한 영화 정보 및 포스터 검색 기능
- **다이어리 작성/관리**: 관람한 영화에 대한 평점, 리뷰 작성 (개인/공개 설정 가능)
- **커뮤니티 피드**: 인기 다이어리 및 다른 사용자의 리뷰 조회
- **댓글 및 좋아요**: 게시물에 대한 상호작용 기능 지원