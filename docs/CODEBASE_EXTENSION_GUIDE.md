# XPENC Codebase Extension Guide

This guide is code-focused: repository structure, runtime flow, and safe extension points.

## 1) Codebase map

- App entry and runtime shell
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/main.dart`
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/app.dart`
- Navigation and shell
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/core/routing/app_router.dart`
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/core/routing/app_shell.dart`
- Data layer (source of persistence + write-time invariants)
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/data/tables.dart`
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/data/database.dart`
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/data/providers.dart`
- Feature modules (UI by slice)
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/features/**`
- Shared core primitives/services
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/core/**`
- Tests
  - `/home/runner/work/XPENC---Extended/XPENC---Extended/test/**`

## 2) Runtime flow

1. `main.dart` boots Flutter and wraps app with Riverpod `ProviderScope`.
2. `XpencApp` (`lib/app.dart`) orchestrates startup:
   - database readiness gate
   - onboarding gate
   - home-widget initialization/sync
   - notification initialization/sync
   - recurring processing
   - message intake scan
   - auto-backup due check
3. Router (`GoRouter` + `StatefulShellRoute`) mounts persistent shell tabs and pushes detail routes above the shell.
4. Feature screens consume Riverpod providers from `lib/data/providers.dart`.
5. Writes go through `AppDatabase` methods in `lib/data/database.dart`; transactional writes update tables + cached balances atomically.
6. Drift reactive streams propagate DB changes back to providers and then UI.
7. Cross-cutting side effects are listener-driven (e.g. ledger changes trigger budget checks, widget refresh, and backup scheduling).

## 3) Safe extension paths

Decide extension type first, then touch only relevant layers.

### A. UI/navigation extension

- Add or update files in `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/features/<feature>/`
- Wire routes in `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/core/routing/app_router.dart`
- Expose/consume screen state through `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/data/providers.dart`

### B. Derived state/reporting extension

- Add provider composition in `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/data/providers.dart`
- Reuse existing DB watch methods where possible
- Keep formatting/presentation decisions in UI layer

### C. Persistence/schema extension

1. Edit `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/data/tables.dart`
2. Bump `schemaVersion` and add migration step in `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/data/database.dart`
3. Regenerate Drift code:
   - `dart run build_runner build --force-jit --delete-conflicting-outputs`
4. Do not hand-edit generated `database.g.dart`

## 4) Layering rules

- Keep persistence/business invariants in `AppDatabase`, not widgets.
- Keep reactive read composition in providers.
- Keep UI concerns in feature screens/components.
- Prefer reusing existing primitives under `lib/core/` for money/currency/theme/routing/security behavior.

## 5) First extension workflow (recommended)

1. Read startup lifecycle:
   - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/app.dart`
2. Read data contract:
   - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/data/database.dart`
   - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/data/providers.dart`
3. Trace one vertical slice end-to-end:
   - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/features/add_transaction/add_transaction_screen.dart`
   - `/home/runner/work/XPENC---Extended/XPENC---Extended/lib/features/transactions/transactions_screen.dart`
4. Implement one small end-to-end change (UI + provider + DB method + test) before larger refactors.

## 6) Validation pipeline

Use the existing project validation commands:

- `flutter analyze`
- `flutter test`

If schema changed, also run:

- `dart run build_runner build --force-jit --delete-conflicting-outputs`
