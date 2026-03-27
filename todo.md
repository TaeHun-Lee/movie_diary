# Movie Diary App — Home Screen Improvement Tasks

> **Project**: Movie Diary (Flutter + NestJS)
> **Target**: Home Screen (`HomeContent`) 개선
> **Created**: 2026-03-27
> **Backend 변경**: 없음 (전체 프론트엔드 작업)

---

## Project Structure

```
D:\Projects\movie_diary\
├── movie_diary_app/          # Flutter frontend
│   └── lib/
│       ├── main.dart
│       ├── constants.dart                    # 디자인 토큰 (kPrimary, kSurface, kHeadlineFont 등)
│       ├── screens/
│       │   ├── main_screen.dart              # 4탭 네비게이션 (홈/탐색/다이어리/프로필)
│       │   ├── home_screen.dart              # 홈 화면 진입점, FutureBuilder로 HomeData fetch
│       │   ├── movie_search_screen.dart
│       │   ├── my_diary_list_screen.dart
│       │   ├── my_page_screen.dart
│       │   ├── movie_detail_screen.dart
│       │   └── diary_write_screen.dart
│       ├── component/
│       │   ├── home_content.dart             # ★ 홈 화면 UI 전체 (1,060줄, 단일 파일)
│       │   └── custom_app_bar.dart
│       ├── data/
│       │   ├── home_data.dart                # HomeData, User, RatedMovie 모델
│       │   ├── diary_entry.dart              # DiaryEntry 모델
│       │   └── movie.dart                    # Movie 모델
│       └── services/
│           └── api_service.dart              # API 호출 (fetchHomeData 등)
└── movie-diary-backend/      # NestJS backend (이번 작업에서 수정 없음)
```

---

## Data Flow

```
MainScreen (_onItemTapped으로 탭 전환)
  └─ HomeScreen (StatefulWidget, GlobalKey로 외부에서 refresh 가능)
      ├─ _fetchHomeData() → ApiService.fetchHomeData()
      │   ├─ GET /auth/me        → User JSON
      │   └─ GET /posts/my       → List<DiaryEntry> JSON
      │   └─ 합쳐서 → HomeData 객체
      └─ FutureBuilder → HomeContent(data: HomeData)
          ├─ _buildGreeting()           인사말 헤더
          ├─ _buildFeaturedCard()       최근 다이어리 히어로 카드
          ├─ _buildStatsBento()         2×2 통계 그리드
          ├─ _buildTopRatedMovies()     평점 8.0+ 가로 스크롤
          ├─ _buildCalendarHeatmap()    이번 달 관람 히트맵
          └─ _buildDiaryCard() × 5     최근 다이어리 리스트
```

---

## HomeData Model (lib/data/home_data.dart)

```dart
class HomeData {
  final User user;
  final int todayCount;
  final int totalCount;
  final List<DiaryEntry> recentEntries;

  // Computed getters (매 접근마다 재계산됨 — Task 3에서 수정 대상):
  int get monthlyCount { ... }           // 이번 달 작성 수
  double get averageRating { ... }       // 전체 평균 평점
  String get topGenre { ... }            // 최다 장르
  List<RatedMovie> get topRatedMovies { ... }  // 평점 8.0+ 영화 리스트
  Map<int, int> get monthlyHeatmap { ... }     // 날짜별 관람 수 맵
}
```

---

## Design System (lib/constants.dart)

| Token | Value | Usage |
|-------|-------|-------|
| `kSurface` | `#F7F9FC` | 배경색 |
| `kSurfaceLow` | `#F0F4F8` | 콘텐츠 영역 |
| `kSurfaceLowest` | `#FFFFFF` | 인터랙티브 카드 |
| `kSurfaceHigh` | `#E3E9EE` | 엘리베이티드 오버레이 |
| `kSurfaceDim` | `#D4DBE1` | 그림자용 |
| `kPrimary` | `#4A50C8` | 주 액센트 (블루/퍼플) |
| `kPrimaryEnd` | `#787FF9` | 그래디언트 끝점 |
| `kPrimaryGradient` | LinearGradient | 배지/버튼 그래디언트 |
| `kOnSurface` | `#2C3338` | 본문 텍스트 |
| `kOnSurfaceVariant` | `#596065` | 보조 텍스트 |
| `kHeadlineFont` | `PlusJakartaSans` | 제목/숫자 |
| `kBodyFont` | `NotoSansKR` | 본문/한국어 |

디자인 스타일: Neumorphic shadows + Glassmorphism (BackdropFilter blur) + Gradient badges

---

## Tasks

### Task 1 — `onDiaryTabTap` 콜백 미전달 버그 수정 [Bug Fix]

**문제**: "내가 사랑한 영화"와 "최근 기록" 섹션의 "전체보기" 버튼이 동작하지 않음.
`HomeContent`는 `onDiaryTabTap` 파라미터를 받지만, `HomeScreen`과 `MainScreen`에서 이 콜백을 전달하는 경로가 없음.

