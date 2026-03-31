# Movie Diary — 프로젝트 분석 및 개선 TODO

> **Project**: Movie Diary (Flutter + NestJS + MySQL)
> **분석일**: 2026-03-28
> **목적**: 프로젝트 전반의 부족한 부분 파악 및 보완 계획 수립

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

## 10. 내비게이션 구조 개선 (Navigation Architecture) — 🔴 긴급

### 10-1. 중첩 내비게이터(Nested Navigator) 도입
- **문제 원인**: 현재 `Navigator.push(context, ...)` 호출 시 루트 내비게이터를 사용하게 되어 `MainScreen` 전체(AppBar, BottomBar 포함)가 새로운 화면으로 덮임.
- **해결 방안**: 각 탭마다 독립적인 내비게이션 스택을 부여하여 탭 내부에서 화면 전환이 일어나도록 변경.

#### 상세 구현 설계 (LLM 구현 가이드)

1. **`TabNavigator` 위젯 생성**:
   - 각 탭을 감싸는 `Navigator` 전용 위젯을 정의합니다.
   - `GlobalKey<NavigatorState>`를 사용하여 각 탭의 상태를 고유하게 관리합니다.
   ```dart
   class TabNavigator extends StatelessWidget {
     final GlobalKey<NavigatorState> navigatorKey;
     final Widget rootPage;

     TabNavigator({required this.navigatorKey, required this.rootPage});

     @override
     Widget build(BuildContext context) {
       return Navigator(
         key: navigatorKey,
         onGenerateRoute: (routeSettings) {
           return MaterialPageRoute(builder: (context) => rootPage);
         },
       );
     }
   }
   ```

2. **`MainScreen` (`main_screen.dart`) 구조 변경**:
   - 4개의 `GlobalKey<NavigatorState>`를 리스트로 관리합니다.
   - `IndexedStack`의 자식들을 `TabNavigator`로 교체합니다.
   ```dart
   final List<GlobalKey<NavigatorState>> _navigatorKeys = [
     GlobalKey<NavigatorState>(),
     GlobalKey<NavigatorState>(),
     GlobalKey<NavigatorState>(),
     GlobalKey<NavigatorState>(),
   ];

   // IndexedStack 내부 예시
   IndexedStack(
     index: _selectedIndex,
     children: [
       TabNavigator(navigatorKey: _navigatorKeys[0], rootPage: HomeScreen(...)),
       TabNavigator(navigatorKey: _navigatorKeys[1], rootPage: MovieSearchScreen(...)),
       TabNavigator(navigatorKey: _navigatorKeys[2], rootPage: MyDiaryListScreen(...)),
       TabNavigator(navigatorKey: _navigatorKeys[3], rootPage: MyPageScreen(...)),
     ],
   )
   ```

3. **시스템 뒤로가기 처리 (`PopScope`)**:
   - 사용자가 안드로이드 뒤로가기 버튼이나 스와이프 제스처를 할 때, 현재 탭의 스택이 비어있지 않으면 탭 내부에서 `pop`을 수행하도록 처리합니다.
   ```dart
   return PopScope(
     canPop: false,
     onPopInvokedWithResult: (didPop, result) async {
       if (didPop) return;
       final NavigatorState? currentNavigator = _navigatorKeys[_selectedIndex].currentState;
       if (currentNavigator != null && currentNavigator.canPop()) {
         currentNavigator.pop();
       } else {
         // 더 이상 pop 할 스택이 없으면 앱 종료 또는 홈 탭으로 이동 로직
       }
     },
     child: Scaffold(...),
   );
   ```

4. **화면별 UI 보완**:
   - **`MyDiaryListScreen`**: 현재 상단 바가 없으므로 `CustomAppBar` 스타일의 `AppBar`를 추가하거나, 내비게이션 바와 일관된 뒤로가기 버튼을 포함하도록 수정합니다.
   - **`AccountSettingsScreen`, `PersonalDiaryScreen`**: `Navigator.push` 호출 시 `context`가 해당 탭의 `TabNavigator` 하위에 있으므로 자연스럽게 탭 내부에서 전환됩니다.

5. **기대 효과**:
   - 프로필 메뉴에서 '계정 설정' 등으로 이동해도 하단 내비게이션 바가 고정되어 있어 다른 탭으로 즉시 이동이 가능해집니다.
   - 앱의 전체적인 흐름이 더 매끄러워지고 네이티브 앱과 유사한 사용자 경험을 제공합니다.

---

## 우선순위 요약 (남은 작업)

| 순위 | 항목 | 긴급도 | 난이도 |
|------|------|--------|--------|
| **1** | **중첩 내비게이터(Nested Navigator) 도입** | 🔴 긴급 | 중 |
| 2 | 다크 모드 지원 | 🟢 개선 | 중 |
| 3 | 푸시 알림 기능 | 🟢 개선 | 중 |
| 4 | 소셜 기능 (팔로우/피드) | 🟢 개선 | 상 |
