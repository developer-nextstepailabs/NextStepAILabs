# On-Device AI Models for Photo Similarity

The app currently uses **perceptual hashing (dHash)** for similar-photo detection—no neural network. To use a real **neural embedding model** on device (offline), you can add one of the following.

---

## Recommended models (offline-capable)

All of these can be **bundled in the app** or **downloaded once** to the device and then run fully offline.

### 1. **MobileNet V2 (feature vector)** — general images

- **Input:** 224×224 RGB, float32, normalized (e.g. in `[0,1]` or `[-1,1]` depending on model).
- **Output:** Feature vector (e.g. 1280 or 1024 dimensions).
- **Use:** Encode each photo to a vector; group photos whose vectors are close (e.g. cosine similarity > 0.9).

**Where to get .tflite:**

- TensorFlow Hub (Lite):  
  `https://tfhub.dev/tensorflow/lite-model/mobilenet_v2_1.0_224/1/default/1`  
  (classification; you can use the last layer before logits as embedding by using a model that outputs a feature vector, or use the one below.)
- **Feature-vector** (embedding) variant, e.g.:  
  `https://tfhub.dev/google/lite-model/imagenet/mobilenet_v2_075_224/feature_vector/2/default/1`  
  Download the `.tflite` and place in `assets/models/` or download at runtime and save with `path_provider`.

### 2. **EfficientNet-Lite** — better quality, still mobile-friendly

- **Input:** 224×224 (or 260×260 for larger lite variants), float32.
- **Output:** Feature vector (e.g. 1280 dims).
- **Use:** Same as MobileNet: encode → compare with cosine similarity.

**Where to get .tflite:**

- Search TensorFlow Hub for `efficientnet lite` and pick a **feature_vector** Lite model, or use the [TensorFlow Lite Task Library – Image Embedder](https://www.tensorflow.org/lite/inference_with_metadata/task_library/image_embedder) compatible models.

### 3. **FaceNet (e.g. facenet_mobilenet.tflite)** — faces only

- **Input:** 160×160 face crop, float32.
- **Output:** 128-dimensional face embedding.
- **Use:** Detect faces (e.g. with ML Kit), crop → run FaceNet → group by embedding similarity. Best for “duplicate/similar **faces**”, not general photos.

**Where to get .tflite:**

- Search for `facenet_mobilenet.tflite` or “FaceNet TFLite”; often found in face-recognition repos or TF Hub.

---

## How to run them on device (offline)

1. **Flutter:** Use the **tflite_flutter** package (`tflite_flutter: ^0.12.1`).
2. **Model location (pick one):**
   - **Bundled:** Put the `.tflite` file in `assets/models/` and load with `Interpreter.fromAsset('assets/models/your_model.tflite')`.
   - **Downloaded on device:** First run (or in settings), download the `.tflite` from a URL (or Firebase Storage) and save to a file in `getApplicationDocumentsDirectory()`. Then load with `Interpreter.fromFile(File(path))`. After that, inference is fully offline.
3. **Inference:** Resize each image to the model’s input size (e.g. 224×224), convert to float32 (and normalize like the model expects), then `interpreter.run(input, output)`. The output is the embedding vector.
4. **Similarity:** Compute **cosine similarity** (or L2 distance) between two embeddings. Photos with similarity above a threshold (e.g. 0.85–0.95) can be grouped as “similar”.

---

## Summary

| Model              | Input size   | Output    | Best for           | Offline |
|-------------------|-------------|-----------|--------------------|--------|
| MobileNet V2 (FV) | 224×224     | 1280/1024 | General photos     | ✅      |
| EfficientNet-Lite | 224 / 260   | 1280      | General photos     | ✅      |
| FaceNet           | 160×160     | 128       | Faces only         | ✅      |

All of these can be **downloaded once to the device** (e.g. on first launch or in a “Download model” step) and then used **offline** for the rest of the pipeline.