**수정 파일 3개**:

#### 1-1. `lib/screens/home_screen.dart`

`HomeScreen` 위젯에 `onDiaryTabTap` 파라미터 추가:

```dart
// 현재:
class HomeScreen extends StatefulWidget {
  final VoidCallback? onSearchTap;
  const HomeScreen({super.key, this.onSearchTap});

// 변경:
class HomeScreen extends StatefulWidget {
  final VoidCallback? onSearchTap;
  final VoidCallback? onDiaryTabTap;    // ← 추가
  const HomeScreen({super.key, this.onSearchTap, this.onDiaryTabTap});  // ← 추가
```

`HomeContent`에 전달:

```dart
// 현재 (build 메서드 내 FutureBuilder):
return HomeContent(
  data: data,
  onRefresh: refresh,
  onSearchTap: widget.onSearchTap,
);

// 변경:
return HomeContent(
  data: data,
  onRefresh: refresh,
  onSearchTap: widget.onSearchTap,
  onDiaryTabTap: widget.onDiaryTabTap,    // ← 추가
);
```

#### 1-2. `lib/screens/main_screen.dart`

`HomeScreen` 생성 시 콜백 전달:

```dart
// 현재:
HomeScreen(
  key: _homeKey,
  onSearchTap: () => _onItemTapped(1),
),

// 변경:
HomeScreen(
  key: _homeKey,
  onSearchTap: () => _onItemTapped(1),
  onDiaryTabTap: () => _onItemTapped(2),    // ← 추가 (다이어리 탭 = index 2)
),
```

**검증**: 앱 실행 후 홈 화면의 "내가 사랑한 영화 > 전체보기", "최근 기록 > 전체보기" 탭 시 다이어리 탭으로 전환되는지 확인.

---

### Task 2 — 이미지 캐싱 적용 [Performance]

**문제**: 모든 이미지가 `Image.network`로 로드되어 스크롤 시 재요청 발생, 깜빡임 있음.

**패키지 추가**:

```yaml
# pubspec.yaml에 추가:
dependencies:
  cached_network_image: ^3.4.1
```

**수정 파일**: `lib/component/home_content.dart`

상단 import 추가:
```dart
import 'package:cached_network_image/cached_network_image.dart';
```

아래 4곳의 `Image.network`를 `CachedNetworkImage`로 교체:

#### 2-1. Featured Card 배경 이미지 (_buildFeaturedCard 내부, 현재 line 178~186)

```dart
// 현재:
if (bgUrl != null)
  Image.network(
    bgUrl,
    fit: BoxFit.cover,
    filterQuality: FilterQuality.high,
    errorBuilder: (_, __, ___) => Container(color: kSurfaceHigh),
  )
else
  Container(color: kSurfaceHigh),

// 변경:
if (bgUrl != null)
  CachedNetworkImage(
    imageUrl: bgUrl,
    fit: BoxFit.cover,
    filterQuality: FilterQuality.high,
    placeholder: (_, __) => Container(color: kSurfaceHigh),
    errorWidget: (_, __, ___) => Container(color: kSurfaceHigh),
  )
else
  Container(color: kSurfaceHigh),
```

#### 2-2. 가로 스크롤 포스터 (_buildTopRatedMovies 내부, 현재 line 556~563)

```dart
// 현재:
rm.movie.posterUrl != null
    ? Image.network(
        rm.movie.posterUrl!,
        fit: BoxFit.cover,
        filterQuality: FilterQuality.high,
        errorBuilder: (_, __, ___) => Container(color: kSurfaceHigh),
      )
    : Container(color: kSurfaceHigh),

// 변경:
rm.movie.posterUrl != null
    ? CachedNetworkImage(
        imageUrl: rm.movie.posterUrl!,
        fit: BoxFit.cover,
        filterQuality: FilterQuality.high,
        placeholder: (_, __) => Container(color: kSurfaceHigh),
        errorWidget: (_, __, ___) => Container(color: kSurfaceHigh),
      )
    : Container(color: kSurfaceHigh),
```

#### 2-3. 다이어리 카드 포스터 (_buildDiaryCard 내부, 현재 line 858~864)

```dart
// 현재:
child: entry.movie.posterUrl != null
    ? Image.network(
        entry.movie.posterUrl!,
        fit: BoxFit.cover,
        errorBuilder: (_, __, ___) => _posterPlaceholder(),
      )
    : _posterPlaceholder(),

// 변경:
child: entry.movie.posterUrl != null
    ? CachedNetworkImage(
        imageUrl: entry.movie.posterUrl!,
        fit: BoxFit.cover,
        placeholder: (_, __) => _posterPlaceholder(),
        errorWidget: (_, __, ___) => _posterPlaceholder(),
      )
    : _posterPlaceholder(),
```

