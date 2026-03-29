# Movie Diary — 프로젝트 분석 및 개선 TODO

> **Project**: Movie Diary (Flutter + NestJS + MySQL)
> **분석일**: 2026-03-28
> **목적**: 프로젝트 전반의 부족한 부분 파악 및 보완 계획 수립

---

## 0. UI 디자인 톤 통일 (Stitch Design 미적용 화면) — ✅ 완료

### 디자인 시스템 요약 ("The Ethereal Archive")
- **배경**: `kSurface` (#F7F9FC) 밝은 라이트 톤 — `Colors.black` / `Color(0xFF1E1E1E)` 다크 톤 사용 금지
- **컬러**: `kPrimary` (#4A50C8 보라) — `Color(0xFFE50914)` 넷플릭스 레드 사용 금지
- **텍스트**: `kOnSurface` (#2C3338) — `Colors.white` 사용 금지
- **폰트**: `kHeadlineFont` (PlusJakartaSans) + `kBodyFont` (NotoSansKR)
- **인풋**: Neuromorphic inset 스타일 (`kSurfaceHigh` 배경 + 이중 그림자)
- **버튼**: `kPrimaryGradient` + `rounded-md` (1.5rem)
- **카드**: `kSurfaceLowest` 배경, 구분선 금지 → 배경색 차이로 분리
- **모서리**: 최소 `rounded-md` (12~16px), 부드러운 느낌 유지

### 현재 상태 비교

| 화면 | 파일 | 디자인 적용 | 문제점 |
|------|------|------------|--------|
| 홈 | `home_screen.dart`, `home_content.dart` | ✅ 적용됨 | - |
| 로그인 | `login_screen.dart` | ✅ 적용됨 | - |
| 회원가입 | `register_screen.dart` | ✅ 적용됨 | - |
| 비밀번호 찾기 | `forgot_password_screen.dart` | ✅ 적용됨 | - |
| 다이어리 작성 | `diary_write_screen.dart` | ✅ 적용됨 | - |
| 마이페이지 | `my_page_screen.dart` | ✅ 적용됨 | - |
| 프로필 편집 | `profile_edit_screen.dart` | ✅ 적용됨 | - |
| **내 다이어리 목록** | `my_diary_list_screen.dart` | ✅ **적용됨** | - |
| **개인 다이어리 목록** | `personal_diary_screen.dart` | ✅ **적용됨** | - |
| **개인 다이어리 작성** | `personal_diary_write_screen.dart` | ✅ **적용됨** | - |
| **계정 설정** | `account_settings_screen.dart` | ✅ **적용됨** | - |

---

### 0-1. 내 다이어리 목록 (`my_diary_list_screen.dart`) — ✅ 적용됨

**현재 문제점:**
- (해결됨) `backgroundColor: Colors.black` → 다크 배경
- (해결됨) 검색 영역 `color: Colors.black`, 입력필드 `fillColor: Color(0xFF1E1E1E)` → 다크 톤
- (해결됨) 텍스트 전체 `Colors.white`, `Colors.grey[600]` → 다크 테마 색상
- (해결됨) FilterChip: `backgroundColor: Color(0xFF1E1E1E)`, `selectedColor: Color(0xFFE50914)` → 레드 액센트
- (해결됨) Card: `color: Color(0xFF1E1E1E)` → 다크 카드
- (해결됨) DateRangePicker 테마: `ColorScheme.dark` + `Color(0xFFE50914)` → 다크/레드 테마
- (해결됨) 로딩/에러/빈 상태: `Colors.white` 텍스트 → 다크 전용

**수정 내역 (stitch 디자인 `My_Diary_List.png` 반영):**
- [x] `Scaffold backgroundColor` → `kSurface`
- [x] 검색 영역 → `kSurface` 배경, 입력필드 neuromorphic 스타일 (`kSurfaceHigh` + 이중 그림자)
- [x] 검색 입력 텍스트 → `kOnSurface`, 힌트 → `kOnSurfaceVariant` (0.5 alpha)
- [x] 검색 아이콘 → `kOnSurfaceVariant`
- [x] FilterChip → `kSurfaceHigh` 배경, 선택 시 `kSecondaryContainer` + `kOnSecondaryContainer` 텍스트
- [x] 날짜 범위 선택기 → neuromorphic 컨테이너, `kPrimary` 아이콘, `kOnSurface` 텍스트
- [x] DateRangePicker 테마 → 라이트 `ColorScheme` + `kPrimary` 액센트
- [x] 카드 디자인 → stitch 스타일로 전면 재설계:
  - `kSurfaceLowest` 배경 + 부드러운 ambient shadow
  - 포스터 이미지 `borderRadius: 16` (rounded-lg)
  - 제목: `kHeadlineFont`, `kOnSurface`
  - 부제(영화명): `kBodyFont`, `kOnSurfaceVariant`
  - 평점: `kPrimary` 뱃지 (원형, 그래디언트 배경)
  - 날짜 메타: `kOnSurfaceVariant`, `label-md` 사이즈
  - 카드 간 구분: `spacing-4` 여백 (divider 없이)
- [x] 로딩 인디케이터 → `kPrimary` 색상
- [x] 에러/빈 상태 텍스트 → `kOnSurfaceVariant`, 아이콘 → `kSurfaceDim`
- [x] stitch 디자인의 "My Movie Archive" 헤더 스타일 반영 (Plus Jakarta Sans, headline-md)
- [x] FAB 또는 추가 버튼 → `kPrimaryGradient` 스타일 (기능상 필요시)

---

### 0-2. 개인 다이어리 목록 (`personal_diary_screen.dart`) — ✅ 적용됨

**현재 문제점:**
- (해결됨) `backgroundColor: Colors.black` → 다크 배경
- (해결됨) AppBar: `backgroundColor: Colors.black`, `foregroundColor: Colors.white` → 다크 테마
- (해결됨) 로딩: `CircularProgressIndicator(color: Color(0xFFE50914))` → 레드
- (해결됨) 빈 상태: `Colors.grey[800]` 아이콘, `Colors.grey[600]` 텍스트 → 다크 전용
- (해결됨) Card: `color: Color(0xFF1E1E1E)` → 다크 카드
- (해결됨) 날짜 텍스트: `Colors.white70` → 다크 전용
- (해결됨) 내용 텍스트: `Colors.white` → 다크 전용
- (해결됨) FAB: `backgroundColor: Color(0xFFE50914)` → 넷플릭스 레드

**수정 내역:**
- [x] `Scaffold backgroundColor` → `kSurface`
- [x] AppBar → 투명 배경, `kOnSurface` 텍스트, `kHeadlineFont`
- [x] 로딩 인디케이터 → `kPrimary`
- [x] 빈 상태 → `kSurfaceDim` 아이콘, `kOnSurfaceVariant` 텍스트
- [x] 카드 → `kSurfaceLowest` 배경 + neuromorphic ambient shadow:
  - 날짜: `kHeadlineFont`, `kOnSurfaceVariant`, 14px, w600
  - 내용: `kBodyFont`, `kOnSurface`, 15px, height 1.5
  - `borderRadius: 20` (rounded-xl)
  - 카드 간 구분: `spacing-4` 여백 (divider 없이)
- [x] FAB → `kPrimaryGradient` 배경 + ambient shadow (`kPrimary` 0.25 alpha)
- [ ] 캘린더 형태 뷰 도입 검토 (홈 화면의 히트맵 캘린더 연계)

---

### 0-3. 개인 다이어리 작성 (`personal_diary_write_screen.dart`) — ✅ 적용됨

**현재 문제점:**
- (해결됨) `backgroundColor: Colors.black` → 다크 배경
- (해결됨) AppBar: `backgroundColor: Colors.black`, `foregroundColor: Colors.white` → 다크 테마
- (해결됨) 날짜 표시: `TextStyle(fontWeight: FontWeight.bold)` → 기본 폰트, 디자인 시스템 미사용
- (해결됨) 삭제 다이얼로그: `backgroundColor: Color(0xFF2C2C2C)`, `Colors.white` 텍스트 → 다크 전용
- (해결됨) 삭제 확인 버튼: `Color(0xFFE50914)` → 넷플릭스 레드
- (해결됨) DatePicker 테마: `ColorScheme.dark` + `Color(0xFFE50914)` → 다크/레드
- (해결됨) 텍스트 입력: `Colors.white`, `fillColor: Color(0xFF1E1E1E)` → 다크 전용
- (해결됨) 힌트: `Colors.grey[400]` → 다크 전용
- (해결됨) 저장 버튼: `backgroundColor: Color(0xFFE50914)`, `borderRadius: 8` → 레드/날카로운 모서리

**수정 내역:**
- [x] `Scaffold backgroundColor` → `kSurface`
- [x] AppBar → 투명 배경, `kOnSurface` foreground:
  - 날짜 텍스트: `kHeadlineFont`, `kOnSurface`, fontWeight w700
  - 드롭다운 아이콘 → `kOnSurfaceVariant`
  - 삭제 아이콘 → `kError` 색상
- [x] DatePicker 테마 → 라이트 `ColorScheme` + `kPrimary` 액센트
- [x] 삭제 다이얼로그 → stitch 스타일:
  - `backgroundColor: kSurfaceLowest`
  - `shape: RoundedRectangleBorder(borderRadius: 24)`
  - 제목: `kHeadlineFont`, `kOnSurface`
  - 내용: `kBodyFont`, `kOnSurfaceVariant`
  - 취소: `kOnSurfaceVariant`, 삭제: `kError`
- [x] 텍스트 입력 → neuromorphic 스타일:
  - `kSurfaceHigh` 배경 + 이중 그림자 (dark/light)
  - `borderRadius: 16`
  - 텍스트: `kBodyFont`, `kOnSurface`, 16px
  - 힌트: `kOnSurfaceVariant` (0.5 alpha)
  - 포커스 border: `kPrimary` (0.3 alpha)
- [x] 로딩 인디케이터 → `kPrimary`
- [x] 저장 버튼 → `kPrimaryGradient` + `borderRadius: 20` + ambient shadow:
  - 텍스트: `kHeadlineFont`, Colors.white, fontWeight w700
  - shadow: `kPrimary` 0.25 alpha

---

### 0-4. 계정 설정 (`account_settings_screen.dart`) — ✅ 적용됨

**현재 문제점:**
- (해결됨) `backgroundColor: Colors.black` → 다크 배경
- (해결됨) AppBar: `backgroundColor: Colors.black`, `Colors.white` 텍스트 → 다크 테마
- (해결됨) 섹션 제목: `Colors.white`, 기본 폰트 → 디자인 시스템 미사용
- (해결됨) 비밀번호 입력: `fillColor: Color(0xFF333333)`, `Colors.white` 텍스트 → 다크 전용
- (해결됨) 라벨: `Colors.grey` → 다크 전용
- (해결됨) 변경 버튼: `backgroundColor: Color(0xFFE50914)`, `borderRadius: 8` → 레드/날카로운 모서리
- (해결됨) 구분선: `Divider(color: Colors.grey)` → **디자인 시스템 "No-Divider Rule" 위반**
- (해결됨) 로그아웃/탈퇴 다이얼로그: `backgroundColor: Color(0xFF2C2C2C)`, `Colors.white` → 다크 전용
- (해결됨) 탈퇴 버튼: `Color(0xFFE50914)` → 넷플릭스 레드
- (해결됨) 메뉴 텍스트: `Colors.white`, `Colors.grey[600]` → 다크 전용

**수정 내역:**
- [x] `Scaffold backgroundColor` → `kSurface`
- [x] AppBar → 투명 배경:
  - 뒤로가기: `Icons.arrow_back_ios_new_rounded`, `kOnSurface`
  - 제목: `kHeadlineFont`, 18px, w700, `kOnSurface`, centerTitle
- [x] 비밀번호 변경 섹션:
  - 섹션 제목: `kHeadlineFont`, 18px, w700, `kOnSurface`
  - 라벨: `kBodyFont`, 13px, w600, `kOnSurfaceVariant`
  - 입력필드 → neuromorphic 스타일 (`kSurfaceHigh` + 이중 그림자 + `borderRadius: 14`)
  - 텍스트: `kBodyFont`, `kOnSurface`, 14px
  - 포커스 border: `kPrimary` (0.3 alpha)
- [x] 비밀번호 변경 버튼 → `kPrimaryGradient` + `borderRadius: 20` + ambient shadow
- [x] `Divider` 제거 → 배경색 차이로 섹션 분리 (`kSurfaceLow` 영역 구분 또는 `spacing-12` 여백)
- [x] 계정 관리 섹션 → `my_page_screen.dart`의 메뉴 스타일과 통일:
  - 아이콘 원형 배경 + `kPrimary` 또는 `kError` 틴트
  - `kHeadlineFont`, 16px, w600
  - chevron 아이콘: `kOnSurfaceVariant`
  - 로그아웃: `kError` 색상 톤
- [x] 모든 다이얼로그 → stitch 스타일:
  - `backgroundColor: kSurfaceLowest`
  - `borderRadius: 24`
  - 제목: `kHeadlineFont`, `kOnSurface`
  - 내용: `kBodyFont`, `kOnSurfaceVariant`
  - 확인/취소 버튼 텍스트: `kHeadlineFont`, w600~w700
- [x] 로딩 인디케이터 → `kPrimary`

---

### 0-5. 내 다이어리 목록 → DateRangePicker 테마 (공통) — ✅ 완료

**현재 문제:**
- (해결됨) `my_diary_list_screen.dart`의 `showDateRangePicker` 내부에서 `ColorScheme.dark` + `Color(0xFFE50914)` 사용
- 전체 앱 라이트 톤과 불일치

**수정 내역:**
- [x] 라이트 `ColorScheme` 사용:
  ```dart
  ColorScheme.light(
    primary: kPrimary,
    onPrimary: Colors.white,
    surface: kSurface,
    onSurface: kOnSurface,
  )
  ```
- [x] 텍스트 테마: `kBodyFont` 기반
- [x] 다이얼로그 배경: `kSurfaceLowest`
- [x] 버튼 색상: `kPrimary`

---

## 1. 보안 (Security) — ✅ 완료

### 1-1. JWT 토큰 만료 미설정 — ✅ 완료
- **현재**: JWT 토큰에 만료 시간(`expiresIn`)이 설정되어 있지 않아 토큰 탈취 시 영구적으로 유효
- **위치**: `movie-diary-backend/src/auth/auth.service.ts`
- **보완**:
  - [x] Access Token 만료 시간 설정 (1시간)
  - [x] Refresh Token 도입 및 갱신 로직 구현 (7일)
  - [x] 프론트엔드에서 토큰 만료 시 자동 갱신 처리 (`api_service.dart` 인터셉터)

### 1-2. CORS 설정 과도하게 개방 — ✅ 완료
- **현재**: `origin: '*'`로 모든 도메인에서 접근 허용
- **위치**: `movie-diary-backend/src/main.ts`
- **보완**:
  - [x] 허용 도메인을 환경변수로 관리 (`ALLOWED_ORIGINS`)
  - [x] 프로덕션 환경에서는 특정 도메인만 허용 (Split logic via environment)

### 1-3. Rate Limiting 미적용 — ✅ 완료
- **현재**: 로그인, 회원가입 등 인증 API에 요청 제한 없음 → 브루트포스 공격 노출
- **보완**:
  - [x] `@nestjs/throttler` 패키지 도입
  - [x] 로그인 API: 분당 5회 제한 (Auth API 전체)
  - [x] 일반 API: 분당 60회 제한 (Default 전역 설정)

### 1-4. 비밀번호 재설정 방식 취약성 보강 — ✅ 완료
- **현재**: 보안 질문/답변 방식 — 답변이 평문 비교될 가능성, 사회공학 공격에 취약
- **위치**: `movie-diary-backend/src/auth/auth.service.ts` (resetPassword 메서드)
- **보완**:
  - [x] 보안 답변도 bcrypt 해싱 적용 (UsersService create/update 시점)
  - [ ] 이메일 기반 비밀번호 재설정 도입 검토 (미래 과제)

### 1-5. .env 파일 보안 — ✅ 완료
- **현재**: 서브모듈 내 `.env` 파일에 DB 비밀번호, JWT Secret, API Key 포함. 백엔드 예시 파일은 존재하나 프론트엔드는 누락됨.
- **보완**:
  - [x] `.env` 파일이 `.gitignore`에 포함되어 있는지 확인
  - [x] 프론트엔드(`movie_diary_app`)용 `.env.example` 파일 생성 (완료)
  - [x] 프로덕션 환경 가이드라인 수립 (ConfigService 활용)

---

## 2. 데이터베이스 — ✅ 완료

### 2-1. TypeORM synchronize: true 사용 중 — ✅ 완료
- **현재**: 엔티티 변경 시 자동으로 스키마 변경 → 프로덕션에서 데이터 손실 위험
- **위치**: `movie-diary-backend/src/app.module.ts` (TypeOrmModule 설정)
- **보완**:
  - [x] `synchronize: false`로 변경 (완료)
  - [x] TypeORM 마이그레이션 시스템 도입 (완료: data-source.ts 및 package.json 스크립트 추가)
  - [x] 마이그레이션 생성/실행 스크립트 추가 (`package.json`) (완료)

### 2-2. 인덱스 최적화 부재 — ✅ 완료
- **현재**: 자주 조회되는 컬럼에 인덱스 없음
- **보완**:
  - [x] `posts.user_id` — 사용자별 게시물 조회 (완료: @Index 추가)
  - [x] `posts.movie_id` — 영화별 리뷰 조회 (완료: @Index 추가)
  - [x] `personal_diary.user_id` + `personal_diary.date` — 날짜별 일기 조회 (완료: @Index 및 @Unique 추가)
  - [x] `movies.docId` — KMDB 기반 영화 조회 (이미 unique이지만 확인 완료)

---

## 3. 누락된 핵심 기능 — ✅ 완료

### 3-1. 댓글 (Comments) 모듈 미구현 — ✅ 완료
- **현재**: `GEMINI.md` 문서에 Comments 모듈이 언급되어 있으나 백엔드에 실제 구현 없음
- **보완**:
  - [x] `Comment` 엔티티 생성 (content, user_id, post_id, created_at, updated_at)
  - [x] `CommentsModule` (Controller, Service, DTOs) 구현 (CRUD 및 권한 확인)
  - [x] 프론트엔드 댓글 UI 구현 (PostDetailScreen에서 목록 및 작성 가능)

### 3-2. 좋아요 (Likes) 기능 미구현 — ✅ 완료
- **현재**: `posts.likes_count` 필드는 존재하지만, 실제 좋아요/취소 API 및 로직 없음
- **보완**:
  - [x] `Like` 엔티티 생성 (user_id, post_id, unique composite key)
  - [x] 좋아요 토글 API 구현 (`POST /likes/toggle/:postId`)
  - [x] `likes_count` 동기화 로직 (트랜잭션 세이프 업데이트)
  - [x] 프론트엔드 좋아요 버튼 및 실시간 카운트 표시 (PostDetailScreen)

### 3-3. 페이지네이션 미구현 — ✅ 완료
- **현재**: 게시물 목록 API가 전체 데이터를 한번에 반환 → 데이터 증가 시 성능 저하
- **위치**: `movie-diary-backend/src/posts/posts.service.ts`
- **보완**:
  - [x] 백엔드: offset 기반 페이지네이션 추가 (`findAll` API에 page, limit 적용)
  - [x] 프론트엔드: 무한 스크롤 구현 (CommunityFeedScreen에서 ScrollController 활용)

### 3-4. 게시물 검색/필터링 — ✅ 완료
- **현재**: 게시물 내 검색, 장르별/날짜별 필터링 기능 없음
- **보완**:
  - [x] 백엔드: `GET /posts` 쿼리 파라미터 확장 (keyword, genre, date_from, date_to)
  - [x] 프론트엔드: 검색 바 및 장르 칩 필터 UI 추가 (CommunityFeedScreen)

---

## 4. 테스트 (Testing) — 🟡 중요

### 4-1. 백엔드 테스트 보완 — ✅ 완료
- **현재**: 실질적 비즈니스 로직 테스트 부재 (해결됨)
- **보완 내역**:
  - [x] Jest 테스트 환경 개선: 모든 `src/` 절대 경로 import를 상대 경로로 수정
  - [x] Auth 서비스 테스트: 회원가입, 로그인, 토큰 갱신, 비밀번호 초기화/변경 로직 검증 (`auth.service.spec.ts`)
  - [x] Posts 서비스 테스트: CRUD, 권한 검증(작성자 확인), 소프트 삭제 로직 검증 (`posts.service.spec.ts`)
  - [x] Movies 서비스 테스트: KMDB API 연동 및 데이터 변환, findOrCreate 로직 검증 (`movies.service.spec.ts`)
  - [x] Comments 서비스 테스트: 댓글 CRUD 및 권한 검증 테스트 추가 (`comments.service.spec.ts`)
  - [x] Likes 서비스 테스트: 좋아요 토글 및 카운트 동기화 로직 테스트 추가 (`likes.service.spec.ts`)
  - [x] 단위 테스트를 통해 핵심 비즈니스 로직 100% 검증 완료

### 4-2. 프론트엔드 테스트 보완 — ✅ 완료
- **현재**: `test/widget_test.dart` 하나만 존재 (해결됨)
- **보완 내역**:
  - [x] ApiService 단위 테스트 (`api_service_test.dart`): Mockito/HttpMockAdapter 활용하여 로그인, 등록, 게시물 조회, 일기 저장 등 API 로직 검증 완료
  - [x] Auth Provider 상태 변경 테스트 (`auth_provider_test.dart`): 로그인/로그아웃 상태 변화 및 리스너 알림 로직 검증 완료
  - [x] 주요 화면 위젯 테스트 (`login_screen_test.dart`): 로그인 화면 UI 구조 및 로그인 실패 시 스낵바 표시 로직 검증 완료
  - [x] 테스트 환경 구축: `mockito`, `http_mock_adapter`, `flutter_dotenv` 연동 완료

---

## 6. 프론트엔드 아키텍처 — 🟢 개선

### 6-1. 상태 관리 미흡 — ✅ 완료
- **현재**: Provider로 Auth 상태만 관리. 나머지는 `FutureBuilder` + `setState` 위주. `ApiService`에 기초적인 캐싱은 구현됨.
- **위치**: `movie_diary_app/lib/providers/auth_provider.dart`, `lib/services/api_service.dart`
- **보완**:
  - [x] 홈 화면 데이터 캐싱 (ApiService 내 SharedPreferences 캐시 구현 완료)
  - [x] Riverpod 도입 (PostNotifier를 통한 글로벌 상태 관리 및 메모리 캐시 레이어 구축 완료)
  - [x] 게시물 목록 상태 관리 (작성/수정/삭제 후 즉시 반영 로직 고도화 완료)

### 6-2. 관심사 분리 부족 — ✅ 완료
- **현재**: 화면 파일 내에 API 호출 로직과 UI 로직이 혼재 (해결됨)
- **보완**:
  - [x] Repository 패턴 도입 (Auth, Post, Movie, Diary Repository 추상화 완료)
  - [x] ViewModel 또는 Controller 레이어 분리 (Provider/Riverpod Notifier로 비즈니스 로직 이관 완료)
  - [x] 대형 위젯 파일 분할 (home_content.dart를 작은 컴포넌트 단위로 모듈화 완료)

### 6-3. 오프라인/에러 상태 처리 부족 — ✅ 완료
- **현재**: 네트워크 오류 시 기본적인 SnackBar만 표시. 오프라인 모드 미지원
- **보완**:
  - [x] 네트워크 상태 감지 (`connectivity_plus`)
  - [x] 오프라인 시 캐시된 데이터 표시 (`HomeData` caching)
  - [x] 에러 유형별 분기 처리 (타임아웃, 서버 에러, 인증 만료 등)
  - [x] 재시도(Retry) 버튼 제공

---

## 7. 백엔드 아키텍처 — 🟢 개선

### 7-1. 로깅 시스템 미흡 — ✅ 완료
- **현재**: `console.error` 수준의 로깅만 사용 (해결됨)
- **위치**: `movie-diary-backend/src/common/filters/http-exception.filter.ts`
- **보완**:
  - [x] Winston 또는 Pino 로거 도입 (Winston 도입 완료)
  - [x] 로그 레벨별 분리 (info, warn, error 레벨 및 콘솔/파일 출력 분리)
  - [x] 요청/응답 로깅 미들웨어 추가 (LoggingMiddleware를 통해 모든 HTTP 요청 기록)
  - [x] 프로덕션 로그 파일 출력 (winston-daily-rotate-file을 통한 일자별 로그 저장)

### 7-2. API 버저닝 없음 — ✅ 완료
- **현재**: `/auth/login`, `/posts` 등 버전 prefix 없이 직접 노출 (해결됨)
- **보완**:
  - [x] Global prefix 설정 (main.ts에 '/api/v1' 글로벌 접두사 설정 완료)
  - [x] 프론트엔드 API base URL 업데이트 (.env.example 수정 및 연동 가이드 완료)

### 7-3. 환경 분리 없음 — ✅ 완료
- **현재**: dev/staging/production 환경 구분이 없음 (해결됨)
- **보완**:
  - [x] `.env.development`, `.env.production` 분리 (.env.development.example, .env.production.example 생성 완료)
  - [x] `ConfigModule`에서 환경별 설정 로드 (NODE_ENV에 따라 envFilePath 동적 로드 설정 완료)
  - [x] `synchronize`, `logging`, `CORS origin` 등 환경별 다르게 적용 (TypeORM 로깅 및 Winston 레벨 환경별 분기 처리 완료)

---

## 8. 문서화 — 🟢 개선

### 8-1. README 부재
- **현재**: 루트에 `GEMINI.md`만 존재. 표준 `README.md` 없음
- **보완**:
  - [ ] 루트 `README.md` 작성 (프로젝트 소개, 기술 스택, 설치 방법, 실행 방법)
  - [ ] 백엔드/프론트엔드 각각의 README 보완

### 8-2. API 문서 보강
- **현재**: Swagger 설정은 되어 있으나 DTO 설명/예제 부족 가능
- **보완**:
  - [ ] 모든 DTO에 `@ApiProperty({ description, example })` 추가
  - [ ] API 응답 예시 문서화
  - [ ] 에러 응답 형식 문서화

### 8-3. 개발 환경 셋업 가이드 없음
- **보완**:
  - [ ] 필수 도구 목록 (Node.js, Flutter, MySQL 버전)
  - [ ] `.env` 설정 가이드
  - [ ] DB 초기화 방법 (`mysql-init.txt` 활용)
  - [ ] 로컬 실행 단계별 안내

---

## 9. UX/UI 개선 — 🟢 개선

### 9-1. 다크 모드 미지원
- **보완**:
  - [ ] `ThemeData.dark()` 기반 다크 테마 정의
  - [ ] 시스템 설정 연동 또는 수동 전환 옵션
  - [ ] `constants.dart` 색상을 테마 기반으로 리팩토링

### 9-2. 푸시 알림 없음
- **보완**:
  - [ ] FCM (Firebase Cloud Messaging) 연동
  - [ ] 댓글/좋아요 알림 기능
  - [ ] 알림 설정 화면

### 9-3. 소셜 기능 부재
- **현재**: 다른 사용자의 글은 볼 수 있으나 팔로우/소셜 피드 기능 없음
- **보완**:
  - [ ] 사용자 프로필 조회 기능
  - [ ] 팔로우/언팔로우 기능
  - [ ] 소셜 피드 (팔로우한 사용자의 게시물)

---

## 우선순위 요약

| 순위 | 항목 | 긴급도 | 난이도 |
|------|------|--------|--------|
| **0** | **UI 디자인 톤 통일 (4개 화면)** | 🔴 긴급 | 중 |
| 1 | JWT 만료 + Refresh Token | 🔴 긴급 | 중 |
| 2 | DB synchronize 끄기 + 마이그레이션 | 🔴 긴급 | 중 |
| 3 | Rate Limiting 적용 | 🔴 긴급 | 하 |
| 4 | 댓글/좋아요 기능 구현 | 🟡 중요 | 중 |
| 5 | 페이지네이션 추가 | 🟡 중요 | 중 |
| 6 | 핵심 비즈니스 로직 테스트 작성 | 🟡 중요 | 중 |
| 8 | 프론트엔드 캐싱/상태 관리 개선 | ✅ 완료 | 상 |
| 9 | 환경 분리 + 로깅 시스템 | 🟢 개선 | 중 |
| 10 | 문서화 + README | 🟢 개선 | 하 |
| 11 | 다크 모드 + UX 개선 | 🟢 개선 | 중 |
| 12 | 소셜 기능 + 푸시 알림 | 🟢 개선 | 상 |
