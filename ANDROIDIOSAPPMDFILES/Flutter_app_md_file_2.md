# AIWise Cleaner – Copilot Instructions

Flutter photo-deduplication app for iOS and Android. Finds duplicate/similar photos using a fast pipeline: parallel thumbnail I/O → SHA256 (exact) → dHash+aHash sliding window (perceptual) → optional TFLite AI.

## Build & Test

```bash
flutter pub get          # install dependencies
flutter run              # run on connected device/simulator
flutter run -d <id>      # target a specific device
flutter test             # unit tests (test/)
flutter analyze          # lint (flutter_lints)
```

## Architecture

```
lib/
├── main.dart                          # Entry point; FlutterForegroundTask.initCommunicationPort() before runApp()
├── models/photo_info.dart             # PhotoInfo, SimilarGroup, DetectionMethod enum
├── services/
│   ├── photo_comparison_service.dart  # Core pipeline (scan, hash, AI)
│   ├── photo_storage_service.dart     # SQLite cache for fingerprints & embeddings (sqflite)
│   ├── ai_embedding_service.dart      # TFLite MobileNet inference
│   ├── scan_foreground_service.dart   # Background scan via flutter_foreground_task
│   └── scan_history_service.dart      # Deletion history (SQLite)
├── screens/
│   ├── home_screen.dart               # Scan trigger, group list, settings toggles
│   ├── groups_screen.dart             # Group viewer + deletion UI
│   ├── history_screen.dart
│   ├── settings_screen.dart
│   └── skipped_screen.dart
├── widgets/slide_to_delete_button.dart
└── theme/app_theme.dart
```

## Pipeline Conventions

- **One decode per photo**: the 256 px thumbnail (`_thumbPx = 256`) is reused for SHA256, dHash, aHash, and TFLite input — never decode at full resolution unless absolutely necessary.
- **Windowed comparison**: photos are sorted by `createDt`; only compare within `_maxTimeGapSec = 600 s` and `_maxForwardNeighbors = 200` neighbors. Never introduce O(n²) full-pairwise logic.
- **Fingerprint cache**: `PhotoStorageService` persists `contentHash`, `dHash`, `aHash`, and `embeddingJson` by asset ID. Always check the cache before recomputing — see `cachedUpToDate` logic in `loadPhotos()`.
- **AI is optional**: skip TFLite above 2500 photos unless `_aiOnLargeLibrary = true`. Max AI candidates = 500 (`_maxAiCandidates`). AI concurrency = 4 (`_aiConcurrency`); IO concurrency = 16 (`_ioConcurrency`).
- **DetectionMethod**: `exact` (SHA256 match) | `perceptual` (dHash+aHash) | `ai` (embedding cosine) | `similars`.

## Key Pitfalls

- **iOS deletion**: `PhotoManager.editor.deleteWithIds(...)` shows a native confirmation sheet. Always check `deletedIds.isEmpty` — that means the user cancelled; do not record a deletion event.
- **iOS permission**: `PhotoManager.requestPermissionExtend()` can return `PermissionState.limited` (partial access). Reflect this in UI.
- **`initialRoute` mismatch in main.dart**: `initialRoute: '/home'` but the routes map only defines `'/'`. This is a known existing issue — do not silently change routes unless asked.
- **TFLite models**: bundled in `lib/assets/models/` (declared in `pubspec.yaml` under `flutter: assets:`). Downloaded models go to `getApplicationDocumentsDirectory()`. See [docs/AI_MODELS.md](../docs/AI_MODELS.md) for model specs.
- **SDK**: Dart `^3.11.0`. Use null safety and `const` constructors throughout.

## Testing

Tests live in `test/`. Unit-test pure logic in `photo_comparison_service.dart` (e.g. `isPerceptualHashPairSimilar`, `isAISimilar`). Widget tests for screens require mocking `PhotoManager` — avoid real device APIs in tests.

## Docs

- [docs/AI_MODELS.md](../docs/AI_MODELS.md) — TFLite model specs, input/output sizes, download URLs
- [README.md](../README.md) — feature overview and project structure