**검증**: 홈 화면 스크롤 후 다시 위로 돌아갔을 때 이미지가 즉시 표시되는지 확인. 네트워크 탭에서 중복 요청이 없는지 확인.

---

### Task 3 — Computed Getter를 `late final`로 변경 [Performance]

**문제**: `HomeData`의 getter 5개가 접근할 때마다 O(n) 재계산됨. 빌드 중 여러 번 참조되어 불필요한 중복 연산 발생.

**수정 파일**: `lib/data/home_data.dart`

모든 `get` 키워드 프로퍼티를 `late final` 필드로 변환:

```dart
// ===== 현재 코드 =====
int get monthlyCount {
  final now = DateTime.now();
  return recentEntries.where((e) {
    return e.createdAt.year == now.year && e.createdAt.month == now.month;
  }).length;
}

double get averageRating {
  if (recentEntries.isEmpty) return 0;
  return recentEntries.fold<double>(0, (s, e) => s + e.rating) /
      recentEntries.length;
}

String get topGenre {
  if (recentEntries.isEmpty) return '-';
  final counts = <String, int>{};
  for (final e in recentEntries) {
    for (final g in e.movie.genres) {
      if (g.isNotEmpty) counts[g] = (counts[g] ?? 0) + 1;
    }
  }
  if (counts.isEmpty) return '-';
  final sorted = counts.entries.toList()
    ..sort((a, b) => b.value.compareTo(a.value));
  return sorted.first.key;
}

List<RatedMovie> get topRatedMovies {
  final best = <String, RatedMovie>{};
  for (final e in recentEntries) {
    if (e.rating >= 8.0) {
      final existing = best[e.movie.docId];
      if (existing == null || e.rating > existing.rating) {
        best[e.movie.docId] = RatedMovie(movie: e.movie, rating: e.rating);
      }
    }
  }
  final list = best.values.toList()
    ..sort((a, b) => b.rating.compareTo(a.rating));
  return list;
}

Map<int, int> get monthlyHeatmap {
  final now = DateTime.now();
  final map = <int, int>{};
  for (final e in recentEntries) {
    final date = DateTime.tryParse(e.watchedDate);
    if (date != null && date.year == now.year && date.month == now.month) {
      map[date.day] = (map[date.day] ?? 0) + 1;
    }
  }
  return map;
}

// ===== 변경 후 =====
late final int monthlyCount = () {
  final now = DateTime.now();
  return recentEntries.where((e) {
    return e.createdAt.year == now.year && e.createdAt.month == now.month;
  }).length;
}();

late final double averageRating = () {
  if (recentEntries.isEmpty) return 0.0;
  return recentEntries.fold<double>(0, (s, e) => s + e.rating) /
      recentEntries.length;
}();

late final String topGenre = () {
  if (recentEntries.isEmpty) return '-';
  final counts = <String, int>{};
  for (final e in recentEntries) {
    for (final g in e.movie.genres) {
      if (g.isNotEmpty) counts[g] = (counts[g] ?? 0) + 1;
    }
  }
  if (counts.isEmpty) return '-';
  final sorted = counts.entries.toList()
    ..sort((a, b) => b.value.compareTo(a.value));
  return sorted.first.key;
}();

late final List<RatedMovie> topRatedMovies = () {
  final best = <String, RatedMovie>{};
  for (final e in recentEntries) {
    if (e.rating >= 8.0) {
      final existing = best[e.movie.docId];
      if (existing == null || e.rating > existing.rating) {
        best[e.movie.docId] = RatedMovie(movie: e.movie, rating: e.rating);
      }
    }
  }
  final list = best.values.toList()
    ..sort((a, b) => b.rating.compareTo(a.rating));
  return list;
}();

late final Map<int, int> monthlyHeatmap = () {
  final now = DateTime.now();
  final map = <int, int>{};
  for (final e in recentEntries) {
    final date = DateTime.tryParse(e.watchedDate);
    if (date != null && date.year == now.year && date.month == now.month) {
      map[date.day] = (map[date.day] ?? 0) + 1;
    }
  }
  return map;
}();
```

**검증**: 기존과 동일하게 동작하는지 확인. 통계 값이 정확히 같은지 비교.

---

### Task 4 — Shimmer/Skeleton 로딩 UI [UX]

**문제**: 데이터 로딩 중 빈 화면 + `CircularProgressIndicator`만 표시됨.

**패키지 추가**:

```yaml
# pubspec.yaml에 추가:
dependencies:
  shimmer: ^3.0.0
```

**수정 파일**: `lib/screens/home_screen.dart`

`FutureBuilder`의 `ConnectionState.waiting` 분기를 skeleton UI로 교체:

