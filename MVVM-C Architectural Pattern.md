# MVVM-C (Model-View-ViewModel-Coordinator) Architecture

## Why MVVM-C?
- **MVC:** too coarse, logic leaks into widgets as app grows → ❌
- **MVP:** ok, but Presenters get bloated; navigation unclear → ❌
- **MVVM:** good, but navigation/role guards end up in views → ⚠️
- **MVVM-C:** best balance—clean ViewModels + clean navigation/guards → ✅ Pick this
- **VIPER:** iOS-centric & ceremony heavy for Flutter → ❌

## Why we use MVVM-C
- Keeps your app organized and scalable
- Makes testing easier
- Keeps navigation logic separate (important for Admin/User split)
- Works perfectly with `Riverpod` + `GoRouter`

## What is MVVM-C?
MVVM-C stands for:<br>
```dart
Model – View – ViewModel – Coordinator
```
It’s an improved version of MVVM, where we add a Coordinator to manage navigation and screen flow (instead of writing navigation in the widget itself).
## Purpose of Each Part
- Model	
    - Data layer (Holds data, talks to APIs or local DB)
- View
    - UI layer (Shows visuals and listens to ViewModel (Widget/Page))
- ViewModel
    - Logic layer (Handles state, updates UI through Riverpod (Notifier/AsyncNotifier))
- Coordinator
    - Navigation layer (Decides which screen opens next (GoRouter/helper class))
