# Problem Definition: Offline Crop Disease Classifier

## 1. Context and user
A farmer or field worker photographs one leaf with a low-end Android phone, often with no internet. They need a likely disease name within a couple of seconds. The app supports their decision; it is not a diagnosis.

## 2. Task
- Input: one RGB photo of one leaf, resized to 224x224 inside the app.
- Output: the top 3 labels with confidence scores, or "not sure" when the top confidence is below a threshold chosen on validation data.
- Labels: the 38 PlantVillage crop-disease and healthy classes (14 crops).

## 4. Evaluation protocol
- Splits are built by us from the original PlantVillage images (not the pre-augmented Kaggle copy), saved as a manifest CSV and checked for leakage.
- Train data fits the model. Validation data makes every choice (backbone, learning rate, threshold). Test data is used once per stage, at the end.
- Augmentation is applied to training images only.
- PlantDoc is never used to choose between models. One part is held out as the final field test set; another part may be used for fine-tuning in the field-aware stage.
- Every experiment records: date, Git commit, data split, settings and metrics.

## 5. Comparison grid
- Backbones: ResNet50 (ceiling), MobileNetV3-Small, MobileNetV3-Large, MobileNetV2, and EfficientNet-Lite0 if our toolchain supports it.
- Precisions: float32, float16, dynamic-range int8, full-integer int8.
- For every combination we report size, latency, and macro-F1 and top-1 on lab and field data.

## 6. Out of scope
- Locating disease inside the image, or grading severity.
- Treatment or pesticide advice.
- Whole-plant or multi-leaf photos, and crops outside the 14 in PlantVillage.
- On-device training, iOS, cloud serving.
- Any claim about real-world accuracy beyond what the PlantDoc test shows.

## 7. Risks and assumptions
- PlantVillage photos are lab-style; backgrounds can leak label information, so high lab scores may not transfer to farms.
- The same leaf can appear several times in the data and leak between splits; without leaf IDs we can reduce this risk but not remove it.
- PlantDoc is small and its classes only partly match PlantVillage.
- A laptop CPU is faster than a low-end phone, so laptop latency is a proxy until we test a real device.
- Small networks may lose more accuracy under int8 quantization.