```dart
// 현재:
if (snapshot.connectionState == ConnectionState.waiting) {
  return const Center(child: CircularProgressIndicator());
}

// 변경: skeleton 위젯으로 교체
if (snapshot.connectionState == ConnectionState.waiting) {
  return _buildSkeletonLoading();
}
```

`HomeScreen` 또는 별도 파일에 `_buildSkeletonLoading()` 메서드 추가. 홈 화면의 실제 레이아웃과 동일한 모양의 회색 placeholder를 Shimmer로 감싸서 구현:

```dart
Widget _buildSkeletonLoading() {
  return Shimmer.fromColors(
    baseColor: kSurfaceLow,
    highlightColor: kSurfaceLowest,
    child: SingleChildScrollView(
      padding: const EdgeInsets.fromLTRB(20, 24, 20, 120),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // 인사말 skeleton
          Container(width: 140, height: 14, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(4))),
          const SizedBox(height: 8),
          Container(width: 200, height: 28, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(4))),
          const SizedBox(height: 20),

          // Featured Card skeleton
          Container(height: 220, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(20))),
          const SizedBox(height: 28),

          // 벤토 그리드 skeleton (2×2)
          Row(children: [
            Expanded(child: Container(height: 80, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(16)))),
            const SizedBox(width: 12),
            Expanded(child: Container(height: 80, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(16)))),
          ]),
          const SizedBox(height: 12),
          Row(children: [
            Expanded(child: Container(height: 80, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(16)))),
            const SizedBox(width: 12),
            Expanded(child: Container(height: 80, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(16)))),
          ]),
          const SizedBox(height: 32),

          // 가로 스크롤 skeleton
          Container(width: 120, height: 18, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(4))),
          const SizedBox(height: 14),
          SizedBox(
            height: 200,
            child: Row(children: List.generate(3, (i) => Padding(
              padding: const EdgeInsets.only(right: 14),
              child: Container(width: 120, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(14))),
            ))),
          ),
          const SizedBox(height: 32),

          // 캘린더 skeleton
          Container(height: 200, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(20))),
          const SizedBox(height: 32),

          // 다이어리 카드 skeleton × 3
          ...List.generate(3, (_) => Padding(
            padding: const EdgeInsets.only(bottom: 14),
            child: Container(height: 100, decoration: BoxDecoration(color: Colors.white, borderRadius: BorderRadius.circular(18))),
          )),
        ],
      ),
    ),
  );
}
```

**검증**: 앱 실행 시 데이터 로딩 중 shimmer 애니메이션이 홈 화면 레이아웃 형태로 표시되는지 확인.

---

### Task 5 — 에러 상태 개선 (리트라이 버튼) [UX]

**문제**: 에러 시 텍스트만 표시되고 재시도 방법이 없음.

**수정 파일**: `lib/screens/home_screen.dart`

```dart
// 현재:
} else if (snapshot.hasError) {
  return const Center(child: Text('데이터를 불러오지 못했습니다.'));
}

// 변경:
} else if (snapshot.hasError) {
  return Center(
    child: Padding(
      padding: const EdgeInsets.all(40),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Container(
            width: 64,
            height: 64,
            decoration: const BoxDecoration(
              color: kSurfaceHigh,
              shape: BoxShape.circle,
            ),
            child: const Icon(
              Icons.wifi_off_rounded,
              size: 30,
              color: kOnSurfaceVariant,
            ),
          ),
          const SizedBox(height: 16),
          const Text(
            '데이터를 불러오지 못했습니다.',
            style: TextStyle(
              fontFamily: kBodyFont,
              fontSize: 14,
              color: kOnSurfaceVariant,
            ),
          ),
          const SizedBox(height: 8),
          Text(
            '네트워크 연결을 확인해주세요.',
            style: TextStyle(
              fontFamily: kBodyFont,
              fontSize: 12,
              color: kOnSurfaceVariant.withValues(alpha: 0.6),
            ),
          ),
          const SizedBox(height: 20),
          GestureDetector(
            onTap: refresh,
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
              decoration: BoxDecoration(
                gradient: kPrimaryGradient,
                borderRadius: BorderRadius.circular(12),
                boxShadow: [
                  BoxShadow(
                    color: kPrimary.withValues(alpha: 0.25),
                    blurRadius: 8,
                    offset: const Offset(0, 4),
                  ),
                ],
              ),
              child: const Text(
                '다시 시도',
                style: TextStyle(
                  fontFamily: kHeadlineFont,
                  fontSize: 14,
                  fontWeight: FontWeight.w700,
                  color: Colors.white,
                ),
              ),
            ),
          ),
        ],
      ),
    ),
  );
}
```

**검증**: 네트워크 끊긴 상태에서 앱 실행 → 에러 UI 표시 → "다시 시도" 탭 → 데이터 재로딩 확인.

---

### Task 6 — 시간대별 인사말 분기 [UX]