## `lib/` Structure
```dart
lib/
├─ main.dart                    // 🚀 App entry point
│                               // Initializes Flutter + Riverpod ProviderScope
│                               // Calls runApp(App()) to start the application
│
├─ app/                         // ⚙️ App-level setup (theme, routes, constants)
│  ├─ app.dart                  // Root widget → MaterialApp.router
│  │                            // Connects GoRouter + Light/Dark themes
│  │
│  ├─ router/                   // 🌐 Navigation system (Coordinator + Guards + Shells)
│  │  ├─ app_router.dart        // Central GoRouter setup as a Riverpod provider
│  │  │                         // Defines all routes (User/Admin/Login)
│  │  │                         // Can refresh automatically when auth changes
│  │  │
│  │  ├─ guards/                // 🔒 Route protection & redirection
│  │  │  └─ auth_guard.dart     // Redirects users to login if not authenticated
│  │  │                         // Also checks user roles (user/admin)
│  │  │
│  │  ├─ shells/                // 🧱 Different app layouts for roles
│  │  │  ├─ user_shell.dart     // Layout for all /u/* pages (AppBar/Nav/Drawer)
│  │  │  └─ admin_shell.dart    // Layout for all /admin/* pages (AppBar/Nav/Drawer)
│  │  │                         // Keeps UI consistent per role
│  │  │
│  │  ├─ user_coordinator.dart  // 🧭 Handles all User-side navigation
│  │  │                         // Example: nav.openProfile() → '/u/profile'
│  │  │                         // Widgets & Controllers never call context.go directly
│  │  │
│  │  └─ admin_coordinator.dart // 🧭 Handles all Admin-side navigation
│  │                            // Example: nav.openDashboard() → '/admin/dashboard'
│  │
│  ├─ themes/                   // 🎨 Global visual themes
│  │  ├─ light_theme.dart       // Light mode ThemeData
│  │  └─ dark_theme.dart        // Dark mode ThemeData
│  │
│  └─ constants/                // 🧩 App-wide constants
│     ├─ colors.dart            // Brand colors (primary, secondary, etc.)
│     ├─ sizes.dart             // Reusable spacing, font sizes, corner radius
│     └─ strings.dart           // Static text or labels used across screens
│
├─ core/                        // 🧠 Core layer → shared logic for entire app
│  ├─ network/                  // 🌐 Networking layer (API setup)
│  │  ├─ api_client.dart        // Dio instance + interceptors (Dio is an HTTP client for Dart that makes it 
│  │  │                         // easy to work with APIs and perform HTTP requests in Flutter)
│  │  │                         // Manages headers, base URLs, logging
│  │  └─ endpoints.dart         // Central file for all REST API paths
│  │
│  ├─ auth/                     // 🔐 Authentication layer
│  │  ├─ auth_service.dart      // Wrapper around Firebase/Auth SDKs
│  │  │                         // Handles login, logout, token refresh
│  │  └─ auth_provider.dart     // Riverpod providers (authState, currentUser, role)
│  │
│  ├─ error/                    // ❌ Centralized error handling
│  │  ├─ app_exception.dart     // Defines AppException (custom error type)
│  │  └─ error_mapper.dart      // Maps Dio/HTTP errors → readable AppException
│  │
│  ├─ extensions/               // 🧩 Dart/Flutter extensions
│  │  └─ string_extensions.dart // Adds helpers like capitalize(), isEmail(), etc.
│  │
│  └─ utils/                    // 🧰 Helper utilities
│     ├─ validators.dart        // Input validators (email/password)
│     ├─ date_format.dart       // Common date/time formatters
│     └─ logger.dart            // Logging helper (for debugging)
│
├─ data/                        // 📊 Local mock/seed data (optional for testing)
│  ├─ alphabet_data.dart        // Example dummy data
│  ├─ user/user_data.dart       // Mock user info for quick testing
│  └─ admin/admin_data.dart     // Mock admin info
│
├─ models/                      // 📦 Shared global data models (used across app)
│  ├─ user_model.dart           // Represents a user object (id, name, role)
│  └─ alphabet_model.dart       // Represents an alphabet or content model
│
├─ features/                    // 🧩 Real app features (each follows MVVM pattern)
│  ├─ user/                     // 👤 All user-side features
│  │  ├─ home/                  // User Home module
│  │  │  ├─ data/               // 💾 Handles fetching/saving data
│  │  │  │  └─ home_repository.dart // Repository (connects to core/network)
│  │  │  │                      // Example: fetchUserStats() → API call
│  │  │  │
│  │  │  ├─ application/        // 🧠 Logic & State layer (ViewModel)
│  │  │  │  └─ home_controller.dart // Riverpod Notifier (fetch + expose UI state)
│  │  │  │                         // Uses repository to get data
│  │  │  │
│  │  │  └─ presentation/       // 🎨 UI (Widgets, Screens)
│  │  │     ├─ home_screen.dart // Builds the actual Home screen layout
│  │  │     │                   // Watches controller for state updates
│  │  │     └─ widgets/         // Small reusable UI parts for this feature only
│  │  │        └─ stats_card.dart // Custom card widget to show user stats
│  │  │
│  │  ├─ login/                 // User Login module
│  │  │  ├─ data/               // Login repository (API call, local auth)
│  │  │  ├─ application/        // Login controller (handles loading/errors)
│  │  │  │  └─ login_controller.dart
│  │  │  └─ presentation/       // Login UI (form + buttons)
│  │  │     ├─ login_screen.dart
│  │  │     └─ widgets/login_form.dart
│  │  │
│  │  └─ dashboard/             // Other user features (same MVVM pattern)
│  │     ├─ data/
│  │     ├─ application/
│  │     └─ presentation/
│  │
│  └─ admin/                    // 🧑‍💼 All admin-side features
│     ├─ home/                  // Admin Home (same 3-layer structure)
│     ├─ login/                 // Admin Login module
│     ├─ dashboard/             // Admin Dashboard
│     └─ progress/              // Admin Progress feature
│        ├─ data/               // Repository to fetch progress stats
│        ├─ application/        // Controller for progress data/state
│        │  └─ progress_controller.dart
│        └─ presentation/       // UI for progress tracking
│           ├─ progress_screen.dart
│           └─ widgets/progress_tile.dart
│
├─ services/                    // 🌍 Third-party integrations
│  ├─ firebase_service.dart     // Firebase helper (init, analytics, etc.)
│  └─ local_db_service.dart     // Local database helper (Hive, SQLite)
│
├─ widgets/                     // 💎 Global reusable UI widgets (Design system)
│  ├─ custom_button.dart        // Common button used across screens
│  ├─ custom_textfield.dart     // Styled text field widget
│  └─ app_logo.dart             // Central app logo widget
│
└─ config/                      // ⚙️ Environment configuration
   ├─ env.dart                  // Stores env variables (baseURL, keys)
   └─ app_config.dart           // Reads env + sets correct config for app
```
## How the whole app fits together
```scss
UI (presentation) ─┐   calls     ┌─> Controller (application / Riverpod)
                   └─────────────┘
                                     calls
Controller (application) ──────────────────────> Repository (data)
                                                     │
                                                     ▼
                                                 API Client (core/network)

Navigation:
UI/Controller ──> Coordinator (app/router/_*_coordinator.dart) ──> GoRouter
Auth/Role:
GoRouter ──> Guard (app/router/guards/auth_guard.dart) ──> core/auth providers
```
## `./app`
### `app/app.dart` (root widget)
- Connects themes and the router to `MaterialApp.router`
- Reads the router from a Riverpod provider so the rest of the app can navigate without context.
    ```dart
    import 'package:flutter/material.dart';
    import 'package:flutter_riverpod/flutter_riverpod.dart';
    import 'router/app_router.dart';
    import 'themes/light_theme.dart';
    import 'themes/dark_theme.dart';

    class App extends ConsumerWidget {
    const App({super.key});
    @override
    Widget build(BuildContext context, WidgetRef ref) {
        final router = ref.watch(appRouterProvider);
        return MaterialApp.router(
        title: 'User/Admin App',
        theme: lightTheme,
        darkTheme: darkTheme,
        routerConfig: router,
        );
    }
    }
    ```
