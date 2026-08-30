# NCH-Sleep-Data-Preprocessing-D
Desaturated vs. undesaturated patients를 대상으로 Sleep Stage 분포를 matching한 후, patient-level 4-fold cross-validation을 통해 EEG 기반 분류 성능을 평가하는 실험. 60/120 Hz noise 제거 및 STFT 전처리 후 ResNet18을 적용하고, Balanced Accuracy와 AUC를 기준으로 성능을 평가한다.