**문제**: 항상 "안녕하세요"로 고정.

**수정 파일**: `lib/component/home_content.dart` — `_buildGreeting()` 메서드

```dart
// 현재:
Text(
  '안녕하세요, ${widget.data.user.nickname}님!',
  ...
),

// 변경:
Text(
  '${_getGreeting()}, ${widget.data.user.nickname}님!',
  ...
),

// 클래스 내에 헬퍼 메서드 추가:
String _getGreeting() {
  final hour = DateTime.now().hour;
  if (hour < 6) return '늦은 밤이에요';
  if (hour < 12) return '좋은 아침이에요';
  if (hour < 18) return '안녕하세요';
  return '좋은 저녁이에요';
}
```

**검증**: 시간대를 변경하며 인사말이 바뀌는지 확인 (또는 코드의 hour 값을 임시로 바꿔서 테스트).

---

### Task 7 — Featured Card 빈 상태 CTA 문구 수정 [UX]

**문제**: "첫 번째 다이어리를 작성해보세요" 라고 하면서 탭 시 검색 탭으로 이동. 문구와 동작 불일치.

**수정 파일**: `lib/component/home_content.dart` — `_buildFeaturedEmptyCTA()` 메서드

```dart
// 현재:
const Text(
  '첫 번째 다이어리를 작성해보세요',
  ...
),
const SizedBox(height: 4),
Text(
  '영화를 검색하고 감상을 기록하세요',
  ...
),

// 변경:
const Text(
  '어떤 영화를 보셨나요?',
  ...
),
const SizedBox(height: 4),
Text(
  '영화를 검색하고 첫 기록을 남겨보세요',
  ...
),
```

아이콘도 검색에 맞게 변경:

```dart
// 현재:
const Icon(Icons.movie_creation_outlined, color: Colors.white, size: 36),

// 변경:
const Icon(Icons.search_rounded, color: Colors.white, size: 36),
```

**검증**: 다이어리가 0개인 계정으로 로그인 시 CTA 문구와 아이콘 확인.

---

### Task 8 — 평점 표기 통일 [UX Consistency]

**문제**: Featured Card와 가로 스크롤에서는 10점 만점 숫자(예: "9.0")로 표시, 다이어리 카드에서는 5개 별 아이콘(★★★★☆)으로 표시. 같은 화면에서 동일 영화가 다르게 보일 수 있음.

**수정 파일**: `lib/component/home_content.dart` — `_buildDiaryCard()` 메서드

다이어리 카드의 별 5개 시스템을 제거하고 숫자 표기로 통일:

```dart
// 현재 (line 901~914): 별 5개 아이콘 시스템
Row(
  children: [
    ...List.generate(5, (i) {
      final v = entry.rating / 2;
      if (v >= i + 1) {
        return const Icon(Icons.star_rounded, color: kPrimary, size: 13);
      } else if (v > i) {
        return const Icon(Icons.star_half_rounded, color: kPrimary, size: 13);
      }
      return Icon(Icons.star_outline_rounded, color: kSurfaceDim, size: 13);
    }),
    const SizedBox(width: 6),
    Text(
      entry.watchedDate,
      ...
    ),
  ],
),

// 변경: 별 1개 + 숫자 + 날짜
Row(
  children: [
    const Icon(Icons.star_rounded, color: kPrimary, size: 14),
    const SizedBox(width: 2),
    Text(
      entry.rating.toStringAsFixed(1),
      style: const TextStyle(
        fontFamily: kHeadlineFont,
        fontSize: 13,
        fontWeight: FontWeight.w700,
        color: kOnSurface,
      ),
    ),
    const SizedBox(width: 8),
    Text(
      entry.watchedDate,
      style: TextStyle(
        fontFamily: kBodyFont,
        fontSize: 10,
        color: kOnSurfaceVariant.withValues(alpha: 0.5),
      ),
    ),
  ],
),
```

**검증**: 다이어리 카드의 평점 표기가 "★ 9.0" 형식으로 통일되는지 확인.

---

### Task 9 — 캘린더 히트맵 날짜 탭 인터랙션 [Feature]

**문제**: 캘린더 셀이 정보 표시만 하고 인터랙션이 없음.

**수정 파일**: `lib/component/home_content.dart` — `_buildCalendarGrid()` 메서드

각 날짜 셀을 `GestureDetector`로 감싸고, 탭 시 해당 날짜의 다이어리를 `showModalBottomSheet`로 표시:

```dart
// _buildCalendarGrid 내부, 날짜 셀 생성 부분:
// 현재의 Container를 GestureDetector로 감싸기:

cells.add(
  GestureDetector(
    onTap: count > 0 ? () => _showDayEntries(context, day) : null,
    child: Container(
      margin: const EdgeInsets.all(2),
      decoration: BoxDecoration(
        color: bgColor,
        borderRadius: BorderRadius.circular(8),
        border: isToday ? Border.all(color: kPrimary, width: 2) : null,
      ),
      child: Center(
        child: Text('$day', style: TextStyle(...)),
      ),
    ),
  ),
);
```