### `app/router/app_router.dart` (central navigation)
- Creates one GoRouter, exposed as a provider.
- Includes ShellRoutes for user and admin layouts.
- Uses an auth guard to redirect by role.
    ```dart
    import 'package:flutter_riverpod/flutter_riverpod.dart';
    import 'package:go_router/go_router.dart';
    import 'guards/auth_guard.dart';
    import 'shells/user_shell.dart';
    import 'shells/admin_shell.dart';
    import '../../core/auth/auth_provider.dart';

    // Example screens
    import '../../features/user/home/presentation/home_screen.dart';
    import '../../features/user/login/presentation/login_screen.dart';
    import '../../features/admin/dashboard/presentation/dashboard_screen.dart';

    final appRouterProvider = Provider<GoRouter>((ref) {
    // Optional: auto-refresh router when auth changes
    final authListenable =
        GoRouterRefreshStream(ref.watch(authStateProvider).asStream());

    return GoRouter(
        initialLocation: '/u',
        refreshListenable: authListenable,
        redirect: (ctx, state) => authRedirect(ref, state.matchedLocation),
        routes: [
        GoRoute(path: '/login', name: 'login', builder: (_, __) => const LoginScreen()),

        // USER SPACE
        ShellRoute(
            builder: (_, __, child) => UserShell(child: child),
            routes: [
            GoRoute(path: '/u',       name: 'userHome',   builder: (_, __) => const UserHomeScreen()),
            // add more user routes here…
            ],
        ),

        // ADMIN SPACE
        ShellRoute(
            builder: (_, __, child) => AdminShell(child: child),
            routes: [
            GoRoute(path: '/admin',   name: 'adminDash',  builder: (_, __) => const AdminDashboardScreen()),
            // add more admin routes here…
            ],
        ),
        ],
    );
    });
    ```
### `app/router/guards/auth_guard.dart` (role & login redirects)
- Runs on every navigation, decides if user can access a route.
    ```dart
    import 'package:flutter_riverpod/flutter_riverpod.dart';
    import '../../../core/auth/auth_provider.dart';

    String? authRedirect(Ref ref, String location) {
    final auth = ref.read(authStateProvider).value; // null if signed out
    final isLogin = location == '/login';
    final isAdminPath = location.startsWith('/admin');

    if (auth == null && !isLogin) return '/login';
    if (auth != null && isLogin)  return auth.role == 'admin' ? '/admin' : '/u';
    if (auth != null && isAdminPath && auth.role != 'admin') return '/u';
    return null;
    }
    ```
