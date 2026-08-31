# AIWise Cleaner

A Flutter app for **iOS** and **Android** that finds and removes duplicate or similar photos using a **fast pipeline**: parallel **256px thumbnails**, **SHA256 of thumbnail** for identical previews, **dHash + aHash** with **time-windowed** comparison (not O(n²)), and optional **MobileNet** on a capped set for refinement.

## Features

- **Fast scan (target: 5000+ photos in ~1 minute on typical phones)**  
  Parallel thumbnail I/O, one decode per photo, windowed similarity (not full pairwise on the whole library).  
  **Exact** groups use identical **thumbnail bytes** (same 256px preview). For byte-identical full files with different previews, use a future “deep hash” mode.
- **Optional MobileNet** – Refines up to **400** remaining photos; **skipped automatically** above **2500** photos unless you enable **AI on large libraries** in the UI.
- **Similar photo groups** – View groups of duplicates/similar images.
- **Select & delete** – Choose which photos in each group to delete (with confirmation).

## Run

```bash
flutter pub get
flutter run
```

For a specific device:

```bash
flutter run -d <device_id>
```

## Permissions

- **iOS**: Photo library access (configured in `ios/Runner/Info.plist`).
- **Android**: `READ_MEDIA_IMAGES` / storage (configured in `android/app/src/main/AndroidManifest.xml`).

Grant photo access when prompted so the app can load and compare images.

## Similarity slider

On the home screen, **Similarity (1–15)** controls how strict the perceptual comparison is:

- **Lower (1–5)** – Only very similar images are grouped.
- **Higher (10–15)** – More images grouped as similar (e.g. same scene, different crop).

## Project structure

```
lib/
├── main.dart                 # App entry, auth gate, routes
├── theme/
│   └── app_theme.dart        # Theme and colors
├── services/
│   └── photo_comparison_service.dart  # Metadata, hash, perceptual hash pipeline
├── models/
│   └── photo_info.dart      # PhotoInfo, SimilarGroup
└── screens/
    ├── home_screen.dart      # Scan photos, list groups
    └── groups_screen.dart   # View group, select photos, delete
```

## On-device AI model (optional)

The app currently uses **perceptual hashing (dHash)** for similar photos. You can add a **real neural embedding model** that runs fully offline:

- **Models:** MobileNet V2 (feature vector), EfficientNet-Lite, or FaceNet. See **[docs/AI_MODELS.md](docs/AI_MODELS.md)** for exact model names, input/output sizes, and where to download the `.tflite` file.
- **Integration:** `lib/services/ai_embedding_service.dart` loads a TFLite model from `assets/models/` or from a file downloaded to app documents, then computes embeddings and cosine similarity. Wire it into the comparison pipeline when you want to use neural similarity instead of (or in addition to) dHash.
- **Offline:** The model can be bundled in the app or downloaded once (e.g. on first launch) and then used offline.

---