**주의**: `_buildCalendarGrid`는 현재 context를 파라미터로 받지 않으므로, 파라미터에 `BuildContext context`를 추가하거나 클래스 레벨의 context를 사용해야 함.

호출부도 수정:
```dart
// 현재:
_buildCalendarGrid(daysInMonth, firstWeekday, heatmap, now.day),

// 변경:
_buildCalendarGrid(context, daysInMonth, firstWeekday, heatmap, now.day),
```

메서드 시그니처 변경:
```dart
// 현재:
Widget _buildCalendarGrid(int daysInMonth, int firstWeekday, Map<int, int> heatmap, int today)

// 변경:
Widget _buildCalendarGrid(BuildContext context, int daysInMonth, int firstWeekday, Map<int, int> heatmap, int today)
```

새 메서드 추가:

```dart
void _showDayEntries(BuildContext context, int day) {
  final now = DateTime.now();
  final dayEntries = widget.data.recentEntries.where((e) {
    final date = DateTime.tryParse(e.watchedDate);
    return date != null &&
        date.year == now.year &&
        date.month == now.month &&
        date.day == day;
  }).toList();

  if (dayEntries.isEmpty) return;

  showModalBottomSheet(
    context: context,
    backgroundColor: Colors.transparent,
    builder: (_) => Container(
      decoration: const BoxDecoration(
        color: kSurfaceLowest,
        borderRadius: BorderRadius.vertical(top: Radius.circular(24)),
      ),
      padding: const EdgeInsets.fromLTRB(20, 16, 20, 32),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // 드래그 핸들
          Center(
            child: Container(
              width: 40, height: 4,
              decoration: BoxDecoration(
                color: kSurfaceDim,
                borderRadius: BorderRadius.circular(2),
              ),
            ),
          ),
          const SizedBox(height: 16),
          // 날짜 헤더
          Text(
            '${now.month}월 ${day}일 관람 기록',
            style: const TextStyle(
              fontFamily: kHeadlineFont,
              fontSize: 18,
              fontWeight: FontWeight.w700,
              color: kOnSurface,
            ),
          ),
          const SizedBox(height: 16),
          // 다이어리 목록
          ...dayEntries.map((e) => _buildDiaryCard(context, e)),
        ],
      ),
    ),
  );
}
```

**검증**: 관람 기록이 있는 날짜 셀 탭 → BottomSheet에 해당 날짜 다이어리 표시 확인. 기록 없는 날짜는 탭 무시 확인.

---

### Task 10 — Hero 애니메이션 트랜지션 [Animation]

**문제**: 포스터/스틸컷을 탭해서 상세 화면으로 이동할 때 화면이 딱딱하게 전환됨. Featured Card, 가로 스크롤 포스터, 다이어리 카드 포스터가 모두 `Navigator.push`만 사용하고 전환 애니메이션이 없음.

**수정 파일**: `lib/component/home_content.dart`

#### 10-1. Featured Card → 다이어리 편집 화면

포스터/스틸컷 이미지에 `Hero` 위젯 감싸기:

```dart
// _buildFeaturedCard 내부, 배경 이미지 (현재 CachedNetworkImage):
// CachedNetworkImage를 Hero로 감싸기

Hero(
  tag: 'diary-hero-${entry.id}',
  child: CachedNetworkImage(
    imageUrl: bgUrl,
    fit: BoxFit.cover,
    filterQuality: FilterQuality.high,
    placeholder: (_, __) => Container(color: kSurfaceHigh),
    errorWidget: (_, __, ___) => Container(color: kSurfaceHigh),
  ),
),
```

#### 10-2. 가로 스크롤 포스터 → MovieDetailScreen

```dart
// _buildTopRatedMovies 내부, 포스터 이미지:

Hero(
  tag: 'movie-poster-${rm.movie.docId}',
  child: ClipRRect(
    borderRadius: BorderRadius.circular(14),
    child: CachedNetworkImage(
      imageUrl: rm.movie.posterUrl!,
      fit: BoxFit.cover,
      filterQuality: FilterQuality.high,
      placeholder: (_, __) => Container(color: kSurfaceHigh),
      errorWidget: (_, __, ___) => Container(color: kSurfaceHigh),
    ),
  ),
),
```

#### 10-3. 다이어리 카드 포스터 → 상세 화면

```dart
// _buildDiaryCard 내부, 포스터 이미지:

Hero(
  tag: 'diary-poster-${entry.id}',
  child: ClipRRect(
    borderRadius: BorderRadius.circular(10),
    child: CachedNetworkImage(
      imageUrl: entry.movie.posterUrl!,
      fit: BoxFit.cover,
      placeholder: (_, __) => _posterPlaceholder(),
      errorWidget: (_, __, ___) => _posterPlaceholder(),
    ),
  ),
),
```