### `app/router/shells/*.dart` (layouts per role)
- Wraps all `/u/*` or `/admin/*` pages with a consistent scaffold (AppBar, drawer, bottom nav, etc.)
    ```dart
    import 'package:flutter/material.dart';

    class UserShell extends StatelessWidget {
    const UserShell({super.key, required this.child});
    final Widget child;

    @override
    Widget build(BuildContext context) {
        return Scaffold(
        appBar: AppBar(title: const Text('User Panel')),
        body: child,
        );
    }
    }
    ```
### `Coordinators` (navigation helpers)
- Widgets and controllers never call context.go().
- They call a coordinator, which uses the router provider.
    ```dart
    import 'package:flutter_riverpod/flutter_riverpod.dart';
    import 'app_router.dart';

    class UserCoordinator {
    UserCoordinator(this.ref);
    final Ref ref;

    void openProfile()         => ref.read(appRouterProvider).go('/u/profile');
    void openOrder(String id)  => ref.read(appRouterProvider).go('/u/orders/$id');
    }
    final userCoordinatorProvider = Provider((ref) => UserCoordinator(ref));
    ```
## `./core`
### `core/network/api_client.dart` (one Dio client for all repos)
- All repositories should use this client (not create their own).
    ```dart
    // core/network/api_client.dart
    import 'package:dio/dio.dart';
    import 'package:flutter_riverpod/flutter_riverpod.dart';

    final apiClientProvider = Provider<Dio>((ref) {
    final dio = Dio(BaseOptions(
        baseUrl: 'https://api.example.com', // replace with config
        connectTimeout: const Duration(seconds: 10),
        receiveTimeout: const Duration(seconds: 15),
    ));
    dio.interceptors.add(LogInterceptor(requestBody: false, responseBody: false));
    return dio;
    });
    ```
### `core/network/endpoints.dart` (central paths)
```dart
class Endpoints {
  static const userStats = '/user/stats';
  static const login     = '/auth/login';
}
```
### `core/auth/auth_provider.dart` (current user & role)
- Exposes an auth stream and user role so guards/coordinators can react.
    ```dart
    import 'package:flutter_riverpod/flutter_riverpod.dart';

    class AuthUser {
    final String uid;
    final String role; // 'user' | 'admin'
    const AuthUser(this.uid, this.role);
    }

    final authStateProvider = StreamProvider<AuthUser?>((ref) async* {
    // TODO: hook to Firebase/Auth SDK stream
    yield null; // signed out by default
    });
    ```
## `./features`
### `features/data/home_repository.dart`
- This file fetches data for the Home screen.
- What happens here:
    - This class talks to the backend (core/network/api_client.dart).
    - It converts raw JSON into your model (UserStats).
    - It hides all API logic from the rest of your app.
        - Think of this as the "Data Factory" — it gives ready-made data to your controller.
```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../../../core/network/api_client.dart';
import '../../../../core/network/endpoints.dart';
import '../../../../core/error/error_mapper.dart';
import '../../../../core/error/app_exception.dart';

/// ✅ Model (you can also import from models/user_stats.dart)
class UserStats {
  final int completed;
  final int pending;
  const UserStats({required this.completed, required this.pending});
}

/// ✅ Repository Provider (injects into controller)
final homeRepositoryProvider = Provider<HomeRepository>((ref) {
  final dio = ref.read(apiClientProvider);
  return HomeRepository(dio);
});

/// ✅ Repository Class — Handles API calls
class HomeRepository {
  HomeRepository(this._dio);
  final Dio _dio;

  Future<UserStats> fetchUserStats() async {
    try {
      final res = await _dio.get(Endpoints.userStats); // call API
      final data = res.data as Map<String, dynamic>;
      return UserStats(
        completed: data['completed'] ?? 0,
        pending: data['pending'] ?? 0,
      );
    } catch (error) {
      throw mapDioError(error); // uses core/error/error_mapper.dart
    }
  }
}
```
--- 