**주의**: 도착 화면(`MovieDetailScreen`, `DiaryWriteScreen`)에도 동일한 `Hero` tag를 가진 위젯이 있어야 함. 해당 화면의 포스터/이미지에도 같은 tag를 적용해야 애니메이션이 동작함.

도착 화면 수정 예시:
```dart
// movie_detail_screen.dart 포스터 부분:
Hero(
  tag: 'movie-poster-${movie.docId}',
  child: CachedNetworkImage(imageUrl: movie.posterUrl!, ...),
),
```

**검증**: 포스터 탭 시 이미지가 자연스럽게 확대/이동하며 상세 화면으로 전환되는지 확인. 뒤로 가기 시 이미지가 원래 위치로 돌아오는지 확인.

---

### Task 11 — 디자인 토큰 통일 (스페이싱 + Border Radius) [Design Consistency]

**문제**: 섹션 간 간격과 Border Radius가 제각각이라 시각적으로 불통일.

현재 상태:
```
섹션 간 간격:  20pt, 28pt, 32pt, 14pt 혼재
Border Radius: 20, 18, 16, 14, 8 혼재
컴포넌트 내부 패딩: 12, 14, 16, 20 혼재
```

**수정 파일 2개**:

#### 11-1. `lib/constants.dart` — 스페이싱/레디어스 토큰 추가

```dart
// constants.dart 하단에 추가:

// ── Spacing Tokens ──
const double kSpacingXS = 4;
const double kSpacingS = 8;
const double kSpacingM = 12;
const double kSpacingL = 16;
const double kSpacingXL = 20;
const double kSpacingXXL = 24;
const double kSpacing3XL = 32;
const double kSpacingNav = 120;    // 하단 네비게이션 여백

// ── Border Radius Tokens ──
const double kRadiusS = 8;        // 뱃지, 히트맵 셀
const double kRadiusM = 12;       // 버튼
const double kRadiusL = 16;       // 통계 카드
const double kRadiusXL = 20;      // 대형 카드 (Featured, 캘린더)
```

#### 11-2. `lib/component/home_content.dart` — 하드코딩된 값을 토큰으로 교체

주요 교체 목록:

```dart
// ── 섹션 간 간격 통일 (모두 kSpacing3XL = 32pt로) ──

// Greeting → Featured Card 간격
// 현재: EdgeInsets.fromLTRB(20, 20, 20, 0)
// 변경: EdgeInsets.fromLTRB(kSpacingXL, kSpacingXXL, kSpacingXL, 0)

// Featured Card → Stats 간격
// 현재: EdgeInsets.fromLTRB(20, 28, 20, 0)
// 변경: EdgeInsets.fromLTRB(kSpacingXL, kSpacing3XL, kSpacingXL, 0)

// Stats → Top Rated 간격
// 현재: EdgeInsets.fromLTRB(20, 32, 20, 0)
// 변경: EdgeInsets.fromLTRB(kSpacingXL, kSpacing3XL, kSpacingXL, 0)

// 섹션 헤더 → 콘텐츠 간격 통일
// 현재: SizedBox(height: 14) 또는 SizedBox(height: 20) 혼재
// 변경: 모두 SizedBox(height: kSpacingL)  // 16pt

// ── Border Radius 통일 ──

// Featured Card
// 현재: BorderRadius.circular(20)
// 변경: BorderRadius.circular(kRadiusXL)

// Stat Cards
// 현재: BorderRadius.circular(16)
// 변경: BorderRadius.circular(kRadiusL)

// Top-rated 포스터
// 현재: BorderRadius.circular(14)
// 변경: BorderRadius.circular(kRadiusL)  // 16으로 올림

// Calendar heatmap 컨테이너
// 현재: BorderRadius.circular(20)
// 변경: BorderRadius.circular(kRadiusXL)

// Diary Card
// 현재: BorderRadius.circular(18)
// 변경: BorderRadius.circular(kRadiusXL)  // 20으로 통일

// Calendar cell
// 현재: BorderRadius.circular(8)
// 변경: BorderRadius.circular(kRadiusS)
```

**검증**: 모든 섹션 간 간격이 균일한지 시각적으로 확인. 카드 모서리 곡률이 일관된지 확인.

---

### Task 12 — home_content.dart 파일 분리 [Code Quality]

**문제**: 단일 파일 1,060줄. 유지보수 어려움.

**작업**: `lib/component/home_content.dart`에서 각 섹션을 별도 위젯 파일로 추출.

분리 후 구조:
```
lib/component/
├── home_content.dart              # CustomScrollView + 섹션 조합 (약 120줄)
├── home_featured_card.dart        # _buildFeaturedCard + _buildFeaturedEmptyCTA
├── home_stats_bento.dart          # _buildStatsBento + _statCard
├── home_top_rated_movies.dart     # _buildTopRatedMovies
├── home_calendar_heatmap.dart     # _buildCalendarHeatmap + _buildCalendarGrid + _showDayEntries
├── home_diary_card.dart           # _buildDiaryCard + _posterPlaceholder
└── home_empty_state.dart          # _buildEmptyState
```

각 파일은 `StatelessWidget`으로 추출하되, 필요한 데이터와 콜백을 파라미터로 전달:

```dart
// 예시: home_featured_card.dart
class HomeFeaturedCard extends StatelessWidget {
  final DiaryEntry? latestEntry;
  final VoidCallback? onSearchTap;
  final VoidCallback? onRefresh;

  const HomeFeaturedCard({
    super.key,
    this.latestEntry,
    this.onSearchTap,
    this.onRefresh,
  });

  @override
  Widget build(BuildContext context) { ... }
}
```

```dart
// home_content.dart는 조합만 담당:
class HomeContent extends StatelessWidget {
  final HomeData data;
  final VoidCallback? onRefresh;
  final VoidCallback? onSearchTap;
  final VoidCallback? onDiaryTabTap;

  @override
  Widget build(BuildContext context) {
    return RefreshIndicator(
      onRefresh: () async => onRefresh?.call(),
      child: CustomScrollView(
        slivers: [
          SliverToBoxAdapter(child: HomeGreeting(nickname: data.user.nickname)),
          SliverToBoxAdapter(child: HomeFeaturedCard(latestEntry: data.recentEntries.isNotEmpty ? data.recentEntries.first : null, ...)),
          SliverToBoxAdapter(child: HomeStatsBento(data: data)),
          if (data.topRatedMovies.isNotEmpty)
            SliverToBoxAdapter(child: HomeTopRatedMovies(movies: data.topRatedMovies)),
          SliverToBoxAdapter(child: HomeCalendarHeatmap(heatmap: data.monthlyHeatmap, entries: data.recentEntries)),
          SliverToBoxAdapter(child: HomeRecentDiaries(entries: data.recentEntries, onDiaryTabTap: onDiaryTabTap)),
        ],
      ),
    );
  }
}
```

**주의**: 이 태스크는 다른 모든 태스크가 완료된 후 마지막에 진행하는 것을 권장. 다른 태스크들이 `home_content.dart`를 직접 수정하므로, 파일 분리를 먼저 하면 나머지 태스크의 수정 위치가 달라짐.

**검증**: 파일 분리 후 앱이 동일하게 동작하는지 전체 기능 확인. Hot reload 정상 동작 확인.

---

## Execution Order

```
Phase 1 — Bug Fix & Core
  Task 1   (onDiaryTabTap 버그)      ← 가장 먼저, 기능이 깨져있음
  Task 3   (late final 변환)          ← 단순 리팩터링, 빠르게 가능

Phase 2 — Performance
  Task 2   (이미지 캐싱)              ← 패키지 추가 필요, 체감 성능 큼
  Task 4   (Shimmer 로딩)            ← 패키지 추가, 별도 위젯 작성

Phase 3 — UX Polish
  Task 5   (에러 상태 리트라이)        ← 작은 변경, 큰 UX 개선
  Task 6   (시간대별 인사말)           ← 1분 작업
  Task 7   (빈 상태 CTA 문구)         ← 1분 작업
  Task 8   (평점 표기 통일)           ← 작은 변경

Phase 4 — Feature & Visual
  Task 9   (캘린더 인터랙션)          ← 새 기능 추가
  Task 10  (Hero 애니메이션)          ← 화면 전환 개선
  Task 11  (디자인 토큰 통일)         ← 전체 일관성 작업

Phase 5 — Refactor (반드시 마지막)
  Task 12  (파일 분리)               ← 다른 태스크 모두 완료 후
```

---

## Packages to Add

```yaml
# pubspec.yaml → dependencies 섹션에 추가:
cached_network_image: ^3.4.1    # Task 2
shimmer: ^3.0.0                 # Task 4
```

`flutter pub get` 실행 필요.

---

## Files Modified (Summary)

| File | Tasks |
|------|-------|
| `lib/screens/main_screen.dart` | 1 |
| `lib/screens/home_screen.dart` | 1, 4, 5 |
| `lib/component/home_content.dart` | 2, 6, 7, 8, 9, 10, 11, 12 |
| `lib/data/home_data.dart` | 3 |
| `lib/constants.dart` | 11 |
| `lib/screens/movie_detail_screen.dart` | 10 (Hero tag 수신측) |
| `lib/screens/diary_write_screen.dart` | 10 (Hero tag 수신측) |
| `pubspec.yaml` | 2, 4 |
